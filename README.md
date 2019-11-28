# Ubuntu 18.04 에서 AspNetCore 3.0 설치 및 https 웹앱 배포

## 무료 도메인 만들기

```html
https://www.freenom.com/
```

## Nginx 설치

```ssh
sudo apt update
sudo apt install nginx
```

## Letsencrypt 설치

```ssh
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx

sudo certbot --nginx -d yourdomain.com
```

## Nginx 설정

```ssh
// sudo vi /etc/nginx/sites-available/yourdomain.com.conf
// sudo ln -s /etc/nginx/sites-available/yourdomain.com.conf /etc/ nginx/sites-enabled/yourdomain.com.conf
```

```ssh
sudo vi /etc/nginx/sites-enabled/yourdomain.com.conf

server {
        server_name yourdomain.com;

        location / {
                proxy_pass https://localhost:5001;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection keep-alive;
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        if ($scheme != "https") {
                return 301 https://$host$request_uri;
        } # managed by Certbot
}
```

## AspNetCore 3.0 sdk 설치

```ssh
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.0
```

## 웹앱 배포 및 게시

### form github

```ssh
sudo git clone https://github.com/songgolmae/Yourapp.git
cd yourapp
sudo dotnet publish -c release -o /var/www/yourapp
```

### form dotnet cmd

```ssh
sudo dotnet new webapp -o yourapp
cd yourapp
sudo dotnet publish -c release -o /var/www/yourapp
```

## 서비스 구성

```ssh
sudo vi /etc/systemd/system/yourapp.service
```

```ssh
[Unit]
Description=ASP.NET Core 3.0 yourapp

[Service]
WorkingDirectory=/home/ubuntu/yourapp
ExecStart=/usr/bin/dotnet /home/ubuntu/yourapp/yourapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-yourapp
User=ubuntu
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

```ssh
sudo systemctl enable yourapp.service
sudo systemctl start yourapp.service
sudo systemctl status yourapp.service
```

### 서비스 설정 변경시

```ssh
sudo systemctl daemon-reload
sudo systemctl restart yourapp.service
```
