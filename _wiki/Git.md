---
layout: wiki 
title: Git
tags: ["Software Engineering"]
last_modified_at: 2025/07/24 18:06:34
---

- [Download a specific tag with git](#download-a-specific-tag-with-git)
- [Combine multiple commits into one](#combine-multiple-commits-into-one)
- [git `-u` flag](#git--u-flag)
- [Remove all commits](#remove-all-commits)

# Download a specific tag with git
```shell
$ git clone --branch b1999 https://github.com/ggerganov/llama.cpp
```
It will download `b1999` tag and detach from HEAD.

# Combine multiple commits into one
```shell
$ git rebase -i HEAD~4
```
`HEAD` 다음에는 전체 갯수를 지정한다. 총 4개의 최신 커밋이 표시된다.
```
pick ae536d3 2	<---- older commit
s cf6a7c7 3
s bc15ec6 4
pick 39d9628 5	<---- newer commit
```
s, squash <commit> = use commit, but meld into previous commit. 이전 커밋으로 녹아든다. 여기서는 2번 커밋으로 된다. 2번과 함께 총 3개의 커밋 로그가 합쳐서 표시된다. 로그를 기입하면 주석은 제거되고 남아있는 문구가 커밋 로그가 된다.


커밋 로그만 수정할때는 `r`을 사용하면 된다. (`e`로 하면 `--continue`로 다시 한 번 진행 필요)

GitHub Desktop에서는 push 하기 전 Undo Commit을 지원한다. CLI에서는 push 이후에도 리셋이 가능하다.
```console
# Pager를 사용하지 않도록 최초 설정
$ git config --global core.pager cat

# 커밋 로그 조회
$ git log --oneline
818c17c (HEAD -> main) 2
d60ac7b 1

$ git reset --soft HEAD~1
```

`--soft`는 커밋만 제거되고 수정사항은 파일에 그대로 남아 있지만 `--hard`로 진행할 경우 수정사항도 모두 초기화되기 때문에 주의가 필요하다. 이후 push로 마무리 하면 되는데 rebase를 했기 때문에 failed to push가 발생하고 `--force`로 진행해야 한다.

```console
$ git push --force
```

참고로 `git revert`는 revert 내역을 명시적으로 남기면서 커밋을 추가하기 때문에 커밋 로그를 정리하는 용도로는 적절치 않다.

# git `-u` flag
첫 push 할 때 다음과 같이 진행하는데:
```console
$ git push -u origin master
```
여기서 `-u` flag는 local branch와 remote branch를 자동으로 연결한다. 그렇지 않으면 매 번 origin과 브랜치명을 기입해주어야 한다. 그 다음 부터는 옵션없이 `git pull`, `git push`가 가능하다.

기타 브랜치 명령:
```console
# 브랜치 확인
$ git branch -va

# 현재 브랜치명 변경:
$ git branch -M master
```

# Remove all commits

```
rm -rf .git
git init
git config user.name "Sang Park"
git config user.email "likejazz@gmail.com"
git add .
git commit -m "."
# git branch -m main
git remote add origin https://github.com/likejazz/likejazz.github.io.git
git config http.postBuffer 524288000
git push --force -u origin master
```

[^fn-1]

[^fn-1]: <https://stackoverflow.com/a/78260070/3513266>