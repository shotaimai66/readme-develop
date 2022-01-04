# Dcoker環境を準備する

## Dockerのインストール

### Windowsの場合

[こちらの記事を参考にDocker Desktop for Windows](https://qiita.com/zaki-lknr/items/db99909ba1eb27803456)をダウンロードする

### Mac OSの場合

**OS X Yosemite 10.10.3 以降**：[Docker CE for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

**それ以前**：[Docker Toolbox](https://docs.docker.com/toolbox/overview/#whats-in-the-box)

![Docker Desktop for Mac 2021-06-15 19-50-36](/Users/imaishota/Downloads/Docker Desktop for Mac 2021-06-15 19-50-36.png)

macをインストールして、起動すると

![image](images/Monosnap%202021-06-15%2019-53-27.png)



シェルでDockerのバージョンを確認する（インストールされたことを確認）

```
[imaishota@MacBook-Pro-3 ~]$ docker -v
Docker version 20.10.7, build f0df350
```







## docker-composeのダウンロード

### Windowsの場合

Dockerをインストールすると、docker-composeも一緒にインストールされるので、何もする必要はありません

### macの場合

ターミナルで、docker-composeの`1.29.2`のバージョンをインストールする

```
sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

コマンドに実行権限を付与

```
sudo chmod +x /usr/local/bin/docker-compose
```

docker-composeのバージョンを確認する

```
[imaishota@MacBook-Pro-3 ~]$ docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```



