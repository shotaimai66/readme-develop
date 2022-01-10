# Dcoker環境を準備する

## 目次
- [Dcoker環境を準備する](#dcoker環境を準備する)
  - [目次](#目次)
  - [🌟自分のPCにDockerをインストールして使いたい場合](#自分のpcにdockerをインストールして使いたい場合)
    - [`<Dockerの要求するPCスペック>`](#dockerの要求するpcスペック)
    - [`<Dockerのインストール>`](#dockerのインストール)
    - [`<docker-composeのダウンロード>`](#docker-composeのダウンロード)
  - [🌟 cloud9を使って開発を行う場合](#-cloud9を使って開発を行う場合)

---
## 🌟自分のPCにDockerをインストールして使いたい場合
### `<Dockerの要求するPCスペック>`
- [Dockerのシステム要件](/Dockerのシステム要件.md)

### `<Dockerのインストール>`
- Windowsの場合
  - [こちらの記事を参考にDocker Desktop for Windows](https://qiita.com/zaki-lknr/items/db99909ba1eb27803456)をダウンロードする
- Mac OSの場合
  - **OS X Yosemite 10.10.3 以降**：[Docker CE for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
  - **それ以前**：[Docker Toolbox](https://docs.docker.com/toolbox/overview/#whats-in-the-box)

macをインストールして、起動すると以下のようなdockerが起動状態になる(macの場合)
![image](images/Monosnap%202021-06-15%2019-53-27.png)



- ターミナルでDockerのバージョンを確認する（インストールされたことを確認）

```
docker -v
```
- 以下のように表示されていればdockerのインストールはOK！
```
Docker version 20.10.7, build f0df350
```

### `<docker-composeのダウンロード>`
- Windowsの場合
  - Dockerをインストールすると、docker-composeも一緒にインストールされるので、何もする必要はありません
- macの場合
  - ターミナルで、docker-composeの`1.29.2`のバージョンをインストールする
  ```
  sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  ```
  - コマンドに実行権限を付与
  ```
  sudo chmod +x /usr/local/bin/docker-compose
  ```
  - docker-composeのバージョンを確認する
  ```
  [imaishota@MacBook-Pro-3 ~]$ docker-compose -v
  docker-compose version 1.29.2, build 5becea4c
  ```

---

## 🌟 cloud9を使って開発を行う場合
- インスタンスの設定とdocker-composeの設定
  1. [cloud9:インスタンスの起動](https://www.youtube.com/watch?v=Z6FO3uoJUJc)
  2. [cloud9:インスタンスの設定](https://www.youtube.com/watch?v=2pqjOTmucRM)
  3. [cloud9:docker-composeコマンドのインストール](https://www.youtube.com/watch?v=A1_2yKtYR7I)
- ssh接続の設定
  1. [参考記事](https://qiita.com/Tech_Leaders/private/41dc06a99436bafd3e11)
  2. リポジトリの環境構築に戻る
