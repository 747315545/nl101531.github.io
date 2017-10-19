---
title: 网站升级HTTPS与HTTP2记录
tags:
  - https    
  - http2
categories: 运维
date: 2017-10-18 22:15:37
updated: 2017-10-18 22:15:44
---
最近看到两篇文章对于HTTPS与HTTP2两者讲解的很详细,分享并实践一下,正好近期捣鼓了一个工具类站点[https://www.itoolshub.com/](https://www.itoolshub.com/),可以用来实验.
**文章地址**
[为什么要把网站升级到HTTPS](https://fed.renren.com/2017/09/03/upgrade-to-https/)
[怎样把网站升级到http/2](https://fed.renren.com/2017/09/23/http2/)

### 升级HTTPS
升级的好处如文章所说,另外这里主要是按照文章中的教程来处理,不过我的服务器是Centos7,大体步骤相同,选择操作系统与服务器后会到该页面[https://certbot.eff.org/#centosrhel7-nginx](https://certbot.eff.org/#centosrhel7-nginx).
要注意`sudo certbot --nginx`命令,该命令默认会去`usr/bin/nginx`与`/etc/nginx`下寻找nginx启动文件与配置文件,因此如果你的nginx位置不对就会报`not install`错误,导致无法进行,解决方案为使用软链接形式创建链接.
```sh
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
ln -s /usr/local/nginx/conf/ /etc/nginx
```
命令都执行完后,nginx的server节点下会多出如下配置,第一部分为监听443端口.即https默认端口,使用该证书.第二部分是对非https请求重定向到https.这样也就学到了nginx是怎么配置https请求.
```conf
    listen 443 ssl; # managed by Certbot
ssl_certificate /etc/letsencrypt/live/itoolshub.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/itoolshub.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot
```

### 升级HTTP2
正如原作者所说HTTP2具有太多的优势,比如多路复用，对同一个域的服务器只建立一次TCP连接，加载多个资源，使用二进制帧传输，同时会对http头部进行压缩,大大提高了传输的效率.
**要注意的是**
Nginx启用http2则需要安装`http_v2_module`模块,并且需要openssl版本大于1.0.2,由于Chrome改变了验证http2的方式,详情可以参考此文章[https://news.cnblogs.com/n/545972/](https://news.cnblogs.com/n/545972/).

**推荐做法**
nginx的模块是支持静态编译的,因此自己下载所需要的软件版本,然后编译时指定配置相应的版本是最佳解决方案.如下脚本,我配置了`http_v2_module`和`/opt/openssl-OpenSSL_1_0_2k`的版本,这样nginx编译时则不会去使用系统自带的openssl.注意不要`make install`,该命令是会执行安装操作,也就是会把你之前安装的nginx覆盖掉.
```sh
./configure \
  --prefix=/usr/local/nginx \
  --conf-path=/etc/nginx/nginx.conf \
  --pid-path=/var/local/nginx/nginx.pid \
  --lock-path=/var/lock/nginx/nginx.lock \
  --error-log-path=/quding/logs/nginx/error.log \
  --http-log-path=/quding/logs/nginx/access.log \
  --with-http_gzip_static_module \
  --http-client-body-temp-path=/var/temp/nginx/client \
  --http-proxy-temp-path=/var/temp/nginx/proxy \
  --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
  --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
  --with-http_ssl_module \
  --http-scgi-temp-path=/var/temp/nginx/scgi \
  --with-http_v2_module \
  --with-openssl=/opt/openssl-OpenSSL_1_0_2k
echo 'make start...'
make
```
脚本执行完毕后,手动copy nginx替换原nginx,使用`nginx -V`查看所使用的openssl版本信息,如下所示则不会有大问题.
![](http://oobu4m7ko.bkt.clouddn.com/1508370655.png?imageMogr2/thumbnail/!100p)

最后在https监听那里加上http2,nginx reload下即可.
```conf
listen 443 ssl http2;
```
对于chrome最可信的调试方式是访问`chrome://net-internals/#http2`,如果显示你的网站使用的协议为h2,那么恭喜你开启了http2

目前[https://www.itoolshub.com/](https://www.itoolshub.com/)已经开启了HTTPS与HTTP2.但是图片是放在七牛云的,七牛的HTTPS收费,所以目前没解决,由于图片并不是很多后期迁到自己的服务器上,或者使用base64形式.

