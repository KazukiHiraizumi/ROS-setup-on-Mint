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

### Nodejsインストール

現時点(Sep18)でのHeadはNode9なので9を入れる
~~~
cd ~
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install nodejs
~~~
よく使うパッケージも追加
~~~
npm install mathjs
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
export NODE_PATH=/usr/lib/node_modules
export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:$PYTHONPATH
source /opt/ros/kinetic/setup.bash
source ~/catkin_ws/devel/setup.bash
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
~~~

### Pythonパッケージ追加

pipがなければまずpipをインストール
~~~
sudo apt install python-pip python-dev
~~~
ROSを入れるとOpenCVなどかなりのパッケージが追加される。それ以外では  
- scipy
~~~
pip install scipy
~~~
- pybind11
~~~
sudo -H pip install pybind11
~~~
- Open3D
~~~
pip install open3d-python
~~~

## ROS GigEカメラインストール
### Aravisライブラリのインストール

http://ftp.gnome.org/pub/GNOME/sources/aravis/0.4/  から  "aravis-0.4.1.tar.xz" をダウンロード。curlコマンドで直接DLしてもいい
~~~
curl http://ftp.gnome.org/pub/GNOME/sources/aravis/0.4/aravis-0.4.1.tar.xz >aravis-0.4.1.tar.xz
~~~
展開してaravis...以下にcdし
~~~
./configure
make
sudo make install
~~~
カメラを接続し
~~~
arv-tool-0.4
~~~
でカメラを認識できればOK

!!エラー

ld.so.conf設定が反映されていないと以下のようなエラーが出る
~~~
arv-tool-0.4: error while loading shared libraries: libaravis-0.4.so.0: cannot open shared object file: No such file or directory
~~~
このようなときはldconfigを実行する
~~~
sudo ldconfig
~~~

### camera_aravisパッケージのインストール
~~~
cd ~/catkin_ws/src
git clone https://github.com/YOODS/camera_aravis.git
~~~
オリジナルのcamera_aravisから以下変更有り
- スキャンタイムを1s⇒0.02s
- Get/SetGregサービス追加

ビルドは
~~~
cd ~/catkin_ws/src
catkin_make
~~~

OKならViewerで見てみる
~~~
rosrun image_view image_view image:=/camera/image_raw
~~~

======
## RoVIのインストール
### 準備
1. rosnodejsをインストール
~~~
cd ~
npm install rosnodejs
~~~
- このパッケージはパッチが要るので注意。以下のようにパッチします。
~~~
cd ~
git clone https://github.com/RethinkRobotics-opensource/rosnodejs
cd ~/node_modules/rosnodejs
rm -rf dist
cp -a ~/rosnodejs/src/ dist
~~~
2. js-yamlをインストール
~~~
cd ~
npm install js-yaml
~~~
### RoVIのインストール
1. Gitからチェックアウト
~~~
 git clone https://github.com/YOODS/rovi.git
 git checkout -m matriel
~~~
2. Eigenをインストール
~~~
cd ~/catkin_ws/src/rovi
wget http://bitbucket.org/eigen/eigen/get/3.3.4.tar.gz
tar xvzf 3.3.4.tar.gz
mkdir include
mv eigen-eigen-5a0156e40feb/Eigen/ include
rm -rf eigen-eigen-5a0156e40feb/ 3.3.4.tar.gz
~~~
3. voxel...をビルド
~~~
cd voxel-noise_reduction
make
cp yodpy2.so ~/catkin_ws/devel/lib/python2.7/dist-packages/
~~~
4. 全部ビルドしてみよう
~~~
cd ~/catkin_ws
catkin_make
~~~

### RoVI実行
1. Network設定
デフォルトのカメラのIPアドレスは**192.168.222.1**です。PC側もそれに合わせて設定します。

カメラはラージパケットで転送してくるので、**MTU=9000**に設定します。

2. Launch
~~~
roslaunch rovi ycamctl.js ycam3
~~~
### Topics(to subscribe)
### Topics(to publish)
