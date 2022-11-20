# SlackAnalytics環境構築手順
SlackAnalyticsBackEnd(Django REST API)とSlackAnalyticsFrontEnd(React)の環境構築手順です。

## 0. node・docker環境の構築
本サービスはnode・Dockerを利用しています。  
[node](https://nodejs.org/ja/download/releases/)のversion16をインストールしてください。
また下記サイトよりDocker Desktopをインストールしてください。
[Docker Desktop](https://www.docker.com/products/docker-desktop/)

## 1. リポジトリの複製
SlackAnalyticsディレクトリを作成してください。  
以下の二つのリポジトリを複製し、SlackAnalytics内に配置してください。

1. [SlackAnalyticsFrontEnd](https://github.com/k1e1n04/SlackAnalyticsFrontEnd)
2. [SlackAnalyticsBackEnd](https://github.com/k1e1n04/SlackAnalyticsBackEnd)

## 2. docker-composeファイルの作成
SlackAnalyticsディレクトリ内にdocker-compose.ymlを作成し、以下を記載してください。
``` docker-compose.yml
version: "3"

services:
  db:
    image: mysql:5.7
    platform: linux/amd64
    restart: always
    command: >
      mysqld --innodb_use_native_aio=0 &&
      mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment: 
      MYSQL_ROOT_PASSWORD: {MYSQL_ROOT_PASSWORD}自分で設定
      MYSQL_DATABASE: {MYSQL_DATABASE}自分で設定
      MYSQL_USER: {MYSQL_USER}自分で設定
      MYSQL_PASSWORD: {MYSQL_PASSWORD}自分で設定
      MYSQL_ROOT_HOST: '%'
      TZ: 'Asia/Tokyo'
    volumes: 
      - ./mysql/data:/var/lib/mysql
      - ./mysql/my.conf:/etc/mysql/conf.d/my.cnf
      - ./mysql/sql:/docker-entrypoint-initdb.d
    ports: 
      - 3306:3306
  api:
    image: slackanalytics-api
    tty: true
    restart: always
    build:
      context: .
      dockerfile: ./SlackAnalyticsBackEnd/Dockerfile
    depends_on:
      - db
    ports:
      - "8000:8000"
    volumes:
      - ./SlackAnalyticsBackEnd/django-api:/django-api
    command: >
            sh -c "/wait &&
            python manage.py migrate --fake-initial &&
            python manage.py runserver 0.0.0.0:8000"
    environment:
      WAIT_HOSTS: db:3306
      WAIT_TIMEOUT: 120 # dbの構築待機のタイムアウト
  front:
    image: slackanalytics-front
    build:
      context: .
      dockerfile: ./SlackAnalyticsFrontEnd/Dockerfile
    restart: always
    volumes:
      - ./SlackAnalyticsFrontEnd/node:/usr/src/app:cached
    command: sh -c "cd slackanalytics_front && yarn start"
    ports:
      - "3000:3000"
volumes:
    mysql:
      driver: local
```

## 3. BackEndの.env作成
API側の環境変数を設定します。
1. SlackAnalyticsBackEnd/django-api/.env.sampleのコピーを作成し".env"としてください。
2. 以下の項目を変更します。
```
SECRET_KEY="" # DjangoのSECRET_KEY
MYSQL_USER="" # docker-composeで設定したもの
MYSQL_PASSWORD=""　# docker-composeで設定したもの
MYSQL_ROOT_PASSWORD="" # docker-composeで設定したもの
DB_NAME="" # docker-composeで設定したもの
```
Djangoのシークレットキーの作成は[こちら](https://blog.kyanny.me/entry/2021/01/27/033507)を参考にしてください。

## 4. フロントエンドの依存関係のあるモジュールのインストール
SlackAnalytics/SlackAnalyticsFrontEnd/node/slackanalytics_frontディレクトリに移動し、下記のコマンドを実行してください。
```
npm install
```
プロジェクトと依存関係のあるモジュールをインストールできます。

## 5. Dockerイメージのビルド
SlackAnalyticsディレクトリ内で以下のコマンドを実行してください。
```
docker-compose build
```
MacOSの場合はDocker Desktopのインストール時にDocker Composeもインストールされています。もしdocker-composeコマンドが利用できない場合は別途インストールしてください。

## 6. mysqlディレクトリの作成
SlackAnalyticsディレクトリ内に下記の構成でディレクトリを作成してください。
```
SlackAnalytics
 |---mysql
        |---data # MySQLのデータ永続化用
        |---sql # MySQL 起動時の初期化スクリプト置き場
```

## 7. MySQLの設定ファイルの作成
作成したmysqlディレクトリ内に「my.conf」というファイルを作成してください。中身は以下の通りです。
``` my.conf
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4   
```

## 8. 作成したイメージの起動
SlackAnalyticsディレクトリ内で以下のコマンドを実行してください。
※ MySQLのポートは3306番を使用しています。ローカル環境でMySQLを起動している場合は停止してください。
```
docker-compose up -d
```
今回はデーモンで起動します。
実行後下記コマンドを実行し、同じ結果になれば起動が完了しています。
```
docker-compose ps
```
```
docker-compose ps   
NAME                     COMMAND                  SERVICE             STATUS              PORTS
slackanalytics-api-1     "sh -c '/wait && pyt…"   api                 running             0.0.0.0:8000->8000/tcp
slackanalytics-db-1      "docker-entrypoint.s…"   db                  running             0.0.0.0:3306->3306/tcp, 33060/tcp
slackanalytics-front-1   "docker-entrypoint.s…"   front               running             0.0.0.0:3000->3000/tcp
```

## 9. サーバーの確認
以下の二つのリンクにアクセスしてください。  
1. [http://localhost:8000/admin](http://localhost:8000/admin)
2. [http://localhost:3000/](http://localhost:3000/)  
「このサイトにアクセスできません」という表示が出なければ環境構築は完了です。

## 10. (初回のみ)データベースに初期データを投入
下記のコマンドを実行し、APIコンテナの中に入ってください。
```
docker exec -it slackanalytics-api-1 /bin/bash
```
その後下記のコマンドを実行しOrganizationとBaseの初期データを投入します。
```
python manage.py loaddata ./analytics/fixtures/seed_data.json
```

## 11. APIのスーパーユーザーを作成します。
APIコンテナの中に入った状態で下記のコマンドを入力し、スーパーユーザーを作成してください。(emailとpasswordはお好きな値を入力してください。)
```
python manage.py createsuperuser
```
上記完了後[http://localhost:8000/admin](http://localhost:8000/admin)にアクセスすることでアドミンページにログインできます。

