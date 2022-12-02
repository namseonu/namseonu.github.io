---
title: Django
author: Nam Seonwoo
date: 2022-12-02
category: server
layout: post
---
---
## 목차
1. [Django 시작하기](#django-시작하기)
2. [Django 명령어](#django-명령어)
<br>

---
## Django 시작하기
### 1. 가상환경 설정하기 
1.1. Anaconda
```bash
# 가상환경 만들기
conda create -n django

# 가상환경 활성화
conda activate django

# (참고) 가상환경 비활성화
conda deactivate
```

1.2. Python
```bash
# 가상환경 만들기
# -m venv: 파이썬 모듈 중에서 venv라는 모듈을 사용하겠다.
python -m venv 가상환경이름

# 가상환경 활성화
source 가상환경이름/bin/activate

# (참고) 가상환경 비활성화
deactivate
```


### 2. 필요한 패키지 다운받기  
2.1. pip 버전 확인
```bash
# 1번째 방법:
pip --version

# 2번째 방법:
pip -V
```

2.2. pip 버전 업그레이드
```bash
# Mac, Windows
python -m pip install --upgrade pip

# Linux
pip install --upgrade pip
```

2.3. 필요한 패키지 다운로드
```bash
pip install django

# 만약 anaconda 가상환경을 사용할 경우 아래 명령어도 입력해 준다.
conda install django
```


### 3. Django 프로젝트 생성 및 불러오기
3.1. 새로운 프로젝트 생성
```bash
# 만들고 싶은 디렉토리에 프로젝트를 생성한다.
django-admin startproject 장고프로젝트명
```

3.2. 이미 있는 프로젝트 불러오기
```bash
git clone 레포지토리주소
```


### 4. Django 프로젝트 열기
파이참에서 만들어진 장고 프로젝트를 연다.
이때 `File-Settings-Project: 프로젝트명-Python Interpreter`에 가서 기존에 만들었던 가상환경으로 설정해 준다.


### 5. Django 서버 실행하기
아래 명령어를 입력해 실행한 서버는 웹 브라우저에 `localhost:8000`으로 확인할 수 있다.
```bash
# 이때 manage.py가 있는 경로에서 해당 명령어를 실행시키거나 manage.py의 경로까지 전부 적어야 한다.
python manage.py runserver
```


## Django 명령어
| 명령어 | 설명 | 예시 |
| ------ | ---- | ---- |
| django-admin startporject 장고프로젝트명 | 프로젝트 생성 | |
| python manage.py runserver | manage.py가 있는 경로에서 실행하며, 서버를 켠다. | |