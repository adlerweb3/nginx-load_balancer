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

Software open source que possui diversas utilizações como: Webserver, proxy reverso, caching, load balancer, media streaming, além de proxy para serviços de email.

### Principais características

- Um dos servidores web mais potentes do mercado
- Performa mais que o Apache
- Consegue suportar alto volume de conexões simultâneas
- Utiliza baixa memória e suporta alta concorrência
- Normalmente é mais utilizado como webserver e proxy reverso
- Mais de 50% dos sites mais acessados do mundo utilizam Nginx
- Possui suporte aos recursos web mais modernos como Websockets, HITP 2, streaming de vídeos, etc.

### Como funciona

![Untitled](.github/Untitled%201.png)

![Untitled](.github/Untitled%202.png)

### Proxy reverso

Proxy reverso é um serviço intermediário que recebe as requisições de diversos *clients*, as processa e as encaminha o serviço correspondente.

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

### Arquivo de configuração base do NGINX `/etc/nginx/nginx.conf`

Focado para uso do sistema/tunning.

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

As configurações do(s) site(s) deverão ser colocadas em arquivos separados atreves de *includes*.

`include /etc/nginx/conf.d/*.conf`

Instalação inicial vem com um arquivo *default*:

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

> ******Neste caso, está direcionando para a página de Welcome do NGINX******
> 

![Untitled](.github/Untitled%205.png)

Arquivo de configuração editado para testes:

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

Primeiro NGINX Container configurado

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
      - "8000:80" #Libera a porta 8000 do localhost e direciona para porta 80 do nginx

  node1:
    image: nginx:alpine
    container_name: node1
    ports:
      - "80" #Libera apenas a porta 80 do nginx

  node2:
    image: nginx:alpine
    container_name: node2
    ports:
      - "80" #Libera apenas a porta 80 do nginx
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

Passando os IPs externos de acesso e não o IP do NGINX PROXY

Os IPs logados nos NODES são os do NGINX PROXY

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