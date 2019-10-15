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

## 3. MySQL + phpMyAdmin の確認
ちょっと時間を置いてアクセス。

http://localhost:4040/

## 4. composer、oilコマンドのインストール
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
```

## 5. fuelphp1.7.3のインストール

```
# fuel phpのインストール
composer create-project fuel/fuel:dev-1.7/master sample
```

インストール中に、Githubでアクセストークンを取得してとエラーが出るはずなので、
指定されてるURLでアクセストークンを発行して、トークンを貼り付けてEnter。

fuelphpをインストール後､ホスト側にsampleというフォルダが作成されていればOKです｡

以前は、このコマンドで1.7.2がインストールされるみたいですが現在は1.7.3がインストールされるみたいです。

# fuelphpの作業準備
## 1. docsディレクトリ削除
sampleの中にfuel、public、docsフォルダが作成される。
docsフォルダは特に必要ないので削除

## 2. apacheの公開ディレクトリにpublicフォルダのシンボリックリンクを作成
httd.confでDocumentRootが/var/www/html/publicを設定している。

fuelphpのpublicフォルダが公開フォルダになるので、
/var/www/html/にpublicフォルダのシンボリックリンクを貼る。

```
ln -s /app/sample/public/ /var/www/html/
```

http://localhost:8080/ にアクセスしてfuelphpの画面が表示されればOK

## 3. ドキュメントルートの設定
/var/www/html/publicから、/sample/publicのindex.phpをみる。
index.phpの下記の場所のディレクトリを通常は変更する必要があるが、
pubilc本体の場所に変更はないので、特に変更の必要はない。

```
/**
 * Path to the application directory.
 */
define('APPPATH', realpath(__DIR__.'/../fuel/app/').DIRECTORY_SEPARATOR);

/**
 * Path to the default packages directory.
 */
define('PKGPATH', realpath(__DIR__.'/../fuel/packages/').DIRECTORY_SEPARATOR);

/**
 * The path to the framework core.
 */
define('COREPATH', realpath(__DIR__.'/../fuel/core/').DIRECTORY_SEPARATOR);
```

## 4. ORMパッケージを有効にする。
/sample/fuel/app/config/config.phpを以下のようにコメントアウトを外す。
```
	/**************************************************************************/
	/* Always Load                                                            */
	/**************************************************************************/
	'always_load'  => array(

		/**
		 * These packages are loaded on Fuel's startup.
		 * You can specify them in the following manner:
		 *
		 * array('auth'); // This will assume the packages are in PKGPATH
		 *
		 * // Use this format to specify the path to the package explicitly
		 * array(
		 *     array('auth'	=> PKGPATH.'auth/')
		 * );
		 */
		'packages'  => array(
			'orm',
		),

		/**
		 * These modules are always loaded on Fuel's startup. You can specify them
		 * in the following manner:
		 *
		 * array('module_name');
		 *
		 * A path must be set in module_paths for this to work.
		 */
		// 'modules'  => array(),

		/**
		 * Classes to autoload & initialize even when not used
		 */
		// 'classes'  => array(),

		/**
		 * Configs to autoload
		 *
		 * Examples: if you want to load 'session' config into a group 'session' you only have to
		 * add 'session'. If you want to add it to another group (example: 'auth') you have to
		 * add it like 'session' => 'auth'.
		 * If you don't want the config in a group use null as groupname.
		 */
		// 'config'  => array(),

		/**
		 * Language files to autoload
		 *
		 * Examples: if you want to load 'validation' lang into a group 'validation' you only have to
		 * add 'validation'. If you want to add it to another group (example: 'forms') you have to
		 * add it like 'validation' => 'forms'.
		 * If you don't want the lang in a group use null as groupname.
		 */
		// 'language'  => array(),
	),
```

## 5. データベースの準備
MySQL にデータベースを作成します。
http://localhost:4040/ のSQLで以下のコマンドを実行。

```
CREATE DATABASE `fuel_dev` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

必要に応じて、/sample/fuel/app/config/development/db.phpの設定を変更する。

マイグレーション時、dsnのdbnameで指定したデータベースにテーブルが作成されます。

## 6. モデル作成
必要なモデルの作成とマイグレーションの実行。

```
cd /app/sample
php oil generate model hoge name:varchar[255] sex:int del_flg:bool
php oil refine migrate
```

http://localhost:4040/ にアクセスして、hogesテーブルができていることを確認します。

## 7. コントローラー作成
```
cd /app/sample
php oil generate controller example index add
```

## 8. TOPページの設定
/sample/fuel/app/config/routes.phpで、TOPページになるコントローラーを指定。

```
<?php
return array(
    '_root_' => 'example/index',
    ...
);
```

## 9. 反映確認
再度 http://localhost:8080/ にアクセスして､以下の画面が表示されれば成功です｡

![image.png (38.9 kB)](https://img.esa.io/uploads/production/attachments/12444/2019/10/14/57955/2b8157d5-ea53-4b65-a97d-232028b6c5a7.png)

# VScodeでxdebugを使う
[Docker × Visual Studio CodeでPHP開発【デバッグ実行】 \- Qiita](https://qiita.com/MasanoriIwakura/items/a310c75e6c5b347adf37)

## 1. VScodeの拡張機能「PHP Debug」をインストール

## 2. /sample/public/index.phpを下記の通りに編集
```
<?php

echo 'Hello World<br>';

// hostname, user, password, db name
$mysqli = new mysqli('mysql', 'root', 'root', 'fuel_dev');

if ($mysqli->connect_error) {
  echo $mysqli->connect_error;
  exit();
} else {
    $mysqli->set_charset("utf8");
}

$sql = "SELECT id, name FROM test";
if ($result = $mysqli->query($sql)) {
    while ($row = $result->fetch_assoc()) {
        echo "ID:" . $row["id"] . " NAME:" . $row["name"] . "<br>";
    }
    $result->close();
}

$mysqli->close();
```

## 3. VScodeでindex.phpを開き、止めたい箇所でブレークポイント追加

## 4. VScode > 左の虫ボタン > 歯車 > PHPを選択 > Listen for XDebugを指定して、▶マークをからデバッグを実行

## 5. http://localhost:8080/ にアクセスして、ブレークポイントで止まれば完了！


## oilコマンドについて
ちょっと混乱したのでメモ。
下記の2種類のoilコマンドがあります。

①/usr/local/bin/oil
→ curl https://get.fuelphp.com/oil | sh で追加

・どこでも使える

②fuel-app/oil
→ oil create fuel-app で追加

・fuel-app配下にいないと使えない。

# とても参考
[PHP5\.3\.3環境を2017年に用意する方法 \- Qiita](https://qiita.com/suin/items/b13df0febf02a61cb5c5)

[docker\-compose で PHP7\.2 \+ Apache \+ MySQL \+ phpMyAdmin 環境を構築 \- Qiita](https://qiita.com/naente_dev/items/d259ea84c172deeff7d8)

[Docker 仮想CentOS6を動かす \- @//メモ](https://hondou.homedns.org/pukiwiki/index.php?Docker%20%B2%BE%C1%DBCentOS6%A4%F2%C6%B0%A4%AB%A4%B9)

[FuelPHP 1\.7\.2のComposerによるインストール — A Day in Serenity \(Reloaded\) — PHP, FuelPHP, Linux or something](http://blog.a-way-out.net/blog/2014/07/14/fuelphp-1-7-2-composer-installation/)

[FuelPHPのインストールから開発までの流れをおさらい \- BTT's blog](http://btt.hatenablog.com/entry/2012/10/31/004104)
