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
