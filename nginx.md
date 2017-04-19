upstream unicorn_freeditorial_production {
  server unix:/tmp/unicorn.freeditorial_production.sock;
}

# redirect all to https
server {
    server_name www.freeditorial.com freeditorial.com;
    return 301 https://freeditorial.com$request_uri;
}

#redirect www.xxx.com to xxx.com, over https
server {
        server_name www.freeditorial.com;
        listen 443;
        ssl on;
        ssl_certificate /etc/ssl/crt.crt;
        ssl_certificate_key /etc/ssl/key.key;
        return 301 https://freeditorial.com$request_uri;
}

server {
        server_name freeditorial.com;
        listen 443;

        root /home/web/apps/freeditorial/current/public;

        ssl on;
        ssl_certificate /etc/ssl/crt.crt;
        ssl_certificate_key /etc/ssl/key.key;

        gzip on;
        gzip_disable "msie6";

        access_log /var/log/nginx/www_freeditorial_com.access.log;
        error_log /var/log/nginx/www_freeditorial_com.error.log;

        client_max_body_size 4G;

        location / {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header    X-Forwarded-Proto https;
                proxy_redirect off;
                proxy_pass http://unicorn_freeditorial_production;
        }

        location ~* \.(?:ico|svg|pdf|flv|jpg|jpeg|png|gif|swf|xml|txt|html|js)$ {
                expires 30d;
                add_header Pragma public;
                add_header Cache-Control "public";
        }

}
