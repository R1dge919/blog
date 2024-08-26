---
title: JNetHack（古のテキストRPGゲームの有志日本語版）をインストールした
published: true
---

## まえがき
ローグライクRPGが遊びたくて色々調べたとき、『NetHack』の存在を知りました。  
コレの有志日本語化版である『JNetHack』をインストールしようと色々試したとき、「色んな人が異なる方法を紹介している」「かつてファイルが公開されていたサイト（OSDN）の不調により、ダウンロードできない場合がある」等の問題があったため、自分が行った方法を残しておきます。

![](/blog/assets/images/jnh01.png)

## 環境
- Debian (VirtualBox)
    - パッケージマネージャ「apt」が使える環境であれば、この方法が使えると思います。

## 手順
```sh
# 必要なパッケージのインストール
$ sudo apt install git gcc make nkf flex bison libncurses-dev -y

# リポジトリのクローン
# （OSDNのリポジトリも使えなくはないのですが、OSDNの不調問題に伴いGitHubに移行されたリポジトリが存在するため、今回はこれを参照します）
$ git clone https://github.com/jnethack/jnethack-release.git
$ cd jnethack-release

# インストール
$ ./configure
$ make install

# （インストール後、ホームディレクトリ以下に専用ディレクトリが配置されます）

# 起動
$ ~/nh/install/games/jnethack
```

## あとがき
- 操作方法とかパラメータとか、まったくわからない