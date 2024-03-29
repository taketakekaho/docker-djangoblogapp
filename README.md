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

# アプリ作成
> docker-compose run --rm web python manage.py startapp sample
```
Starting docker-django_db_1 ... done
```
### ファイル構成
```
docker-django
├── Dockerfile                      //Dockerfile：Python環境設定
├── docker-compose.yml              //docker-composeファイル：WEBサーバー、DBサーバー構築
├── requirements.txt                //パッケージインストールリスト
├── example                         //【プロジェクト】
│     ├── __init__.py               //pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
│     ├── __pycache__               //キャッシュ
│     ├── settings.py               //★全般設定ファイル
│     ├── urls.py                   //★URL設定ファイル
│     └── wsgi.py                   //汎用API
├── maange.py                       //コマンドラインユーティリティ　コマンド実行時に使用する
└── sample                          //【アプリケーション】
　　　 ├── admin.py                 //▲Django 管理画面モデル登録
　　　 ├── apps.py                  //アプリケーションモジュールのロード
　　　 ├── __init__.py              //pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 ├── migrations
　　　 │     ├── 0001_initial.py    //DBマイグレーションファイル
　　　 │     ├── __init__.py        //pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 │     └── __pycache__        //キャッシュ
　　　 ├── models.py                //★モデル：DBラッパークラス
```

## view変更
sample/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello world.")
```

sample/urls.py
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
example/urls.py
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('sample/', include('sample.urls')),
    path('admin/', admin.site.urls),
]
```
> docker-compose up     //http://localhost:8000/sample/ で"hello world"表示

## DB接続
### ファイル構成
```
docker-django
├── Dockerfile                                 #Dockerfile：Python環境設定
├── docker-compose.yml                         #docker-composeファイル：WEBサーバー、DBサーバー構築
├── requirements.txt                           #パッケージインストールリスト
├── example                                    #【プロジェクト】
│     ├── __init__.py                         #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
│     ├── __pycache__                         #キャッシュ
│     ├── settings.py  #★★★★全般設定ファイル
│     ├── urls.py                             #URL設定ファイル
│     └── wsgi.py                             #汎用API
├── maange.py                                  #コマンドラインユーティリティ　コマンド実行時に使用する
└── sample                                     #【アプリケーション】
　　　 ├── admin.py   #★★★★Django 管理画面モデル登録
　　　 ├── apps.py                             #アプリケーションモジュールのロード
　　　 ├── __init__.py                         #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 ├── migrations #★★★★自動生成場所
　　　 │     ├── 0001_initial.py              #DBマイグレーションファイル
　　　 │     ├── __init__.py                  #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 │     └── __pycache__                  #キャッシュ
　　　 ├── models.py  #★★★★モデル：DBラッパークラス
```
### モデル作成
sample/models.py
```
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
### モデル有効化
example/settings.py
```
INSTALLED_APPS = [
    'sample.apps.SampleConfig',    #ここを追加
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
### マイグレーションファイル作成
> docker-compose run --rm web python manage.py makemigrations sample
```
Starting docker-django_db_1 ... done
Migrations for 'sample':
  sample/migrations/0001_initial.py
    - Create model Choice
    - Create model Question
    - Add field question to choice
>
```
### SQL確認用
> docker-compose run --rm web python manage.py sqlmigrate sample 0001
```
Starting docker-django_db_1 ... done
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "sample_choice" ("id" serial NOT NULL PRIMARY KEY, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE "sample_question" ("id" serial NOT NULL PRIMARY KEY, "question_text" varchar(200) NOT NULL, "pub_date" timestamp with time zone NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE "sample_choice" ADD COLUMN "question_id" integer NOT NULL;
CREATE INDEX "sample_choice_question_id_1e820721" ON "sample_choice" ("question_id");
ALTER TABLE "sample_choice" ADD CONSTRAINT "sample_choice_question_id_1e820721_fk_sample_question_id" FOREIGN KEY ("question_id") REFERENCES "sample_question" ("id") DEFERRABLE INITIALLY DEFERRED;COMMIT;
```
### マイグレーション
> docker-compose run --rm web python manage.py migrate
```
Starting docker-django_db_1 ... done
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sample, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying sample.0001_initial... OK
  Applying sessions.0001_initial... OK
```

## ユーザー作成
> docker-compose run --rm web python manage.py createsuperuser
```
Starting docker-django_db_1 ... done
Username (leave blank to use 'root'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.
```
## sampleアプリのモデルをadmin上で編集できるようにする
sample/admin.py
```
from django.contrib import admin

from .models import Question
from .models import Choice

admin.site.register(Question)
admin.site.register(Choice)
```
## 画面
### ファイル構成
```
docker-django
├── Dockerfile                                 #Dockerfile：Python環境設定
├── docker-compose.yml                         #docker-composeファイル：WEBサーバー、DBサーバー構築
├── requirements.txt                           #パッケージインストールリスト
├── example                                    #【プロジェクト】
│     ├── __init__.py                         #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
│     ├── __pycache__                         #キャッシュ
│     ├── settings.py                         #全般設定ファイル
│     ├── urls.py                             #URL設定ファイル
│     └── wsgi.py                             #汎用API
├── manange.py                                  #コマンドラインユーティリティ　コマンド実行時に使用する
└── sample                                     #【アプリケーション】
　　　 ├── admin.py                            #Django 管理画面モデル登録
　　　 ├── apps.py                             #アプリケーションモジュールのロード
　　　 ├── __init__.py                         #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 ├── migrations                          #自動生成場所
　　　 │     ├── 0001_initial.py              #DBマイグレーションファイル
　　　 │     ├── __init__.py                  #pythonスクリプトがあるディレクトリを表すまたは初期化時のimportモジュール
　　　 │     └── __pycache__                  #キャッシュ
　　　 ├── models.py                           #モデル：DBラッパークラス
　　　 ├── __pycache__                         #キャッシュ
　　　 ├── tests.py 	                        #アプリケーションのテストを書く場所
　　　 ├── templates                           #テンプレートエンジン
　　　 │     └── sample                       #テンプレート
　　　 │            ├── index.html  #★★★★  一覧画面
　　　 │            ├── detail.html  #★★★★ 詳細画面
　　　 │            └── results.html #★★★★ 結果画面
　　　 ├── urls.py  #★★★★URLの設定ファイル
　　　 └── views.py #★★★★ビュー：HTTPレスポンスやりとり
```

## index.html作成
sample/templates/sample/index.html
```
{% if latest_question_list %}
<ul>
{% for question in latest_question_list %}
    <li><a href="{% url 'sample:detail' question.id %}">{{ question.question_text }}</a></li>
{% endfor %}
</ul>
{% else %}
<p>No sample are available.</p>
{% endif %}
```

sample/templates/sample/detail.html
```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'sample:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

sample/templates/sample/results.html
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'sample:detail' question.id %}">Vote again?</a>
```
sample/views.py
```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question

class IndexView(generic.ListView):
    template_name = 'sample/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = 'sample/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'sample/results.html'

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'sample/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('sample:results', args=(question.id,)))
```
sample/urls.py
```
from django.urls import path

from . import views

app_name = 'sample'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
