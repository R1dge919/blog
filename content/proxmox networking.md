---
title: 家のなかにProxmoxを設置した話
tags:
  - 備忘録
---
## まえがき
Amazonで安売りされていた格安小型PCを買って、Proxmox（仮想化プラットフォーム）をインストール、何台か仮想マシンを動かし始めました。広告ブロック用DNSサーバー（Pi-Hole）等、試験的に動かしたい仮想マシンを咄嗟に用意できるので、あると何かと便利です。

ただ、初期設定のまま仮想マシンを立てると、自宅ネットワークの中に＋1台増えるような振る舞いをしてしまい、ちょっと都合が悪いです。そこで、自宅ネットワーク（192.168.0.0/24）に対し、仮想マシン用ネットワーク（10.0.0.0/24）を用意して、その中のマシンへと通信を確立させるための設定を行いました。

調べて試しても駄目で、他の情報を調べて……と試行錯誤したので、もし今後同様の設定をすることがあったときのために、とりあえずメモを残しておきます。
ネットワーク周りはド素人で、調べて出てきたコマンドを言われるがまま実行しているような状態ですので、もっとマシな設定方法があるかもしれません。

結論からいうと、[公式ドキュメントの設定](https://pve.proxmox.com/wiki/Network_Configuration#sysadmin_network_masquerading)をそのまま参考にしました。
## 状況
- 自宅ネットワーク：192.168.0.0/24
	- Proxmox：192.168.0.254

## 設定
### ①Proxmox本体の設定
Proxmox本体について、自宅ネットワーク（192.168.0.0/24）だけでなく、仮想ネットワーク（10.0.0.0/24）にも所属するよう設定します。
ここでは、`/etc/network/interfaces`を編集して、以下のように記述します。
```ini
auto lo
iface lo inet loopback

# 自宅ネットワーク用
# 環境によっては「enp3s0」でない可能性がある
auto enp3s0
iface enp3s0 inet static
        address 192.168.0.254/24
        gateway 192.168.0.1

# 無線LAN（つかわない）
iface wlp1s0 inet manual

# 仮想ネットワーク用
auto vmbr0
iface vmbr0 inet static
        address 10.0.0.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o enp3s0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUING -s '10.0.0.0/24' -o enp3s0 -j MASQUERADE
```

### ②ルーターの設定
ルーター側の設定を弄り、「10.0.0.0/24（VM用ネットワーク宛）の通信は、192.168.0.254（Proxmoxの自宅ネットワーク側IPアドレス）に流す」よう、静的経路設定を行います。

TP-Linkのルーターだとこんな感じ
![[Pasted image 20250622130109.png]]