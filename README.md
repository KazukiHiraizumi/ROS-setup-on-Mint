# ROSをMintで使おう

Unityの独りよがりに嫌気がさした人は、純正Ubuntuはぶち消して、Mintをインストールしよう。  
MintもUbuntu派生なので、ROSのセットアップはほぼ同じ。  
Desktopは**普通のもの**が使える。これ大事！
 
## インストール LinuxMint

### USBブート
- Mint18.2 SonyaのUSBブータブルUSBを用意する
- Winならrufusというので作れる

### インストール
- インターネット接続がWifiなら、先にネットワーク設定をして接続する
- Installアイコンで開始

### アカウント設定
- user yoods
- password yama

### Grubアップデート

grubをアップデート(/etc/default/grubのリロード)する。
~~~
sudo update-grub
~~~

### 再起動

再起動を確認。！！もし起動できないときは、
1. USBから起動
- ブートUSBから起動しgrubメニューを出す(起動時にESCキーを押すなど)
- カーネルオプションを編集する("e"キーを押すなど、画面指示を見ること)
- オプションの"boot=casper"を"root=/dev/sda2"に変更
- 起動する(F10を押すなど、画面指示を見ること)
- 再起動できたら、4のupdate-grubのログを精査する

2. BIOS設定
- SecureBootをDisableにする
- 先にUEFIブートをDisableにしないとできないBIOSもある
- どちらもできないなら一旦すべてのBIOS設定をDefaultに戻してから再起動

### 画面チラつき

あまりメジャーでないGraphicsコントローラでは、画面チラつきが出ることがある。このときはgrubにてカーネルパラメータを変更する。
- /etc/default/grubを開く
~~~
sudo /etc/default/grub
~~~
- GRUB_CMDLINE_LINUX_DEFAULT=...の行を探して、その下に
~~~
GRUB_CMDLINE_LINUX_DEFAULT="nomodeset"
~~~
を追加
~~~
sudo update-grub
~~~
でエラーがないことを確認し再起動。

======
## 初期設定 LinuxMint

### アップグレード
~~~
sudo apt upgrade
~~~
ついでに
~~~
sudo apt update
~~~

### ツール・ライブラリの追加

後々のために以下を追加する
  - g++
  - git
  - automake
  - intltool
  - libgsreamer*-dev
~~~
sudo apt install g++ git automake intltool libgstreamer*-dev
~~~

### 電源管理

スタートメニュー⇒設定⇒電源管理
  - 一般  
  スリープモードは"ハイバネート"を選択
  - システム  
  同じく"ハイバネート"
  - ディスプレイ  
  □電源管理を行う、はOff
  - Secuity  
  LightLocker"しない"を選択

### デスクトップ切り替え

タスクバー⇒パネル⇒新しいアイテムの追加  
ワークスペーススイッチャを追加

## ソフトウェア追加
### Chrome　　
デフォルトのFoxがGithubのサポート外になっているので、ブラウザを更新
~~~
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt-get update 
sudo apt-get install google-chrome-stable
~~~

### Nodejsインストール　　
現時点(Sep18)でのHeadはNode9なので9を入れる
~~~
cd ~
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install nodejs
~~~

======

## インストール ROS

ROSのリリースはKineticが前提です。  
http://wiki.ros.org/kinetic/Installation/Ubuntuに手順がありますが、Mintに入れるので若干修正が要ります。

### Repos設定

先のページでは
~~~
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
~~~
ですが、Mintだと"lsb_release -sc"が"sonya"になっしまうので、あとから"/etc/apt/sources.list.d/ros-latest.list"の"sonya"を"xenial"に修正するか、最初から
~~~
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu xenial main" > /etc/apt/sources.list.d/ros-latest.list'
~~~
とします。

### Key設定
~~~
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
~~~
もしエラーになったら、キーサーバを"hkp://pgp.mit.edu:80"とか"hkp://keyserver.ubuntu.com:80"でやってみるそう。


### インストール

フルセットでOKです
~~~
sudo apt-get update
sudo apt-get install ros-kinetic-desktop-full
~~~

### rosdep初期化
~~~
sudo rosdep init
rosdep update
~~~
もしエラーになるときは"--os"オプションを付加する
~~~
rosdep --os=ubuntu:xenial  ....
~~~

### catkin初期化
~~~
cd ~
mkdir -p catkin_ws/src
cd catkin_ws/src
catkin_init_workspace
~~~

### アカウントの初期設定
.bashrcの最後に以下を入れる
~~~
source /opt/ros/kinetic/setup.bash
source ~/catkin_ws/devel/setup.bash
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
export NODE_PATH=/usr/lib/node_modules
export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:$PYTHONPATH
~~~

### Pythonパッケージ追加

pipがなければまずpipをインストール
~~~
sudo apt install python-pip python-dev
~~~
pipのversionは9.0.1以上が必要になる。低い場合はアップデートする。
~~~
pip install pip==9.0.3 --user
~~~

======
## RoVIのインストール
1. Gitからチェックアウト(israfelブランチ)
~~~
 git clone -b israfel --depth 1 https://github.com/YOODS/rovi.git
~~~
あとはroviのREADMEのとおりインストールを続ける
