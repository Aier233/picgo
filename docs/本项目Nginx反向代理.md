# 本项目Nginx的反向代理与负载均衡

利用nginx反向代理可以实现客户端对代理的无感知性，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 `IP`地址。这样我们可以用我们所有的多台服务器来对客户的请求进行相应，而且在某一台服务器挂了之后也可以马上用另一台进行热后备，真正实现高并发、高可用。

参考链接：

https://zhuanlan.zhihu.com/p/451825018

https://www.runoob.com/w3cnote/nginx-proxy-balancing.html

## Nginx代理服务的配置

### nginx.conf文件的内容

nginx.conf 配置文件可分为三个部分：*（以下文件内容均来自ubuntu apt命令安装的nginx）*

1. **全局块**

   从配置文件开始到 events 块之间的内容，主要会设置一些影响 Nginx 服务器整体运行的配置指令，主要包括：配置运行 Nginx 服务器的用户（组）、允许生成的 worker_process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

   ```
   user www-data;
   worker_processes auto;
   pid /run/nginx.pid;
   include /etc/nginx/modules-enabled/*.conf;
   ```

2. **events 块**

   events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括：是否开启对多 work_process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 work_process 可以同时支持的最大连接数等。

   以下例子就表示每个 work_process 支持的最大连接数为 768。这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

   ```
   events {
   	worker_connections 768;
   	# multi_accept on;
   }
   ```

3. **http 块**（重要）

   此模块及其重要且复杂，代表了Nginx中用作Web服务器、代理、缓存、日志等绝大多数功能的配置。以下只介绍在本项目中使用到的Nginx的反向代理与负载均衡功能。

   ~~~
   http {
   
   	##
   	# Basic Settings
   	##
   
   	sendfile on;
   	tcp_nopush on;
   	tcp_nodelay on;
   	keepalive_timeout 65;
   	types_hash_max_size 2048;
   	# server_tokens off;
   
   	# server_names_hash_bucket_size 64;
   	# server_name_in_redirect off;
   
   	include /etc/nginx/mime.types;
   	default_type application/octet-stream;
   
   	##
   	# SSL Settings
   	##
   
   	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
   	ssl_prefer_server_ciphers on;
   
   	##
   	# Logging Settings
   	##
   
   	access_log /var/log/nginx/access.log;
   	error_log /var/log/nginx/error.log;
   
   	##
   	# Gzip Settings
   	##
   
   	gzip on;
   
   	# gzip_vary on;
   	# gzip_proxied any;
   	# gzip_comp_level 6;
   	# gzip_buffers 16 8k;
   	# gzip_http_version 1.1;
   	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
   
   	##
   	# Virtual Host Configs
   	##
   
   	include /etc/nginx/conf.d/*.conf;
   	include /etc/nginx/sites-enabled/*;#可以看到在此处引入server块
   }
   
   ~~~

   *nginx.conf的配置文件所有内容均在此处*

   文件末尾有一行 `include /etc/nginx/sites-enabled/*;`

   由此可知在/etc/nginx/sites-enabled/目录下所有文件均会被引入

   于是我们可以在此目录下编写代理配置文件

   ## Nginx代理配置

   ```
   # greenfarm
   # 文件名是不重要的，重要的是文件的位置
   # 文件编码问题显示中文有误，因此没有中文注释
   upstream mysers {
   
           # One of our remote server:
   
           server 180.76.233.143:8086;
   
   
           # Since localhost 8086 should be the proxy,
           # we choose 8089 to run our backend project
   
           server 1.12.249.224:8089;
   
   
           # We choose the ip_hash as our Load balancing strategy
           # So that we can keep our session
   
           ip_hash;
   }
   
   server {
   
           # Whoever ask for 1.12.249.224:8086,nginx proxy it
   
           listen 8086;
           server_name 1.12.249.224;
   
           location / {
   
                   # Add our proxy strategy
                   proxy_pass http://mysers;
   
                   # Add some identification to the NetWork ResposeHeader
                   # So that we can see the ip on Edge using F12~
                   add_header backendIP $upstream_addr;
                   add_header backendCode $upstream_status;
           }
   }
   ```

   以上文件说明几点：

   1. 我们的后端运行在 1.12.249.224:8089 以及 180.76.233.143:8086上
   2. 1.12.249.224作为代理服务器，8086为它的监听端口
   3. Nginx采用ip_hash的负载均衡策略，它会将每个ip来的请求哈希到某一台特定的服务器上。这么做是为了保持每个设备对后端资源访问的会话。
   4. 每当有请求访问1.12.249.224:8086，Nginx会首先由它的ip地址根据哈希算法归类到某一台服务器上去，然后将此请求代理至那台服务器
   5. 若两台服务器中有一台临时挂了。另一台可以马上充当它的热后备

   配置好文件后，使用``sudo nginx -s reload``让Nginx把这些配置文件热部署即可
