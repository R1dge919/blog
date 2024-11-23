---
title: HTB(Starting Point) Meow
draft: false
tags:
---
一通りStarting Point（無料でできる範囲）をやってみて、難易度EasyのActiveマシンに挑んでみたら、難しすぎて絶望しました。
再履修ついでに、メモを残しておきます。

……とはいえ、Task 5まではほぼ知識問題です。


---
### Task 1:  What does the acronym VM stand for? 
> 翻訳：VMの略語は何を意味しますか？

Answer: `Virtual Machine`

### Task 2: What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell. 
> 翻訳：オペレーティングシステムと対話して、VPN接続を開始するためのコマンドなどをコマンドラインで発行するために使用するツールは何ですか？コンソールやシェルとも呼ばれます。

Answer: `terminal`

※厳密には少々違うらしい？　自分もよく分かっていない。
　ユーザーとコンピューターを仲介するコマンドラインインターフェースが「シェル」であり、これを操作するためのアプリが「ターミナル」らしい。


### Task 3:  What service do we use to form our VPN connection into HTB labs? 
> 翻訳：HTBラボへのVPN接続を形成するためにどのサービスを使用しますか?

Answer: `openvpn`

### Task 4: What tool do we use to test our connection to the target with an ICMP echo request?
> 翻訳：ターゲットへの接続をICMPエコーリクエストでテストするために使用するツールは何ですか？

Answer: `ping`

### Task 5: What is the name of the most common tool for finding open ports on a target?
> 翻訳：ターゲットのオープンポートを見つけるための最も一般的なツールの名前は何ですか？

Answer: `nmap`

### Task 6:  What service do we identify on port 23/tcp during our scans? 
> 翻訳：スキャン中にポート23/tcpで識別されるサービスは何ですか？

上述のnmapを使ってみます
```sh
└─$ nmap 10.129.44.177      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 22:32 JST
Nmap scan report for 10.129.44.177
Host is up (0.84s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
23/tcp open  telnet

Nmap done: 1 IP address (1 host up) scanned in 32.52 seconds
```

Answer: `telnet`

### Task 7:  What username is able to log into the target over telnet with a blank password?
> 翻訳：どのユーザー名が空のパスワードでターゲットにtelnetでログインできますか？

telnetで繋いでみると、ログインを要求されます
```sh
└─$ telnet 10.129.44.177
Trying 10.129.44.177...
Connected to 10.129.44.177.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: a
Password: 
```

基本的にはパスワードが要求されますが、あるユーザーのみパスワードが要求されないようです。
解答欄の値を見ると、ユーザー名は`***t`？
（この次のタスク名のせいで、正直答えは丸分かり状態です）

Answer: `root`
※「根」を意味する単語。
　UNIX系OSにおいて、システムに組み込まれている管理者アカウントのこと
　
### Submit root flag

先ほどのログインにより、rootアカウントでMeowマシンに対する操作が行えるようになったはず。
ホームディレクトリにある「flag.txt」を開けば良い

```sh
root@Meow:~# pwd
/root
root@Meow:~# ls
flag.txt  snap
root@Meow:~# cat flag.txt
b40abdfe23665f766f9c61ecba8a4c19
```

