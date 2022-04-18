server {
    listen 80;
    listen [::]:80;

    server_name example.com;

    gzip on;
    gzip_types
      application/javascript
      application/x-javascript
      application/json
      application/rss+xml
      application/xml
      image/svg+xml
      image/x-icon
      application/vnd.ms-fontobject
      application/font-sfnt
      text/css
      text/plain;
    gzip_min_length 256;
    gzip_comp_level 5;
    gzip_http_version 1.1;
    gzip_vary on;

    location ~ ^/.well-known/(webfinger|nodeinfo|host-meta) {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect off;
    }

    location ~ ^/(css|img|js|fonts)/ {
        root /var/www/example.com/static;
        # Optionally cache these files in the browser:
        # expires 12M;
    }

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect off;
    }
}

description "WriteFreely Instance"
author "Write.as "

start on runlevel [2345]
stop on runlevel [016]
respawn

chdir /var/www/example.com
exec /var/www/example.com/writefreely >> /var/log/writefreely.log 2>&1

sudo start writefreely

tail -F /var/log/writefreely.log

/etc/systemd/system/writefreely.service:
[Unit]
Description=WriteFreely Instance
After=syslog.target network.target
# If MySQL is running on the same machine, uncomment the following 
# line to use it, instead. 
#After=syslog.target network.target mysql.service

[Service]
Type=simple
StandardOutput=syslog
StandardError=syslog
WorkingDirectory=/var/www/example.com
ExecStart=/var/www/example.com/writefreely
Restart=always

[Install]
WantedBy=multi-user.target
sudo systemctl start writefreely
sudo journalctl -f -u writefreely
