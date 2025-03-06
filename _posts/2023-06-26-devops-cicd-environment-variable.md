---
title: CI/CD 환경 변수 설정하기

categories:
  - DevOps

toc: true
toc_sticky: true

date: 2023-06-26
last_modified_at: 2023-06-26
---

이번 글에서는 CI/CD의 전반적인 내용과 구축 방법을 알려주지는 않는다.
이번 글은 이전에 내가 CI/CD를 구축하는 과정에서 경험했던 트러블 슈팅 내용을 정리하기 위함이다.
종종 서버에서 (클라이언트도 마찬가지) 민감한 정보를 숨겨야 하는 경우가 있는데, 이 정보를 CI/CD 파이프라인을 구축할 때는 어떻게 숨겨야 하는지를 정리한 내용이다.

| 구분                         | 개발 환경     |
| ---------------------------- | ------------- |
| 운영체제                     | CentOS 7 (VM) |
| 백엔드(Back-end) 프레임 워크 | Spring Boot   |

<br>

## 목차

<!-- 목차 -->

1. [CI/CD란?](#1-cicd란)  
   1.1 [CI/CD의 개념](#11-cicd의-개념)  
   1.2 [CI/CD의 등장 배경](#12-cicd의-등장-배경)
2. [Back-end의 민감한 정보를 숨겨야 하는 이유](#2-back-end의-민감한-정보를-숨겨야-하는-이유)
3. [CI/CD 파이프라인에서 환경 변수를 설정하는 방법](#3-cicd-파이프라인에서-환경-변수를-설정하는-방법)
4. [결론](#4-결론)
5. [참고 자료](#5-참고-자료)

<br>

## 1. CI/CD란?

### 1.1 CI/CD의 개념

- CI(Continuous Integration): 지속적인 통합
- CD(Continuous Delivery/Deployment): 지속적인 전달/배포

먼저 CI/CD에서 CI는 Continuous Integration의 약자로, 지속적인 통합을 의미한다. 지속적인 통합이란 하나의 애플리케이션을 여러 개발자가 같이 개발하고 있는 경우 각자의 결과물을 통합하기 위한 형상 관리 혹은 통합된 코드를 빌드하고 테스트하는 과정 자체를 의미한다.

다음으로 지속적인 배포를 의미하는 CD는 개발자가 수정한 내용을 배포된 서버, 즉 실행 환경에 자동으로 반영해 주는 것을 의미한다. 이러한 CD는 Continupus Delivery와 Continuous Deployment, 이렇게 두 가지로 나눌 수 있다. 이 두 가지는 패키지화된 코드 결과물을 실행 환경에 어떻게 배포하는지에 따라서 구분된다. 만약 패키지화된 코드 결과물을 실행 환경에 수작업으로 배포해야 한다면 Continuous Delivery이며, 관리자의 개입 없이 실행 환경까지 완벽하게 자동화 되어 있는 배포를 Continuous Deployment라고 한다.

<img src="../../images/cicd_230626_01.png" alt="CD의 개념" height=200/>

<br>

### 1.2 CI/CD의 등장 배경

그렇다면 이러한 CI/CD의 개념은 왜 등장하게 되었을까? 오늘날 소프트웨어 산업이 발달함에 따라 소프트웨어가 거대해지고 복잡해지면서 대부분의 개발 작업은 팀 단위로 이루어지게 되었다. 팀원들끼리의 분업과 협업은 선택이 아닌 필수가 되었지만, 각자의 코드를 합치는 과정에서 많은 사람들이 불편함을 느끼게 되었다. (CI와 관련된 부분)

또, 고객들의 변화되는 요구에 맞추어 서비스를 개발해야 했고, 변화된 요구를 적용시키는 과정에서 서비스를 중단시키지 않고 안정적이면서 신속하게 변경된 내용을 반영해야 했다. 즉, 업데이트 배포의 위험을 최소화하고, 자동화시켜 무중단으로 서비스를 계속 고객들에게 제공해야 한다는 것이다. (CD와 관련된 부분)

CI/CD는 이러한 환경과 요구 속에서 등장하게 되었다. CI, 지속적인 통합으로 수월하고 신속한 협업을 이루어낼 수 있다. CD, 지속적인 배포로 안정적이고 빠른 서비스 제공을 이루어낼 수 있다.

<br>

## 2. Back-end의 민감한 정보를 숨겨야 하는 이유

나는 Spring Boot로 해당 실습을 진행하였기 때문에 아래의 내용은 모두 Spring Boot 프레임워크를 기준으로 작성되어 있다.
하지만 모든 클라이언트/서버 개발에서 민감한 정보를 숨겨야 하는 건 공통적으로 적용되는 내용이다.

Spring Boot의 application.yml 파일을 예로 들어 설명해보겠다.

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${DATABASE_USER_NAME}
    password: ${DATABASE_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: update
    generate-ddl: true
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    properties:
      hibernate:
        format_sql: true
    show-sql: true

  data:
    rest:
      base-path: /api
      default-page-size: 10
      max-page-size: 10

  main:
    allow-bean-definition-overriding: true

logging:
  level:
    root: info
    ctype.survey.hivey: debug
```

위의 application.yml 파일에서 우리가 숨겨야 할 민감한 정보는 데이터베이스와 관련된 정보이다. 데이터베이스의 URL, 사용자의 이름, 비밀번호 등은 외부로 유출이 되면 트래픽 등 다양한 공격을 받을 수 있기 때문에 외부에 유출되지 않도록 주의해야 한다. 특히 AWS 같은 유료 클라우드 플랫폼을 사용하는 경우는 과금의 위험성도 있으니 더 주의해야 한다.

위에서 `${}`로 표시된 부분이 바로 민감한 정보를 가리기 위해 환경 변수를 사용한 것이다.

요즘 대부분의 개발자가 Git, GitHub, GitLab을 사용하는 만큼 외부로 소스 코드가 공개되는 경우가 종종 있는데, 이때 이러한 민감한 정보를 가리는 것을 간과해서는 안 된다. 참고로 가려야 하는 민감한 정보의 예로는 데이터베이스 정보, API 키 값 등이 있다.

<br>

## 3. CI/CD 파이프라인에서 환경 변수를 설정하는 방법

그럼 이제 CI/CD 파이프라인에서 환경 변수를 어떻게 설정해야 이러한 민감한 정보를 잘 숨길 수 있는지를 알아보도록 하겠다.
나는 여기서 CI로는 GitLab을, CD로는 Jenkins의 Freestyle project를 사용하였다. (실무에서는 Pipeline을 더 많이 사용한다고 한다.)

해당 글에서는 CI/CD 파이프라인에서 환경 변수를 어떻게 설정하는지만 정리할 예정이며, CI/CD의 구축 과정은 추후 정리하도록 하겠다.

- CI: GitLab
- CD: Jenkins (Freestyle project)

<br>

### ① Jenkins 프로젝트 설정 들어가기

먼저 Jenkins를 실행시킨 후, Jenkins에서 생성한 프로젝트의 설정으로 들어간다.
아래와 같이 'Dashboard > 원하는 프로젝트 > Configuration'으로 들어가면 된다. (참고로 내가 이때 사용한 프로젝트명은 아래에 보이는 것처럼 hivey-backend-springboot라는 프로젝트이다.)

<img src="../../images/cicd_230626_02.png" alt="Jenkins 프로젝트 설정 들어가기" />

<br>

### ② '이 빌드는 매개변수가 있습니다' 체크한 후 매개변수 추가하기

Configuration에 들어간 후, General 부분에 있는 '이 빌드는 매개변수가 있습니다'를 찾아 체크한다.

<img src="../../images/cicd_230626_03.png" alt="Jenkins 환경 변수 설정 ①" />

<br>

'이 빌드는 매개변수가 있습니다.'를 체크한 후에는 아래 이미지처럼 '매개변수 추가 > 원하는 형식'을 선택해서 매개변수를 설정하면 된다. 이때 여기서의 매개변수가 바로 환경 변수 역할을 하게 된다.
아래와 같이 설정하게 되면 Jenkins에서 프로젝트를 빌드할 때 환경 변수가 필요한 부분에 알맞은 값을 넣어줄 수 있다.

| 구분          | 설명                                              |
| ------------- | ------------------------------------------------- |
| 매개변수 명   | .yml 파일 등에 작성한 `${}` 안에 있는 매개변수 명 |
| Default Value | 환경 변수가 위치한 곳에 넣을 값                   |

<img src="../../images/cicd_230626_04.png" alt="Jenkins 환경 변수 설정 ②" />

<br>

### ③ Build Steps 부분에 명령어 작성하기

위의 단계에서 매개변수 추가를 완료하였다면, 이제 마지막으로 Build Steps 부분에 알맞은 명령어를 작성하면 된다. 'Build Steps > Add build step > Execute shell' 부분에 아래 이미지처럼 내용을 작성해 주어야 한다.

```bash
echo '설명'
sed -i "s/\${환경 변수 명}/${매개 변수 명}/" "${WORKSPACE}/src/main/resources/application.yml"
```

이 단계는 우리가 위에서 추가한 매개변수가 위치한 곳에 알맞은 값을 넣어줌으로써 흔히 잘 알고 있는 환경 변수로서의 역할을 수행할 수 있도록 하기 위한 부분이라고 보면 된다. 즉, Spring Boot의 yml 파일에 환경 변수로서 처리된 부분을 찾아 Jenkins에서 추가한 매개변수의 값을 넣어주기 위함이다.

이렇게 한 후 저장하면 CI/CD 파이프라인에서 Jenkins의 매개 변수를 활용해 환경 변수를 설정함으로써 민감한 정보를 숨길 수 있다.

<img src="../../images/cicd_230626_05.png" alt="Jenkins 환경 변수 설정 ③" />

<br>

<br>

## 4. 결론

사실 환경변수를 설정해서 빌드 및 배포를 하는 과정에서 정말 어마무시한 삽질을 했다. CI/CD를 처음 접해서 진행했던 나는 환경변수를 어디에서 설정해줘야 하는지도 몰랐다. CI인 GitLab에서 해줘야 하는지, CD인 Jenkins에서 해줘야 하는지, 심지어 단순히 그냥 Spring Boot application.yml에서 뭔가 다른 처리를 해줘야 하는 건지도 잘 몰랐다.

CI/CD 파이프라인 구축 및 WAS 연결 과제가 바로 내가 이번 실습을 진행하게 된 계기였는데, 이 과제는 데이터베이스 연결까지는 포함하고 있지 않았다. 하지만 서버에 데이터베이스 연결은 거의 필수적이라고 생각했던 나는 기왕 하는 거 제대로 해보고 싶어서 도전해보았던 것이었다.

그런데 중간에 너무 오랜 삽질을 거치니 정말 포기하고 싶어서 그냥 데이터베이스 내용을 넣지 말까? 부터 application.yml 파일에 민감한 정보를 그대로 노출시킬까? 하는 생각까지 했었다. (개발자로서 절대 하면 안 되는 생각이다. 이건 단순히 과제였기 때문에 생각했던 것이었다.) 하지만 스스로 용납을 못했다. 기왕 시작한 거 끝까지 제대로 하고 싶었다. 민감한 정보를 그대로 공개하는 것은 보안적인 큰 이슈를 불러올 수 있기 때문에 백엔드 개발자가 절대 하면 안 되는 행동이라고 생각했다.

오랜 삽질과 고민을 거친 결과 Jenkins의 매개 변수를 이용해 환경변수를 무사히 설정하고 나니 그 뿌듯함을 이루 말할 수 없었다. 이때 CI/CD의 매력에 흠뻑 빠졌는데, 다음에는 보다 더 자세히, 꼼꼼하게 정리해볼 수 있었으면 좋겠다.

<br>

## 5. 참고 자료

- [Red Hat, 「CI/CD(Continuous Integration/Continuous Delivery)란?」, 2023. 04. 26.](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)
- [Dowon Lee, 「Spring Cloud로 개발하는 마이크로서비스 애플리케이션 (MSA)」, 인프런, 2022. 06. 29.](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4/dashboard)

<br>
