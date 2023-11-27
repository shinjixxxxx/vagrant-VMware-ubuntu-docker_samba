# Node開発環境用Vagrantfile(M2Mac - vmware fusion 対応)

## モチベーション
webpackなどを使ったタスクランナー環境をvagrant上に短時間で簡単に構築できるように作りました。
npm install時のファイル名長やシンボリックリンクの問題を避けるためゲストosのファイルシステムを使い、sambaでホストとファイル共有します。

## 概要
M2Mac - vmware fusion 対応した新しいバージョンです。
vagrant 環境を利用してnodeのタスクランナーを使った開発を行うための
環境をプロビジョニングするVagrantfileです。
Vagrantfileを配置してvagrant upするだけで環境がnodeによる開発環境が作成されます。
フォルダ(/dockerd)はSambaでファイル共有されるので、vscodeなどで自由に編集できます。
フォルダ(/dockerd)は webサーバ(ポート8080)で動作確認が行えます。
phpがインストールされてます。
node npm のstableがインストールされてます。
n を使って node のバージョンを管理できます。

### 使用法
ダウンロードしたVagrantdを配置したフォルダから vagrant up するだけです。
sambaでゲストマシンの/dockerdが共有されるのでVscodeやSublimeTextなどを使ってソースを編集し、
ゲストマシンのnode環境を使ってwebpackなどでトランスパイルします。
結果をブラウザで確認します。
http://192.168.33.10:8080
```bash
  $ ### 使用例：
  $ # vagrant用のディレクトリを作ってVagrantfileをクローン → vagrant up実行
  $ mkdir vagrantUbuntuDirectory
  $ cd vagrantUbuntuDirectory
  $ git clone https://github.com/shinjixxxxx/Vaglantfile-ubuntu.git
  $ vagrant up
  ......install
  $ vagrant ssh
  Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-96-generic x86_64)
  System information as of Wed Apr 22 01:26:18 UTC 2020
  vagrant@ubuntu:~$
  vagrant@ubuntu:~$
  vagrant@ubuntu:~$ cd /dockerd
  vagrant@ubuntu:/dockerd$ mkdir pj
  vagrant@ubuntu:/dockerd$ cd pj
  vagrant@ubuntu:/dockerd/pj$ # npm を使って開発環境を初期化
  vagrant@ubuntu:/dockerd/pj$ npm init -y
  vagrant@ubuntu:/dockerd/pj$ npm imstall react react-dom
  vagrant@ubuntu:/dockerd/pj$ npm imstall -D webpack webpack-cli babel-loader @babel/core @babel/preset-env @babel/preset-react
  vagrant@ubuntu:/dockerd/pj$ mkdir src
  vagrant@ubuntu:/dockerd/pj$ touch src/index.jsx
  vagrant@ubuntu:/dockerd/pj$ touch webpack.config.js
  vagrant@ubuntu:/dockerd/pj$

```
  
  
---
## 詳細
linuxの環境とファイルシステムでnodeを稼働するため、
windowsやmacintoshなどの環境の違いによって生じる問題を回避します。
ホストマシンの環境に影響されないようにゲストマシンのファイルはsambaで共有します。
Samba共有にはdockerのdeperson/sambaを使用します。

- 開発ディレクトリ（/vagrantd）を http://192.168.33.10:8080 で見えるようにします。
- 開発ディレクトリ（/vagrantd）を deperson/sambaでwindows ネットワーク共有で
  ホストのシステムとファイル共有します。

### 開発ディレクトリ
8080ポートでhttpアクセスできます。
sambaでファイル共有されます。
```
/dockerd
```
### ファイルへのアクセス
#### Macintoshの場合
1. ファインダーのメニューから
[移動] ＞ [サーバーへ接続]
を選んで、以下のアドレスに接続。
```
smb://192.168.33.10
```
2. [◉ゲスト] を選んで[接続]
*※ 一定時間で共有が外れるので接続しなおしてください。*
   
#### windowsの場合
エクスプローラのアドレスに以下のアドレスへアクセス。
```
¥¥192.168.33.10
```
### インストールソフト 
最低限の開発環境として以下のアプリがインストールされます。
- apache2
- php
- node
- npm 
- n
- docker
- emacs
  
### httpアクセス
```
  $ curl http://192.168.33.10:8080
```
  
### 環境
```
vagrant@ubuntu-bionic:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.4 LTS
Release:	18.04
Codename:	bionic
```
