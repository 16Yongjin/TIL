
# `code-server` 설정

### 설치

```
curl -fsSL https://code-server.dev/install.sh | sh -s -- --dry-run

curl -fsSL https://code-server.dev/install.sh | sh
```

### 자동시작을 위해 서비스 파일 생성

```
sudo vim /lib/systemd/system/code-server.service
```

```
[Unit]
Description=code-server
After=nginx.service

[Service]
Type=simple
Environment=PASSWORD=your_password
ExecStart=/usr/bin/code-server --bind-addr 127.0.0.1:8080 --user-data-dir /var/lib/code-server --auth password
Restart=always

[Install]
WantedBy=multi-user.target
```

### 서비스 시작

```
sudo systemctl start code-server
sudo systemctl status code-server
```

### 자동시작 설정

```
sudo systemctl enable code-server
```

### nginx 설정

```
server {
    listen 80;
    listen [::]:80;

    server_name code-server.your-domain;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```

업그레이드 헤드 설정을 하지 않으면 웹소켓 에러가 발생한다.
