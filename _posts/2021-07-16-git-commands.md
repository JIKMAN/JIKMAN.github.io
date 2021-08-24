---
title:  "Git 구성요소 및 커맨드"

categories:
  - Git
tags:
  - [Blog, git]
toc: true
toc_sticky: true
---

# Git

> #### 구성요소

1. __working directory__

   * 현재 작업을 하고있는 프로젝트 디렉토리

2. __staging area__

   * repository에 저장될 변경이력들을 스냅샷(snap shot)의 형태로 올려두는 곳

3. __repository__

   * 버전에 따른 변경 과정이 저장되는 공간

   * 최초 init후 생성되는`.git` 디렉토리가 레포지토리임

     ```
     # 커밋(commit)이란 
     : 프로젝트 디렉토리의 특정 모습을 하나의 버전으로 repository에 남기는 행위
     ```

     

     커밋(commit)이란 프로젝트 디렉토리의 특정 모습을 하나의 버전으로 repository에 남기는 행위

   ```bash
   # local repository
   : 지역저장소. 현재 PC의 프로젝트 디렉토리
   
   # remote repository
   : 원격저장소. GitHub상의 프로젝트 디렉토리
   ```

   



> #### 명령어

* `git init`
  * 레포지토리(`.git`)를 생성
* `git config user.name "본인이름"`
  * 레포지토리에 커밋을 하기 위해 본인의 이름 설정
* `git config user.email "본인 이메일"`

  * 본인의 이메일 설정
* `git add [파일명]`
  * 레포지토리에 커밋을 하기 위해 staging area에 파일을 추가
* `git add .`

  * 변경된 모든 사항을 커밋에 반영
* `git commit -m "변경 사항에 대한 설명"`
  - 변경사항을 staging area에서 repository로 커밋함
* `git commit --amend`
  * 최신 커밋을 다시 되돌려서 커밋 메세지를 수정할 수 있음
* `git status`
  * 프로젝트 디렉토리의 현재 상태를 알려줌
* `git push --set-upstream origin master`

  * local repository의 내용을 처음으로 GitHub remote repository로 보낼 때
* `git push`

  * local repository에서 remote repository로 커밋 반영
* `git pull`

  * remote repository에서 local repository로 커밋 반영
* `git clone [github 프로젝트의 주소]`

  * 해당 주소의 프로젝트의 레포지토리를 가져옴

* `git log`
  * 지금까지의 커밋의 히스토리를 보여줌
* `git log --oneline`
  * 커밋 히스토리를 한줄로 보여줌
* `git show [커밋명]`
  * 현재 커밋과 이전 커밋을 비교할 수 있음(해시는 4자 정도만 써줘도 됌)
* `git diff [커밋명1] [커밋명2]`
  * 두 커밋간의 차이를 비교할 수 있음
* `git help [커맨드명]`

  * 커맨드의 자세한 사용법 설명이 나타남
* `git reset --hard [커밋명]`

  * __HEAD__가 과거의 커밋을 가리키게 하고, working directory의 내용도 과거 커밋의 모습으로 돌아가게 함

```bash
# HEAD란
: 현재 브랜치를 가르키는 포인터이다. 현재 브랜치의 마지막 커밋의 스냅샷

# git reset의 option
--soft : 해당 커밋 이후의 작업이 ropository상에서 사라지고 staging area, working directory에는 남아 있음
--mixed : 해당 커밋 이후의 작업이 repository, staging area에서 사라지나 working directory 에는 남아 있음
--hard : 해당 커밋 이후로 한 작업이 전부 사라짐 (작업이 전부 사라져도 상관없을 때 사용)
```

* `HEAD^`

  * HEAD 바로 이전 커밋을 나타냄
  * `git reset --hard HEAD^` : HEAD 바로 이전 커밋으로 reset

* `HEAD~2`

  * HEAD 2단계 전 커밋을 나타냄
  * `git reset --hard HEAD~3` : HEAD 3단계 전의 커밋으로 reset

* `git tag [태그 이름] [커밋명]`

  * 커밋에 태그를 지정할 수 있음

* `git tag`

  * 모든 태그 조회

* `git show [태그 이름]`

  * 해당 태그를 가진 커밋의 내용을 볼수 있음

  

> #### branch

협업 시, 브랜치를 이용하면 하나의 프로젝트를 여러 갈래로 나누어서 관리할 수 있음

최초 commit을 할 때 master 브랜치가 생기고, 그 위에 다른 갈래로 나누어 지는 브랜치를 생성 및 원래의 master 브랜치로 병합할 수 있음

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Ft3Ih1%2FbtqFvuOfQFZ%2FHrlGV5obNZ9NNRmOnEbkok%2Fimg.png)

* `git branch [브랜치명]`
  * 브랜치 생성
* `git checkout [브랜치명]`
  * 브랜치로 이동
* `git checkout -b [브랜치명]`
  * 브랜치를 생성 & 해당 브랜치로 이동

* `git branch -d [브랜치명]`
  * 브랜치 삭제

* `git merge [브랜치명]`
  * 다른 브랜치의 내용을 가져와 현재 브랜치에 병합
  * `git merge master` : 현재 브랜치에 master 브랜치의 내용을 합침

```shell
# Conflict 발생

- A 브랜치와 B 브랜치를 병합하려고 할 때, 서로 다른 내용이 있다면 충돌이 발생할 수 있다.
이 때는 서로 다른 내용을 merge의 결과가 되었으면 하는 모습으로 코드를 수정해주면
충돌을 해결할 수 있다.

- git merge --abort : merge 자체를 취소하고 merge를 시도하기 이전 상태로 돌아감
```

```bash
# Detached HEAD

- git checkout [브랜치명]으로 HEAD가 브랜치를 가르키게 할 수 있다. 
또한, git checkout [커밋명]을 통해 HEAD가 브랜치를 가르키지 않고 커밋을 직접적으로 가르키게 할 수 있다.
이 때, HEAD가 branch가 아닌 commit을 직접 가르키는 경우의 HEAD를 'Detached HEAD'라고 한다. 
해당 HEAD에서 git branch [브랜치명]을 통해 새로운 브랜치를 만들 수 있다.
```



> #### Git 협업

```
local repository에 새로운 커밋을 remote repo에 push 했을 때, 
이미 다른 누군가가 remote repo의 내용을 변경했다면 Conflict가 발생할 수 있다.
그렇기 때문에 local repo 작업을 하기 전 git pull을 이용해 
remote repo의 프로젝트의 변경사항을 먼저 가져온 후 작업을 진행하는 편이 좋다.
```

* `git fetch`

  * remote repo에 있는 브랜치의 내용을 가져오기만 하고 merge는 하지 않음

    일단 가져와서 살펴본 후에 merge하고 싶을 때 사용

* `git diff`

  * 두 커밋 / 두 브랜치 간의 차이를 비교

* `git blame`

  * 어떤 파일의 특정 부분을 누가 작성했는지 찾아내기 위한 커맨드

  

> #### Git 심화

* `git revert [커밋명]`

  * 이미 remote repository에 올라간 커밋을 취소한 revert 커밋을 생성

    ```bash
    # reset을 사용하지 않는 이유
    
    마지막 커밋을 취소하고 싶은데 remote repository에 이미 push를 해버렸다면
    reset을 이용하여 되돌린 후 push를 다시 하는 것은 불가능하다. 
    (local repo를 reset 한다면 remote repo의 내용이 local repo의 내용 보다 최신버전이 되기 때문)
    이 때, revert를 통해 이전 내용이 담긴 revert 커밋을 생성한 후 
    remote repository에 push할 수 있다. 
    (revert커밋이란 이전 내용으로 돌아가는 것이 아니라 이전 내용이 담긴 새로운 커밋을 만드는 것)
    ```



* `git revert [커밋명1]..[커밋명2]`
  * 커밋1의 ___다음 커밋___ 부터 커밋2 까지 취소한 여러 revert 커밋을 생성
* `git reflog`
  * HEAD가 이동한 로그를 추적하는 데 이용

```bash
# git reset을 하고나서 돌아오려면?

git reset --hard를 통해 커밋이 reset 되었을 때, reset된 커밋의 이름을 알고 있다면 
git reset --hard [리셋된 커밋명]을 통해 되돌아 갈 수 있다.
그러나 커밋명을 알지 못하는 경우 'git reflog(reference log)'를 통해 지난 커밋의 이름을 확인할 수 있다. 
정확히 말하면 지금까지 HEAD가 이동한 로그를 추적할 수 있다.
```



* `--graph`
  * 커밋 히스토리가 각 브랜치와의 관계가 잘 드러나도록 그래프로 출력
  * `git log --oneline --all --graph`

* `git rebase`
  * 두 갈레로 갈라져 있는 브랜치를 합병하는 merge 대신 두 브랜치를 덮어씌워 하나의 갈레로 만듬
  * merge와 결과는 똑같으나 merge로 만들어진 커밋 히스토리보다 깔끔함

* `git stash`
  * 부득이하게 현재 브랜치의 작업을 아직 커밋하지 않았는데, 잠시 중단하고 다른 브랜치로 이동하여 작업을 해야하는 상황에서, 작업 중이던 내용을 잠깐 저장하고 싶을 때 사용
  * working directory에서 작업하던 내용을 stack에 따로 보관함
* `git stash list`
  * `git stash`에 의해 저장되있던 내용의 리스트 확인
* `git stash apply`
  * `git stash`에 저장되어있던 이전 작업 내용을 다시 불러옴

```bash
# 잘못된 브랜치에서 작업하고 있었을 때도?

1. git stash로 stack에 작업 내용을 저장한다.
2. 올바른 브랜치로 가서 다시 git stash apply를 한다.
```

* `git stash drop [stack내의 작업 내용명]`
  * stack 내에 쌓인 작업 내용을 삭제

```bash
# git stash pop [작업 내용명]
- stack에 저장한 후 apply할 경우에 작업 내용이 나중에 또 쓸 필요가 없다면,
'git stash pop'을 이용하면 작업내용도 가져오고 따로 'drop'을 이용해 삭제할 필요 없이 stack에서 바로 삭제한다.
즉, pop = apply + drop
```



* `git cherry-pick [커밋명]`

  * 한 브랜치에서 다양한 작업을 했을 때, 원하는 작업이 들어있는 특정 커밋을 골라서 현재 커밋에 반영

  

```bash
# .gitignore

이 폴더 안에 명시된 조건에 해당하는 파일은 
working directory에 있음에도 불구하고 Git에 의해 무시되는 파일들이다.
만약 working directory에서 버전 관리를 할 필요가 없는 것들이 있다면 
.gitignore 파일에 그 이름을 추가하여 관리가 가능
```

- 프로그램에 따라 권장되는 gitignore를 만들어 주는 사이트 https://www.gitignore.io/



> #### 백업

1. GitHub 원격 저장소 생성

* create repository를 통해 원격 저장소를 생성한다.

2. remote repository 정보 입력

* `git remote add [저장소명] [저장소 주소]`
  * 지역 저장소를 원격 저장소와 연결
  * `git remote add origin https://github.com/JIKMAN/git-start.git`

* `git remote -v`
  * 원격 저장소 정보 상세 출력

3. remote repository에 push하기

* `git push origin master`
  * `git push --set-upstream origin master` : 최초 push시 해당 명령어로 push를 하면 이후는 `git push` 명령어로 커밋 push가 가능

