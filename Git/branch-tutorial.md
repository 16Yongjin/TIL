# 깃 브랜치 사용법

## 새 브랜치 생성하기

```
git branch testing
```

`git branch` 명령은 브랜치를 만들기만 하고 브랜치를 옮기지 않는다.

`git log --oneline --decorate` 명령어로 브랜치가 어떤 커밋을 가리키는지 확인할 수 있다.

보면 아직 `HEAD`가 `master`를 가리키고 있다.

## 브랜치 이동하기

```
git checkout testing
```

이렇게 하면 `HEAD`는 `testing` 브랜치를 가리킨다.

이 상태에서 수정 후 커밋한다.

다시 `git checkout master`를 실행하면 `master` 브랜치의 커밋을 `HEAD`도 가리키게 하고 파일들도 그 시점으로 되돌려 놓는다.

(엄청 빨라서 놀랐다..)

## 브랜치 생성과 체크아웃을 동시에

`git checkout` 명령에 `-b`라는 옵션을 추가한다.

```
git checkout -b iss53
```

## Fast Forward Merge 방식

A 브랜치와 B 브랜치 Merge 시

B 브랜치가 A 브랜치 이후의 커밋을 가리키고 있으면 (Upstream 브랜치라면)

A 브랜치가 B 브랜치와 동일한 커밋을 가리키도록 이동시킨다.

## 브랜치 삭제

필요없는 브랜치는 아래 명령어로 삭제한다.

```
git branch -d testing
```

## Recursive Merge 방식

서로 다른 방향으로 뻣은 커밋을 Merge할 때는

각 브랜치가 가리키는 커밋 2개와 공통 조상 1개를 사용하는 3-way Merge를 한다.

그냥 브랜치 포인터만 최신 커밋으로 옮기는 게 아니라

3-way Merge의 결과를 별도의 커밋으로 만들고 Merge 하는 브랜치가 그 커밋을 가리키도록 이동시킨다.

## Merge하지 않는 브랜치 삭제하기

Merge하지 않은 커밋을 담고 있는 브랜치는 `git branch -d` 명령으로 삭제되지 않는다.

`git branch -D` 명령으로 Merge하지 않은 브랜치를 강제로 삭제할 수 있다.
