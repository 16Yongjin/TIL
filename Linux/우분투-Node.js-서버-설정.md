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

## SCP로 `id_rsa` 옮기기

```
scp id_rsa username@127.0.0.1:/home/username
```
