---
title: LinuxでThinkPadのファンが回ってなかった話
author: r1dge
date: 2024-06-23
category: 備忘録
layout: post
---

## まえがき
表題の通りです。
ThinkPadを触ってると、1時間ほど触ってるとキーボードが熱くなってきます。
……ちょっと待て、ファン自体が回ってないんじゃないのか！？　ということに先ほど気づきました。Linuxむずかしいね……

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

### 回転速度の設定
- `$ echo level auto | sudo tee /proc/acpi/ibm/fan`
    - `auto`の部分は置換可能
        - 0〜7（定数での回転速度設定　0停止〜7高速）
        - `auto`（自動）
        - `disengaged`（最高速度）
            - 7との違いは不明

## おまけ
- `thinkfan`等のファン制御デーモンを使用することで、ファン回転数の制御を細かく制御できるようになるそうです
    - ただ、参考にしたWebサイトに記載されていた設定ファイル内容と、現行の設定ファイルの形式が異なるのか、うまく設定できませんでした……
        - 気が向いたら色々弄ってみたい。そのときは記事を更新予定

## 参考
- [ファンスピード制御 - ArchWiki](https://wiki.archlinux.jp/index.php/ファンスピード制御#ThinkPad_ノートパソコン)
- [\[ThinkPad X1\]Ubuntu GnomeでCPUファンをコントロール  |  BOOLEE STREET.net'](https://booleestreet.net/archives/11263)