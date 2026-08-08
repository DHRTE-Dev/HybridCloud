# Git 기본 명령어 및 원격 저장소 활용 가이드

## 목차
1. [Git 환경 설정 및 저장소 초기화](#1-git-환경-설정-및-저장소-초기화)
	- [패키지 설치 및 사용자 정보 설정](#패키지-설치-및-사용자-정보-설정)
	- [저장소 초기화 및 상태 확인](#저장소-초기화-및-상태-확인)
	- [원격 저장소 복제 (Git Clone)](#원격-저장소-복제-git-clone)
2. [파일 추적 및 커밋 관리](#2-파일-추적-및-커밋-관리)
	- [스테이징 영역 추가 및 관리](#스테이징-영역-추가-및-관리)
	- [커밋 생성 및 이력 조회](#커밋-생성-및-이력-조회)
	- [작업 취소 및 커밋 되돌리기](#작업-취소-및-커밋--되돌리기)
3. [브랜치 관리 및 작업 분기](#3-브랜치-관리-및-작업-분기)
	- [브랜치 조회 및 생성](#브랜치-조회-및-생성)
	- [브랜치 전환 및 병합](#브랜치-전환-및-병합)
	- [브랜치 재배치 (Rebase)](#브랜치-재배치-rebase)
4. [원격 저장소 연동 및 동기화](#4-원격-저장소-연동-및-동기화)
	- [원격 저장소 연결 및 설정](#원격-저장소-연결-및-설정)
	- [작업 브랜치 업로드 및 Upstream 설정](#작업-브랜치-업로드-및-upstream-설정)
	- [코드 다운로드 및 동기화](#코드-다운로드-및-동기화)

---

## 1. Git 환경 설정 및 저장소 초기화

### 패키지 설치 및 사용자 정보 설정

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `yum -y install git` | 패키지 관리자를 통해 **Git**을 시스템에 설치합니다.<br>경량화된 설치가 필요한 경우 `git-core` 패키지를 지정할 수 있습니다. |
| `git config --global user.name <이름>` | 전역 **사용자 이름**을 설정합니다.<br>모든 커밋 작성자 정보에 표시됩니다. |
| `git config --global user.email <이메일>` | 전역 **사용자 이메일**을 설정합니다.<br>원격 저장소 계정과 일치하도록 설정합니다. |
| `git config --global core.editor <에디터>` | 커밋 메시지 작성 시 사용할 **기본 텍스트 에디터**를 지정합니다. |
| `git config --global init.defaultBranch <이름>` | `git init` 실행 시 생성되는 **기본 브랜치 이름**을 지정합니다. |
| `git config --list` | 현재 적용된 모든 **Git 설정 항목**을 조회합니다. |
| `git config --get <키>` | 특정 설정 키에 대한 **설정값만 단독 조회**합니다. |
| `cat ~/.gitconfig` | 사용자 홈 디렉터리의 **전역 설정 파일 내용**을 직접 확인합니다. |

### 자주 사용되는 예제
- git config --global user.name "User_Name"
- git config --global user.email "User_Email@gmail.com"
- git config --global init.defaultBranch main

### 저장소 초기화 및 상태 확인

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git init` | 현재 작업 디렉터리를 **Git 로컬 저장소**로 초기화합니다.<br>실행 시 관리용 `.git` 숨김 디렉터리가 생성됩니다. |
| `git status` | **작업 디렉터리**와 **스테이징 영역**의 변경 상태를 상세 조회합니다. |
| `git status -s` | 파일 변경 상태를 **요약된 형태(Short Format)**로 간결하게 출력합니다. |
| `git status -b` | 요약 출력 시 현재 **브랜치 및 Upstream 상태 정보**를 함께 표시합니다. |
| `tree .git` | 생성된 `.git` 디렉터리의 **내부 구조 및 메타데이터 파일**을 트리 형태로 출력합니다. |

### 자주 사용되는 예제
- git init
- git status -s -b

### 원격 저장소 복제 (Git Clone)

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git clone <URL>` | 원격 저장소의 전체 커밋 이력과 프로젝트 파일들을 내 컴퓨터로 **복제**합니다.<br>로컬 저장소 초기화 및 원격 연결(`origin`)이 자동으로 수행됩니다. |
| `git clone <URL> <디렉터리명>` | 지정한 **새 디렉터리 이름**으로 프로젝트를 복제하여 저장합니다. |
| `git clone -b <브랜치명> <URL>` | 원격 저장소의 특정 **단일 브랜치**만 지정하여 복제합니다. |
| `git clone --depth <개수> <URL>` | 최근 커밋 이력을 **지정한 개수만큼만 가져오는 얕은 복제(Shallow Clone)**를 수행합니다. |

### 자주 사용되는 예제
- git clone https://github.com/GitHub_Name/Repository_Name.git
- git clone -b feature/login https://github.com/GitHub_Name/Repository_Name.git

---

## 2. 파일 추적 및 커밋 관리

### 스테이징 영역 추가 및 관리

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git add <파일명>` | 지정한 특정 파일의 변경 사항을 **스테이징 영역**에 추가합니다. |
| `git add .` | 현재 디렉터리 및 하위 디렉터리의 모든 변경 사항을 **스테이징 영역**에 추가합니다. |
| `git add -A` | 작업 공간 전체의 생성, 수정, 삭제된 모든 파일을 **스테이징 영역**에 일괄 추가합니다. |
| `git add -u` | 기존에 **추적 중인 파일의 변경 및 삭제 사항**만 스테이징 영역에 반영합니다. |
| `git add -p` | 파일의 변경 내용을 **대화형 덩어리(Patch) 단위**로 확인하며 선택적으로 스테이징합니다. |
| `git rm --cached <파일명>` | 파일을 로컬 디렉터리에는 유지하고 **스테이징 영역 및 추적 대상에서만 제거**합니다. |
| `git restore --staged <파일명>` | 스테이징 영역에 올라간 파일을 **언스테이징(Unstage)** 상태로 되돌립니다. |

### 자주 사용되는 예제
- git add README.md
- git add . / git add -A
- git rm --cached secrets.txt
- git restore --staged README.md

### 커밋 생성 및 이력 조회

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git commit -m "<메시지>"` | 스테이징된 변경 사항을 지정한 **커밋 메시지**와 함께 로컬 저장소에 기록합니다. |
| `git commit -am "<메시지>"` | 추적 중인 파일의 변경 사항을 **자동 스테이징 후 커밋**합니다. |
| `git commit --amend` | **직전 커밋을 수정**하거나 커밋 메시지를 변경합니다.<br>추가된 스테이징 내역을 기존 커밋에 병합합니다. |
| `git log` | 저장소의 **전체 커밋 이력**을 상세하게 출력합니다. |
| `git log --oneline` | 커밋 해시와 메시지를 **한 줄로 간결하게 요약**하여 출력합니다. |
| `git log --graph` | 브랜치 분기와 병합 흐름을 **ASCII 그래픽 형태**로 출력합니다. |
| `git log --all` | 현재 브랜치뿐만 아니라 **모든 브랜치의 커밋 이력**을 함께 출력합니다. |
| `git log -n <숫자>` | 최신 커밋부터 **지정한 개수만큼의 이력만 출력**합니다. |
| `git log --stat` | 각 커밋별로 **변경된 파일 및 라인 수 통계**를 포함하여 출력합니다. |

### 자주 사용되는 예제
- git commit -m "first commit"
- git commit --amend --no-edit

### 작업 취소 및 커밋 되돌리기

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git reset --soft <커밋해시>` | 지정한 커밋으로 돌아가며, 이후의 변경 사항은 **스테이징 영역에 유지**합니다. |
| `git reset --mixed <커밋해시>` | 지정한 커밋으로 돌아가며, 이후의 변경 사항은 **작업 디렉터리에만 유지**하고 언스테이징 처리합니다. (기본 옵션) |
| `git reset --hard <커밋해시>` | 지정한 커밋으로 돌아가며, 이후의 **모든 변경 사항을 완전히 삭제**합니다. |
| `git revert <커밋해시>` | 특정 커밋의 변경 사항을 되돌리는 **새로운 커밋을 생성**하여 이력을 보존하면서 작업을 취소합니다. |

### 자주 사용되는 예제
- git reset --soft HEAD~1
- git reset --hard HEAD~2
- git revert HEAD

---

## 3. 브랜치 관리 및 작업 분기

### 브랜치 조회 및 생성

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git branch` | 현재 로컬 저장소에 존재하는 **브랜치 목록**을 조회합니다. |
| `git branch -v` | 각 브랜치의 **최신 커밋 해시 및 메시지**를 포함하여 상세 조회합니다. |
| `git branch -a` | 로컬 및 원격 추적 브랜치를 포함한 **모든 브랜치**를 조회합니다. |
| `git branch -r` | **원격 저장소의 브랜치 목록만** 필터링하여 조회합니다. |
| `git branch <브랜치명>` | 현재 작업 중인 커밋을 기준으로 **새로운 브랜치**를 생성합니다. |
| `git branch -M <새 이름>` | 현재 위치한 브랜치의 이름을 **강제로 변경**합니다. |
| `git branch -d <브랜치명>` | 병합이 완료된 **안전한 브랜치를 삭제**합니다. |
| `git branch -D <브랜치명>` | 병합 여부와 상관없이 **브랜치를 강제 삭제**합니다. |

### 자주 사용되는 예제
- git branch -M main

### 브랜치 전환 및 병합

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git switch <브랜치명>` | 작업 공간을 지정한 **브랜치로 전환**합니다. |
| `git switch -c <브랜치명>` | 새로운 브랜치를 **생성함과 동시에 해당 브랜치로 전환**합니다. |
| `git checkout <브랜치명>` | 지정한 **브랜치로 전환**합니다. |
| `git checkout -b <브랜치명>` | 새로운 브랜치를 **생성하고 전환**합니다. |
| `git merge <브랜치명>` | 지정한 브랜치의 변경 이력을 **현재 위치한 브랜치로 병합**합니다. |
| `git merge --no-ff <브랜치명>` | Fast-Forward 관계이더라도 **명시적인 병합 커밋(Merge Commit)**을 생성합니다. |
| `git merge --abort` | 병합 과정에서 충돌이 발생했을 때 **병합 작업을 취소**하고 이전 상태로 되돌립니다. |

### 자주 사용되는 예제
- git switch main
- git switch -c feature/login
- git merge feature/login
- git merge --no-ff feature/login
- git merge --abort

### 브랜치 재배치 (Rebase)

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git rebase <목표브랜치>` | 현재 브랜치의 커밋들을 지정한 타겟 브랜치의 최신 커밋 이후로 **재배치하여 선형적인 이력**을 만듭니다. |
| `git rebase -i <커밋해시>` | 대화형 모드(Interactive)로 여러 커밋을 **합치거나(squash) 수정, 삭제**하는 등 이력을 정리할 수 있습니다. |
| `git rebase --continue` | Rebase 중 발생한 충돌을 해결한 후 **재배치 작업을 계속 진행**합니다. |
| `git rebase --abort` | Rebase 도중 문제가 발생하여 **작업을 취소하고 이전 상태로 복구**합니다. |

### 자주 사용되는 예제
- git rebase main
- git rebase -i HEAD~3
- git rebase --continue

---

## 4. 원격 저장소 연동 및 동기화

### 원격 저장소 연결 및 설정

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git remote add <별칭> <URL>` | 로컬 저장소에 **원격 저장소 URL**을 연결합니다.<br>기본 원격지 별칭으로 `origin`을 주로 사용합니다. |
| `git remote -v` | 연결된 원격 저장소의 **별칭과 단방향 URL**을 조회합니다. |
| `git remote show <별칭>` | 지정한 원격 저장소의 **상세 정보 및 추적 브랜치 상태**를 확인합니다. |
| `git remote remove <별칭>` | 지정한 **원격 저장소 연결을 해제**합니다. |
| `git remote rename <구별칭> <신별칭>` | 원격 저장소의 **별칭 이름을 변경**합니다. |
| `git remote set-url <별칭> <신규 URL>` | 기존 원격 저장소의 **접속 URL을 새로운 주소로 변경**합니다. |

### 자주 사용되는 예제
- git remote add origin https://github.com/GitHub_Name/Repository_Name.git
- git remote -v
- git remote show origin
- git remote set-url origin https://github.com/GitHub_Name/New_Repository_Name.git

### 작업 브랜치 업로드 및 Upstream 설정

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git push -u <원격지> <브랜치>` | 로컬 브랜치를 원격지에 푸시하며 **Upstream 관계를 설정**합니다.<br>원격 저장소에 해당 브랜치가 없으면 **자동으로 새로 생성**됩니다. |
| `git push <원격지> <브랜치>` | 지정한 원격 저장소의 **특정 브랜치로 커밋을 업로드**합니다. |
| `git push` | Upstream이 설정된 상태에서 **현재 연결된 원격 브랜치로 자동 푸시**합니다. |
| `git push <원격지> --delete <브랜치>` | 원격 저장소에 존재하는 **원격 브랜치를 삭제**합니다. |
| `git push --force-with-lease` | 안전성을 검증하며 원격 저장소의 커밋 이력을 **강제로 덮어씁니다**. |

### 자주 사용되는 예제
- git push -u origin main
- git push -u origin feature/login
- git push origin --delete feature/login

### 코드 다운로드 및 동기화

| 명령어 / 옵션 | 기능 및 상세 설명 |
| --- | --- |
| `git pull <원격지> <브랜치>` | 원격 저장소의 최신 변경 사항을 가져와 **현재 로컬 브랜치와 병합**합니다. |
| `git pull --rebase` | 원격 저장소의 변경 사항을 가져올 때 병합 커밋 대신 **Rebase 방식으로 정렬**합니다. |
| `git fetch <원격지>` | 원격 저장소의 최신 커밋 이력 및 브랜치 정보를 가져오되 **로컬 코드와 병합하지 않습니다**. |
| `git fetch --all` | 등록된 **모든 원격 저장소의 최신 정보**를 가져옵니다. |
| `git fetch --prune` | 원격 저장소에서 이미 삭제된 브랜치의 **로컬 추적 가지를 정리**합니다. |

### 자주 사용되는 예제
- git pull origin main
- git pull origin feature/login
- git pull --rebase origin main
- git fetch origin
- git fetch --prune
