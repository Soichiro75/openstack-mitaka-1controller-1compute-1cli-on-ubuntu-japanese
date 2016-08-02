# 02_サービス事前準備

## OpenStack パッケージ

- 既存パッケージの更新
[対象: controller01, compute01, cli01]

```
# apt-get update

# <dist-upgrade 10分程度>
# apt-get -y dist-upgrade

# reboot
```

- 既存パッケージの更新
[対象: controller01, compute01, cli01]

```
# apt-get update

# <dist-upgrade 10分程度>
# apt-get -y dist-upgrade

# reboot
```

- 自動アップデートの無効確認

```
#<>
# dpkg -l unattended-upgrades
========> dpkg -l ここから
||/ Name  Version  Architecture  Description
+++-===-===-===-===
ii  unattended-upgrades  0.90  all  automatic installation of security upgrades
========< dpkg -l ここまで

aaaaaaaaaa

```
