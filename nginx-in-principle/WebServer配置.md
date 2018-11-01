## nginx Web Server配置

Nginx可以作为一个web server，提供http服务。

Nginx提供静态网站服务： <br>
nginx.conf(最基本配置)

```

http{
    server {
        listen       80;   ## 监听的端口
        server_name  localhost; ## 虚拟服务器名称

        location / {   ##请求匹配
            root   /var/nginx_data/www;
            index  index.html index.htm;
        }

        location /images/ {
           root /var/nginx_data;
        }
    }
}
```

以上配置可以实现访问localhost/ 访问到/var/nginx_data/www/index.html;访问localhost/images/xxx.jpg可以访到/var/nginx_data/images/xxx.jpg

各项详解 TODO：
http :
server :
location配置：