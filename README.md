# openstack-mitaka-1controller-1compute-1cli-on-ubuntu-japanese

</br>
**本手順の作成は中止。**

**[「Liberty on CentOS」](https://github.com/Soichiro75/openstack-liberty-on-centos-japanese)を変わりに作成予定。**

---

この手順では、
[OpenStack Docs Installation Guide for Ubuntu](http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/) を参考に、1Controller, 1Compute, 1Cli の構成でOpenStackのPoC環境を構築する。

「ssh root ログイン を許可する」など、基本的にセキュリティは考慮しない。

設定は基本的にrootユーザーにて実施している。

パスワードは基本的に`Password123$`を使用する。


## Helpful Links

- [OpenStack Docs Installation Guide for Ubuntu](http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/)
- [OpenStack構築手順書Mitaka版(日本仮想化技術株式会社)](http://www.slideshare.net/VirtualTech-JP/openstackmitaka)

## 環境

### 今回使用したサーバースペック

|   |controller01|compute01|cli01|
|---|---|---|---|
|CPU|2 cores|16 cores|2 cores|
|Memory|16 GB|88 GB|2 GB|
|HDD|2 TB|900 GB|120 GB|
|Nic|2 nics|2 nics|1 nics|

### (参考)最小構成

[OpenStack ドキュメント 環境 について](http://docs.openstack.org/mitaka/ja/install-guide-ubuntu/environment.html)

コアなサービスと CirrOS のインスタンスをいくつか動かす程度の検証環境(PoC)であれば、以下の最小要件で動作する

|   |controller01|compute01|cli01不要</br>(controller01で操作する)|
|---|---|---|---|
|CPU|1 core|1 core| - |
|Memory|4 GB|2 GB| - |
|HDD|5 GB|10 GB| - |
|Nic|1 nic|1 nic| - |


### 構成図

<img src="https://github.com/Soichiro75/openstack-mitaka-1controller-1compute-1cli-on-ubuntu-japanese/blob/master/images/xxxxxxxxx.png" width="320px" title="OpenStack全体構成図">
