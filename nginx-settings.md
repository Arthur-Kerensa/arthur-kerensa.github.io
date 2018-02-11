server {

        root /var/www/html;
        
        index index.html index.htm index.nginx-debian.html;

        server_name wiki.org.uk www.wiki.org.uk;
        
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/wiki.org.uk-0001/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/wiki.org.uk-0001/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_next_upstream error timeout http_502 http_503 http_504;
        try_files $uri $uri/ =404;
        }

        location /hello {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        }
}

server {
    if ($host = www.wiki.org.uk) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = wiki.org.uk) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

        listen 80 default_server;
        listen [::]:80 default_server;

        server_name wiki.org.uk www.wiki.org.uk;
    return 404; # managed by Certbot
}
