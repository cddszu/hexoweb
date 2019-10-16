---
title: mac 安装nginx
---


> 最近因为客户现场会进行nginx转发，为了保证客户现场的nginx转发没有问题，所以需要提前自测

1. 安装brew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
```
这里有提示目录不存在，要创建需要按return键，window下的回车键 安装成功后，
```
<!-- 更新brew-->
brew update

<!-- 查看安装信息-->
brew -v

<!-- 查看是否安装了nginx-->
brew info nginx 
```

2. 安装nginx
```
brew install nginx
```
> 这两部分的安装都会花费比较长的时间

3. nginx 相关命令
```
<!--启动nginx-->
nginx
<!--重启nginx-->
nginx -s reload
<!--停止nginx-->
nginx -s strop

```
4. 查看nginx的安装目录
```
open /usr/local/Cellar/nginx/
```
5. 查看nginx的配置文件 nginx.conf
```
open /usr/local/etc/nginx/
```

6. 在nginx中添加代理，编辑器打开 nginx.conf
```
<!--修改http里面的内容-->
http {
    server {
    listen 8000;
    server_name server_name;

    location /path {
      proxy_pass http://ip:port;
    }
    
    ...
    <!--只需要重复添加server就能有不同的代理-->
    server {
    listen 8000;
    server_name server_name;

    location /path {
      proxy_pass http://ip:port;
    }
}

```
> 修改nginx.conf 后需要重启nginx

7. 解决 `ERR_CONTENT_LENGTH_MISMATCH`报错
```
<!--修改nginx.conf-->
<!--第一行-->
user root owner;
```
修改后执行 `sudo nginx -s reload`，后续的启动都需要加上 `sudo` 命令