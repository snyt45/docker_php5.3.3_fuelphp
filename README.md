# docker_php5.3.3_fuelphp
[【Docker環境構築】PHP5\.3\.3 \+ Apache \+ MySQL \+ phpMyAdminにFuelPHPをインストール \- Qiita](https://qiita.com/snyt45/items/b54ce2ac5be2a474d9b6)

# fuelphpインストールまでの流れ
## 1. git clone
```
git clone https://github.com/snyt45/docker_php5.3.3_fuelphp.git
cd docker_php5.3.3_fuelphp/
```

## 2. コンテナ作成と起動
```
docker-compose up -d
```
`docker ps`でコンテナの稼働状況を確認

## 3. Apacheの動作確認
http://localhost:8080/

## 4. MySQL + phpMyAdmin の確認
http://localhost:4040/

## 5. composerのインストール～furlphpのインストールまで
```
# コンテナID確認
docker ps

# phpコンテナに入る
docker exec -it docker_php533_fuelphp_php_1 /bin/bash

# composerのインストール
curl -sS https://getcomposer.org/installer | php

# composerをコマンドで扱うために環境変数で使えるディレクトリ配下に移動
mv composer.phar /usr/local/bin/composer

# composerのバージョン確認
composer --version

# fuel phpのoilコマンドインストール
curl https://get.fuelphp.com/oil | sh

# fuel phpのoilコマンドインストール確認
which oil

# fuel phpのインストール
oil create fuel-app

```

fuelphpをインストール後､ホスト側にfuel-appというフォルダが作成されていればOKです｡

http://localhost:8080/ にアクセスすると､まだApacheの画面が表示されると思います｡



## 6. oilコマンドについて
下記の2種類のoilコマンドがあります。

①/usr/local/bin/oil
→ curl https://get.fuelphp.com/oil | sh で追加

・どこでも使える

②fuel-app/oil
→ oil create fuel-app で追加

・fuel-app配下にいないと使えない。

## 7. httpd.confの修正

ホスト側のhttpd.confを下記のように変更してください｡

```httpd.conf
変更点
DocumentRoot "/app"
↓
DocumentRoot "/app/fuel-app/public"
```

以下のコマンドで設定を反映させます｡

```
docker-compose stop
docker-compose up -d
```


## 6. 反映確認
再度 http://localhost:8080/ にアクセスして､以下の画面が表示されれば成功です｡

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163887/584f6e04-8a32-98bf-f696-cad1160af4c6.png)


# とても参考
[PHP5\.3\.3環境を2017年に用意する方法 \- Qiita](https://qiita.com/suin/items/b13df0febf02a61cb5c5)
[docker\-compose で PHP7\.2 \+ Apache \+ MySQL \+ phpMyAdmin 環境を構築 \- Qiita](https://qiita.com/naente_dev/items/d259ea84c172deeff7d8)
[Docker 仮想CentOS6を動かす \- @//メモ](https://hondou.homedns.org/pukiwiki/index.php?Docker%20%B2%BE%C1%DBCentOS6%A4%F2%C6%B0%A4%AB%A4%B9)
