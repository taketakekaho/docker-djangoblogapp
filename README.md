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
## docker-compose起動
> docker-compose起動
```
docker-django_db_1 is up-to-date
Creating docker-django_web_1 ... done
Attaching to docker-django_db_1, docker-django_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".
db_1   |
db_1   | Data page checksums are disabled.
db_1   |
db_1   | fixing permissions on existing directory /var/lib/postgresql/data ... ok
db_1   | creating subdirectories ... ok
db_1   | selecting default max_connections ... 100
db_1   | selecting default shared_buffers ... 128MB
db_1   | selecting dynamic shared memory implementation ... posix
db_1   | creating configuration files ... ok
db_1   | running bootstrap script ... ok
db_1   | performing post-bootstrap initialization ... ok
db_1   | syncing data to disk ... ok
db_1   |
db_1   | Success. You can now start the database server using:
db_1   |
db_1   |     pg_ctl -D /var/lib/postgresql/data -l logfile start
db_1   |
db_1   |
db_1   | WARNING: enabling "trust" authentication for local connections
db_1   | You can change this by editing pg_hba.conf or using the option -A, or
db_1   | --auth-local and --auth-host, the next time you run initdb.
db_1   | ****************************************************
db_1   | WARNING: No password has been set for the database.
db_1   |          This will allow anyone with access to the
db_1   |          Postgres port to access your database. In
db_1   |          Docker's default configuration, this is
db_1   |          effectively any other container on the same
db_1   |          system.
db_1   |
db_1   |          Use "-e POSTGRES_PASSWORD=password" to set
db_1   |          it in "docker run".
db_1   | ****************************************************
db_1   | waiting for server to start....2019-06-02 15:56:02.211 UTC [42] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db_1   | 2019-06-02 15:56:02.235 UTC [43] LOG:  database system was shut down at 2019-06-02 15:56:01 UTC
db_1   | 2019-06-02 15:56:02.240 UTC [42] LOG:  database system is ready to accept connections
db_1   |  done
db_1   | server started
db_1   |
db_1   | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
db_1   |
db_1   | waiting for server to shut down....2019-06-02 15:56:02.276 UTC [42] LOG:  received fast shutdown request
db_1   | 2019-06-02 15:56:02.280 UTC [42] LOG:  aborting any active transactions
db_1   | 2019-06-02 15:56:02.282 UTC [42] LOG:  background worker "logical replication launcher" (PID 49) exited with exit code 1
db_1   | 2019-06-02 15:56:02.284 UTC [44] LOG:  shutting down
db_1   | 2019-06-02 15:56:02.327 UTC [42] LOG:  database system is shut down
db_1   |  done
db_1   | server stopped
db_1   |
db_1   | PostgreSQL init process complete; ready for start up.
db_1   |
db_1   | 2019-06-02 15:56:02.400 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
db_1   | 2019-06-02 15:56:02.400 UTC [1] LOG:  listening on IPv6 address "::", port 5432
db_1   | 2019-06-02 15:56:02.412 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db_1   | 2019-06-02 15:56:02.434 UTC [51] LOG:  database system was shut down at 2019-06-02 15:56:02 UTC
db_1   | 2019-06-02 15:56:02.440 UTC [1] LOG:  database system is ready to accept connections
web_1  | Performing system checks...
web_1  |
web_1  | System check identified no issues (0 silenced).
web_1  |
web_1  | You have 15 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
web_1  | Run 'python manage.py migrate' to apply them.
web_1  | June 02, 2019 - 16:23:32
web_1  | Django version 2.1.8, using settings 'example.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
```

止める時は[Ctrl]+C
```
racefully stopping... (press Ctrl+C again to force)
Stopping docker-django_web_1 ... done
Stopping docker-django_db_1  ... done
>
```

