# rails7テンプレリポジトリデプロイ方法
### herokuアプリの作成
```ruby
# herokuへのログイン
heroku login

# heroku.yml​ マニフェスト内で定義されている setup​ セクションからアプリを作成するには、beta​ アップデートチャネルから heroku-manifest​ プラグインをインストールしてください。
heroku update beta
heroku plugins:install @heroku-cli/plugin-manifest

# いつでも、stable アップデートストリームに戻して、プラグインを削除できます。
heroku update stable
heroku plugins:remove manifest

# 次に、--manifest​ フラグを使用してアプリを作成します。 アプリのスタックは自動的に container​ に設定されます。
heroku create 好きなアプリ名 --manifest

# herokuリポジトリへのpush(dockerイメージのビルド、デプロイをやってくれる)
git push heroku main
```

### RAILS_MASTER_KEYの生成
SECRET_KEY_BASEについて
- 新規に作成する際には、master.keyとcredentials.yml.encを削除して、コンソールで以下のコマンドでコンテナの中に入って、
```
docker-compose run web bash
```
- 以下のコマンドを打ち込みmaster.keyを作成する
```
EDITOR="vi" bin/rails credentials:edit
```

### 環境変数の設定とdb操作
```ruby

# 環境変数の設定
heroku config:add RAILS_MASTER_KEY='85dedc5f3959aa6b91d71f60bb91673c' --app 好きなアプリ名
heroku config:add RAILS_APP_HOST='好きなアプリ名.herokuapp.com' --app 好きなアプリ名

# rails db:migrate db:seed
heroku run rake db:migrate db:seed --app 好きなアプリ名

# アプリを確認
heroku open --app 好きなアプリ名

# その後は開発をしてpush
# herokuリポジトリへのpush(dockerイメージのビルド、デプロイをやってくれる)
git add -A
git commit -m"開発"
git push heroku master

```

以上でデプロイができる