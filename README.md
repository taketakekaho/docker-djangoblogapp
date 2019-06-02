# docker-djangoblogapp

## Docker環境
> docker -v
Docker version 18.09.2, build 6247962
> docker-compose -v
docker-compose version 1.23.2, build 1110ad01

## Docker関連ファイル作成
> touch Dockerfile
```
FROM python:3.5
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```
> touch docker-compose.yml
```
Django>=2.1,<2.2
psycopg2
```
> touch requirements.txt
```
version: '3'

services:
  db:
    image: postgres
  web:
    build: .
    command: python3 manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
```

## Djangoプロジェクト作成
> docker-compose run --rm web django-admin.py startproject example .
```
Creating network "docker-django_default" with the default driver
Creating docker-django_db_1 ... done
Building web
Step 1/7 : FROM python:3.5
 ---> 2bf898387bbd
Step 2/7 : ENV PYTHONUNBUFFERED 1
 ---> Using cache
 ---> c190ee1a2a9f
Step 3/7 : RUN mkdir /code
 ---> Using cache
 ---> 56f3907454c7
Step 4/7 : WORKDIR /code
 ---> Using cache
 ---> 3b6e2f1dfff6
Step 5/7 : ADD requirements.txt /code/
 ---> Using cache
 ---> 965a1a72867b
Step 6/7 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> f25a6288a90b
Step 7/7 : ADD . /code/
 ---> ef7287f865b8
Successfully built ef7287f865b8
Successfully tagged docker-django_web:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
```
### ファイル構成
```
docker-django
├── Dockerfile              //Dockerfile：Python環境設定
├── docker-compose.yml      //docker-composeファイル：WEBサーバー、DBサーバー構築
├── requirements.txt        //パッケージインストールリスト
├── example                 //【プロジェクト】
│     ├── __init__.py       //pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
│     ├── __pycache__       //キャッシュ
│     ├── settings.py       //★全般設定ファイル
│     ├── urls.py           //★URL設定ファイル
│     └── wsgi.py           //汎用API
└── manage.py               //Djangoに対する様々な操作を行うためのコマンドラインユーティリティ
```
## DB接続、ALLOWED_HOST設定
example/settings.py
```
♯28行目
#公開するドメイン名を設定（今回はワイルドカード）
ALLOWED_HOSTS = ["*"]

#76行目
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```
