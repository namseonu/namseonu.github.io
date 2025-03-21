---
title: 컴포넌트 스캔과 의존성 주입

categories:
  - Spring Boot

toc: true
toc_sticky: true

date: 2024-08-10
last_modified_at: 2024-08-10
---

**Table of Contents**

1. 스프링 빈을 등록하는 방법
2. 컴포넌트 스캔
   1. @ComponentScan 사용하기
   2. 컴포넌트 스캔 기본 대상
   3. 컴포넌트 스캔과 필터
   4. 빈 중복 등록과 충돌 문제 해결
3. 의존성 주입

# 1. 스프링 빈을 등록하는 방법

스프링에서 스프링 빈을 등록하는 방법에는 크게 3가지가 있다.

**스프링 빈을 등록하는 방법**

① 자바에서 @Configuration과 @Bean 메서드를 이용한다.

② XML 파일에서 <bean>을 이용한다.

③ <u>스프링에서 제공하는 컴포넌트 스캔 기능을 이용한다.</u> (+의존 관계를 자동으로 주입하는 @Autowired)

자바와 XML 파일을 이용하는 경우, 빈이 새로 추가될 때마다 자바와 XML 파일에도 추가된 빈을 등록해줘야 한다는 단점이 있다. 따라서 스프링에서 제공하는 컴포넌트 스캔 기능을 이용하는 것이 좋아 보인다. 해당 내용은 '3. 컴포넌트 스캔에서 의존 관계를 자동으로 주입하기' 목차에서 자세하게 다룰 예정이다.

참고: 자바에서 스프링 빈을 등록하는 방법 (AppConfig.java)

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

	@Bean
	public MemberService memberService() {
		return new MemberServiceImpl(memberRepository());
	}

	@Bean
	public MemoryMemberRepository memberRepository() {
		return new MemoryMemberRepository(); // 구현체를 바꾸고 싶을 때 해당 부분만 바꾸면 된다.
	}

	@Bean
	public OrderService orderService() {
		System.out.println("call AppConfig.orderService");
		return new OrderServiceImpl(
				memberRepository(),
				discountPolicy());
	}

	@Bean
	public DiscountPolicy discountPolicy() {
//		return new FixDiscountPolicy();
		return new RateDiscountPolicy();
	}
}
```

참고: XML 파일에서 스프링 빈을 등록하는 방법 (appConfig.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="memberService" class="hello.core.member.MemberServiceImpl">
		<constructor-arg name="memberRepository" ref="memberRepository" />
	</bean>

	<bean id="memberRepository" class="hello.core.member.MemoryMemberRepository" />

	<bean id="orderService" class="hello.core.order.OrderServiceImpl">
		<constructor-arg name="memberRepository" ref="memberRepository" />
		<constructor-arg name="discountPolicy" ref="discountPolicy" />
	</bean>

	<bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy" />
</beans>
```

# 2. 컴포넌트 스캔

여기서는 스프링에서 제공하는 컴포넌트 스캔 기능을 이용하여 스프링 빈을 등록하는 방법에 대해 자세히 알아보려고 한다. 컴포넌트 스캔을 사용하려면 먼저 @ComponentScan을 설정 정보에 붙여주면 된다.

## 2.1 @ComponentScan 사용하기

- @Component가 붙은 클래스를 모두 찾아서 스프링 빈으로 등록한다.

  - @Component가 붙은 클래스들은 싱글톤으로 스프링 빈에 등록된다.
  - 스프링 빈의 기본 이름은 클래스 명을 사용하되 맨 앞 글자는 소문자로 사용한다.
    - 빈 이름 기본 전략: MemberServiceImpl 클래스 → memberServiceImpl
    - 특수한 경우에 빈 이름 직접 지정: `@Component("memberService2")`

- 컴포넌트 스캔을 사용하면 @Configuration 애너테이션이 붙은 설정 정보도 자동으로 등록된다.

  - @Configuration 애너테이션 내부적으로 @Component를 사용하기 때문이다.

- @ComponentScan을 사용하기 위해 스프링 빈으로 등록할 클래스에 @Component를 붙여준다.
  - @Component는 스프링 빈으로 등록하기만 할 뿐 의존 관계를 자동으로 주입하지는 않는다.
  - 그렇다면 스프링 빈으로 등록하는 클래스 사이에 필요한 의존 관계 주입은 어떻게 해야 할까? ('2.2 @Autowired' 참고)

**@ComponentScan 속성**

| 속성                     | 설명                                                                                                                                                                                                                                         | 예시                                                                                                                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| basePackages (\*1)       | 컴포넌트 스캔으로 탐색할 패키지의 시작 위치를 지정한다.<br />패키지의 시작 위치를 따로 지정하지 않으면 라이브러리를 포함한 모든 패키지를 스캔하므로 시간이 오래 걸린다.<br />콤마(,)를 통해 여러 개의 패키지를 시작 위치로 지정할 수도 있다. | hello.core.member 패키지와 그 하위 패키지를 대상으로 컴포넌트 스캔을 진행한다.<br />`basePackages = "hello.core.member"`                                                       |
| basePackageClasses (\*1) | 지정한 클래스의 패키지를 탐색 위치로 지정한다.                                                                                                                                                                                               | AutoAppConfig.java 클래스가 위치한 패키지를 탐색 위치로 지정한다.<br />`basePackageClasses = AutoAppConfig.class`                                                              |
| includeFilters           | 컴포넌트 스캔 대상을 추가로 지정한다.<br />거의 사용하지 않는다.                                                                                                                                                                             | MyIncludeComponent 애너테이션이 붙은 클래스를 추가로 지정한다.<br />`includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)` |
| excludeFilters           | 컴포넌트 스캔에서 제외할 대상을 지정한다.<br />필요할 때 간혹 사용할 때가 있다.                                                                                                                                                              | MyExcludeComponent 애너테이션이 붙은 클래스를 제외한다.<br />`excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)`             |
| useDefaultFilters        | 기본적으로 켜져 있는데, 이 옵션을 끄면 기본 스캔 대상들이 제외된다. 여기서 말하는 기본 스캔 대상은 '2.2 컴포넌트 스캔 기본 대상'에서 소개한 @Component, @Controller, @Service, @Repository, @Configuration을 의미한다.                       |                                                                                                                                                                                |

참고로, 위에서 @Configuration 애너테이션이 붙은 클래스를 빈 등록 시 제외시키는 이유는 스프링 강의 시간에 진행한 예제 코드를 보존하기 위함이므로 특별한 이유가 없으면 사용하지 않아도 된다. 보통 설정 정보를 컴포넌트 스캔 대상에서 제외하지는 않는다.

자바에서 @ComponentScan을 사용하는 방법 (AutoAppConfig.java)

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class AutoAppConfig {
}
```

> **\*1) 컴포넌트 스캔의 탐색 패키지 기본 값**
>
> basePackages, basePackageClasses 속성의 기본 값은 @ComponentScan이 붙은 설정 정보 클래스의 패키지 명이다. 따라서 굳이 명시하지 않아도 @ComponentScan이 붙은 클래스 패키지를 시작으로 그 하위 패키지를 탐색하게 된다.
>
> 이러한 관례를 따랐을 때 권장하는 방법은 패키지 위치를 지정하지 않고, @ComponentScan이 붙는 설정 정보 클래스를 프로젝트 최상단에 위치시키는 것이다.
>
> 최근 스프링 부트도 이 방법을 @ComponentScan을 가지고 있는 @SpringBootApplication 애너테이션을 통해 기본으로 제공한다. 참고로, @SpringBootApplication이 붙은 메인 클래스는 프로젝트 최상단에 위치한다.

## 2.2 컴포넌트 스캔 기본 대상

| 컴포넌트 스캔 대상 애너테이션 | 설명                             | 스프링에서 제공하는 부가 기능                                                                 |
| ----------------------------- | -------------------------------- | --------------------------------------------------------------------------------------------- |
| @Component                    | 컴포넌트 스캔에서 사용           | -                                                                                             |
| @Controller                   | 스프링 MVC 컨트롤러에서 사용     | 스프링 MVC 컨트롤러로 인식                                                                    |
| @Service                      | 스프링 비즈니스 로직에서 사용    | -                                                                                             |
| @Repository                   | 스프링 데이터 접근 계층에서 사용 | 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 추상화하여 스프링 예외로 변환한다. |
| @Configuration                | 스프링 설정 정보에서 사용        | 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 처리한다.                        |

@Controller, @Service, @Repository 모두 @Componet 애너테이션을 내부적으로 포함하고 있다.

```java
@Component
public @interface Controller {
}

@Component
public @interface Service {
}

@Component
public @interface Repository {
}

@Component
public @interface Configuration {
}
```

참고로, @interface는 애너테이션을 의미한다. 여기서 주의해야 할 점은 애너테이션은 상속 관계가 없기 때문에 특정 애너테이션이 가지고 있는 다른 애너테이션을 인식할 수 있는 것은 자바가 아닌 스프링이 지원하는 기능이다.

## 2.3 컴포넌트 스캔과 필터

- @ComponentScan 애너테이션의 includeFilters, excludeFilters 속성을 통해 필터를 관리할 수 있다.
- 하지만 컴포넌트 스캔 필터 기능은 거의 사용하지 않으며, 필요할 때 ASSIGNABLE_TYPE 정도만 사용한다.
- 옵션을 변경하는 것보다는 스프링 부트가 제공하는 컴포넌트 스캔 기본 기능에 맞춰 사용하는 것이 좋다.

| 속성           | 설명                                      |
| -------------- | ----------------------------------------- |
| includeFilters | 컴포넌트 스캔 대상을 추가로 지정한다.     |
| excludeFilters | 컴포넌트 스캔에서 제외할 대상을 지정한다. |

**FilterType 옵션**

아래에서 사용된 org.springframework.context.annotation.FilterType에는 총 5개의 옵션이 있다.

| 옵션            | 설명                                                              | 예시                       |
| --------------- | ----------------------------------------------------------------- | -------------------------- |
| ANNOTATION      | 기본 값이므로 생략 가능하다.<br />애너테이션을 인식해서 동작한다. | org.example.SomeAnnotation |
| ASSIGNABLE_TYPE | 지정한 타입과 자식 타입을 인식해서 동작한다.                      | org.example.SomeClass      |
| ASPECTJ         | AspectJ 패턴을 사용한다.                                          | org.example..\*Service+    |
| REGEX           | 정규 표현식을 사용한다.                                           | org\\.example\\.Default.\* |
| CUSTOM          | TypeFilter라는 인터페이스를 구현하여 처리한다.                    | org.example.MyTypeFilter   |

컴포넌트 스캔 시 추가/제외 필터 추가 (ComponentFilterAppConfig.java)

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
        includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
)
public class ComponentFilterAppConfig {
}
```

@MyIncludeComponent 애너테이션을 사용하는 BeanA를 빼고 싶은 경우(ComponentFilterAppConfig.java)

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
        includeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)
        },
        excludeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class),
            @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BeanA.class)
        }
)
public class ComponentFilterAppConfig {
}
```

## 2.4 빈 중복 등록과 충돌 문제 해결

컴포넌트 스캔 시 같은 이름을 가진 빈을 중복해서 등록하면 어떻게 될까?

### ① 자동 빈 등록 vs. 자동 빈 등록

- 컴포넌트 스캔에 의해 자동으로 등록된 빈 중에서 이름을 같은 빈이 2개 이상 있는 경우, 스프링은 ConflictingBeanDefinitionException 예외를 발생시킨다.

아래처럼 MemberServiceImpl 클래스와 OrderServiceImpl 클래스에 정의된 빈 이름이 동일할 경우, 컴포넌트 스캔 과정에서 ConflictingBeanDefinitionException 예외가 발생한다.

MemberServiceImpl.java

```java
@Component("service")
public class MemberServiceImpl implements MemberService {
    // ...
}
```

OrderServiceImpl.java

```java
@Component("service")
public class OrderServiceImpl implements OrderService {
    // ...
}
```

### ② 수동 빈 등록 vs. 자동 빈 등록

- @Component에 의해 자동으로 등록된 빈과 사용자가 수동으로 등록한 빈의 이름이 중복될 경우 스프링은 수동 빈 등록 우선권을 가지므로, 수동 빈이 자동 빈을 오버라이딩한다. 즉, 오류가 발생하지 않는다.
- 하지만 수동 빈이 자동 빈을 오버라이딩하는 경우 개발자가 예상하지 못한 버그가 발생할 수 있으므로, 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본 값을 바꾸었다. (이를 확인하려면 테스트가 아닌 스프링 부트를 통해서 실행해야 한다.)

MemoryMemberRepository.java

```java
@Component
public class MemoryMemberRepository implements MemberRepository {
}
```

AutoAppConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
		basePackages = "hello.core.member",
		basePackageClasses = AutoAppConfig.class,
		excludeFilters = @ComponentScan.Filter(
				type = FilterType.ANNOTATION,
				classes = Configuration.class
		)
)
public class AutoAppConfig {

	@Bean(name = "memoryMemberRepository")
	MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}
}
```

**위와 같은 상황에서 테스트 코드 실행 시**

이미 MemoryMemberRepository 빈이 @Component로 등록되어 있는 상황에서 @Bean 애너테이션으로 수동으로 동일한 이름의 빈을 등록한 경우, 수동 빈 등록이 우선권을 가진다. 즉, 수동 빈이 자동 빈을 오버라이딩한다.

```bash
Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing [Generic bean: class [hello.core.member.MemoryMemberRepository]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in file [/Users/namseonu/Documents/study/spring/course-inflearn-spring-basic/practice/out/production/classes/hello/core/member/MemoryMemberRepository.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=autoAppConfig; factoryMethodName=memberRepository; initMethodNames=null; destroyMethodNames=[(inferred)]; defined in hello.core.AutoAppConfig]
```

**위와 같은 상황에서 스프링 부트 실행 시**

스프링 부트 실행 시 아래와 같은 오류를 발생시킨다.

스프링은 오버라이딩을 가능하도록 되어 있지만, 스프링 부트는 오바라이딩을 막도록 설정되어 있다.

```bash
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'memoryMemberRepository', defined in class path resource [hello/core/AutoAppConfig.class], could not be registered. A bean with that name has already been defined in file [/Users/namseonu/Documents/study/spring/course-inflearn-spring-basic/practice/out/production/classes/hello/core/member/MemoryMemberRepository.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

만약 스프링 부트에서 빈 중복 등록 문제의 오버라이딩을 허용하도록 변경하고 싶다면, application.yml 혹은 application.properties에서 아래와 같이 설정해주면 된다.

application.yml

```xml
spring:
    main:
        allow-bean-definition-overriding: true
```

application.properties

```bash
spring.main.allow-bean-definition-overriding=true
```

# 3. 의존성 주입

### @Autowired

- 스프링 컨테이너가 의존 관계를 자동으로 주입한다.
  - 스프링 컨테이너의 기본 조회 전략은 타입이 같은 빈을 찾아서 주입하는 방식이다. (=`getBean(MemberRepository.class)`)
  - <mark>만약 같은 타입의 빈이 2개 이상 등록되어 있는 경우, </mark>
- 컴포넌트 스캔을 사용하여 스프링 빈을 등록하면 자동으로 의존 관계 주입이 안 되는데, 이때 @Autowired를 사용하여 의존 관계 문제를 해결할 수 있다.
  - AppConfig.java 클래스에서 @Bean으로 직접 설정 정보를 작성하여 의존 관계를 명시하지 않기 때문이다.
  - 설정 정보 클래스가 별도로 존재하지 않기 때문에 의존 관계 주입도 @Component를 붙인 클래스에서 해결해야 한다.
  - 이때 @Autowired를 사용하면 @Component 클래스 안에서 의존 관계를 자동으로 주입할 수 있다.

#### ① 생성자 주입 방법

- 생성자에서 여러 의존 관계를 한 번에 주입할 수 있다.
