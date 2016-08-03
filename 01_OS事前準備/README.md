# 01_OS事前準備

## OSインストール

[対象: controller01, compute01, cli01]

- 環境について [(OpenStack ドキュメント 環境)](http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/environment.html)
  - インターネットに接続可能な環境であること
  - 64bit CPU であること
  - BIOSにて`Virtualization Technology(VT-x or AMD-V)` を有効にしていること
  - ハイパーバイザー上(VM)で構築検証したい場合は、以下を確認すること</br>
      ※ 本手順ではオンプレミスにて構築する
   - nestedの設定をしていること
   - ハイパーバイザーがプロバイダーネットワークインターフェースに MAC アドレスフィルタリングを無効化する方法を提供していること
  - 以下スペック以上であること
    - コントローラーノード: 1 CPU、4 GB メモリ、5 GB ストレージ
    - コンピュートノード: 1 CPU、2 GB メモリ、10 GB ストレージ

- Ubuntu Server 14.04 (LTS) 64bit を[http://releases.ubuntu.com/14.04/ubuntu-14.04.4-server-amd64.iso](http://releases.ubuntu.com/14.04/ubuntu-14.04.4-server-amd64.iso)からダウンロード
- Ubuntu Servier のイメージをDVDに焼く
- controller01, compute01, cli01 とするサーバーに最小構成でインストールする
  - インストール時のパラメーター
    - Ubuntuインストール時に使用した言語がインストール後にも使用される。日本語環境では何かと問題が起きるので、英語環境とするために英語インストールを推奨
  - Block Storage のようなオプションサービスをインストールする場合には Logical Volume Manager (LVM) 推奨
  - ディスクに既に別のデータが入っている場合、パーティション選択時に内容を削除すること
  - 以下xxxについて、

|設定項目|設定例|
|---|---|
|Language|English|
|-|Install Ubuntu Server|
|Language|English|
|Location|other > Asia > Japan|
|Locales|United States - en_US.UTF-8|
|Keyboard (Detect keyboard layout?)|No|
|Keyboard|Japanese / Japanese|
|(DHCPが環境にない場合)|manual configuration|
|(DHCPが環境にない場合)nic|xxxxxxxx|
|(DHCPが環境にない場合)IP|192.168.101.x/24|
|(DHCPが環境にない場合)Gateway|192.168.101.254|
|(DHCPが環境にない場合)DNS|8.8.8.8|
|Hostname|xxx.st.local|
|New user|user01|
|UserName for your account|user01|
|Password|Password123$|
|Encrypt your home directory|No|
|Asia/Tokyo Is this time zone correct?|Yes|
|Partitioning method|Guided - use entire disk and set up LVM|
|Disk|SCSI3 (0,0,0) (sda) - xxxGB|
|Write the changes to disks and configuration LVM?|Yes|
|Amount of volume group|max|
|Write the changes to disk?|Yes|
|Proxy|(空)|
|Manage upgrades|No automatic updates|
|Software|OpenSSH server|
|Install GRUB boot loader?|Yes|
|(Diskが複数ある場合)Chose xxx?|sda|
|Installation Complete. Restart?|Continue|


## ログイン設定

- ログイン
[対象: controller01, compute01, cli01]

```
login: user01
Password: Password123$ (← 入力時 表示されない)
```


- rootパスワード
[対象: controller01, compute01, cli01]

```
$ sudo su -
password for user01: Password123$

# passwd
Enter password: Password123$
Retype password: Password123$

# exit
$ exit
```


- rootログイン確認
[対象: controller01, compute01, cli01]

```
login: root
Password: Password123$
```


- ssh root ログイン
[対象: controller01, compute01, cli01]

```
# vi /etc/ssh/sshd_config
========> vi ここから
PermitRootLogin without-password
  ↓
PermitRootLogin yes
========< vi ここまで

# service ssh restart
```

- 以降teraterm等からの操作可能
[対象: controller01, compute01, cli01]


## ネットワーク設定

- 優先nicの設定確認
[対象: controller01, compute01, cli01]

```
# cat /etc/network/interfaces
========> controller01の例 cat ここから
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp3s0
iface enp3s0 inet static
        address 192.168.101.1
        netmask 255.255.255.0
        network 192.168.101.0
        broadcast 192.168.101.255
        gateway 192.168.101.254
        # dns-* options are implemented by the resolvconf package, if installed
        dns-nameservers 8.8.8.8
        dns-search st.local
========< cat ここまで

```

- プロバイダーインターフェースの設定
[対象: controller01, compute01]
  - `INTERFACE_NAME` は環境に合わせて置き換えること</br>
    ※ 私の環境ではcontroller01でh「enp6s0」、compute01では「enp2s0f1」
  - メモ: `iface <config_name> <address_family> <method_name>`
    - iface は スタンザと呼ばれる複数行からなる設定(他には、mapping, auto, source, allow-hotplug など)
    - method_name = static: 固定の設定を入れたい時に使用。address, netmask は必須。その他、gatewayなど。
    - method_name = manual: 自前で全部設定したい時に使用(今回の様に、address(IP)の設定なしでinterfaceの起動をしたい時などに使用)。 `/etc/network/if-\*.d/`以下のスクリプトで設定する。

```
# vi /etc/network/interfaces
========> 以下を追加 vi ここから
# The provider network interface
auto INTERFACE_NAME
iface INTERFACE_NAME inet manual
up ip link set dev $IFACE up
down ip link set dev $IFACE down
========< vi ここまで
```

- 名前解決
[対象: controller01, compute01, cli01]
  - (注意) ディストリビューションによっては`127.0.1.1`の記載が存在する、`127.0.1.1`が存在した場合は、不具合を防ぐため削除(コメントアウト)すること
  - `127.0.0.1` については削除しないこと
```
vi /etc/hosts
========> 以下を参考に編集 vi ここから
# 127.0.1.1     xxxx.st.local     xxxx
127.0.0.1       localhost
192.168.101.1   controller01.st.local     controller01
192.168.101.2   compute01.st.local     compute01
========< vi ここまで
```

- 接続確認
[対象: controller01, compute01, cli01]

  - hosts動作確認

```
# ping -c 4 controller01
# ping -c 4 controller01.st.local
# ping -c 4 compute01
# ping -c 4 compute01.st.local
```

  - インターネット接続確認

```
# ping -c 4 openstack.org
```

## NTP設定

controller01をNTPサーバーとして、その他のノードはcontroller01を参照して時刻同期する

### NTPサーバー設定

- Chrony インストール
[対象: controller01]

```
# apt-get install chrony
    Do you want to continue? [Y/n] Y
```

- Chrony 設定
[対象: controller01]
  - メモ: `iburst` オプション:サーバに到達できない場合、ポーリング間隔ごとに、通常のパケット1個の代わりに、パケット8個をバースト的に送信する。これによって、初期化時間を短縮出来る。

```
# vi /etc/chrony/chrony.conf
========> 以下を参考に編集 vi ここから
# pool 2.debian.pool.ntp.org offline iburst
server ntp.nict.jp iburst
allow 0/0
========< vi ここまで
```

- Chrony 設定反映
[対象: controller01]

```
# service chrony restart
```

- Chrony 同期確認
[対象: controller01]

```
# chronyc sources
設定したNTPサーバー(nict)に"^*"が付いていること
========> chronyc sources 結果 ここから
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ntp-a2.nict.go.jp             1   6    17     3    +12us[ +121us] +/- 3852us
========< chronyc sources 結果 ここまで
```


### NTP クライアント設定

- Chrony インストール
[対象: compute01, cli01]

```
# apt-get install chrony
    Do you want to continue? [Y/n] Y
```

- Chrony 設定
[対象: compute01, cli01]

```
# vi /etc/chrony/chrony.conf
========> 以下を参考に編集 vi ここから
# pool 2.debian.pool.ntp.org offline iburst
server controller01 iburst
========< vi ここまで
```

- Chrony 設定反映
[対象: compute01, cli01]

```
# service chrony restart
```

- Chrony 同期確認
[対象: compute01, cli01]

```
# chronyc sources
controller01 に "^*" が付いていること
========> chronyc sources 結果 ここから
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* controller01.st.local         2   6    17     3   -763ns[  -13us] +/- 4261us
========< chronyc sources 結果 ここまで
```
