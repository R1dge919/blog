---
title: GitHub Pages上にObsidianの記事を載せる
draft: false
tags:
  - 備忘録
  - 環境構築
---
### 参考情報
- [Welcome to Quartz 4](https://quartz.jzhao.xyz/)
- [Quartz4で無料でObsidian Publishを代替する](https://masaki39.github.io/Quartz4%E3%81%A7%E7%84%A1%E6%96%99%E3%81%A7Obsidian-Publish%E3%82%92%E4%BB%A3%E6%9B%BF%E3%81%99%E3%82%8B)
- [How to publish Obsidian notes with Quartz on GitHub Pages - Fork My Brain](https://notes.nicolevanderhoeven.com/How+to+publish+Obsidian+notes+with+Quartz+on+GitHub+Pages)

### まえがき
　「Quartz4」と呼ばれる静的サイトジェネレーターを用いることで、Obsidian Publishと同じようなことをGitHub Pages上で行えると聞いて、先駆者の情報をもとに再現してみました。
　一部で詰まることがあったため、私自身も先駆者に倣い備忘録として書き残すことにします。
　最低限、ページとして閲覧できるようになるまでを目標としているため、ページの名前変更方法や、記事の追加方法などは省略します。

---
### 事前準備
> このあたりはOS等によって導入手順が異なるため、本記事では省略します
- node.jsとnpmのインストール
- gitのインストール
- GitHubへのSSH公開鍵の登録
	- プッシュ等の際にHTTPS/SSHのどちらか選ぶのですが、HTTPSのほうは「トークン」の登録が必要なようです
	- Git、GitHubよく分かっていないので、自分はSSHのほうで作業を行いました

### Quartz4のクローン
- 今回はリポジトリ名「test_quartz」として複製します
	- 今後も頻繁に登場します。適宜自分のリポジトリ名に置き換えてください。
```sh
$ git clone https://github.com/jackyzha0/quartz.git test_quartz
$ cd test_quartz
$ npm i
$ npx quartz create
```
- 最後のコマンドを実行した際、2つの問いに対して、十字キー等での選択を要求されます
	- 「記事用フォルダの中身を他からコピーするか」「リンク方式はどうするか」といった問いになります
	- 2択とも初期項目を選択しました

### ローカルでの確認
```sh
$ npx quartz build --serve
（省略）
Started a Quartz server listening at http://localhost:8080
（省略）
```
- Webブラウザ上で、サイトがどのような感じか確認することができます

### GitHubでの空リポジトリ作成
- ページ用のリポジトリを作成します
- このとき、READMEは作成しないようにしてください
- SSHの接続情報をコピーしておきます

### リモートリポジトリを書き換える
- 現在使用しているリポジトリはQuartz4をクローンしたものであるため、このままだとQuartz4のオリジナルのリポジトリを追跡しようとします
- 1つ前の手順で作成したリポジトリを追跡するよう、設定を変更します
```sh
$ git remote -v                                                
origin	https://github.com/jackyzha0/quartz.git (fetch)
origin	https://github.com/jackyzha0/quartz.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)

$ git remote rm origin

# 先ほどコピーした内容を貼り付ける
$ git remote add origin git@github.com:R1dge919/test_quartz.git

$ git remote -v                                                
origin	git@github.com:R1dge919/test_quartz.git (fetch)
origin	git@github.com:R1dge919/test_quartz.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)
upstream	https://github.com/jackyzha0/quartz.git (push)
```
### GitHub Pages用の設定ファイル配置
- https://quartz.jzhao.xyz/hosting#github-pages
- GitHub Pages用の設定ファイルの内容が記載されています
- この内容をコピーして、リポジトリ内`test_quartz/.github/workflows/deploy.yml`を作成します
	- 必ず上記リンク（公式サイト）のものを使用しましょう
	- 自分は先駆者が転記していたものを使おうとしたせいで、バージョンが古いなどのエラーが発生していっぱい怒られました……


### リモートリポジトリとの初回同期

```sh
$ npx quartz sync --no-pull
```

### GitHub Pagesの設定
- Settings -> Pages から、Sourceを「GitHub Actions」に設定する
- ![[Pasted image 20241112233026.png]]
### 同期
- 以降、GitHub Pages側に変更を適用させたいタイミングで、以下のコマンドを実行する
```sh
$ npx quartz sync
```

### ページの確認
- `ユーザー名.github.io/リポジトリ名`
- 自分の場合、　`r1dge919.github.io/test_quartz/`
	- ※テスト用に一時的に作成したものであるため、現在は何もありません
- ![[Pasted image 20241112233340.png]]