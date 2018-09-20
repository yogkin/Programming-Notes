# nginx推流服务器搭建

- 先下载安装  nginx 和 nginx-rtmp 编译依赖工具

```
sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev
```

- 下载 nginx 和 nginx-rtmp源码（wget是一个从网络上自动下载文件的自由工具）

```Shell
wget http://nginx.org/download/nginx-1.7.5.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
```

- 安装unzip工具，解压下载的安装包

```
sudo apt-get install unzip
```

- 解压 nginx 和 nginx-rtmp安装包

```
tar -zxvf nginx-1.7.5.tar.gz
-zxvf分别是四个参数
x : 从 tar 包中把文件提取出来
z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压
v : 显示详细信息
f xxx.tar.gz :  指定被处理的文件是 xxx.tar.gz

unzip master.zip

切换到 nginx-目录
cd nginx-1.7.5
```


- 添加 nginx-rtmp 模板编译到 nginx
```
./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master
```

- 编译安装 
```
make
sudo make install
```

- 安装nginx init脚本
```
sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
#开启启动
sudo update-rc.d nginx defaults
```

- 启动和停止nginx 服务，生成配置文件
```
sudo service nginx start
sudo service nginx stop
```

- 安装 FFmpeg
```
下载源码
 ./configure --disable-yasm
make
make install
```

- 配置 nginx-rtmp 服务器

```
打开 /usr/local/nginx/conf/nginx.conf，在末尾添加如下配置

rtmp{
    server {
        listen 1935;
        chunk_size 4096;

        application live {
                    live on;
                    record off;
                    exec ffmpeg -i rtmp://localhost/live/$name -threads 1 -c:v libx264 -profile:v baseline -b:v 350K -s 640x360 -f flv -c:a aac -ac 1 -strict -2 -b:a 56k rtmp://localhost/live360p/$name;
                    }
        application live360p {
                    live on;
                    record off;
                    }
        }
    }
}

```

- 保存上面配置文件，然后重新启动nginx服务

```
sudo service nginx restart
```

- 如果你使用了防火墙，请允许端口 tcp 1935

- 使用客户端，使用rtmp协议进行视频实时采集

```
推流客户端：推送到rtmp://39.108.56.76:1935/live/test
播放客户端：使用ffplay播放视频流：ffpaly rtmp://39.108.56.76:1935/live/test
```
