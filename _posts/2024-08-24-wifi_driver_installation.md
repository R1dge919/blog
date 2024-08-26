---
title: Kali LinuxでAircrack-ngを使うためにUSBドングルを設定する
published: true
---

## まえがき
- 一部の機能に対応した無線LANアダプタを用意すると、Aircrack-ngと呼ばれるツールで色々できると聞き、先駆者の情報をもとに「TP-Link Archer T2U Nano」を調達しました。
    - ![](/blog/assets/images/oi5qX_4i.jpeg)
    
- ……このアダプタを使用しようと思うと、デバイスドライバを導入する必要があるのですが、先駆者の情報通りにデバイスドライバを導入しても、自分の環境では使うことができませんでした……(;_;)

- 他のデバイスドライバを導入するなどして、自分の環境で動作できるようになったときの記録を残しておきます。

## 結論から言うと……
- このデバイスドライバが使えました
    - [https://github.com/morrownr/8821au-20210708]()
        - このリポジトリのREADMEを見て導入しただけです
- 3つの環境（ThinkPad T14 Gen5、自作PC、自作PC内のVirtualBoxマシン）で試したのですが、この3つすべてで正常に使えることを確認しました

## 実施手順
- アップデート
    - `sudo apt update && sudo apt upgrade -y`
- 必要なパッケージの導入
    - `sudo apt install -y linux-headers-$(uname -r) build-essential bc dkms git libelf-dev rfkill iw`
- デバイスドライバのリポジトリを複製
    - `git clone https://github.com/morrownr/8821au-20210708.git`
    - `cd 8821au-20210708`
- インストール用スクリプトを実行
    - `sudo ./install-driver.sh`
    - スクリプトを実行すると、途中で設定ファイルを開くかどうか、その後に再起動を行うかどうか聞かれます
        - 指示に従い、設定ファイルの編集・再起動を実施することを勧めます
            > （再起動するまで、デバイスドライバが使用できないため）

### 設定ファイルの例（自分の設定内容）
> 初期設定では、「rtw_led_ctrl=1」（無線LANのLEDドングル＝点滅）のみ記載されています  
> コメントの説明文を参考に、無線LANドングルのLEDを常時点灯するよう変更、省電力モードの無効化、の2つを設定してみました
```conf
# /etc/modprobe.d/8821au.conf
#
# Purpose: Allow easy access to specific driver options.
#
# Edit the following line to change, add or delete driver options:
#
options 8821au rtw_led_ctrl=2 rtw_power_mgnt=0
#
# Note: The above `options` line is a good default for managed mode. Below
# are examples for AP mode. Modify as required after reading the documentation:
#
# Band 1 (2.4 GHz)
#options 8812au rtw_switch_usb_mode=2
#
# Band 2 (5 GHz)
#options 8812au rtw_switch_usb_mode=1 rtw_vht_enable=2 rtw_dfs_region_domain=1 rtw_country_code=US
#
# After editing is complete, save this file (if using nano: Ctrl + x, y, Enter)
# and reboot to activate the changes.
#
# Note: hostapd information is located at the end of this file.
#
# Documentation:
#
# -----
#
# Log options ( rtw_drv_log_level )
#
# 0 = NONE (default)
# 1 = ALWAYS
# 2 = ERROR
# 3 = WARNING
# 4 = INFO
# 5 = DEBUG
# 6 = MAX
#
# Note: You can save a log file that only includes RTW log entries by running
# the following in a terminal:
#
# sudo ./save-log.sh
#
# Note: A log option greater than 1 must be set. The name of the log
# file will be `rtw.log`.
#
# -----
#
# LED options ( rtw_led_ctrl )
#
# 0 = Always off
# 1 = Normal blink (default)
# 2 = Always on
#
# -----
#
# VHT options ( rtw_vht_enable )
#
#  0 = Disable
#  1 = Enable (default)
#  2 = Force auto enable (use only for 5 GHz AP mode)
#
# Notes:
# - A non-default setting can degrade performance greatly in managed mode.
# - Option 2 allows 80 MHz channel width for 5GHz AP mode, such as when
#   you are using hostapd.
#
# -----
#
# Power options ( rtw_power_mgnt )
#
# 0 = Disable power saving
# 1 = Power saving on, minPS (default)
# 2 = Power saving on, maxPS (not recommended for AP mode)
#
# -----
#
# DFS Options ( rtw_dfs_region_domain )
#
# 0 = NONE (default)
# 1 = FCC
# 2 = MKK
# 3 = ETSI
#
# Notes:
# - Activates DFS channels in AP mode.
# - DFS FCC 80 MHz channels for hostapd: 52(58), 100(106), 116(122) and 132(138)
# - For more information: https://en.wikipedia.org/wiki/List_of_WLAN_channels
#
# Note: An AP needs to listen on a DFS channel for a period of 60 seconds
# before transmitting on the channel. If any radar pulses are detected,
# the AP cannot use that channel and will have to try a different channel.
#
# -----
#
# Wireless Mode options ( rtw_wireless_mode )
#
# 1  = 2.4GHz 802.11b
# 2  = 2.4GHz 802.11g
# 3  = 2.4GHz 802.11b/g
# 4  = 5GHz 802.11a
# 8  = 2.4Hz 802.11n
# 11 = 2.4GHz 802.11b/g/n
# 16 = 5GHz 802.11n
# 20 = 5GHz 802.11a/n
# 64 = 5GHz 802.11ac
# 84 = 5GHz 802.11a/n/ac
# 95 = 2.4GHz 802.11b/g/n 5GHz 802.11a/n/ac (default)
#
# -----
#
# Country Code options ( rtw_country_code )
#
# Note: Allows the Country Code to be set in cases where it is unable to
# be obtained otherwise.
#
# URL: http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
#
# Example for the US: rtw_country_code=US
# Example for Panama: rtw_country_code=PA
# Example for Norway: rtw_country_code=NO
# Example for Kuwait: rtw_country_code=KW
# Example for Taiwan: rtw_country_code=TW
#
# -----
#
# To see all driver options that are available:
#
# $ ls /sys/module/8821au/parameters/
#
# -----
#
# To see the values that are in use:
#
# $ grep [[:alnum:]] /sys/module/8821au/parameters/*
#
# -----
#
# hostapd setup information for rtl8821au
# Note: The best settings can vary but the following may be a good place to start.
#
# /etc/modprobe.d/8821au.conf
# options 8821au rtw_drv_log_level=0 rtw_led_ctrl=0 rtw_vht_enable=2 rtw_power_mgnt=1 rtw_dfs_region_domain=1
#
# Note: The best setting for `rtw_dfs_region_domain=` will depend on your location.
#
# /etc/hostapd/hostapd.conf
#
# hw ht capab: 0x962
# ht_capab=[HT40+][HT40-][SHORT-GI-20][SHORT-GI-40][RX-STBC1][MAX-AMSDU-7935]
#
# hw vht capab: 0x3c00122
# vht_capab=[MAX-MPDU-11454][SHORT-GI-80][RX-STBC-1][HTC-VHT][MAX-A-MPDU-LEN-EXP7]
#
# -----
```

## 最後に（動作確認）
- KDEでの例
    - 家のWiFiが受信できるようになりました！
    - ![](/blog/assets/images/Screenshot_20240825_222317.png)