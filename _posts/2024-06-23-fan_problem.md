---
title: LinuxでThinkPadのファンが回ってなかった話
published: true
---

## まえがき
表題の通りです。  
ThinkPadを触ってると、1時間ほど触ってるとキーボードが熱くなってきます。  
……ちょっと待て、ファン自体が回ってないんじゃないのか！？　ということに先ほど気づきました。  

## 環境
- PC: Lenovo ThinkPad T14 Gen 5 AMD    
    - OS: Kali Linux 2024.2（Debian系）

## 対応
### ファンの有効化
- `/etc/modprobe.d/thinkpad_acpi.conf`（存在しなければ新規作成）に、以下の1行を追加
    ```txt
    options thinkpad_acpi fan_control=1
    ```
- 再起動
- 確認
    ```sh
    $ cat /proc/acpi/ibm/fan                    

    status:		enabled
    speed:		0
    level:		auto
    commands:	level <level> (<level> is 0-7, auto, disengaged, full-speed)
    commands:	enable, disable
    commands:	watchdog <timeout> (<timeout> is 0 (off), 1-120 (seconds))
    ```

### 回転速度の手動設定
- `$ echo level auto | sudo tee /proc/acpi/ibm/fan`
    - `auto`の部分は置換可能
        - 0〜7（定数での回転速度設定　0停止〜7高速）
        - `auto`（自動……ただし挙動が微妙？　あまり回ってない気がする）
        - `disengaged`（最高速度）
            - 7との違いは不明
- 回転速度を固定するのであれば、これだけでOK
    - CPU温度に応じた制御を行うのであれば、以下の作業を参照

### ※CPU温度に応じた回転設定
- `thinkfan`のインストール
    - `$ sudo apt install thinkfan`
- `/etc/thinkfan.conf`の編集
    - 例　参考にしたサイトから丸ごとコピー　自環境ではこれで動きました
        ```yaml
        sensors:
            - tpacpi: /proc/acpi/ibm/thermal
            indices: [0]  # 各種センサーの温度情報を持つ。0番目（CPU）の情報を使うよう指定している
        
        fans:
            - tpacpi: /proc/acpi/ibm/fan # 制御対象ファンを指定
        
        levels: # 温度に応じた回転レベルを設定
            # [レベル値, 最低温度, 最大温度]
            # 温度範囲は重ねておくことが望ましいらしい
            # （2つの範囲にまたがる温度のときは、上位レベルを参照しているように見える）
            - [0, 0,  41]
            - [1, 38, 51]
            - [2, 45, 56]
            - [3, 51, 61]
            - [4, 55, 64]
            - [5, 60, 66]
            - [6, 63, 68]
            - [7, 65, 74]
            - [127, 70, 32767] # 127は最大速度（disengagedと同じ）らしい
        ```

## 参考
- [ファンスピード制御 - ArchWiki](https://wiki.archlinux.jp/index.php/ファンスピード制御#ThinkPad_ノートパソコン)
- [\[ThinkPad X1\]Ubuntu GnomeでCPUファンをコントロール  \|  BOOLEE STREET.net](https://booleestreet.net/archives/11263)
- [thinkfan - Gentoo Wiki](https://wiki.gentoo.org/wiki/Fan_speed_control/thinkfan#Configuration)