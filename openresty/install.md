# 1、安装Openresty
```bash
# root 执行安装依赖包
dnf install -y gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel

# 切换用户编译安装
su - gateway
wget https://openresty.org/download/openresty-1.25.3.2.tar.gz
ls /usr/local/gateway/
tar -xf openresty-1.25.3.2.tar.gz 
cd openresty-1.25.3.2/
./configure --prefix=/usr/local/gateway/openresty                 --with-luajit                 --without-http_redis2_module                 --with-http_iconv_module
# --with-luajit 开启 Lua JIT 支持
# --without-http_redis2_module 这个选项用于禁用 http_redis2_module 模块
# --with-http_iconv_module 模块用于提供字符编码转换功能

gmake && gmake install
echo $?
wget https://github.com/doujiang24/lua-resty-kafka/archive/master.zip
unzip master.zip
cp -rf lua-resty-kafka-master/lib/resty/kafka /usr/local/gateway/openresty/lualib/resty/

# systemd root 用户执行
[ec2-user@ip-172-31-10-10 nginx]$ vim /usr/lib/systemd/system/openresty.service 
[Unit]
Description=OpenResty Service
After=network.target

[Service]
Type=forking
User=gateway
Group=gateway
AmbientCapabilities=CAP_NET_BIND_SERVICE
PIDFile=/usr/local/gateway/openresty/nginx/logs/nginx.pid
ExecStartPre=/usr/local/gateway/openresty/nginx/sbin/nginx -t
ExecStart=/usr/local/gateway/openresty/nginx/sbin/nginx
Restart=on-failure

[Install]
WantedBy=multi-user.target


# nginx 配置修改
user gateway; 
# 其余配置文件不变



systemctl daemon-reload && systemctl enable openresty.service 
systemctl start openresty.service
systemctl status openresty.service
# 启动会失败： 在大多数 Linux 系统中，使用 1024 以下的端口需要超级用户（root）权限。为了让一个非超级管理员用户（如 gateway 用户）能够在 1024 以下的端口上监听
# AmbientCapabilities=CAP_NET_BIND_SERVICE
```
#### 2、验证
http://192.168.1.1