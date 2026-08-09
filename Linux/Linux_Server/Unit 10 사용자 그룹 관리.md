# 사용자 및 그룹 관리

## 목차

- [1. 사용자 관리](#1-사용자-관리)
	- [사용자 계정과 UID](#사용자-계정과-uid)
	- [사용자 정보 파일](#사용자-정보-파일)
	- [etcpasswd 필드](#etcpasswd-필드)
	- [etcshadow 필드](#etcshadow-필드)
	- [useradd 명령어](#useradd-명령어)
	- [usermod 명령어](#usermod-명령어)
	- [userdel 명령어](#userdel-명령어)
	- [etcskel 디렉토리](#etcskel-디렉토리)
	- [사용자 계정 기본값](#사용자-계정-기본값)
	- [사용자 계정 로그인 제한](#사용자-계정-로그인-제한)
	- [자주 사용하는 사용자 관리 예제](#자주-사용하는-사용자-관리-예제)
- [2. 그룹 관리](#2-그룹-관리)
	- [그룹의 개념](#그룹의-개념)
	- [etcgroup 파일](#etcgroup-파일)
	- [groupadd 명령어](#groupadd-명령어)
	- [groupmod 명령어](#groupmod-명령어)
	- [groupdel 명령어](#groupdel-명령어)
	- [주 그룹과 보조 그룹 관리](#주-그룹과-보조-그룹-관리)
	- [gpasswd 명령어](#gpasswd-명령어)
	- [그룹을 이용한 권한 관리](#그룹을-이용한-권한-관리)
	- [자주 사용하는 그룹 관리 예제](#자주-사용하는-그룹-관리-예제)
- [3. 패스워드 사용 제한 관리](#3-패스워드-사용-제한-관리)
	- [Password Aging 개념](#password-aging-개념)
	- [chage 명령어](#chage-명령어)
	- [패스워드 만료 정책](#패스워드-만료-정책)
	- [자주 사용하는 패스워드 관리 예제](#자주-사용하는-패스워드-관리-예제)
- [4. 패스워드 복잡도 관리](#4-패스워드-복잡도-관리)
	- [PAM 설정](#pam-설정)
	- [pwquality.conf](#pwqualityconf)
	- [주요 패스워드 복잡도 옵션](#주요-패스워드-복잡도-옵션)
- [5. 사용자 및 그룹 관리 핵심 정리](#5-사용자-및-그룹-관리-핵심-정리)

---

## 1. 사용자 관리

### 사용자 계정과 UID

**사용자 계정(User Account)**은 사용자와 실행 중인 프로세스 사이의 보안 경계를 제공한다.

Linux에서는 사용자를 이름으로 관리하는 것처럼 보이지만 실제 시스템 내부에서는 **UID(User ID)**를 이용하여 사용자를 식별한다.

사용자 계정은 다음과 같은 시스템 보안 요소와 직접 관련된다.

| 항목 | 설명 |
|---|---|
| 사용자 이름 | 사용자가 로그인할 때 사용하는 이름 |
| UID | 시스템에서 사용자를 식별하는 고유 번호 |
| GID | 사용자의 **주 그룹(Primary Group)**을 식별하는 번호 |
| 홈 디렉토리 | 사용자의 개인 작업 공간 |
| 로그인 쉘 | 로그인 후 실행되는 기본 쉘 |
| 파일 소유권 | 파일 및 디렉토리에 대한 접근 권한 판단에 사용 |
| 프로세스 | 모든 프로세스는 특정 사용자 권한으로 실행됨 |

### UID 범위

CentOS Stream 10에서는 일반적으로 다음과 같은 UID 범위를 사용한다.

| UID 범위 | 구분 | 설명 |
|---|---|---|
| **0** | root | 슈퍼유저 계정 |
| **1~200** | 정적 시스템 사용자 | 시스템 프로세스에 정적으로 할당되는 계정 |
| **201~999** | 동적 시스템 사용자 | 시스템 서비스 및 소프트웨어에서 사용하는 계정 |
| **1000 이상** | 일반 사용자 | 일반 사용자를 위한 UID 범위 |

> 이전 CentOS/RHEL 버전에서는 시스템 사용자와 일반 사용자의 UID 기준이 달랐으므로 버전별 UID 정책을 구분해야 한다.

### 사용자 정보 파일

| 파일 | 주요 내용 | 특징 |
|---|---|---|
| **/etc/passwd** | 사용자 기본 정보 | 사용자 이름, UID, GID, 홈 디렉토리, 쉘 등 |
| **/etc/shadow** | 패스워드 및 패스워드 만료 정보 | 암호 해시와 Password Aging 정보 저장 |
| **/etc/group** | 그룹 정보 | 그룹 이름, GID, 그룹 구성원 정보 |
| **/etc/gshadow** | 그룹 인증 정보 | 그룹 패스워드 및 그룹 관리자 정보 |

### /etc/passwd 필드

`/etc/passwd`의 기본적인 형식은 다음과 같다.

`사용자명:x:UID:GID:Comment:홈디렉토리:로그인쉘`

| 필드 | 의미 |
|---|---|
| 1 | **사용자 이름(Login Name)** |
| 2 | 패스워드 필드. 일반적으로 실제 암호는 `/etc/shadow`에 저장되므로 `x`가 표시됨 |
| 3 | **UID(User ID)** |
| 4 | **GID(Group ID)**. 사용자의 주 그룹 |
| 5 | **Comment/GECOS**. 사용자 설명 정보 |
| 6 | **홈 디렉토리** |
| 7 | **로그인 쉘** |

사용자 정보를 확인할 때는 `grep 사용자명 /etc/passwd` 또는 `getent passwd 사용자명`을 사용할 수 있다.

### /etc/shadow 필드

`/etc/shadow`는 사용자 패스워드와 패스워드 사용 기간을 관리한다.

기본적인 형식은 다음과 같다.

`사용자명:암호해시:LastChange:Min:Max:Warn:Inactive:Expire:Reserved`

| 필드 | 의미 |
|---|---|
| 1 | 사용자 이름 |
| 2 | **암호 해시** |
| 3 | 마지막 패스워드 변경일. 1970-01-01부터의 날짜 수 |
| 4 | 패스워드 변경 후 다시 변경할 수 있을 때까지의 **최소 일수(MIN)** |
| 5 | 패스워드를 사용할 수 있는 **최대 일수(MAX)** |
| 6 | 패스워드 만료 전 경고 기간 **WARN** |
| 7 | 패스워드 만료 후 계정을 비활성화하기까지의 유예 기간 **INACTIVE** |
| 8 | 계정 자체의 만료일 **EXPIRE DATE** |
| 9 | 예약 필드 |

### CentOS Stream 10 패스워드 해시

CentOS Stream 10의 기본 패스워드 해시 방식은 **yescrypt**이다.

| 식별자 | 알고리즘 |
|---|---|
| `$1$` | MD5 |
| `$5$` | SHA-256 |
| `$6$` | SHA-512 |
| `$y$` | **yescrypt** |

CentOS Stream 10에서는 기존 SHA-512 해시도 인식할 수 있지만 새 패스워드 생성 및 변경 시에는 **yescrypt**가 기본적으로 사용된다.

### useradd 명령어

**useradd**는 사용자 계정을 생성하는 명령어이다.

형식: `useradd [옵션] LOGIN`

| 옵션 | 설명 |
|---|---|
| `-b, --base-dir BASE_DIR` | 홈 디렉토리를 생성할 기본 디렉토리 지정 |
| `-c, --comment COMMENT` | GECOS/Comment 정보 지정 |
| `-d, --home-dir HOME_DIR` | 홈 디렉토리 지정 |
| `-D, --defaults` | 사용자 생성 기본 설정 확인 및 변경 |
| `-e, --expiredate EXPIRE_DATE` | 계정 만료일 지정 |
| `-f, --inactive INACTIVE` | 패스워드 만료 후 비활성화까지의 기간 지정 |
| `-g, --gid GROUP` | 주 그룹 지정 |
| `-G, --groups GROUPS` | 보조 그룹 지정 |
| `-k, --skel SKEL_DIR` | `/etc/skel` 대신 사용할 디렉토리 지정 |
| `-K, --key KEY=VALUE` | `/etc/login.defs` 기본값 재정의 |
| `-m, --create-home` | 홈 디렉토리 생성 |
| `-M` | 홈 디렉토리를 생성하지 않음 |
| `-r` | 시스템 계정 생성 |
| `-o, --non-unique` | 중복 UID 사용 허용 |
| `-p, --password PASSWORD` | 암호화된 패스워드 지정 |
| `-s, --shell SHELL` | 로그인 쉘 지정 |
| `-u, --uid UID` | UID 직접 지정 |
| `-Z, --selinux-user SEUSER` | SELinux 사용자 매핑 지정 |

### useradd 기본 동작

일반적인 사용자 생성 시 다음과 같은 기본 설정이 사용된다.

| 항목 | 일반적인 기본값 |
|---|---|
| UID | 사용 가능한 일반 사용자 UID |
| 주 그룹 | 사용자 이름과 동일한 개인 그룹을 생성하여 사용 |
| 홈 디렉토리 | `/home/사용자명` |
| 로그인 쉘 | `/bin/bash` |
| Comment | 지정하지 않으면 비어 있음 |
| SKEL | `/etc/skel` |
| Mail Spool | 설정에 따라 `/var/spool/mail/사용자명` 생성 |

### usermod 명령어

**usermod**는 기존 사용자 계정의 정보를 변경한다.

형식: `usermod [옵션] LOGIN`

| 옵션 | 설명 |
|---|---|
| `-a, --append` | 기존 보조 그룹을 유지하면서 그룹 추가. **`-G`와 함께 사용** |
| `-c, --comment COMMENT` | Comment 변경 |
| `-d, --home HOME_DIR` | 홈 디렉토리 변경 |
| `-e, --expiredate EXPIRE_DATE` | 계정 만료일 변경 |
| `-f, --inactive INACTIVE` | 패스워드 만료 후 비활성화 기간 변경 |
| `-g, --gid GROUP` | 주 그룹 변경 |
| `-G, --groups GROUPS` | 보조 그룹 목록 변경 |
| `-l, --login NEW_LOGIN` | 로그인 이름 변경 |
| `-L, --lock` | 사용자 패스워드 잠금 |
| `-m, --move-home` | 변경된 홈 디렉토리로 기존 홈 내용 이동. `-d`와 함께 사용 |
| `-o, --non-unique` | 중복 UID 허용 |
| `-s, --shell SHELL` | 로그인 쉘 변경 |
| `-u, --uid UID` | UID 변경 |
| `-U, --unlock` | 사용자 패스워드 잠금 해제 |
| `-Z, --selinux-user` | SELinux 사용자 매핑 변경 |

**주의:** `usermod -G`는 기존 보조 그룹 목록을 **교체**한다. 기존 그룹을 유지하면서 그룹을 추가하려면 `usermod -aG`를 사용한다.

### userdel 명령어

**userdel**은 사용자 계정을 삭제한다.

형식: `userdel [옵션] LOGIN`

| 옵션 | 설명 |
|---|---|
| `-f, --force` | 강제로 사용자 삭제 |
| `-r, --remove` | 홈 디렉토리와 Mail Spool까지 삭제 |
| `-h, --help` | 도움말 출력 |

| 명령어 | 삭제 범위 |
|---|---|
| `userdel 사용자명` | 계정 정보 삭제. 홈 디렉토리 등 사용자 자원은 남을 수 있음 |
| `userdel -r 사용자명` | 계정 정보와 홈 디렉토리 및 Mail Spool 삭제 |

사용자를 삭제한 뒤에도 사용자 소유 파일이 남아 있으면 소유자가 UID 숫자로 표시될 수 있다. 이는 `/etc/passwd`에 해당 UID의 사용자 정보가 더 이상 없기 때문이다.

### /etc/skel 디렉토리

`/etc/skel`은 새로운 사용자 계정의 홈 디렉토리를 생성할 때 기본적으로 참조되는 **Skeleton Directory**이다.

| 항목 | 설명 |
|---|---|
| `/etc/skel` | 신규 사용자 홈 디렉토리에 복사할 기본 파일 보관 |
| `/home/사용자명` | 새 사용자의 홈 디렉토리 |
| 적용 시점 | **새 사용자가 생성될 때** |
| 기존 사용자 | `/etc/skel` 변경 내용이 자동으로 반영되지 않음 |

따라서 `/etc/skel/.bashrc` 등을 변경하면 이후 생성되는 사용자에게 적용되며 기존 사용자의 홈 디렉토리에는 자동 반영되지 않는다.

### 사용자 계정 기본값

`useradd -D` 명령으로 사용자 생성 시 적용되는 기본값을 확인하거나 변경할 수 있다.

| 명령어 | 기능 |
|---|---|
| `useradd -D` | 현재 기본값 확인 |
| `useradd -D -s /bin/sh` | 기본 로그인 쉘 변경 |
| `useradd -D -b /users` | 기본 홈 디렉토리 생성 위치 변경 |

주요 설정은 `/etc/default/useradd`에서 확인할 수 있다.

| 설정 | 의미 |
|---|---|
| `GROUP` | 기본 그룹 관련 설정 |
| `HOME` | 홈 디렉토리의 기본 위치 |
| `INACTIVE` | 패스워드 만료 후 비활성화 기간 |
| `EXPIRE` | 계정 만료 관련 기본값 |
| `SHELL` | 기본 로그인 쉘 |
| `SKEL` | 기본 Skeleton 디렉토리 |
| `CREATE_MAIL_SPOOL` | 사용자 Mail Spool 생성 여부 |

### 사용자 계정 로그인 제한

사용자의 로그인을 제한하는 대표적인 방법은 **패스워드 잠금**, **nologin 쉘**, **계정 만료**이다.

| 방법 | 로그인 | `su` 사용자 전환 | 해제 방법 |
|---|---|---|---|
| `usermod -L 사용자명` | 차단 | 가능 | `usermod -U 사용자명` |
| `usermod -s /sbin/nologin 사용자명` | 차단 | 일반적인 대화형 전환도 차단 | `usermod -s /bin/bash 사용자명` |
| `usermod -e 0 사용자명` | 차단 | 가능 | 계정 만료일을 정상 값으로 변경 |

**`usermod -L`**은 패스워드 인증을 잠그는 방식이므로 임시적인 계정 잠금에 적합하다.

**`/sbin/nologin`**은 대화형 로그인을 차단하기 위한 쉘이며, 애플리케이션 서비스용 계정 등에 주로 사용한다.

**계정 만료**는 계정 자체의 사용 기간을 제한하는 방식이다.

### 자주 사용하는 사용자 관리 예제

사용자 생성: `useradd UserName` → 패스워드 설정: `passwd UserName`

홈 디렉토리를 직접 지정하여 사용자 생성: `useradd -d /users/UserName UserName`

홈 디렉토리를 생성하지 않고 특정 경로를 사용자 홈으로 지정: `useradd -M -d /application UserName`

시스템 계정 생성: `useradd -r -s /sbin/nologin UserName`

UID를 직접 지정하여 사용자 생성: `useradd -u 2000 UserName`

주 그룹과 보조 그룹을 지정하여 사용자 생성: `useradd -g PrimaryGroup -G SecondaryGroup UserName`

UID 변경: `usermod -u 2000 UserName`

로그인 쉘 변경: `usermod -s /bin/zsh UserName`

Comment 변경: `usermod -c "User Description" UserName`

홈 디렉토리와 사용자 이름을 함께 변경: `usermod -l NewUserName -d /home/NewUserName -m OldUserName`

사용자 삭제: `userdel UserName`

홈 디렉토리까지 삭제: `userdel -r UserName`

---

## 2. 그룹 관리

### 그룹의 개념

**그룹(Group)**은 여러 사용자를 하나의 집합으로 묶어 파일 및 시스템 자원에 대한 접근 권한을 효율적으로 관리하기 위한 단위이다.

사용자는 하나의 **주 그룹(Primary Group)**을 가지며 여러 개의 **보조 그룹(Secondary Group)**에 속할 수 있다.

| 구분 | 설명 |
|---|---|
| **Primary Group** | 사용자의 기본 그룹. `/etc/passwd`의 GID로 확인 |
| **Secondary Group** | 추가적으로 소속된 그룹. `/etc/group`에서 확인 |
| GID | 그룹을 식별하는 고유 번호 |
| 그룹 이름 | 관리자가 사람이 이해하기 쉽게 사용하는 그룹 식별자 |

일반 사용자 생성 시 사용자 이름과 동일한 개인 그룹을 생성하고 이를 주 그룹으로 사용하는 방식이 일반적이다.

### /etc/group 파일

`/etc/group`은 그룹 정보를 저장하는 파일이다.

기본적인 형식은 다음과 같다.

`그룹명:x:GID:그룹구성원목록`

| 필드 | 의미 |
|---|---|
| 1 | 그룹 이름 |
| 2 | 그룹 패스워드 필드 |
| 3 | **GID** |
| 4 | 보조 그룹에 속한 사용자 목록 |

사용자의 그룹 정보를 확인할 때는 `id UserName` 또는 `groups UserName`을 사용할 수 있다.

### groupadd 명령어

**groupadd**는 새로운 그룹을 생성한다.

형식: `groupadd [옵션] GROUP`

| 옵션 | 설명 |
|---|---|
| `-f, --force` | 지정한 그룹이 이미 존재해도 성공 상태로 종료 |
| `-g, --gid GID` | GID 지정 |
| `-r` | 시스템 그룹 생성 |
| `-o, --non-unique` | 중복 GID 허용 |
| `-K, --key KEY=VALUE` | `/etc/login.defs` 기본값 재정의 |
| `-h, --help` | 도움말 출력 |

### groupmod 명령어

**groupmod**는 기존 그룹의 정보를 변경한다.

형식: `groupmod [옵션] GROUP`

| 옵션 | 설명 |
|---|---|
| `-g GID` | 그룹 GID 변경 |
| `-n NEW_GROUP_NAME` | 그룹 이름 변경 |
| `-o` | 중복 GID 허용 |

GID를 변경하면 기존 GID를 소유자로 가지고 있는 파일의 그룹 ID가 자동으로 모두 변경되는 것은 아니다. 따라서 관련 파일의 그룹 소유권을 별도로 확인해야 한다.

### groupdel 명령어

**groupdel**은 그룹을 삭제한다.

형식: `groupdel GROUP`

그룹 삭제 전 해당 그룹을 주 그룹으로 사용하는 사용자가 있는지 확인하는 것이 안전하다.

### 주 그룹과 보조 그룹 관리

사용자의 그룹 관계를 확인할 때는 `id UserName` 또는 `groups UserName`을 사용한다.

| 명령어 | 의미 |
|---|---|
| `useradd -g Group UserName` | 사용자의 **주 그룹** 지정 |
| `useradd -G Group1,Group2 UserName` | 사용자의 **보조 그룹** 지정 |
| `usermod -g Group UserName` | 주 그룹 변경 |
| `usermod -G Group1,Group2 UserName` | 보조 그룹 목록을 지정한 목록으로 변경 |
| `usermod -aG Group UserName` | 기존 보조 그룹을 유지하면서 그룹 추가 |
| `usermod -G "" UserName` | 보조 그룹 목록 제거 |

**`-g`는 주 그룹**, **`-G`는 보조 그룹**을 의미한다.

### wheel 그룹

CentOS/RHEL 계열에서 **wheel 그룹**은 관리자 권한 관리에 널리 사용된다.

사용자를 wheel 그룹에 추가하는 방법은 `usermod -aG wheel UserName`과 같다.

실제 `sudo` 사용 가능 여부는 `/etc/sudoers` 및 `/etc/sudoers.d/`의 정책에 의해 결정되므로 wheel 그룹에 속했다는 사실만으로 모든 시스템에서 동일한 sudo 정책이 보장되는 것은 아니다.

### gpasswd 명령어

**gpasswd**는 그룹을 중심으로 그룹 패스워드와 그룹 구성원을 관리한다.

형식: `gpasswd [옵션] GROUP`

| 옵션 | 설명 |
|---|---|
| `-a UserName` | 그룹에 사용자 추가 |
| `-d UserName` | 그룹에서 사용자 삭제 |
| `-M User1,User2,...` | 그룹 구성원 목록을 지정한 목록으로 설정 |
| `-r` | 그룹 패스워드 제거 |

| 관리 방식 | 대표 명령어 |
|---|---|
| **사용자 중심 관리** | `usermod` |
| **그룹 중심 관리** | `gpasswd` |

관련 파일은 **`/etc/group`**과 **`/etc/gshadow`**이다.

### 그룹을 이용한 권한 관리

그룹을 이용하면 여러 사용자에게 동일한 시스템 권한을 일괄적으로 적용할 수 있다.

예를 들어 관리자 그룹을 생성한 뒤 해당 그룹을 `/etc/sudoers.d/`의 정책에 등록하면 그룹 구성원에게 동일한 sudo 권한을 부여할 수 있다.

SSH 원격 접속 역시 `AllowGroups` 정책을 이용하여 특정 그룹에 속한 사용자만 허용하도록 구성할 수 있다.

| 설정 | 목적 |
|---|---|
| `/etc/sudoers.d/파일명` | 특정 그룹에 sudo 권한 부여 |
| `AllowGroups GROUP` | 특정 그룹에 속한 사용자만 SSH 접속 허용 |
| `systemctl restart sshd` | SSH 설정 변경 후 서비스 재시작 |

### 자주 사용하는 그룹 관리 예제

그룹 생성: `groupadd GroupName`

특정 GID로 그룹 생성: `groupadd -g 2000 GroupName`

시스템 그룹 생성: `groupadd -r GroupName`

그룹 GID 변경: `groupmod -g 3000 GroupName`

그룹 이름 변경: `groupmod -n NewGroupName OldGroupName`

그룹 삭제: `groupdel GroupName`

사용자를 보조 그룹에 추가: `usermod -aG GroupName UserName`

사용자의 그룹 정보 확인: `id UserName`

그룹에 사용자 추가: `gpasswd -a UserName GroupName`

그룹에서 사용자 삭제: `gpasswd -d UserName GroupName`

그룹 구성원을 일괄 지정: `gpasswd -M UserName1,UserName2 GroupName`

---

## 3. 패스워드 사용 제한 관리

### Password Aging 개념

**Password Aging**은 사용자의 패스워드를 일정 기간 동안만 사용할 수 있도록 제한하고 패스워드 변경을 강제하는 기능이다.

주요 관리 항목은 다음과 같다.

| 항목 | 의미 |
|---|---|
| **Last Change** | 마지막 패스워드 변경일 |
| **MIN** | 패스워드 변경 후 다시 변경할 수 있을 때까지의 최소 일수 |
| **MAX** | 패스워드를 사용할 수 있는 최대 일수 |
| **WARN** | 패스워드 만료 전 경고 기간 |
| **INACTIVE** | 패스워드 만료 후 계정 비활성화까지의 유예 기간 |
| **EXPIRE DATE** | 계정 자체의 만료일 |

### chage 명령어

**chage**는 사용자 패스워드의 Aging 정책을 변경하는 명령어이다.

형식: `chage [옵션] UserName`

| 옵션 | 설명 |
|---|---|
| `-d, --lastday LAST_DAY` | 마지막 패스워드 변경일 설정 |
| `-E, --expiredate EXPIRE_DATE` | 계정 만료일 설정 |
| `-I, --inactive INACTIVE` | 패스워드 만료 후 계정 비활성화까지의 기간 설정 |
| `-l, --list` | 현재 Password Aging 정보 확인 |
| `-m, --mindays MIN_DAYS` | 패스워드 변경 최소 일수 설정 |
| `-M, --maxdays MAX_DAYS` | 패스워드 최대 사용 일수 설정 |
| `-W, --warndays WARN_DAYS` | 패스워드 만료 전 경고 기간 설정 |
| `-h, --help` | 도움말 출력 |

### 패스워드 만료 정책

예를 들어 **MAX 30일 + WARN 7일** 정책을 적용하면 패스워드는 최대 30일까지 사용할 수 있으며 만료 7일 전부터 사용자에게 경고가 표시된다.

| 정책 | 의미 |
|---|---|
| `MIN=0` | 패스워드 변경 후 즉시 다시 변경 가능 |
| `MIN=7` | 패스워드 변경 후 7일 동안 다시 변경 불가 |
| `MAX=30` | 패스워드를 최대 30일 사용 |
| `WARN=7` | 만료 7일 전부터 경고 |
| `INACTIVE=30` | 패스워드 만료 후 30일의 유예 기간 |
| `EXPIRE DATE` | 지정된 날짜가 지나면 계정 자체가 만료 |

### 자주 사용하는 패스워드 관리 예제

현재 Password Aging 확인: `chage -l UserName`

패스워드 최소 변경 기간 설정: `chage -m 7 UserName`

패스워드 최대 사용 기간 설정: `chage -M 30 UserName`

패스워드 만료 경고 기간 설정: `chage -W 7 UserName`

MAX 30일 + WARN 7일 정책 설정: `chage -M 30 -W 7 UserName`

계정 만료일 설정: `chage -E 2026-12-31 UserName`

패스워드의 마지막 변경일 설정: `chage -d 0 UserName`

`chage -d 0 UserName`은 사용자가 다음 로그인 시 패스워드를 변경하도록 설정할 때 활용할 수 있다.

전역 패스워드 정책은 `/etc/login.defs`의 `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE` 등의 설정을 통해 관리할 수 있으며, 개별 사용자의 정책은 **`chage`**를 통해 `/etc/shadow`에 반영된다.

---

## 4. 패스워드 복잡도 관리

### PAM 설정

**PAM(Pluggable Authentication Modules)**은 Linux의 인증 기능을 모듈화하여 관리하는 체계이다.

패스워드 변경과 관련된 주요 PAM 설정 파일은 다음과 같다.

| 파일 | 설명 |
|---|---|
| `/etc/pam.d/system-auth` | 시스템 인증 관련 PAM 정책 |
| `/etc/pam.d/password-auth` | 패스워드 인증 관련 PAM 정책 |
| `/etc/security/pwquality.conf` | 패스워드 복잡도 정책 |

CentOS Stream 10에서는 인증 설정이 **authselect**에 의해 관리될 수 있으므로 PAM 파일을 직접 수정하기 전에 현재 시스템의 authselect 구성을 확인하는 것이 중요하다.

### pwquality.conf

`/etc/security/pwquality.conf`에서는 패스워드의 길이, 문자 종류, 반복 문자, 사용자 이름 포함 여부, 사전 단어 사용 여부 등을 설정할 수 있다.

### 주요 패스워드 복잡도 옵션

| 옵션 | 설명 |
|---|---|
| `minlen` | 최소 패스워드 길이 |
| `dcredit` | 숫자에 대한 요구 조건 |
| `ucredit` | 대문자에 대한 요구 조건 |
| `lcredit` | 소문자에 대한 요구 조건 |
| `ocredit` | 특수문자에 대한 요구 조건 |
| `minclass` | 필요한 문자 종류의 최소 개수 |
| `maxrepeat` | 동일 문자의 연속 반복 제한 |
| `maxclassrepeat` | 동일 문자 종류의 연속 반복 제한 |
| `reject_username` | 사용자 이름을 포함한 패스워드 거부 |
| `usercheck` | 사용자 이름 포함 여부 검사 |
| `usersubstr` | 사용자 이름의 일부 문자열 검사 |
| `dictcheck` | 사전 단어 포함 여부 검사 |
| `dictpath` | 패스워드 사전 파일 경로 |
| `maxsequence` | 연속된 문자 또는 숫자 사용 제한 |
| `difok` | 이전 패스워드와의 차이 수준 |
| `retry` | 패스워드 입력 재시도 횟수 |
| `enforcing` | 패스워드 품질 검사를 실제로 강제할지 여부 |
| `enforce_for_root` | root 패스워드에도 품질 검사 적용 |
| `local_users_only` | 로컬 사용자에 대해서만 검사 |

### 패스워드 복잡도 설정 예시

강의자료에서 제시한 보안 정책의 핵심은 다음과 같다.

| 설정 | 정책 예 |
|---|---|
| `minlen` | 최소 10~12자 |
| `dcredit` | 숫자 최소 1개 |
| `ucredit` | 대문자 최소 1개 |
| `lcredit` | 소문자 최소 1개 |
| `ocredit` | 특수문자 최소 1개 |
| `maxrepeat` | 동일 문자 반복 제한 |
| `maxsequence` | 연속 문자 사용 제한 |
| `dictcheck` | 사전 단어 사용 제한 |
| `reject_username` 또는 `usercheck` | 사용자 이름 포함 제한 |
| `retry` | 패스워드 입력 재시도 횟수 제한 |

`dcredit=-1`, `ucredit=-1`, `lcredit=-1`, `ocredit=-1`과 같이 음수 값을 사용하면 각각 해당 문자 종류를 **최소 1개 이상 요구**하는 방식으로 동작한다.

---

## 5. 사용자 및 그룹 관리 핵심 정리

### 사용자 관리 핵심 명령어

| 명령어 | 핵심 기능 | 주요 옵션 |
|---|---|---|
| **useradd** | 사용자 생성 | `-u`, `-g`, `-G`, `-c`, `-d`, `-m`, `-M`, `-r`, `-s`, `-e` |
| **usermod** | 사용자 정보 변경 | `-u`, `-g`, `-G`, `-aG`, `-c`, `-d`, `-l`, `-s`, `-e`, `-L`, `-U` |
| **userdel** | 사용자 삭제 | `-r`, `-f` |
| **passwd** | 사용자 패스워드 관리 | 패스워드 변경 및 설정 |
| **chage** | Password Aging 관리 | `-m`, `-M`, `-W`, `-I`, `-E`, `-l` |

### 그룹 관리 핵심 명령어

| 명령어 | 핵심 기능 | 주요 옵션 |
|---|---|---|
| **groupadd** | 그룹 생성 | `-g`, `-r`, `-o` |
| **groupmod** | 그룹 정보 변경 | `-g`, `-n`, `-o` |
| **groupdel** | 그룹 삭제 | 주요 옵션 없음 |
| **gpasswd** | 그룹 구성원 및 그룹 패스워드 관리 | `-a`, `-d`, `-M`, `-r` |

### 주요 파일 정리

| 파일 | 관리 대상 |
|---|---|
| **`/etc/passwd`** | 사용자 기본 정보 |
| **`/etc/shadow`** | 사용자 패스워드 및 Password Aging |
| **`/etc/group`** | 그룹 정보 |
| **`/etc/gshadow`** | 그룹 패스워드 및 그룹 관리자 정보 |
| **`/etc/skel`** | 신규 사용자 홈 디렉토리 기본 파일 |
| **`/etc/default/useradd`** | useradd 기본 설정 |
| **`/etc/login.defs`** | 사용자 및 패스워드 관련 전역 기본 정책 |
| **`/etc/pam.d/system-auth`** | PAM 인증 정책 |
| **`/etc/pam.d/password-auth`** | PAM 패스워드 인증 정책 |
| **`/etc/security/pwquality.conf`** | 패스워드 복잡도 정책 |

### 사용자 관리에서 반드시 구분할 옵션

| 구분 | 의미 |
|---|---|
| `useradd -g` | **주 그룹 지정** |
| `useradd -G` | **보조 그룹 지정** |
| `usermod -G` | 보조 그룹 목록을 **지정 목록으로 교체** |
| `usermod -aG` | 기존 보조 그룹을 유지하면서 **그룹 추가** |
| `userdel` | 계정 정보 삭제 |
| `userdel -r` | 계정 + 홈 디렉토리 + Mail Spool 삭제 |
| `usermod -L` | 패스워드 인증 잠금 |
| `usermod -U` | 패스워드 인증 잠금 해제 |
| `usermod -s /sbin/nologin` | 대화형 로그인 차단 |
| `usermod -e 0` | 계정 만료를 통한 로그인 차단 |
| `chage -m` | 패스워드 **최소 사용 기간** |
| `chage -M` | 패스워드 **최대 사용 기간** |
| `chage -W` | 패스워드 **만료 경고 기간** |
| `chage -I` | 패스워드 만료 후 **비활성화 유예 기간** |
| `chage -E` | **계정 만료일** |
| `chage -l` | Password Aging 정보 확인 |

### 핵심 명령어 암기

사용자 생성: `useradd UserName`

사용자 정보 변경: `usermod [옵션] UserName`

사용자 삭제: `userdel -r UserName`

그룹 생성: `groupadd GroupName`

그룹 정보 변경: `groupmod [옵션] GroupName`

그룹 삭제: `groupdel GroupName`

사용자 그룹 확인: `id UserName`

보조 그룹 추가: `usermod -aG GroupName UserName`

Password Aging 확인: `chage -l UserName`

패스워드 최대 사용 기간 30일 + 경고 7일: `chage -M 30 -W 7 UserName`

계정 만료일 설정: `chage -E YYYY-MM-DD UserName`

패스워드 복잡도 설정: `/etc/security/pwquality.conf`

### 강의자료의 주요 오탈자 및 내용 정정

| 원문에서 주의할 부분 | 정리 시 수정한 내용 |
|---|---|
| `usermod -d 0 UserName`으로 다음 로그인 시 패스워드 변경 | **`chage -d 0 UserName`**으로 수정 |
| `groupmod -aU UserName GroupName` | 일반적인 `groupmod`의 그룹 구성원 관리 방식으로 보지 않고 **`usermod -aG` 또는 `gpasswd -a`**로 정리 |
| SSH 사용자 생성 예에서 `useradd -G sshuser2`처럼 그룹/사용자 인수가 혼재 | **그룹을 먼저 생성하고 `useradd -G 그룹명 사용자명` 형식**으로 정리 |
| SSH 설정 삭제 경로에 `sshd-config.d` 표기 | **`/etc/ssh/sshd_config.d/`**로 정정 |
| `passwd --stdin` 중심의 사용자 생성 예제 | CentOS Stream 10에서의 일반적인 관리 방식에 맞춰 **`passwd` 또는 `chpasswd`** 중심으로 정리 |
| 예제의 특정 사용자명 및 서버명 | **UserName, GroupName 등 일반적인 명칭**으로 변경 |
| `MAX Change/MAX days` 등 혼동되는 표현 | **MAX AGE / Maximum number of days**로 의미를 명확하게 구분 |
| 패스워드 해시 설명에서 구버전 SHA 계열을 CentOS 10의 기본값처럼 표현 | **CentOS Stream 10의 기본 해시 방식은 yescrypt**로 정리 |