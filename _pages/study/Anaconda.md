---
title: Anaconda
author: Nam Seonwoo
date: 2022-12-02
category: study
layout: post
---

## 목차
1. [명령어](#명령어)
2. [트러블슈팅](#트러블슈팅)
</br>

## 명령어

가상환경 목록 확인:
```bash
conda env list
```

가상환경 생성:
```bash
# 일반적인 가상환경 만들기
conda create -n 가상환경이름 python

# 특정 파이썬 버전(예: 3.7)의 가상환경 만들기
conda create -n 가상환경이름 python=3.7

# 기본 라이브러리 함께 설치하면서 가상환경 만들기
# 예를 들어 anaconda를 붙이면 대다수의 파이썬 라이브러리가 자동으로 설치된다.
# 그렇지 않으면 numpy, matplotlib, pandas 등 수동 설치가 필요하다.
conda create -n 가상환경이름 python anaconda
```

가상환경 삭제:
```bash
conda env remove -n 가상환경이름
```

가상환경 활성화:
```bash
conda activate 가상환경이름
```

가상환경 비활성화:
```bash
conda deactivate
```

가상환경 업데이트:
<span style="color: orange">여기서 base가 아니라 다른 가상환경이름이 들어가도 되는지 알아 보기</span>
```bash
conda update -n base -c defaults conda
```
</br>

## 트러블슈팅
> PackagesNotFoundError: The following packages are not available from current channels:
> \- 패키지이름\==버전
```bash
# Anaconda 환경인 경우 아래 명령어를 실행한다.
conda install -c conda-forge 패키지이름

# 만약 위의 명령어로도 해결이 되지 않은 경우 아래 명령어를 실행한다.
pip install 패키지이름
```
여기서 `conda-forge`는 여러 자발적인 기여자들이 모인 conda-forge라는 커뮤니티에서 운영 및 관리하는 conda용 채널이다. 이때 채널(channel)이란 어디서 패키지를 가져와 설치할지를 지정하는 데 사용되는 것으로, 패키지들이 저장되어 있으면서 호스팅되는 위치라고 생각하면 된다. (참고로 conda는 https://repo.anaconda.com/pkgs/ 를 default channel로 사용한다.)  

`conda-forge`는 conda의 기본 채널보다 보다 다양하고 안정적인 패키지를 제공한다. 특히 머신러닝, 컴퓨터 비전, 수치해석, 그리고 django 등을 이용한 RESTful API 개발 등이 대표적인 예시다. 원하는 패키지가 `conda-forge`에서도 보이지 않는다면 `pip`를 이용하는 수밖에 없는데, `conda-forge`에서 지원하지 않는다는 것은 해당 패키지의 사용률이 굉장히 낮음을 의미하므로 사용을 지양하는 것이 좋다.  

출처: [miniconda의 기본 채널 변경 : conda-forge](https://dsaint31.tistory.com/261)
