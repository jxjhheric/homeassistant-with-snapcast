# homeassistant-with-snapcast   
multi-room system with homeassistant、modify and snapcast on pogo e02 and pogo v4

**WIP project！！！！！**

**硬件列表**
```
1.pogoplug e02 
2.pogoplug v4
3.usb DAC声卡: pcm2704芯片产品
4.usb DAC声卡： C-Medai HS-100B芯片产品
5.旧安卓手机
```
![](https://github.com/jxjhheric/homeassistant-with-snapcast/blob/master/snapcast%E5%A4%9A%E9%9F%B3%E4%B9%90%E7%B3%BB%E7%BB%9F.png?raw=true)

**升级系统到最新的 Debian Stretch**
```
apt-get update
apt-get upgrade
```
**让USB DAC正常工作**

    apt-get install alsa-utils
编辑alsa.conf文件

    vi /usr/share/alsa/alsa.conf 
把

    defaults.ctl.card 0  
    defaults.pcm.card 0
中的0改成1

编辑/创建/etc/asound.conf文件

    vi /etc/asound.conf 
输入如下代码：
```
pcm.!default  {
 type hw card 1
}
ctl.!default {
 type hw card 1
}
```
测试是否工作：

    speaker-test -c2
调节音量：

    alsamixer -c 1
DAC一切就绪后就可以开始下面的工作了！！！
==============================================================================================
**安装mopify**  
首先安装mopify来当做音乐播放器  

    apt-get install mopidy
    pip install Mopidy-Local-SQLite #安装数据库插件

**配置mopify**

如果是以mopify身份运行程序，配置文件如下

    vi /etc/mopidy/mopidy.conf
  
如果以root身份运行程序，配置文件如下

    vi /root/.config/mopidy/mopidy.conf
  
内容都统一修改为：
```
[m3u]
playlists_dir = /var/lib/mopidy/playlists


[core]
cache_dir = /var/cache/mopidy
config_dir = /etc/mopidy
data_dir = /var/lib/mopidy
max_tracklist_length = 10000
restore_state = false

[logging]
color = true
console_format = %(levelname)-8s %(message)s
debug_format = %(levelname)-8s %(asctime)s [%(process)d:%(threadName)s] %(name)s
debug_file = /var/log/mopidy/mopidy-debug.log
config_file = /etc/mopidy/logging.conf

[audio]
mixer = software
mixer_volume =
#output = autoaudiosink
output = audioresample ! audio/x-raw,rate=48000,channels=2,format=S16LE ! audioconvert ! wavenc ! filesink location=/tmp/snapfifo  #输出音频到snapserver
buffer_time =

[proxy]
scheme =
hostname =
port =
username =
password =

[mpd]
enabled = true
hostname = ::
port = 6600
password =
max_connections = 20
connection_timeout = 60
zeroconf = Mopidy MPD server on $hostname
command_blacklist =
  listall
  listallinfo
default_playlist_scheme = m3u

[http]
enabled = true
hostname = ::
port = 6680
static_dir =
zeroconf = Mopidy HTTP server on $hostname

[stream]
enabled = true
protocols =
  http
  https
  mms
  rtmp
  rtmps
  rtsp
metadata_blacklist =
timeout = 5000

[m3u]
enabled = true
base_dir =
default_encoding = latin-1
default_extension = .m3u8
playlists_dir = /var/lib/mopidy/playlists

[softwaremixer]
enabled = true

[file]
enabled = true
media_dirs =
  $XDG_MUSIC_DIR|Music
  ~/|Home
excluded_file_extensions =
  .jpg
  .jpeg
show_dotfiles = false
follow_symlinks = false
metadata_timeout = 1000

[local]
enabled = true
#library = json
library = sqlite  #安装Mopidy-Local-SQLite后使用此配置，会把乐库的歌曲索引成数据库，读写速度更快，扫描后不用重启程序
#media_dir = /var/lib/mopidy/media
media_dir = /home/Music #根据自己的文件夹修改
scan_timeout = 1000
scan_flush_threshold = 100
scan_follow_symlinks = false
excluded_file_extensions =
  .directory
  .html
  .jpeg
  .jpg
  .log
  .nfo
  .png
  .txt
```
确认配置文件是否正确：

    mopidyctl config
  
没问题后直接运行扫描程序

    mopidyctl local scan

第一次运行完成后重启下mopidy：

    /etc/init.d/mopidy restart
  
正确的话可以看到文件夹中的音乐文件了。

Android端推荐使用程序: [malp](https://github.com/gateship-one/malp/releases/download/release-25/malp-1.1.16.apk)

**本地音乐库的歌曲来源**

推荐一个网易音乐的下载项目：https://github.com/codezjx/netease-cloud-music-dl
![](https://github.com/codezjx/netease-cloud-music-dl/raw/master/preview.gif)

首先下载源码：

    $ git clone https://github.com/codezjx/netease-cloud-music-dl.git
进入根目录，然后执行：

    $ python setup.py install
最终显示以下log，表示顺利安装：
```
running install
running bdist_egg
running egg_info
...
...
Finished processing dependencies for netease-cloud-music-dl==x.x.x
```

后续直接在命令行中通过ncm指令即可快速调用相关功能，**Warning: 目前只支持Python3.x版本**

**为了能播放网络上的音乐，这里安装一个UPnP Renderer ----upmpdcli ，可以直接通过DLNA来播放音乐（苹果用户可以安装[shairport-sync](https://github.com/mikebrady/shairport-sync)）**

**安装upmpdcli**

    vi /etc/apt/sources.list.d/upmpdcli.list
 填入以下：
 ```
deb http://www.lesbonscomptes.com/upmpdcli/downloads/raspbian/ stretch main
deb-src http://www.lesbonscomptes.com/upmpdcli/downloads/raspbian/ stretch main
         
```
保存后再运行一下命令：
```
sudo apt-get update
sudo apt-get install upmpdcli
```
当然也可以直接编译：
```
sudo apt-get install build-essential devscripts debhelper quilt cdbs
sudo apt-get install fakeroot
sudo apt-get install dh-autoreconf autoconf automake libtool pkg-config
sudo apt-get install libexpat1-dev libcurl4-gnutls-dev g++
sudo apt-get install libmpdclient-dev libmicrohttpd-dev python-requests libjsoncpp-dev

# Only for wheezy:
# sudo apt-get install hardening-wrapper

# Only for jessie and later
sudo apt-get install dh-systemd

mkdir build # Or any name you fancy
cd build 

mkdir libupnp6
cd libupnp6
apt-get source libupnp6
cd libupnp-1.6.20.jfd5/
debuild -us -uc
cd ..
sudo dpkg -i *.deb
cd ..

# Current dir is now build/

mkdir libupnpp4
cd libupnpp4
apt-get source libupnpp4
cd libupnpp4-0.16.0
debuild  -us -uc
cd ..
sudo dpkg -i *.deb

# Current dir is now build/

mkdir upmpdcli
cd upmpdcli
apt-get source upmpdcli
cd upmpdcli-1.2.15/
debuild  -us -uc
cd ..
sudo dpkg -i *.deb
```
安装完成后无缝与MPD对接，安卓手机端的网易云音乐等app中就可以看到Umpd的播放设备了！！
==================================================================================
**下载并安装snapserver/snapclient**
```
cd /home
wget http://ftp.cn.debian.org/debian/pool/main/s/snapcast/snapserver_0.13.0-1_armel.deb
wget http://ftp.cn.debian.org/debian/pool/main/s/snapcast/snapclient_0.13.0-1_armel.deb
dpkg -i snapclient*
apt-get -f install #安装缺失的依赖
```
运行程序：

    /etc/init.d/snapserver start
    /etc/init.d/snapclient start
    
[安卓端apk](https://github.com/badaix/snapcast/releases/download/v0.13.0/Snapcast_0.13.0.apk)

**snapcast、mopify接入homeassistant：**

在configuration.yaml中加入：

    media_player:
      - platform: snapcast
        host: xxx.xxx.xxx.xxx
      - platform: mpd
        host: xxx.xxx.xxx.xxx
        name: Main Controller
        
**homeassistant 自动化**

~~todo：~~
~~- 音乐随人动~~
把musicfollowme.yaml放入homeassistant的config/packages文件夹中，重启即可！
