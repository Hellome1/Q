# linux 安装nginx

### 选定目录
选择 /usr/local
新建 nginx 目录
nginx 目录下新建 install 目录
进入该目录

### 解压缩
```shell
tar -zxvf nginx-1.22.1.tar.gz
tar -zxvf openssl-1.1.1v.tar.gz
tar -zxvf pcre-8.41.tar.gz
tar -zxvf zlib.tar.gz
```

### 安装 openssl
```shell
cd openssl-1.1.1v/
./config
make && make install
```

### 配置 nginx 并安装
```shell
cd ..
cd nginx-1.22.1/
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.3.1 --with-openssl=../openssl-1.1.1v
make && make install
```

### 查看 nginx 能否使用
```shell
cd /usr/local/nginx/sbin
./nginx -v
```

### 防火墙操作
```shell
firewall-cmd --zone=public --add-port=8101/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-ports
```

## 特殊的配置 https
将本目录下的 https 里面的证书拷贝到 nginx/conf 目录下
```nginx.conf
server {
  listen       8101 ssl;
  server_name  localhost;

  ssl_certificate      cert.pem;
  ssl_certificate_key  cert.key;

  ssl_session_cache    shared:SSL:1m;
  ssl_session_timeout  5m;

  ssl_ciphers  HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers  on;

  #charset koi8-r;

  #access_log  logs/host.access.log  main;

  location / {
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host:$server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,userid,tpye';
    proxy_pass https://10.8.1.200:1443;
  }
}
```