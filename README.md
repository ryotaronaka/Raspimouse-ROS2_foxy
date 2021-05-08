# Raspimouse : RapsberryPi3 + Ubuntu 20.04 + ROS2 foxy
これはRaspimouseをRapsberryPi3 + Ubuntu 20.04 + ROS2 foxyで動かすための手順です。<br>
基本的にはRT-Shopさんの下記参考URLの1番と2番の手順をなぞれば良いのですが、ROS2のサンプルであるraspimouse2のソースをRapsberryPi3でビルドするとRaspimouseパッケージが25%で止まってしまう現象があります。<br>
この現象を回避する方法をtshellさんが記載してくれていたのですが、おそらくホストPCもUbuntuだと思われましたので、Windows10のDockerを使ってインストールする手順を記載します。<br>
初心者にもわかるようにUbuntu20.04のインストール方法から順番に記載します。ただし、Linuxに由来するコマンドや知識はご自身で補完してください。

![IMG_0723](https://user-images.githubusercontent.com/34445043/117532437-7ef08280-b022-11eb-8467-5362277e8733.jpg)

## 参考URL
1. Inastall Ubuntu 20.04 and ROS2 for RasberryPi3 : https://www.rt-shop.jp/blog/archives/11061
2. Build the device driver and raspimouse2.git : https://www.rt-shop.jp/blog/archives/11249
3. tshell's blog(build raspimouse2) : https://tshell.hatenablog.com/entry/2021/03/08/222557
4. tshell's blog(install ROS2) : https://tshell.hatenablog.com/entry/2021/03/07/224537

## raspimouse2のビルドが面倒だという方へ
このGitページに保管している rsp.tar がビルド済みのraspimouse2になります。<br>
こちらを展開すればすぐに使えます。（Linux初心者のため環境が変わると使えるかわかりません。）下記Requirementsと同じ仕様ならOKだと思います。<br>

rsp.tarの中身はフォルダで構成されていて、<br><br>
```
rsp.tar
 ├ build
 ├ install
 ├ log
 └ src
```
になっています。<br><br>

これらのフォルダの配置は
```
/
 └home
  └ubuntu
   └ros2_ws
     ├ build
     ├ install
     ├ log
     └ src
```
となるようにします。<br><br>

そのため、rsp.tarを/home/ubutu/ros2_wsに置いて、
```
$ cd /home/ubuntu/ros2_ws
$ tar xvf rsp.tar
```
とやれば、上記フォルダ配置になります。<br><br>
あとはROS2とraspimouse2を使うだけになります。<br>
詳しくはこの記事を読んでください。

# Requirements
* Raspberry Pi Mouse(Raspimouse)
  - https://rt-net.jp/products/raspberrypimousev3/
  - RT Robot Shop
  - 私はRaspimouse V2 を使いました。
* RaspberryPi
  - Raspiemouseのバージョンに合わせて、準備してください。V2の場合はRaspberryPi3、V3の場合はRaspberryPi4。
  - microSD : 16GB 以上が良いと思います。
* Linux for RaspberryPi3
  - ubuntu-20.04.2-preinstalled-server-arm64+raspi.img.xz
  - https://ubuntu.com/download/raspberry-pi
* Device Driver
  - https://github.com/rt-net/RaspberryPiMouse
* ROS2
  - https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html
* Windows10 
  - version : 20H2
  - Docker Desktop
  - Visual Studio Code(Docker extentionをインストールしておくこと)
  - Powershell
  - Etcher(https://www.balena.io/etcher/) : microSDにUbuntuイメージを書き込むソフト
  - WinSCP(FTPツール) : https://ja.osdn.net/projects/winscp/

# RapsberryPi3 に Ubuntu 20.04 をインストールする
Ubuntu.comからRaspberryPiのArm64bitに対応したイメージがダウンロードできるので、ダウンロードしてください。<br>
2021.05.08時点ではUbuntu Server 20.04.2 LTSが選べました。LTSというタイプを選ぶのが良いそうです。このあたりはLinux系の詳しい情報を調べてください。<br>
ダウンロードしたイメージをEtcherというソフトを使ってイメージを書き込みます。<br>
書き込みが終わったら、microSDカードをRaspberryPiに差し込んで起動させます。
  - 立ち上げる前にUSBキーボード、有線LAN、モニター(HDMI)を接続しておいてください。立ち上げて、後から接続しても検知されないので、その場合は電源を無理やり切りましょう。本当は良くないのですが、大丈夫です。
  - RaspberryPi本体は、まだRaspimouseに搭載しなくても良いですが、デバイスドライバーをビルドする時点では搭載した方が良いです。
 
ubuntu server の場合、最初はlogin idもパスワードも「ubuntu」なので、それでログインしてください。<br>
ログインするとこんな感じになります。
```
ubuntu@ubuntu:`$ 
```
以降、コマンドを入力する部分を
```
$ 
```
と記載します。

パスワードはこのままではまずいので、変更します。パスワードはご自身で決めてください。
```
$ passwd
Changing password for pi.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
```

# Windows10からRasberryPiのUbuntuへSSH接続するための準備
SSH接続は、暗号化された状態でサーバー間やPC間を通信する方式です。<br>
Ubuntuが立ち上がったRaspberryPiで作業します。<br>
まず、RaspberryPiのIPアドレスを調べます。IPアドレスはサンプルです。
```
$ ip addr show eth0
3: eth0: ...
   inet 192.168.1.9/24 ...
```
次にSSHの準備です。IPアドレスはご自身のアドレスに読み替えてください。
```
$ ssh ubuntu@192.168.1.9
(略)
Are you sure you want to conotinue connecting (yes/no)?
```
となるので、
```
Are you sure you want to conotinue connecting (yes/no)? yes
```
として、Enterを押してください。<br>
これでWindows10からSSH接続できます。<br><br>

Windows10からはPowershellでアクセスします。Usernameの部分はご自身の環境に読み替えてください。
```
PS C:\Users\Username>
```
SSH接続するには
```
PS C:\Users\Username>ssh ubuntu@192.168.1.9
ubuntu@192.168.1.9's password:yourpassword
(略)
ubuntu@ubuntu:~$
```
でアクセスできます。

# RaspberryPi Ubuntuのソフトアップデート
おまじないのように下記を実行してください。
```
$ sudo apt uodate
$ sudo apt upgrade
```
sudo は管理者権限で実行するためのコマンドです。パスワードを聞かれたらログインした際のubuntuユーザーのパスワードを記入すればOKです。

# デバイスドライバーのインストール
Ubuntuが立ち上がったRaspberryPiで作業します。<br>
デバイスドライバのソースファイルをGitHubからクローンして、クローン時にできたディレクトリを移動します。
```
$ cd /home/ubuntu
$ git clone https://github.com/rt-net/RaspberryPiMouse.git
$ cd RaspberryPiMouse/utils
$ sudo apt install linux-headers-$(uname -r) build-essential
$ ./build_install.bash
```
このビルドが上手くいくとピッという音が鳴り、光センサが一度光ると思います。これでビルド(インストール)成功です。<br>
ビルドしたものの場所はこちらです。
```
$ cd /home/ubuntu/RaspberryPiMouse/src/drivers
$ ls -l
lrwxrwxrwx 1 ubuntu ubuntu    17 Apr 30 14:10 Makefile -> Makefile.ubuntu14
-rw-rw-r-- 1 ubuntu ubuntu   268 Apr 30 11:28 Makefile.raspbian
-rw-rw-r-- 1 ubuntu ubuntu   477 Apr 30 11:28 Makefile.ubuntu14
-rw-rw-r-- 1 ubuntu ubuntu     0 Apr 30 14:11 Module.symvers
-rw-rw-r-- 1 ubuntu ubuntu    53 Apr 30 14:11 modules.order
-rw-rw-r-- 1 ubuntu ubuntu 61288 Apr 30 11:28 rtmouse.c
-rw-rw-r-- 1 ubuntu ubuntu 55752 Apr 30 14:11 rtmouse.ko
-rw-rw-r-- 1 ubuntu ubuntu    53 Apr 30 14:11 rtmouse.mod
-rw-rw-r-- 1 ubuntu ubuntu  2819 Apr 30 14:11 rtmouse.mod.c
-rw-rw-r-- 1 ubuntu ubuntu  7008 Apr 30 14:11 rtmouse.mod.o
-rw-rw-r-- 1 ubuntu ubuntu 51136 Apr 30 14:11 rtmouse.o
```
この中のrtmouse.koがカーネルモジュールと呼ばれるもので、ビルドしてできたデバイスドライバーになります。カーネルはLinux用語なので調べてみてください。<br><br>

ビルドした直後はrtmouse.koが実行されており、
```
$ ls -l /dev/rt*
crw-rw-rw- 1 root root 510, 0 May  8 10:30 /dev/rtbuzzer0
crw-rw-rw- 1 root root 511, 0 May  8 10:30 /dev/rtled0
crw-rw-rw- 1 root root 511, 1 May  8 10:30 /dev/rtled1
crw-rw-rw- 1 root root 511, 2 May  8 10:30 /dev/rtled2
crw-rw-rw- 1 root root 511, 3 May  8 10:30 /dev/rtled3
crw-rw-rw- 1 root root 508, 0 May  8 10:30 /dev/rtlightsensor0
crw-rw-rw- 1 root root 504, 0 May  8 10:30 /dev/rtmotor0
crw-rw-rw- 1 root root 506, 0 May  8 10:30 /dev/rtmotor_raw_l0
crw-rw-rw- 1 root root 507, 0 May  8 10:30 /dev/rtmotor_raw_r0
crw-rw-rw- 1 root root 505, 0 May  8 10:30 /dev/rtmotoren0
crw-rw-rw- 1 root root 509, 0 May  8 10:30 /dev/rtswitch0
crw-rw-rw- 1 root root 509, 1 May  8 10:30 /dev/rtswitch1
crw-rw-rw- 1 root root 509, 2 May  8 10:30 /dev/rtswitch2
```
のように表示されます。このrtxxxというファイルがデバイスファイルと呼ばれ、Raspimouseはデバイスファイルを通じて操作することができます。詳しくは上田隆一先生の著書「RaspberryPiで学ぶ　ROSロボット入門」をご覧ください。デバイスファイルの直接的な動かし方などが解説されています。ROSであろうがROS2であろうが、デバイスファイルの使い方は同じです。<br><br>

RasberryPiを再起動させると、自動ではデバイスドライバーが読み込まれないので、細工します。setup.bashというシェルスクリプトを準備して、cronを使って起動時にシェルスクリプトを読みこむことでデバイスドライバが起動時に読み込まれるようします。(cd は cd /home/ubuntu と同じ意味で、ホームディレクトリに移動します。)
```
$ cd
$ mkdir pimouse_setup
$ cd pimouse_setup
```
setup.bashを作ります。viというエディタを使うのが一般的だと思いますが、癖があって慣れが必要です。使い方は調べてください。
```
$ sudo vi setup.bash
```
中身は以下のように記載してください。
```
#!/bin/bash -xve

exec 2> /tmp/setup.log

cd /home/ubuntu/RaspberryPiMouse/src/drivers/
/sbin/insmod rtmouse.ko

sleep 1
chmod 666 /dev/rt*

echo 0 > /dev/rtmotoren0
```
exec 2> /tmp/setup.log は /tmp にログを出すためのコマンドです。<br>
/sbin/insmod rtmouse.ko　は、rtmouse.ko を実行してカーネルにデバイスファイルをねじ込むコマンドです。<br>
デバイスドライバがすぐに出来ないのでsleep 1で1秒待ちます。<br>
chmod 666 /dev/rt* はroot以外の権限(今はubuntuユーザーです)でもデバイスファイルが使えるように権限を変更します。
echo 0 > /dev/rtmotoren0 は安全のためモーターの電源を切るコマンドを送っています。<br>
記載した中身の確認は
```
$ cat setup.bash
```
で中身を見ることができます。<br><br>

次にcron設定です。
```
$ sudo vi crontab.conf
```
中身は以下のように記載してください。
```
@reboot /home/ubuntu/pimouse_setup/setup.bash
```
crontabにセットします。
```
$ sudo crontab crontab.conf
```
これで以下のようなファイル構成になります。
```
/
 └home
  └ubuntu
   ├RaspberryPiMouse
   │ ├src
   │ ├utils
   │ ├ ・・・
   └pimouse_setup
     ├setup.bash
     └crontab.conf
```
