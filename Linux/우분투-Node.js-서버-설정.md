# 우분투 Node.js 서버 설정

## 유저 추가

```
sudo adduser username
```

## 유저 변경

```
su username
```

## zsh 설치

```
sudo apt-get install zsh
```

## oh my zsh 설치

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" //
```

## 기본 쉘을 zsh로

```
chsh -s `which zsh`
```

## zsh 테마

`.zshrc`에 아래 코드 추가

```
ZSH_THEME=agnoster
```

## username 감추기

```
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
   # prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```

## node.js 설치

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt-get install nodejs
```

## postgres 설정

설치

```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib -y
```

생성된 postgres 계정 접속 후, DB 생성과 DB 이름과 동일한 계정 생성

```
sudo -i -u postgres
createdb alarm
sudo adduser alarm
```

## postgres 비번 변경

```
sudo -u postgres psql

ALTER USER postgres PASSWORD 'postgres';

```

## SCP로 `id_rsa.pub` 옮기기

```
scp id_rsa.pub username@127.0.0.1:/home/username/.ssh
```

## `id_rsa.pub`의 내용을 `authorized_keys`로 옮기기

```
cat id_rsa.pub >> authorized_keys
```

## nginx 설치

```
sudo apt-get install nginx

```

## nginx 설정

```
vim /etc/nginx/sites-enabled/default

~~

server {
       listen 80;
       listen [::]:80;

       server_name deploy.lecture.hufs.app;

       location / {
                proxy_pass http://127.0.0.1:3001;
       }
}
```

## certbot 설정

[cerbot 설정](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update


sudo apt-get install certbot python-certbot-nginx

sudo certbot --nginx
```

## unable to resolve host 에러 시

`/etc/hostname`으로 장치이름 알아내기

`/etc/hosts` 파일 아래와 같이 수정하기

```
 127.0.0.1    localhost.localdomain localhost
 127.0.1.1    my-machine
```

## ssh 터널링

ssh -L [로컬에서 사용할 포트]:[최종적으로 접근할 곳][ssh server 주소]

ex:

```
ssh -L 8585:127.0.0.1:80 192.168.1.201
```

## redis 설치

```
sudo apt-get install redis-server

sudo systemctl enable redis-server.service
```

## code-server 설정

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
