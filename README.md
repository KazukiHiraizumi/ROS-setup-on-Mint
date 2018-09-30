# ROSのPCのセットアップ
====
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
- ブートUSBから起動しgrubメニューを出す(起動時にESCキーを押すなど)
- カーネルオプションを編集する("e"キーを押すなど、画面指示を見ること)
- オプションの"boot=casper"を"root=/dev/sda2"に変更
- 起動する(F10を押すなど、画面指示を見ること)
- 再起動できたら、4のupdate-grubのログを精査する

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

====
## 初期設定 LinuxMint

1. アップグレード
~~~
sudo apt upgrade
~~~
ついでに
~~~
sudo apt update
~~~

2. ツール・ライブラリの追加

後々のために以下を追加する
  - g++
  - git
  - automake
  - intltool
  - libgsreamer*-dev

3. 電源管理

スタートメニュー⇒設定⇒電源管理
  - 一般  
  スリープモードは"ハイバネート"を選択
  - システム  
  同じく"ハイバネート"
  - ディスプレイ  
  □電源管理を行う、はOff
  - Secuity  
  LightLocker"しない"を選択

4. デスクトップ切り替え

タスクバー⇒パネル⇒新しいアイテムの追加  
ワークスペーススイッチャを追加

5. Nodejsインストール

現時点(Sep18)でのHeadはNode9なので9を入れる
~~~
cd ~
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install nodejs
~~~
====

## インストール ROS

ROSのリリースはKineticが前提です。  
http://wiki.ros.org/kinetic/Installation/Ubuntuに手順がありますが、Mintに入れるので若干修正が要ります。

1. Repos設定

先のページでは
~~~
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
~~~
ですが、Mintだと"lsb_release -sc"が"sonya"になっしまうので、あとから"/etc/apt/sources.list.d/ros-latest.list"の"sonya"を"xenial"に修正するか、最初から
~~~
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu xenial main" > /etc/apt/sources.list.d/ros-latest.list'
~~~
とします。

2. Key設定
~~~
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
~~~
もしエラーになったら、キーサーバを"hkp://pgp.mit.edu:80"とか"hkp://keyserver.ubuntu.com:80"でやってみるそう。


3. インストール

フルセットでOKです
~~~
sudo apt-get update
sudo apt-get install ros-kinetic-desktop-full
~~~

4. rosdep初期化
~~~
sudo rosdep init
rosdep update
~~~
もしエラーになるときは"--os"オプションを付加する
~~~
rosdep --os=ubuntu:xenial  ....
~~~




①Ubuntuバージョン

ROSの推奨バージョンはkineticいうやつ
対応するUbuntuは15.10か16.04(xenial)。16.04がLTSなのでこっち。Mintなら18.2(sonya)

②追加で必要なもの
・G++(sudo apt-get install g++
・Git(sudo apt-get inslall git

③インストールkinetic
http://wiki.ros.org/kinetic/Installation/Ubuntu
のとおり・・・
！apt updateの前に
/etc/apt/sources.list.d/ros-latest.list
のソースがsonyaになっていたらxenialに書き換える

④apt updateで arm64..404 エラーが出る
Jetpackがインストール済だとこういうややこしいことになる。
とりあえず以下のように一旦Armのクロス環境を消すしかない？？
<ans>
dpkg --get-selections | grep i386 | awk '{print $1}'

And then if happy with them being removed, run

apt-get remove --purge `dpkg --get-selections | grep i386 | awk '{print $1}'`

And then retry the

dpkg --remove-architecture i386

apt-get updatesource /opt/ros/kinetic/setup.bash
source catkin_ws/devel/setup.bash
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311

</ans>

⑤アカウントの初期設定
.bashrcの最後に以下を入れた方がよい。ROSワークスペースは*_wsとする。最後の2行はスタンドアロンで使うとき。
source /opt/ros/kinetic/setup.bash
for rcsetup in *_ws/devel/setup.bash
do
	source $rcsetup
done
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311


