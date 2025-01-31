[OpenTracker](https://erdgeist.org/arts/software/opentracker/)
=======

-------
That should leave you with an exectuable called opentracker and one debug version opentracker.debug.

Some variables in opentracker's Makefile control features and behaviour of opentracker. Here they are:

-DWANT_V6 makes opentracker an IPv6-only tracker. More in the v6-section below.
opentracker can deliver gzip compressed full scrapes. Enable this with -DWANT_COMPRESSION_GZIP option.
Normally opentracker tracks any torrent announced to it. You can change that behaviour by enabling ONE of -DWANT_ACCESSLIST_BLACK or  -DWANT_ACCESSLIST_WHITE. Note, that you have to provide a whitelist file in order to make opentracker do anything in the latter case. More in the closed mode section below.
opentracker can run in a cluster. Enable this behaviour by enabling -DWANT_SYNC_LIVE. Note, that you have to configure your cluster before you can use opentracker when this option is on.
Some statistics opentracker can provide are sensitive. You can restrict access to these statistics by enabling -DWANT_RESTRICT_STATS. See section statistics for more details.
Fullscrapes is bittorrent's way to query a tracker for all tracked torrents. Since it's in the standard, it is enabled by default. Disable it by commenting out -DWANT_FULLSCRAPE.
By default opentracker will only allow the connecting endpoint's IP address to be announced. Bittorrent standard allows clients to provide an IP address in its query string. You can make opentracker use this IP address by enabling -DWANT_IP_FROM_QUERY_STRING.
Some experimental or older, deprecated features can be enabled by the -DWANT_LOG_NETWORKS, -DWANT_SYNC_SCRAPE or -DWANT_IP_FROM_PROXY switch.
Currently there is some packages for some linux distributions and OpenBSD around, but some of them patch Makefile and default config to make opentracker closed by default. I explicitly don't endorse those packages and will not give support for problems stemming from these missconfigurations.


-------
Invocation
opentracker can be run by just typing ./opentracker. This will make opentracker bind to 0.0.0.0:6969 and happily serve all torrents presented to it. If ran as root, opentracker will immediately chroot to . (or any directory given with the -d option) and drop all priviliges after binding to whatever tcp or udp ports it is requested.

When options were few, opentracker used to accept all of them from command line. While this still is possible for most options, using them is quite unhandy: an example invocation would look like ./opentracker -i 23.23.23.7 -p 80 -P 80 -p 6969 -i 23.23.23.8 -p 80 -r http://www.mytorrentsite.com/ -d /usr/local/etc/opentracker -w mytorrents.list -A 127.0.0.1.

opentracker now uses a config file that you can provide with the -f switch.


-------
Config file
opentracker's config file is very straight forward and a very well documented example config can be found in the file opentracker.conf.sample.


-------
Closed mode
While personally I like my tracker to be open, I can see that there's people that want to control what torrents to track – or not to track. If you've compiled opentracker with one of the accesslist-options (see Build instructions above), you can control which torrents are tracked by providing a file that contains a list of human readable info_hashes. An example whitelist file would look like

0123456789abcdef0123456789abcdef01234567
890123456789abcdef0123456789abcdef012345
To make opentracker reload it's white/blacklist, send a SIGHUP unix signal.


-------
Statistics
Given its very network centric approach, talking to opentracker via http comes very naturally. Besides the /announce and /scrape paths, there is a third path you can access the tracker by: /stats. This request takes parameters, for a quick overview just inquire /stats?mode=everything`.

Statistics have grown over time and are currently not very tidied up. Most modes were written to dump legacy-SNMP-style blocks that can easily be monitored by MRTG. These modes are: peer, conn, scrp, udp4, tcp4, busy, torr, fscr, completed, syncs. I'm not going to explain these here.

The statedump mode dumps non-recreatable states of the tracker so you can later reconstruct an opentracker session with the -l option. This is beta and wildly undocumented.

You can inquire opentracker's version (i.e. CVS versions of all its objects) using the version mode.


下面中文的，方便讲解使用
Centos 6.9 x64位安装说明
``` markdown
yum -y install unzip wget gcc zlib-devel
wget https://github.com/lj5645/OpenTracker/archive/master.zip -O /root/OpenTracker.zip
unzip OpenTracker.zip
mv OpenTracker-master /home
cd /home
cd OpenTracker-master
cd libowfat
make
cd /home/OpenTracker-master
cd opentracker
make

```

运行程序，并且监听tcp和udp端口的8080，并且自动后台工作
``` markdown
./opentracker -p 8080 -P 8080 &
```

推荐使用8080端口，基本上免备案不会扫描，而且CloudFlare也支持使用CDN隐藏IP来使用。

多端口可以加多个即可
``` markdown
./opentracker -p 8080 -P 8080 -p 6961 -P 6961 -p 2710 -P 2710 &
```


输入下方命令可以查看是否在工作中
``` markdown
top -b -n 1 |grep opentracker
```

通过浏览器访问程序的统计功能
``` markdown
http://ip:8080/stats
http://ip:8080/stats?mode=everything
```

软件的自带帮助说明
``` markdown
Usage: ./opentracker [-i ip] [-p port] [-P port] [-r redirect] [-d dir] [-u user] [-A ip] [-f config] [-s livesyncport]
	-f config include and execute the config file
	-i ip     specify ip to bind to (default: *, you may specify more than one)
	-p port   specify tcp port to bind to (default: 6969, you may specify more than one)
	-P port   specify udp port to bind to (default: 6969, you may specify more than one)
	-r redirecturlspecify url where / should be redirected to (default none)
	-d dir    specify directory to try to chroot to (default: ".")
	-u user   specify user under whose priviliges opentracker should run (default: "nobody")
	-A ip     bless an ip address as admin address (e.g. to allow syncs from this address)

Example:   ./opentracker -i 127.0.0.1 -p 6969 -P 6969 -f ./opentracker.conf -i 10.1.1.23 -p 2710 -p 80
```

间隔可以在编译前进行修改，默认我改成了2小时，以便降低服务器宽带开销
``` markdown
trackerlogic.h:#define OT_CLIENT_REQUEST_INTERVAL (60*30)#(60*120)，客户端默认间隔请求时间
trackerlogic.h:#define OT_CLIENT_TIMEOUT_SEND (60*15)#(60*30)，客户端最小间隔请求时间，部分客户端的可能不会准守
trackerlogic.h:##define OT_PEER_TIMEOUT 45#144，服务端删除peer时间，单位分钟
trackerlogic.h:##define OT_CLIENT_REQUEST_VARIATION (60*6)#服务端下发随机客户端间隔请求时间调整，提高性能，默认允许误差随机6分钟内，保持默认无修改
```


utorrent中制作种子过程tracker写
http://服务器ip:8080/announce

udp://服务器ip:8080/announce


反映：http://bbs.itzmx.com/thread-18214-1-1.html
-------
程序来自官网
https://erdgeist.org/arts/software/opentracker/
mail:erdgeist@erdgeist.org
