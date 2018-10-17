### Nginx

Во-первых, следует подготовить главный файл настроек:

```
#
# /etc/nginx/nginx.conf
#
worker_processes auto;
pcre_jit on;
error_log /var/log/nginx/error_log warn;

events {
  worker_connections 768;
  use epoll;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main
    '$remote_addr - $remote_user [$time_local] '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$gzip_ratio"';

  client_header_timeout 2m;
  client_body_timeout 5m;
  send_timeout 3m;
  client_max_body_size 20m;

  client_header_buffer_size 1k;
  large_client_header_buffers 4 4k;

  gzip on;
  gzip_min_length 1100;
  gzip_buffers 4 8k;
  gzip_types text/plain;

  output_buffers 4 256k;
  postpone_output 1460;

  sendfile on;
  aio off;
  tcp_nopush on;
  tcp_nodelay on;

  keepalive_timeout 75 40;
  server_tokens off;
  include /etc/nginx/sites-enabled/*;
}
```

Во-вторых, рекомендуется создать 3 виртуальных хоста: непосредственно для самой АСР, для Kibana'ы и для Flower.

```
#
# /etc/nginx/sites-available/<some.domain.name>
#
upstream uwsgi_pp {
  server uwsgi:8001 fail_timeout=20s;
}

server {
  listen 80;
  server_name <some.domain.name>;

  access_log /var/log/nginx/<some.domain.name>.no_ssl.access_log main;
  error_log /var/log/nginx/<some.domain.name>.no_ssl.error_log info;

  client_max_body_size 48m;

  location / {
    rewrite ^/favicon.ico$ /static/favicon.ico permanent;
    try_files $uri @proxy;
  }

  location /api/key/customer/new/ {
    uwsgi_pass uwsgi_pp;
    include /etc/nginx/uwsgi_params;
    uwsgi_read_timeout 300;
  }

  location @proxy {
    uwsgi_pass uwsgi_pp;
    include /etc/nginx/uwsgi_params;
    uwsgi_read_timeout 600;
    proxy_intercept_errors on;
  }

  location /static {
    alias /static;
    aio on;
    directio 512;
    output_buffers 8 256k;

    sendfile on;
    sendfile_max_chunk 1m;
    tcp_nopush on;
    tcp_nodelay on;

    expires 30d;
    access_log off;
  }
}
```

```
#
# /etc/nginx/sites-available/kibana.<domain.name>
#
upstream kibana_pp {
  server kibana:5601 fail_timeout=20s;
}

server {
  listen 80;
  server_name kibana.<domain.name>;

  access_log /var/log/nginx/kibana.<domain.name>.access_log main;
  error_log /var/log/nginx/kibana.<domain.name>.error_log info;

  location / {
    proxy_pass http://kibana_pp;
  }
}
```

```
#
# /etc/nginx/sites-available/flower.<domain.name>
#
upstream flower_pp {
  server flower:5555 fail_timeout=20s;
}

server {
  listen 80;
  server_name flower.<domain.name>;

  access_log /var/log/nginx/flower.<domain.name>.access_log main;
  error_log /var/log/nginx/flower.<domain.name>.error_log info;

  location / {
    proxy_pass http://flower_pp;
  }
}
```

В завершение необходимо создать ссылки на файлы виртуальных хостов командами:

    $ cd /etc/nginx
    $ sudo /bin/bash -c 'for file in `ls sites-available`; do /bin/ln -s "../sites-available/$file" sites-enabled/$file; done'



