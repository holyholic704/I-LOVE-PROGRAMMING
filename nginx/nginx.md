# 简介

Nginx 是一个 轻量级、高性能的反向代理 Web 服务器、反向代理服务器及电子邮件（IMAP/POP3/SMTP）代理服务器

### 优点

- 跨平台、配置简单、扩展性强

- 支持热部署，启动简单

- 成本低廉，且开源

- 响应速度快

- 非阻塞、高并发连接：能处理高达 5 万个并发连接数
  
  - 采用了多进程和 I/O 多路复用（epoll）的底层实现

- 内存消耗小：开启 10 个 Nginx 才占 150M 内存

- 稳定性高，宕机的概率非常小：采用的是多进程模式运行，有一个 master 主进程和多个 worker 进程

- 内置的健康检查功能：如果有一个服务器宕机，会做一个健康检查，再发送的请求就不会发送到宕机的服务器了，重新将请求提交到其他的节点上

### 反向代理

- 正向代理（forward proxy）：客户端把请求发给代理服务器，代理服务器把请求转发给外部服务器，外部服务器处理完成后通过代理服务器将结果返回给客户端
  
  - 提高访问速度：通常代理服务器都设置一个较大的缓冲区，当有外界的信息通过时，同时也将其保存到缓冲区中，当其他用户再访问相同的信息时， 则直接由缓冲区中取出信息，传给用户，以提高访问速度
  
  - 控制对内部资源的访问
  
  - 过滤内容
  
  - 隐藏真实 IP
  
  - 突破网站访问限制

- 反向代理（reverse proxy）：代理服务器接受请求，将请求转发给内部服务器，内部服务器处理完成后通过代理服务器将结果返回给客户端
  
  - 隐藏服务器真实 IP
  
  - 负载均衡
  
  - 提高访问速度
    
    - 对于静态内容及短时间内有大量访问请求的动态内容提供缓存服务
    
    - 对一些内容进行压缩，以节约带宽或为网络带宽不佳的网络提供服务
  
  - 提供安全保障
    
    - 作为应用层防火墙，为网站提供对基于Web的攻击行为（例如DoS/DDoS）的防护，更容易排查恶意软体等
    
    - 为后端服务器统一提供加密和 SSL 加速（如SSL终端代理）
    
    - 提供HTTP访问认证

![](.\md.assets\640.png)

![](.\md.assets\640 (1).png)

#### 正向代理与反向代理的区别

1. 正向代理是客户端的代理，反向代理是服务器的代理

2. 正向代理一般由客户端架设，反向代理一般由服务器架设

3. 正向代理中，服务器不知道真正的客户端；反向代理中，客户端不知道真正的服务器

4. 正向代理主要为了解决访问限制，反向代理主要为了负载均衡和安全防护

# 安装使用

```shell
# 下载
wget http://nginx.org/download/nginx-1.18.0.tar.gz

# 解压
tar -zxvf nginx-1.18.0.tar.gz

# 执行编译前的环境配置（默认安装路径为：/usr/local/nginx）
./configure
# 编译及安装
make && make install

# 依赖（如果需要）
#   gcc编译工具
#   pcre库
#   zlib库
#   openssl库
yum install -y gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel

# 启动，进入/usr/local/nginx/sbin
./nginx
```

- 执行成功后，默认会产生两个服务进程，一个 master 进程，一个 worker 进程

- 默认端口号为 80

### 卸载

```shell
# 关闭进程
./nginx -s stop

# 删除文件夹
rm -rf /usr/local/nginx

# 清除编译环境，进入到源代码解压后的目录
make clean
```

### 常用命令

```shell
# 启动
./nginx

# 查看版本
./nginx -V

# 立即关闭服务
./nginx -s stop

# 执行完当前的任务后再关闭服务
./nginx -s quit

# 重新加载服务
./nginx -s reload

# 检查配置文件
./nginx -t
```

# 配置文件

### 全局块

```nginx
# 指定可以运行nginx服务的用户和用户组
# 在Windows上不生效
user  [user] [group];
# user	nobody;

# 指定工作线程数
# auto会创建与核心数相同的数量的工作线程
worker_processes number | auto;
# worker_processes	1;

# 指定错误日志的路径和日志级别
# debug级别的日志需要编译时使用--with-debug开启debug开关
error_log  [path] [debug | info | notice | warn | error | crit | alert | emerg];
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 指定pid文件存放的路径
pid        logs/nginx.pid;
```

### events 块

```nginx
# 设置允许每个工作线程同时开启的最大连接数，当每个工作进程接受的连接数超过这个值时将不再接收连接
# 当所有的工作进程都接收满时，连接进入logback，logback满后连接被拒绝
# 值不能超过超过系统支持打开的最大文件数，也不能超过单个进程支持打开的最大文件数
worker_connections	1024;

# 指定使用哪种网络IO模型
# 可以不设置，nginx会根据操作系统选择合适的模型
use	[select | poll | kqueue | epoll | rtsig | /dev/poll | eventport]

# 客户端请求头部的缓冲区大小
client_header_buffer_size  4k;

# 防止惊群现象：当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接，其他进程获取控制权只能重新进入睡眠状态，造成性能浪费
# 默认开启，即对多个进程接收连接进行序列化，防止多个进程对连接的争抢
accept_mutex on;

# 是否允许每个工作进程同时接受多个网络连接
# 默认关闭，即一个工作进程只能同时接受一个新的连接
multi_accept off;
```









