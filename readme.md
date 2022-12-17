# NGINX

Category: Youtube
Created time: December 15, 2022 11:55 PM
Source: FullCycle
Status: Done
Tags: Nginx
Type: Page

[Nginx: do básico ao Load Balancer](https://youtu.be/pPlcC5hDMCs)

[Advanced Load Balancer, Web Server, & Reverse Proxy - NGINX](https://www.nginx.com/)

![Untitled](.github/Untitled.png)

Open source software that has several uses such as: Webserver, reverse proxy, caching, load balancer, media streaming, as well as proxy for email services.

### Main features

- One of the most powerful web servers on the market
- Performs better than Apache
- Can support high volume of simultaneous connections
- Uses low memory and supports high concurrency
- Usually used as a webserver and reverse proxy
- More than 50% of the most accessed websites in the world use Nginx
- Supports the most modern web resources such as Websockets, HITP 2, video streaming, etc.

### How it works

![Untitled](.github/Untitled%201.png)

![Untitled](.github/Untitled%202.png)

### Reverse proxy

Reverse proxy is an intermediary service that receives requests from different *clients*, processes them and forwards them to the corresponding service.

![Untitled](.github/Untitled%203.png)

### Load Balancing

![Untitled](.github/Untitled%204.png)

# 📝 Goals Commits

[https://github.com/adlerweb3/nginx-load_balancer](https://github.com/adlerweb3/nginx-load_balancer)

⚽ [GOAL] NGINX | Reverse Proxy: 1st container DONE

⚽ [GOAL] NGINX | Node1: 2nd container DONE

⚽ [GOAL] NGINX | Node2: 3rd container and NGINX Load Balancer DONE

⚽ [GOAL] NGINX | Enable Logs DONE

# NGINX | Reverse Proxy

### Docker | 📂 docker-compose.yml

```yaml
version: '3'

services:

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8000:80" #Libera a porta 8000 do localhost e direciona para porta 80 do nginx
```

### CLI

```bash
# run nginx containers
docker compose up -d

# install bash and vim inside nginx container
docker compose exec nginx apk add bash vim

# Enter to nginx container bash
docker compose exec nginx bash
```

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

### NGINX base configuration file `/etc/nginx/nginx.conf`

Focused for system use/tunning.

```bash
# open nginx.conf file inside nginx container
vim /etc/nginx/nginx.conf
```

```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

The settings of the site(s) must be placed in separate files through *includes*.

`include /etc/nginx/conf.d/*.conf`

Initial installation comes with a *default* file:

```bash
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

> ******In this case, it is redirecting to the Welcome page of NGINX******
> 

![Untitled](.github/Untitled%205.png)

Edited configuration file for testing:

```bash
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

![Untitled](.github/Untitled%206.png)

![Untitled](.github/Untitled%207.png)

First NGINX Container Configured

# NGINX | Node1

### Docker | 📂 docker-compose.yml

```yaml
version: '3'

services:

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8000:80" #Libera a porta 8000 do localhost e direciona para porta 80 do nginx

  node1:
    image: nginx:alpine
    container_name: node1
    ports:
      - "80" #Libera apenas a porta 80 do nginx
```

### CLI

```bash
# run nginx containers
docker compose up -d

# install bash and vim inside nginx container
docker compose exec node1 apk add bash vim

# Enter to nginx container bash
docker compose exec node1 bash
```

Edit to identify Node1 correctly:

```bash
vim /usr/share/nginx/html/index.html
```

```html
<h1> NODE 1 </h1>
```

### ByPassing 1st NGINX to 2nd

```bash
server {
    listen       80;
    server_name  localhost;

    location / {
        # root   /usr/share/nginx/html;
        # index  index.html index.htm;

        # byPassing to another container || nginx || service ...
        proxy_pass http://node1;
    }
}
```

NGINX - default.conf

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

![Untitled](.github/Untitled%208.png)

# NGINX | Node2

### Docker | 📂 docker-compose.yml

```yaml
version: '3'

services:

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8000:80" # Release localhost port 8000 and direct to nginx port 80

  node1:
    image: nginx:alpine
    container_name: node1
    ports:
      - "80" # Release only nginx port 80

  node2:
    image: nginx:alpine
    container_name: node2
    ports:
      - "80" # Release only nginx port 80
```

Edit to identify Node2 correctly:

```bash
vim /usr/share/nginx/html/index.html
```

```html
<h1> NODE 2 </h1>
```

# NGINX | Load Balancer

```bash
upstream nodes {
    server node1;
    server node2;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://nodes;
    }
}
```

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

# Logs

All access to NGINX create a log

```bash
upstream nodes {
    server node1;
    server node2;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://nodes;
    }

    access_log /var/log/nginx/nginx-access.log;
}
```

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

```bash
path: /var/log/nginx/access.log
```

![Untitled](.github/Untitled%209.png)

### Tunning ON Logs

Node1 

```bash
nano /etc/nginx/conf.d/default.conf
```

![Untitled](.github/Untitled%2010.png)

```bash
access_log  /var/log/nginx/nginx-access.log main;
```

```bash
tail —f /var/log/nginx/nginx—access.log
```

## IP Configs

Passing the access external IPs and not the NGINX PROXY IP

The IPs logged into the NODES are those of the NGINX PROXY

![Untitled](.github/Untitled%2011.png)

![Untitled](.github/Untitled%2012.png)

### NGINX | Load Balancer

```bash
upstream nodes {
    server node1;
    server node2;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://nodes;
        proxy_set_header X—Real—IP $remote_addr;
    }

    access_log /var/log/nginx/nginx-access.log;
}
```

<aside>
⚠️ Use CLI `nginx -t` to confirm it’s all ok.

</aside>

<aside>
⚠️ Use CLI `nginx -s reload` to restart nginx with updates.

</aside>

### Node1 && Node2

📂 `nano /etc/nginx/nginx.conf`

```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    log_format  main  '$http_x_real_ip - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```