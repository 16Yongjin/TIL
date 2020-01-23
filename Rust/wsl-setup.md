# Windows Subsystem for Linux에서 러스트 개발환경 구성하기

## 설치

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## 환경변수 설정

아래 코드를 `.bash_profile`이나 `.zshrc`에 추가

```
export PATH="$HOME/.cargo/bin:$PATH"
```

## 프로젝트 생성

```
cargo new guessing_game --bin
```

## 에디터 열기

```
code guessing_game
```

## 프로젝트 실행

```
cargo run
```
