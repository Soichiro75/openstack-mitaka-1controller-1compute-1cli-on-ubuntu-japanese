# 02_サービス事前準備

## OpenStack パッケージのインストール

http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/environment-packages.html

### 既存パッケージ更新 と 自動アップデート無効化

- 既存パッケージの更新
[対象: controller01, compute01, cli01]

```
# apt-get update

# <dist-upgrade 10分程度>
# apt-get -y dist-upgrade

# reboot
```

- 自動アップデートの無効化
[対象: controller01, compute01, cli01]

OpenStack 環境に影響を与える可能性があるため自動アップデートを無効化する

```
# <自動更新パッケージの有無確認>
# dpkg -l unattended-upgrades
========> dpkg -l ここから
||/ Name  Version  Architecture  Description
+++-===-===-===-===
ii  unattended-upgrades  0.90  all  automatic installation of security upgrades
========< dpkg -l ここまで

# <自動更新 設定確認>
# cat /etc/apt/apt.conf.d/10periodic
# cat /etc/apt/apt.conf.d/20auto-upgrades
# cat /etc/apt/apt.conf.d/50unattended-upgrades
# grep "Periodic::Unattended-Upgrade" /etc/apt/apt.conf.d/*
========> grep ここから
/etc/apt/apt.conf.d/20auto-upgrades:APT::Periodic::Unattended-Upgrade "0";
========> grep ここまで


# <自動更新 無効化>
# vi /etc/apt/apt.conf.d/20auto-upgrades
========> 以下を参考に編集 vi ここから
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
========> vi ここまで
```

### OpenStack パッケージ

- OpenStack リポジトリーの有効化
[対象: controller01, compute01, cli01]

```
# <software-properties-common は add-apt-repository をインストールするため>
# apt-get install software-properties-common

# <他のバージョンを選択したい場合はmitakaの部分を書き換え>
# add-apt-repository cloud-archive:mitaka
  Press [ENTER] to continue or ctrl-c to cancel adding it ← [ENTER]押下
```

- パッケージのアップグレード
[対象: controller01, compute01, cli01]

```
# apt-get update && apt-get dist-upgrade
# reboot
```

- OpenStackクライアントのインストール
[対象: controller01, compute01, cli01]

```
# apt-get install python-openstackclient
```



## SQL データベース

自動起動とかいらないの？？？？？？？？？？？？？

http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/environment-sql-database.html

ほとんどの OpenStack のサービスは、情報を保存するために SQL データベースを使用する。データベースは、一般的にコントローラーノードで実行する。 この手順では MariaDB を使用する。OpenStack のサービスは、MySQL、 PostgreSQL などの他の SQL データベースもサポートしている。

### データベースのインストール

- パッケージのインストール
[対象: controller01]

```
# apt-get install mariadb-server python-pymysql
  Do you want to continue? [Y/n] Y
```


### データベース の rootパスワード設定

- rootパスワード設定
[対象: controller01]

```
# <MariaDBにログイン>
# <最初のログイン時はパスワードが設定されていない>
# mysql -u root


# <ユーザーとパスワード確認>
# SELECT User, Host, Password FROM mysql.user;
========>
+------+-----------+----------+
| User | Host      | Password |
+------+-----------+----------+
| root | localhost |          |
+------+-----------+----------+
1 row in set (0.00 sec)
========<


# <rootパスワードの設定>
MariaDB [(none)]> UPDATE mysql.user SET Password = PASSWORD('Password123$') WHERE User = 'root';


# <ユーザーとパスワード確認>
MariaDB [(none)]> SELECT User, Host, Password FROM mysql.user;
========>
+------+-----------+-------------------------------------------+
| User | Host      | Password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *6C12FC2B651683A59ADD31A5C49E1E6F801D399F |
+------+-----------+-------------------------------------------+
1 row in set (0.00 sec)
========<


# <変更の反映>
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit;

# <ログイン 確認>
# mysql -u root -p
Enter password: Password123$
MariaDB [(none)]> exit;

############################## 自分メモ 後で対応
######################
なぜかパスワードなしでもログイン出来てしまう。。。。
mysql -u root
MariaDB [(none)]> exit;

パスワードを間違えてもログイン出来てしまう。。。。
mysql -u root -p
Enter password: aaaaaaaaaaaaaaaa
MariaDB [(none)]> exit;

何か、おかしい。。。。。
cat /etc/mysql/debian.cnf
========>
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = root
password =             # <==== 本来ここにパスワードが入っているべきなのに入っていない
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = root
password =             # <==== 本来ここにパスワードが入っているべきなのに入っていない
socket   = /var/run/mysqld/mysqld.sock
basedir  = /usr
========<


パスワードに関する問題事象は多発しているようだ。。。。。
https://jyn.jp/ubuntu-16-04-mariadb-password-bug/


今、Ubutnu16でやっているので、
試しに、
Ubutnu14でやり直し中。。。。。。。。


```
