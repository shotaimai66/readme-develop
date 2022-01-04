# heroku✖︎docker

# 本番環境にあげる手順

1. エラーに立ち向かう決意をする
2. ローカルで開発環境を最初から構築してエラーが出ないか確認する
3. ローカルでアプリをproduction環境で立ち上げる
    1. 以下のようなエラーや壁が立ちはだかるがここをクリアする必要がある
        1. RAILS_ENV=production
        2. database.ymlの設定
        3. rails assets:precompileの設定
        4. その他。。。
4. 本番環境（heroku）へのデプロイに挑戦する
    1. 以下のような手順がある
        1. **事前にpuma.rbやdatabase.ymlの修正が必要（後続の資料を確認）**
        2. herokuにアプリを作成
        3. DBを作成
        4. herokuのアプリに環境変数設定
            1. DB関連
            2. RAILS_ENV
            3. RACK_ENV
            4. SECRET_KEY_BASE
            5. その他（GamilとかAPI関連のシークレットなど必要に応じて）
        5. Dockerイメージを作成して、herokuにpush(`heroku container:push web`)
        6. DBの設定（`heroku run rails db:migrate, heroku run rails db:seed`）
        7. herokuの環境にリリース（`heroku container:push web`）
        8. アセットのプリコンパイル（`heroku run rails assets:precompile`）
        9. 発生するエラーに後続の資料を参考に、対処する


# herokuのコンテナアプリ関連コマンド

```ruby
# herokuにログイン
heroku login

# herokuのコンテナレジストリにログイン
heroku container:login

# 新しいappを作成
heroku create
# --appオプションでアプリ名を指定できる
heroku create --app test-app

# イメージを作成してコンテナレジストリにpush
heroku container:push web --app test-app

# postgresqlの場合
heroku addons:create heroku-postgresql:hobby-dev --app test-app
# mysqlの場合
heroku addons:create cleardb:ignite --app test-app

# 環境変数の設定（herokuのコンソール画面からも設定できる）
# DBの接続情報はherokuのコンソールから確認できる
heroku config:add DB_NAME='heroku_5e3a3ed0ee0833b' --app test-app
heroku config:add USERNAME='bcae81c6135300' --app test-app
heroku config:add PASSWORD='c173bf24' --app test-app
heroku config:add HOSTNAME='us-cdbr-east-04.cleardb.com' --app test-app
heroku config:add RACK_ENV='production' --app test-app
heroku config:add RAILS_ENV='production' --app test-app
heroku config:add SECRET_KEY_BASE='85dedc5f3959aa6b91d71f60bb91673c' --app test-app

# DBセットアップ
heroku run rails db:migrate --app test-app
heroku run rails db:seed --app test-app

# イメージをherokuへデプロイ
heroku container:release web --app test-app

# assets:precompile
heroku run rails assets:precompile --app test-app

# 実際にアクセスしてアプリを確認してみる
heroku open --app test-app
```

# herokuのDB情報の確認

- 以下のコマンドか、herokuのアプリ管理画面の環境変数の設定する箇所に接続情報が書かれている


```ruby
# herokuのアプリ一覧
heroku apps

heroku pg:credentials:url --app tech-video-on-docker-on-heroku
```

# その他のコマンド

```ruby
# アプリの概要を表示
heroku info --app test-app

# コンフィグを表示
heroku config --app test-app

# herokuのログを確認
heroku logs --app test-app

# アプリの再起動
heroku restart --app test-app
```

# 各種エラーや環境変数の対応について

### 環境変数について

- db関連はdatabase.ymlに設定されている
- RAILS_ENV RACK_ENVはrailsアプリを本番環境で動かすときに必要
- SECRET_KEY_BASEも本番環境でアプリを動かすときに必要。クッキーの暗号化の際などに使用する。この値は、厳密な管理が必須。絶対にgitにあげないようにする必要がある。

### SECRET_KEY_BASEについて

- 新規に作成する際には、`master.key`と`credentials.yml.enc`を削除して、コンソールで以下のコマンドを打ち込む

```ruby
EDITOR="vi" bin/rails credentials:edit
```

- だがdockerでは、コンテナ内でコマンドを打つ必要がある。以下のコマンドでコンテナを起動してrailsコンテナの中に入る。その後上のコマンドを打ち込む

```ruby
docker-compose run web bash
```

- コンテナ内にvimがインストールされていないと一番上のコマンドは打ち込めない。なので、Dockerfileにviをインストールするように追記

```dockerfile
FROM ruby:3.0.2

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update -qq && apt-get install -y \
  nodejs \
  yarn \
  imagemagick \
  build-essential \
  libpq-dev \
  postgresql-client \
  vim                 # <=これを追記する

WORKDIR /app

COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

RUN bundle install

COPY . /app

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

CMD bash -c "rm -f tmp/pids/server.pid && bundle exec puma -C config/puma.rb"
```

### SECRET_KEY_BASEを本番環境の環境変数に設定

- SECRET_KEY_BASEを本番環境の環境変数に設定するときは、ローカルの情報を取得必要がある

```ruby
docker-compose run web bash
EDITOR="vi" bin/rails credentials:edit
```

- 上のコマンドを打ち込むと以下のような表示になるので、ここの `secret_key_base`の値を環境変数SECRET_KEY_BASEに設定する

```ruby
credentials.yml.enc
aws:
  access_key_id: 123
  #=> Rails.application.credentials.aws[:access_key_id]

secret_key_base: 8be8e637d755f79c799048bed8be0c...
#=> Rails.application.credentials.secret_key_base
```

### puma.rbの編集

- herokuの利用の際には以下のようにpuma.rbを編集する必要がある

```ruby
workers Integer(ENV.fetch("WEB_CONCURRENCY") { 2 })

max_threads_count = Integer(ENV.fetch("RAILS_MAX_THREADS") { 5 })
min_threads_count = Integer(ENV.fetch("RAILS_MIN_THREADS") { max_threads_count })
threads min_threads_count, max_threads_count

preload_app!

rackup      DefaultRackup
port        ENV.fetch("PORT") { 3000 }
environment ENV.fetch("RACK_ENV") { "development" }

on_worker_boot do
  ActiveRecord::Base.establish_connection
end
```

### herokuデプロイ時のエラー

- rails6のホスト名のエラー
    - 以下のようなエラーが出た時は、 `config/environments/production.rb`　に以下のように自身のherokuのURLを追記する。

    ```ruby
    Rails.application.configure do
    ## 上は省略
    	config.hosts << "stormy-fjord-62228.herokuapp.com" # <=これを追記
    ## したは省略
    end
    ```


### 本番環境のdatabase.ymlの修正

productionの部分を環境変数に変えて、herokuに事前にセットする

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  user: postgres
  port: 5432
  password:
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: product_register_development

test:
  <<: *default
  database: product_register_test

production:
  <<: *default
  host: <%= ENV['DB_HOST'] %>
  database: <%= ENV['DB_NAME'] %>
  user: <%= ENV['DB_USER'] %>
  password: <%= ENV['DB_PASS'] %>
```

### 本番環境のassets読み込み関連

- 本番環境では事前に `heroku run rails assets:precompile`するので、以下の設定が必要。

```ruby
config.assets.compile = true #動的コンパイルを有効化
config.assets.css_compressor = :sass # sass-rails gemを使用している場合コメントアウトを外す
config.public_file_server.enabled = true # publicディレクトリ以下のアセットを返す設定
```

### 自動デプロイ参考記事

[https://qiita.com/gakinchoy7/items/ae31107ef56efb16fe7e](https://qiita.com/gakinchoy7/items/ae31107ef56efb16fe7e)

### heroku run rails assets:precompileがherokuのサーバーのメモリ不足でエラーになる場合


- ローカル環境でdockerイメージをbuildする際に、assets:precompileを行う必要がある。

```dockerfile
# ここを追加
RUN SECRET_KEY_BASE=placeholder bundle exec rails assets:precompile \
 && yarn cache clean \
 && rm -rf node_modules tmp/cache
```

```dockerfile
FROM ruby:3.0.2

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update -qq && apt-get install -y \
  nodejs \
  yarn \
  imagemagick \
  build-essential \
  libpq-dev \
  postgresql-client \
  vim                 # <=これを追記する

WORKDIR /app

COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

RUN bundle install

COPY . /app

# アセットのプリコンパイル （以下を追加する）
RUN SECRET_KEY_BASE=placeholder bundle exec rails assets:precompile \
 && yarn cache clean \
 && rm -rf node_modules tmp/cache

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Configure the main process to run when running the image
CMD ["rails", "server", "-b", "0.0.0.0"]
```

### webpackerのエラーが出る時（rails6とかは出るかも）

- これもrails assets:precompileができていない。だが、してもcssだけコンパイルされない現象が起きることがある。以下の対応を行い再度`heroku container:release`を行う


## 本番環境のエラー対応その他

- ローカルでproduction環境を指定して起動させれば同じエラーが発生することが多い。

```dockerfile
# 環境変数のRAILS_ENV=productionを指定すればproduction環境で起動できる
rails s -e production

# Dockerの場合
version: "3"

volumes:
  db-data:
  bundle:
  node_modules:

services:
  web:
    build: .
    command: >
      bash -c "rm -f tmp/pids/server.pid &&
               bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - '3000:3000'
    volumes:
      - '.:/app'
      - 'bundle:/usr/local/bundle:cached'
      - 'node_modules:/app/node_modules'
      - '/app/vendor'
      - '/app/tmp'
      - '/app/log'
      - '/app/.git'
    environment:
      - 'DATABASE_PASSWORD=postgres'
      - 'RAILS_ENV=production'# <=ここで指定する
    tty: true
    stdin_open: true
    depends_on:
      - db
    links:
      - db

  db:
    image: postgres
    environment:
      - 'POSTGRES_USER=postgres'
      - 'POSTGRES_PASSWORD=postgres'
    volumes:
      - 'db-data:/var/lib/postgresql/data'
```

- あとdatabaseだけは開発環境のDBに繋ぐように設定すればOK

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  user: postgres
  port: 5432
  password: postgres
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: product_register_development

test:
  <<: *default
  database: product_register_test

# for heroku deployment 以下の環境変数を追加
production:
  <<: *default
  database: product_register_development
  # <<: *default
  # database: <%= ENV['APP_DATABASE'] %>
  # username: <%= ENV['APP_DATABASE_USERNAME'] %>
  # password: <%= ENV['APP_DATABASE_PASSWORD'] %>
  # host: <%= ENV['APP_DATABASE_HOST'] %>
  # secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>  #for devise / credentials.yml.enc で確認できる
```

# 開発用と本番用のDockerfileを分ける

- なぜ必要?
    - 本番用のDockerfileでは、assets:precompileなどの重い処理を記載することがあるので、開発しているときにdocker-compose buildなどが時間がかかってしまう。
- どうする？
    - ファイルを分けると解決（以下のような名前にする）
        - 開発用　`DEV.Dockerfile`
        - 本番用　`Dockerfile`
    - docker-compose.ymlの設定


- どうなる？
    - 開発時のdocker-comopose buildやupなどでは開発用のDockerfileをシステムが見るようになり、herokuへのデプロイ時には、本番用のDockerfileを見るようになります。