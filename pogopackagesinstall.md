```
apt update
apt upgrade
apt install build-essential devscripts debhelper quilt cdbs fakeroot dh-autoreconf autoconf automake libtool pkg-config libmpdclient-dev libmicrohttpd-dev python-requests libjsoncpp-dev cifs-utils python-pip locales ttf-wqy-zenhei mopidy screen alsa-utils git-core
dpkg-reconfigure tzdata #时区调整为Asia Shanghai
dpkg-reconfigure locales #中文乱码调整编码为en_US.UTF8 zh_CN GB2312 zh_CN GBK GBK zh_CN UTF-8 UTF-8，修改locale文件
vi /etc/default/locale   #加入LANG=en_US.UTF-8
```
### Upmpdcli 编译 ###

#### libupnp编译 ####

```
wget https://www.lesbonscomptes.com/upmpdcli/downloads/libupnp-1.6.20.jfd5.tar.gz
tar -xzvf libupnp-1.6.20.jfd5.tar.gz
cd libupnp-1.6.20.jfd5
./configure --prefix=/usr --sysconfdir=/etc
make
make install
```

#### libupnpp编译 ####

 ```
 git clone https://@opensourceprojects.eu/git/p/libupnpp/code libupnpp-code
 cd libupnpp-code
 sh autogen.sh
 ./configure --prefix=/usr --sysconfdir=/etc
 make
 make install
 ```
 
 #### upmpdcli编译 ####
 ```
 git clone https://@opensourceprojects.eu/git/p/upmpdcli/code upmpdcli-code
 cd upmpdcli-code
 sh autogen.sh
 ./configure --prefix=/usr --sysconfdir=/etc
 make
 make install
 cp debian/upmpdcli.ini /etc/init.d/upmpdcli
 chmod a+x /etc/init.d/upmpdcli
 ```
