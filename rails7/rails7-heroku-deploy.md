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
# mainブランチ以外のブランチをデプロイする場合
git push heroku <現在いるブランチ名>:master
```

### RAILS_MASTER_KEYの生成(新規に生成する場合のみ必要な手順です)
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
heroku config:add RAILS_MASTER_KEY='config/master.keyの値をここに入れる' --app 好きなアプリ名
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

# その他

### heroku create 好きなアプリ名 --manifestについて
- このコマンドの`--manifest`オプションは手順にもあるように、heroku-manifest​ プラグインをインストールする必要があるので注意が必要。
- またこのコマンドによって、heroku側でheroku.ymlの設定から自動的にDockerを使ってアプリ設定になるので、`git push heroku main`でDockerfileのビルドが始まり、デプロイされる。

### dbの設定について
- heroku.ymlの設定で、`heroku create 好きなアプリ名 --manifest`コマンドで自動的に、mysql(jawsdb)が作られる。mysqlが作られると同時に、herokuの環境変数に、JAWSDB_URLという環境変数がセットされる。その環境変数をdatabase.ymlで使用しているので特にDB関連の環境変数の手動でのセットは必要ない。

### 実際の運用時の注意
- メールの設定やドメインの設定が必要になると思う


以上でデプロイができる