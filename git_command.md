# git 명령어 레퍼런스

> 기준 버전: **git 2.56.0** (`git --version`)
> 각 명령의 전체 옵션은 `git <command> -h` 또는 `git help <command>`로 확인할 수 있습니다.

---

## 목차

1. [초기 설정](#1-초기-설정)
2. [저장소 시작](#2-저장소-시작)
3. [기본 작업 흐름 (add / commit)](#3-기본-작업-흐름-add--commit)
4. [상태 및 이력 확인](#4-상태-및-이력-확인)
5. [브랜치](#5-브랜치)
6. [원격 저장소 (remote / push / pull)](#6-원격-저장소-remote--push--pull)
7. [병합과 리베이스](#7-병합과-리베이스)
8. [되돌리기](#8-되돌리기)
9. [임시 저장 (stash)](#9-임시-저장-stash)
10. [태그](#10-태그)
11. [.gitignore](#11-gitignore)
12. [자주 만나는 에러와 해결](#12-자주-만나는-에러와-해결)
13. [자주 쓰는 워크플로](#13-자주-쓰는-워크플로)

---

## 1. 초기 설정

```bash
git config --global user.name "이름"
git config --global user.email "email@example.com"
git config --global init.defaultBranch main   # 새 저장소 기본 브랜치를 main으로
git config --global core.autocrlf true        # Windows 줄바꿈(CRLF) 자동 변환
git config --global core.quotepath false      # 한글 파일명 깨짐 방지
git config --list                             # 현재 설정 확인
git config --global --edit                    # 전역 설정 파일 편집
```

---

## 2. 저장소 시작

```bash
git init                     # 현재 디렉터리를 git 저장소로
git init -b main             # 기본 브랜치를 main으로 지정해 생성
git clone <URL>              # 원격 저장소 복제
git clone <URL> mydir        # 디렉터리 이름 지정
git clone --depth 1 <URL>    # 최신 커밋만 얕게 복제
```

---

## 3. 기본 작업 흐름 (add / commit)

| 명령 | 설명 |
|---|---|
| `git add` | 변경 사항을 스테이징 영역에 올림 |
| `git commit` | 스테이징된 변경 사항을 커밋 |
| `git rm` | 파일 삭제 후 스테이징 |
| `git mv` | 파일 이동/이름 변경 후 스테이징 |

```bash
git add file.py              # 특정 파일
git add .                    # 현재 디렉터리 이하 전체
git add -p                   # 변경 덩어리(hunk)별로 골라서 추가

git commit -m "메시지"
git commit -am "메시지"       # 추적 중인 파일은 add 생략하고 바로 커밋
git commit --amend           # 직전 커밋 수정 (push 전에만 사용 권장)
git commit --amend --no-edit # 메시지는 그대로, 내용만 추가

git rm file.py
git rm --cached file.py      # 파일은 남기고 git 추적만 해제
git mv old.py new.py
```

---

## 4. 상태 및 이력 확인

```bash
git status                   # 작업 트리 상태
git status -s                # 짧은 형식

git diff                     # 스테이징 안 된 변경 사항
git diff --staged            # 스테이징된 변경 사항
git diff main..feature       # 두 브랜치 비교

git log                      # 커밋 이력
git log --oneline            # 한 줄 요약
git log --oneline --graph --all   # 전체 브랜치 그래프
git log -p file.py           # 파일의 변경 이력과 diff
git log --author="이름"

git show <커밋>              # 특정 커밋 내용
git blame file.py            # 줄별 마지막 수정자/커밋
git reflog                   # HEAD 이동 이력 (잃어버린 커밋 복구용)
```

---

## 5. 브랜치

```bash
git branch                   # 로컬 브랜치 목록
git branch -a                # 원격 포함 전체 목록
git branch feature           # 브랜치 생성
git branch -M main           # 현재 브랜치 이름을 main으로 변경 (강제)
git branch -d feature        # 병합된 브랜치 삭제
git branch -D feature        # 강제 삭제

git switch feature           # 브랜치 이동
git switch -c feature        # 생성 후 이동
git switch -                 # 직전 브랜치로 이동

git checkout feature         # (구 방식) 브랜치 이동
git checkout -b feature      # (구 방식) 생성 후 이동
```

---

## 6. 원격 저장소 (remote / push / pull)

| 명령 | 설명 |
|---|---|
| `git remote` | 원격 저장소 관리 |
| `git fetch` | 원격 변경 사항 가져오기 (병합 안 함) |
| `git pull` | fetch + merge (또는 rebase) |
| `git push` | 로컬 커밋을 원격에 업로드 |

```bash
git remote -v                          # 원격 목록과 주소
git remote add origin <URL>            # 원격 추가 (이미 있으면 에러)
git remote set-url origin <URL>        # 기존 원격 주소 변경
git remote rename origin upstream      # 원격 이름 변경
git remote remove origin               # 원격 삭제

git fetch origin
git fetch --prune                      # 원격에서 삭제된 브랜치 정리

git pull                               # 현재 브랜치의 upstream에서 pull
git pull origin main
git pull --rebase                      # merge 대신 rebase로 합치기
git pull origin main --allow-unrelated-histories   # 이력이 다른 저장소 합치기

git push -u origin main                # 최초 push + upstream 설정
git push                               # upstream 설정 후에는 이것만
git push origin master:main            # 로컬 master → 원격 main
git push origin --delete feature       # 원격 브랜치 삭제
git push --force-with-lease            # 안전한 강제 push (남의 커밋 덮어쓰기 방지)
```

> GitHub 저장소 주소 형식: `https://github.com/<사용자명>/<저장소명>.git`
> `https://github.com/users/<사용자명>/projects/...`는 Projects(작업 보드) 주소라서 push할 수 없습니다.

---

## 7. 병합과 리베이스

```bash
git merge feature            # 현재 브랜치에 feature 병합
git merge --no-ff feature    # 항상 병합 커밋 생성
git merge --abort            # 충돌 중 병합 취소

git rebase main              # 현재 브랜치를 main 위로 재배치
git rebase --continue        # 충돌 해결 후 계속
git rebase --abort           # 리베이스 취소

git cherry-pick <커밋>       # 특정 커밋만 현재 브랜치에 적용
```

충돌 해결 순서:

```bash
git status                   # 충돌 파일 확인
# 파일에서 <<<<<<< ======= >>>>>>> 구간을 직접 수정
git add <파일>
git commit                   # merge의 경우 (rebase는 git rebase --continue)
```

---

## 8. 되돌리기

| 상황 | 명령 |
|---|---|
| 작업 중인 파일 변경 취소 | `git restore file.py` |
| 스테이징 취소 (add 취소) | `git restore --staged file.py` |
| 직전 커밋 취소, 변경은 스테이징에 유지 | `git reset --soft HEAD~1` |
| 직전 커밋 취소, 변경은 작업 트리에 유지 | `git reset HEAD~1` |
| 직전 커밋과 변경 모두 버림 ⚠️ | `git reset --hard HEAD~1` |
| 이미 push한 커밋 되돌리기 | `git revert <커밋>` |
| 추적 안 되는 파일 삭제 ⚠️ | `git clean -fd` (`-n`으로 미리보기) |

> ⚠️ `reset --hard`, `clean -fd`는 되돌릴 수 없습니다. 실수했다면 `git reflog`로 이전 커밋을 찾아 `git reset --hard <커밋>`으로 복구할 수 있습니다 (커밋된 내용만 가능).
> 이미 push한 커밋은 `reset` 대신 `revert`를 사용하세요.

---

## 9. 임시 저장 (stash)

```bash
git stash                    # 변경 사항 임시 저장
git stash -u                 # 추적 안 되는 파일 포함
git stash push -m "메모"
git stash list               # 목록
git stash pop                # 가장 최근 것 적용 후 삭제
git stash apply stash@{1}    # 적용만 (삭제 안 함)
git stash drop stash@{0}     # 삭제
git stash clear              # 전체 삭제
```

---

## 10. 태그

```bash
git tag                      # 목록
git tag v1.0.0               # 가벼운 태그
git tag -a v1.0.0 -m "릴리스"   # 주석 태그
git push origin v1.0.0       # 태그 push
git push origin --tags       # 모든 태그 push
git tag -d v1.0.0            # 로컬 태그 삭제
git push origin --delete v1.0.0 # 원격 태그 삭제
```

---

## 11. .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
.venv/
dist/
*.egg-info/

# 환경 변수
.env

# IDE
.vscode/
.idea/
```

```bash
git check-ignore -v file.py  # 어떤 규칙 때문에 무시되는지 확인
git rm -r --cached .venv     # 이미 추적 중인 파일을 무시 목록에 반영
```

> uv 프로젝트에서는 `uv.lock`을 커밋하고 `.venv/`는 무시하는 것이 일반적입니다.

---

## 12. 자주 만나는 에러와 해결

| 에러 메시지 | 원인 | 해결 |
|---|---|---|
| `src refspec main does not match any` | 로컬에 `main` 브랜치가 없음 (`master`이거나 커밋이 없음) | `git branch -M main` 후 push, 또는 먼저 커밋 |
| `remote origin already exists` | `origin`이 이미 등록됨 | `git remote set-url origin <URL>` |
| `'orgin' does not appear to be a git repository` | 원격 이름 오타 또는 미등록 | `git remote -v`로 이름 확인 |
| `Updates were rejected ... fetch first` | 원격에 로컬에 없는 커밋이 있음 | `git pull --rebase` 후 push |
| `refusing to merge unrelated histories` | 이력이 다른 두 저장소 | `git pull origin main --allow-unrelated-histories` |
| `Repository not found` | 주소 오류 또는 접근 권한 없음 | 저장소 URL과 로그인 계정 확인 |
| `Authentication failed` | GitHub는 비밀번호 인증 불가 | Personal Access Token 또는 Git Credential Manager 사용 |
| `'origin' : 용어가 cmdlet ... 인식되지 않습니다` | 명령이 아닌 출력 결과를 터미널에 입력 | `git` 으로 시작하는 명령만 실행 |

---

## 13. 자주 쓰는 워크플로

### 로컬 프로젝트를 GitHub에 처음 올리기

```bash
git init -b main
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<사용자명>/<저장소명>.git
git push -u origin main
```

### 기능 브랜치 작업

```bash
git switch -c feature/login
# 작업...
git add .
git commit -m "Add login"
git push -u origin feature/login
# GitHub에서 Pull Request 생성 → 병합 후
git switch main
git pull
git branch -d feature/login
```

### 작업 전 최신 상태로 맞추기

```bash
git switch main
git pull --rebase
```

### 다른 브랜치로 급하게 이동해야 할 때

```bash
git stash -u
git switch hotfix
# 작업 후
git switch -
git stash pop
```
