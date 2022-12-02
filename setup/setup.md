# SlackAnalytics環境構築手順
SlackAnalyticsAPI(Express)とSlackAnalyticsFrontEnd(React)の環境構築手順です。  

## 0. node・docker環境の構築
本サービスはnode・Dockerを利用しています。  
[node](https://nodejs.org/ja/download/releases/)のversion16をインストールしてください。
また下記サイトよりDocker Desktopをインストールしてください。
[Docker Desktop](https://www.docker.com/products/docker-desktop/)

## 1. リポジトリの複製
SlackAnalyticsディレクトリを作成してください。  
以下のリポジトリを複製し、SlackAnalytics内に配置してください。

1. [SlackAnalyticsFrontEnd](https://github.com/k1e1n04/SlackAnalyticsFrontEnd)
2. [SlackAnalyticsAPI](https://github.com/k1e1n04/SlackAnalyticsAPI)
3. [SlackAnalyticsAPI_Nginx](https://github.com/k1e1n04/SlackAnalyticsAPI_Nginx)
4. [SlackAnalyticsFrontEnd_Nginx](https://github.com/k1e1n04/SlackAnalyticsFrontEnd_Nginx)

## 2. docker-composeファイルの作成
SlackAnalyticsディレクトリ内にdocker-compose.ymlを作成し、以下を記載してください。
``` docker-compose.yml
version: "3"

services:
  db:
    image: mysql:5.7
    container_name: mysql
    platform: linux/amd64
    restart: always
    command: >
      mysqld --innodb_use_native_aio=0 &&
      mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment: 
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: slackanalytics
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_HOST: '%'
      TZ: 'Asia/Tokyo'
    volumes: 
      - ./mysql/data:/var/lib/mysql
      - ./mysql/my.conf:/etc/mysql/conf.d/my.cnf
      - ./mysql/sql:/docker-entrypoint-initdb.d
    ports: 
      - 3306:3306
    networks:
    - api_network
  api:
    image: slackanalytics-api
    container_name: slackanalytics-api
    tty: true
    restart: always
    environment: 
      DB_NAME: slackanalytics
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      WAIT_HOSTS: db:3306
      WAIT_TIMEOUT: 120 # dbの構築を120秒間待機する
    build:
      context: .
      dockerfile: ./SlackAnalyticsAPI/express_api/Dockerfile
    depends_on:
      - db
    volumes:
      - ./SlackAnalyticsAPI/express_api:/usr/src/app
      - ./SlackAnalyticsAPI/express_api/node_modules:/usr/src/app/node_modules
    networks:
      - api_network
  api-server:
    container_name: nginx_api
    build:
      context: ./SlackAnalyticsAPI_Nginx/.
      dockerfile: Dockerfile.dev
    ports:
      - "8080:80"
    depends_on:
      - api
    networks:
      - api_network
  front:
    image: slackanalytics-front
    container_name: slackanalytics-front
    build:
      context: .
      dockerfile: ./SlackAnalyticsFrontEnd/Dockerfile
    restart: always
    volumes:
      - ./SlackAnalyticsFrontEnd/node:/usr/src/app:cached
    command: sh -c "cd slackanalytics_front && yarn start"
    ports:
      - "3000:3000"
    networks:
      - front_network
  frontend-server:
    container_name: nginx_front
    build:
      context: ./SlackAnalyticsFrontEnd_Nginx/.
      dockerfile: Dockerfile.dev
    ports:
      - "4200:80"
    depends_on:
      - front
    networks:
      - front_network
    environment:
      WAIT_HOSTS: front:3000
      WAIT_TIMEOUT: 120 # reactの構築を120秒間待機する
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    depends_on:
      - db
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOSTS=db
      - PMA_USER=root
      - PMA_PASSWORD=password
    ports:
      - "81:80"
    volumes:
      - ./phpmyadmin/sessions:/sessions
    networks:
      - api_network
networks:
  # nginx + apiのネットワーク
  api_network:
    driver: bridge
  # nginx + reactのネットワーク
  front_network:
    driver: bridge
volumes:
    mysql:
      driver: local
```

## 3. FrontEndの.env作成
Front側の環境変数を設定します。
SlackAnalyticsFrontEnd/node/slackanalytics_front/.env.sampleのコピーを作成し".env"としてください。

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

## 8. phpmyadminディレクトリの作成
SlackAnalyticsディレクトリ内に下記の構成でディレクトリを作成してください。
```
SlackAnalytics
 |---phpmyadmin
```
その後phpmyadminディレクトリ内に"sessions"という名前でファイルを作成してください。

## 9. 作成したイメージの起動
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
NAME                          COMMAND                  SERVICE             STATUS              PORTS
mysql                         "docker-entrypoint.s…"   db                  running             0.0.0.0:3306->3306/tcp, 33060/tcp
nginx_api                     "nginx -g 'daemon of…"   api-server          running             0.0.0.0:8080->80/tcp
nginx_front                   "nginx -g 'daemon of…"   frontend-server     running             0.0.0.0:4200->80/tcp
slackanalytics-api            "docker-entrypoint.s…"   api                 running             8000/tcp
slackanalytics-front          "docker-entrypoint.s…"   front               running             0.0.0.0:3000->3000/tcp
slackanalytics-phpmyadmin-1   "/docker-entrypoint.…"   phpmyadmin          running             0.0.0.0:81->80/tcp
```

## 10. サーバーの確認
以下の二つのリンクにアクセスしてください。  
1. [API](http://localhost:8080/)
2. [FrontEnd](http://localhost:4200/)  
3. [phpmyadmin](http://localhost:81/) 
「このサイトにアクセスできません」という表示が出なければ環境構築は完了です。


