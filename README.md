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

**下载并安装snapserver/snapclient**
```
cd /home
wget http://ftp.cn.debian.org/debian/pool/main/s/snapcast/snapserver_0.13.0-1_armel.deb
wget http://ftp.cn.debian.org/debian/pool/main/s/snapcast/snapclient_0.13.0-1_armel.deb
dpkg -i snapclient*
apt-get -f install #安装缺失的依赖
```
