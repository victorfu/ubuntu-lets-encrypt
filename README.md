
# 1. Download certbot (https://certbot.eff.org)
```
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto
```

# 2. Install nginx
```
$ sudo apt-get install nginx
```

# 3. Edit nginx configuration
```
$ sudo vim /etc/nginx/sites-available/default
```

In `server{}`,
```
location ~ /.well-known {
  allow all;
}
```

# 4. Restart nginx
```
$ sudo service nginx reload
```

# 5. Install Let's Encrypt with webroot (need to enter domain name)
```
$ ./certbot-auto certonly --webroot -w /usr/share/nginx/html
```

# 6. Generate Diffie-Hellman (DH) key
```
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

# 7. Edit nginx configuration again
```
$ sudo vim /etc/nginx/sites-available/default
```
```
server {
# listen 80 default_server; 
# listen [::]:80 default_server ipv6only=on;

# root /usr/share/nginx/html; 
# index index.html index.htm;

listen 443 ssl;
server_name your.domain.com;

# Configure Let's Encrypt certification
ssl_certificate /etc/letsencrypt/live/your.domain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/your.domain.com/privkey.pem;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;

# Use generated dhparam.pem
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
ssl_session_timeout 1d; 
ssl_session_cache shared:SSL:50m;
ssl_stapling on;
ssl_stapling_verify on;
add_header Strict-Transport-Security max-age=15768000;

# Redirect HTTPS to 8080 (assuming the web server is hosting at port 8080)
location / {
proxy_pass http://localhost:8080;
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

}
# Redirect HTTP(80 port) to TTPS
server {
listen 80;
server_name your.domain.com;
return 301 https://$host$request_uri;

}
```

# 8. Reload nginx
```
$ sudo service nginx reload
```

