# 소프트웨어 관리

## 목차

1. [RPM 패키지 관리](#1-rpm-패키지-관리)
	- [RPM 개요](#rpm-개요)
	- [RPM 패키지 파일명 구조](#rpm-패키지-파일명-구조)
	- [RPM 패키지 설치 및 업데이트](#rpm-패키지-설치-및-업데이트)
	- [RPM 패키지 조회 및 정보 확인](#rpm-패키지-조회-및-정보-확인)
	- [RPM 패키지 삭제](#rpm-패키지-삭제)
	- [RPM 패키지 검증 및 서명](#rpm-패키지-검증-및-서명)
	- [RPM 패키지 추출](#rpm-패키지-추출)
	- [자주 사용하는 RPM 예제](#자주-사용하는-rpm-예제)

2. [DNF 패키지 관리](#2-dnf-패키지-관리)
	- [DNF 개요](#dnf-개요)
	- [CentOS Stream 10의 DNF와 YUM 관계](#centos-stream-10의-dnf와-yum-관계)
	- [패키지 검색 및 확인](#패키지-검색-및-확인)
	- [패키지 설치·업데이트·삭제](#패키지-설치업데이트삭제)
	- [DNF 주요 명령어](#dnf-주요-명령어)
	- [DNF 주요 옵션](#dnf-주요-옵션)
	- [패키지 다운로드 및 캐시 관리](#패키지-다운로드-및-캐시-관리)
	- [DNF 트랜잭션 이력 관리](#dnf-트랜잭션-이력-관리)
	- [자주 사용하는 DNF 예제](#자주-사용하는-dnf-예제)

3. [DNF 저장소 관리](#3-dnf-저장소-관리)
	- [DNF Repository 개념](#dnf-repository-개념)
	- [저장소 목록 확인](#저장소-목록-확인)
	- [저장소 활성화 및 비활성화](#저장소-활성화-및-비활성화)
	- [Repository 설정 파일](#repository-설정-파일)
	- [CD/DVD 기반 로컬 저장소](#cddvd-기반-로컬-저장소)
	- [HTTP 기반 DNF 저장소](#http-기반-dnf-저장소)
	- [자주 사용하는 Repository 예제](#자주-사용하는-repository-예제)

4. [그룹 패키지 관리](#4-그룹-패키지-관리)
	- [그룹 패키지 개념](#그룹-패키지-개념)
	- [그룹 목록 및 정보 확인](#그룹-목록-및-정보-확인)
	- [그룹 패키지 설치 및 삭제](#그룹-패키지-설치-및-삭제)
	- [Development Tools](#development-tools)
	- [자주 사용하는 그룹 패키지 예제](#자주-사용하는-그룹-패키지-예제)

5. [운영체제 전체 업데이트](#5-운영체제-전체-업데이트)
	- [업데이트 확인](#업데이트-확인)
	- [전체 패키지 업데이트](#전체-패키지-업데이트)
	- [커널 업데이트와 재부팅](#커널-업데이트와-재부팅)
	- [업데이트 전후 확인](#업데이트-전후-확인)
	- [자주 사용하는 업데이트 예제](#자주-사용하는-업데이트-예제)

6. [Flatpak 애플리케이션 관리](#6-flatpak-애플리케이션-관리)
	- [Flatpak 개요](#flatpak-개요)
	- [Flatpak 저장소 관리](#flatpak-저장소-관리)
	- [애플리케이션 관리](#애플리케이션-관리)
	- [업데이트 및 검색](#업데이트-및-검색)
	- [권한 관리](#권한-관리)
	- [런타임 관리](#런타임-관리)
	- [사용자 설치와 시스템 설치](#사용자-설치와-시스템-설치)
	- [자주 사용하는 Flatpak 예제](#자주-사용하는-flatpak-예제)

7. [RPM 패키지 제작](#7-rpm-패키지-제작)
	- [RPM 패키지 제작 개요](#rpm-패키지-제작-개요)
	- [RPM 빌드 디렉터리](#rpm-빌드-디렉터리)
	- [소스 아카이브 생성](#소스-아카이브-생성)
	- [SPEC 파일](#spec-파일)
	- [SPEC 파일 검사](#spec-파일-검사)
	- [RPM 패키지 빌드](#rpm-패키지-빌드)
	- [패키지 설치 및 테스트](#패키지-설치-및-테스트)
	- [GPG 키 생성 및 패키지 서명](#gpg-키-생성-및-패키지-서명)
	- [RPM Repository 구축](#rpm-repository-구축)
	- [자주 사용하는 RPM 제작 예제](#자주-사용하는-rpm-제작-예제)

---

# 1. RPM 패키지 관리

### RPM 개요

**RPM(Red Hat Package Manager)**은 RPM 형식의 패키지를 설치·삭제·조회·검증하기 위한 패키지 관리 시스템이다.

RPM의 특징은 다음과 같다.

| 항목 | 설명 |
|---|---|
| 패키지 형식 | **`.rpm`** |
| 주요 대상 | RHEL, Fedora, CentOS, Rocky Linux 등 RPM 계열 |
| 장점 | 소스 코드를 직접 컴파일하지 않고 패키지를 설치할 수 있음 |
| 한계 | 패키지 간 **의존성(dependency)**을 직접 해결해야 하는 경우가 있음 |
| 주요 명령 | `rpm` |
| 저장소 연계 | 저장소 기반 의존성 해결은 **DNF**가 담당 |

RPM은 개별 패키지 파일을 직접 관리하는 저수준 패키지 관리 도구이고, **DNF는 저장소와 의존성 해결을 포함한 고수준 패키지 관리 도구**이다.

### RPM 패키지 파일명 구조

RPM 패키지는 일반적으로 다음과 같은 형식을 사용한다.

`name-version-release.architecture.rpm`

| 구성 요소 | 설명 |
|---|---|
| name | 패키지 이름 |
| version | 패키지 버전 |
| release | 동일 버전에 대한 패키지 릴리스 번호 |
| architecture | 패키지가 대상으로 하는 CPU 또는 범용 아키텍처 |
| rpm | RPM 패키지 확장자 |

주요 아키텍처는 다음과 같다.

| 아키텍처 | 설명 |
|---|---|
| `x86_64` | 64비트 x86 계열 |
| `i386` | 32비트 Intel 호환 계열 |
| `i586` | Pentium 계열 32비트 |
| `i686` | 686 계열 32비트 |
| `noarch` | 특정 CPU 아키텍처에 종속되지 않는 패키지 |
| `src` | 소스 RPM |

### RPM 패키지 설치 및 업데이트

| 명령어 | 설명 |
|---|---|
| `rpm -ivh PKG.rpm` | 기존 패키지를 유지하면서 패키지 설치 |
| `rpm -Fvh PKG.rpm` | 기존에 설치된 패키지가 있는 경우에만 업데이트 |
| `rpm -Uvh PKG.rpm` | 설치되어 있지 않으면 설치하고, 설치되어 있으면 업데이트 |
| `rpm -ivh --nodeps PKG.rpm` | 의존성 검사를 수행하지 않고 설치 |
| `rpm -Uvh --force PKG.rpm` | 충돌 상황에서도 강제로 처리 |
| `rpm -e PKG` | 패키지 삭제 |
| `rpm -e --nodeps PKG` | 의존성 검사를 하지 않고 삭제 |

**`--nodeps`와 `--force`는 일반적인 패키지 관리에서는 신중하게 사용해야 한다.** 의존성 관계를 무시하면 다른 패키지가 정상적으로 동작하지 않을 수 있다.

### RPM 패키지 조회 및 정보 확인

| 명령어 | 설명 |
|---|---|
| `rpm -qa` | 설치된 모든 패키지 조회 |
| `rpm -qa \| grep PKG` | 특정 패키지가 설치되어 있는지 검색 |
| `rpm -q PKG` | 특정 패키지의 설치 여부 확인 |
| `rpm -qi PKG` | 설치된 패키지의 상세 정보 확인 |
| `rpm -ql PKG` | 패키지가 설치한 파일 목록 확인 |
| `rpm -qc PKG` | 패키지가 설치한 설정 파일 목록 확인 |
| `rpm -qf FILE` | 지정한 파일이 어느 패키지에 속하는지 확인 |
| `rpm -qi -p PKG.rpm` | 설치하지 않은 RPM 파일의 상세 정보 확인 |
| `rpm -ql -p PKG.rpm` | 설치하지 않은 RPM 파일의 파일 목록 확인 |
| `rpm -q --scripts PKG` | 설치·삭제 전후에 실행되는 스크립트 확인 |

### RPM 질의 옵션

| 옵션 | 설명 |
|---|---|
| `-q`, `--query` | 패키지 정보 질의 |
| `-a`, `--all` | 모든 설치 패키지를 대상으로 질의 |
| `-f`, `--file` | 특정 파일을 포함하는 패키지 질의 |
| `-p`, `--package` | 설치된 패키지가 아닌 RPM 파일을 대상으로 질의 |
| `-l`, `--list` | 패키지에 포함된 파일 목록 출력 |
| `-c`, `--configfiles` | 설정 파일 목록 출력 |
| `-d`, `--docfiles` | 문서 파일 목록 출력 |
| `-s`, `--state` | 파일 상태 출력 |

### RPM 패키지 검증 및 서명

RPM은 설치된 파일의 상태와 패키지 서명을 확인할 수 있다.

| 명령어 | 설명 |
|---|---|
| `rpm -V PKG` | 설치된 패키지 파일의 변경 여부 검증 |
| `rpm -Va` | 설치된 모든 패키지 검증 |
| `rpm -Vf FILE` | 특정 파일이 포함된 패키지 검증 |
| `rpm -K PKG.rpm` | RPM 패키지 서명 검증 |
| `rpm --checksig PKG.rpm` | RPM 패키지 서명 검증 |
| `rpm --import KEYFILE` | 공개 GPG 키 가져오기 |

### RPM 데이터베이스 관리

| 명령어 | 설명 |
|---|---|
| `rpm --initdb` | RPM 데이터베이스 초기화 |
| `rpm --rebuilddb` | 설치된 패키지 헤더를 이용하여 RPM 데이터베이스 재구축 |

### RPM 패키지 추출

**`rpm2cpio`**는 RPM 패키지를 CPIO 아카이브로 변환한다.

| 명령어 | 설명 |
|---|---|
| `rpm2cpio PKG.rpm \| cpio -tv` | 패키지 내부 파일 목록 확인 |
| `rpm2cpio PKG.rpm \| cpio -idv` | 패키지 전체 파일 추출 |
| `rpm2cpio PKG.rpm \| cpio -idv "./경로/파일"` | 특정 파일만 추출 |

`cpio` 주요 옵션:

| 옵션 | 설명 |
|---|---|
| `-i` | 표준 입력에서 아카이브 추출 |
| `-d` | 필요한 하위 디렉터리 생성 |
| `-v` | 처리되는 파일을 상세하게 표시 |
| `-t` | 파일 목록만 확인 |

### 자주 사용하는 RPM 예제

`rpm -qa | grep PKG` — 특정 패키지의 설치 여부를 빠르게 확인한다.

`rpm -qi PKG` — 설치된 패키지의 버전·릴리스·아키텍처 등의 상세 정보를 확인한다.

`rpm -ql PKG` — 특정 패키지가 설치한 파일의 위치를 확인한다.

`rpm -qf /usr/bin/FILE` — 특정 실행 파일이 어떤 RPM 패키지에 의해 설치되었는지 확인한다.

`rpm -qi -p PKG.rpm` — 설치하지 않은 RPM 파일의 정보를 확인한다.

`rpm -Uvh PKG.rpm` — 로컬 RPM 파일을 설치하거나 기존 버전을 업데이트한다.

`rpm -e PKG` — 설치된 패키지를 삭제한다.

`rpm2cpio PKG.rpm | cpio -tv` — RPM 내부 파일 목록을 확인한다.

`rpm2cpio PKG.rpm | cpio -idv` — RPM 내부 파일을 현재 디렉터리에 추출한다.

---

# 2. DNF 패키지 관리

### DNF 개요

**DNF(Dandified YUM)**는 RPM 기반 Linux에서 패키지를 관리하는 고수준 패키지 관리자이다.

RPM이 개별 RPM 파일 중심으로 동작하는 것과 달리 DNF는 **원격 저장소의 패키지 정보와 의존성 관계를 이용하여 패키지를 자동으로 설치·업데이트·삭제**할 수 있다.

| 기능 | RPM | DNF |
|---|---|---|
| 로컬 RPM 설치 | 가능 | 가능 |
| 패키지 조회 | 가능 | 가능 |
| 패키지 삭제 | 가능 | 가능 |
| 의존성 자동 해결 | 제한적 | **지원** |
| 저장소 이용 | 직접 지정 필요 | **자동 검색 및 이용** |
| 다수 시스템 관리 | 상대적으로 불편 | **효율적** |

### CentOS Stream 10의 DNF와 YUM 관계

CentOS Stream 9 및 10에서는 **DNF가 기본 패키지 관리 도구**이다.

자료의 CentOS Stream 10 환경에서는 다음과 같은 관계를 확인할 수 있다.

| 명령 | 관계 |
|---|---|
| `dnf` | DNF 패키지 관리자 |
| `dnf-3` | DNF의 실행 구현 |
| `yum` | DNF 호환 명령 |
| `dnf4` | DNF 실행 파일을 가리키는 명령 |

따라서 **CentOS Stream 10에서는 새로운 명령을 작성할 때 DNF를 기준으로 사용하는 것이 적절하다.**

### 패키지 검색 및 확인

| 명령어 | 설명 |
|---|---|
| `dnf list` | 설치 및 사용 가능한 패키지 목록 확인 |
| `dnf list all` | 전체 패키지 목록 확인 |
| `dnf list installed` | 설치된 패키지 목록 확인 |
| `dnf list available` | 설치 가능한 패키지 목록 확인 |
| `dnf list recent` | 최근 추가된 패키지 확인 |
| `dnf info PKG` | 패키지 상세 정보 확인 |
| `dnf info installed PKG` | 설치된 패키지의 상세 정보 확인 |
| `dnf search KEYWORD` | 패키지 이름 및 설명 등을 검색 |
| `dnf provides '*/FILE'` | 특정 파일을 제공하는 패키지 검색 |
| `dnf repoquery PKG` | 저장소의 패키지 질의 |

### 패키지 설치·업데이트·삭제

| 명령어 | 설명 |
|---|---|
| `dnf install PKG` | 패키지 설치 및 의존성 해결 |
| `dnf -y install PKG` | 설치 확인 질문에 자동으로 `yes` 응답 |
| `dnf localinstall PKG.rpm` | 로컬 RPM 파일 설치 및 의존성 해결 |
| `dnf update` | 전체 패키지 업데이트 |
| `dnf update PKG` | 특정 패키지 업데이트 |
| `dnf upgrade PKG` | 패키지 업그레이드 |
| `dnf reinstall PKG` | 패키지 재설치 |
| `dnf remove PKG` | 패키지 삭제 |
| `dnf erase PKG` | 패키지 삭제 |
| `dnf autoremove` | 더 이상 필요하지 않은 의존 패키지 제거 |
| `dnf downgrade PKG` | 패키지를 이전 버전으로 다운그레이드 |
| `dnf distro-sync` | 설치된 패키지를 저장소의 최신 버전에 맞춰 동기화 |

### DNF 주요 명령어

| 명령어 | 설명 |
|---|---|
| `dnf check` | 패키지 데이터베이스 문제 확인 |
| `dnf check-update` | 업데이트 가능한 패키지 확인 |
| `dnf clean` | 캐시 데이터 삭제 |
| `dnf makecache` | 저장소 메타데이터 캐시 생성 |
| `dnf history` | 패키지 트랜잭션 이력 확인 |
| `dnf history info ID` | 특정 트랜잭션 상세 정보 확인 |
| `dnf history undo ID` | 특정 트랜잭션 실행 취소 |
| `dnf history redo ID` | 특정 트랜잭션 다시 실행 |
| `dnf history rollback ID` | 지정한 트랜잭션 시점 이후의 변경을 되돌림 |
| `dnf repolist` | 활성화된 저장소 목록 확인 |
| `dnf repolist all` | 활성·비활성 저장소 전체 확인 |
| `dnf provides '*/FILE'` | 파일을 제공하는 패키지 검색 |
| `dnf download PKG` | 패키지를 설치하지 않고 다운로드 |
| `dnf group` | 패키지 그룹 관리 |

### DNF 주요 옵션

| 옵션 | 설명 |
|---|---|
| `-y`, `--assumeyes` | 모든 질문에 자동으로 `yes` 응답 |
| `-q`, `--quiet` | 출력 최소화 |
| `-v`, `--verbose` | 상세 출력 |
| `--version` | DNF 버전 표시 |
| `-C`, `--cacheonly` | 시스템 캐시만 사용 |
| `--refresh` | 메타데이터를 만료된 것으로 처리하고 갱신 |
| `--skip-broken` | 의존성 문제 패키지를 건너뜀 |
| `--allowerasing` | 의존성 해결을 위해 설치된 패키지 제거 허용 |
| `--best` | 가능한 최상의 패키지 버전 사용 |
| `--nobest` | 최상의 후보만 사용하지 않음 |
| `--nodocs` | 문서 파일 설치 제외 |
| `--noplugins` | 플러그인 사용 안 함 |
| `--enablerepo=REPO` | 특정 저장소를 임시 활성화 |
| `--disablerepo=REPO` | 특정 저장소를 임시 비활성화 |
| `--repo=REPO` | 지정한 저장소만 사용 |
| `-x PKG`, `--exclude PKG` | 특정 패키지 제외 |
| `--downloadonly` | 패키지를 다운로드만 하고 설치하지 않음 |
| `--downloaddir=DIR` | 다운로드한 패키지를 지정 디렉터리에 저장 |
| `--nogpgcheck` | GPG 서명 검사를 비활성화 |

**`--nogpgcheck`는 패키지 무결성과 출처 검증을 약화시키므로 일반적인 운영 환경에서는 사용을 피하는 것이 좋다.**

### 패키지 다운로드 및 캐시 관리

**`dnf download`**는 패키지를 설치하지 않고 RPM 파일을 다운로드한다.

**`dnf install --downloadonly`**는 의존성을 해결하면서 패키지를 다운로드하지만 설치하지 않는다.

| 명령어 | 용도 |
|---|---|
| `dnf download PKG` | 지정 패키지 다운로드 |
| `dnf install --downloadonly PKG` | 패키지 및 필요한 의존 패키지 다운로드 |
| `dnf install --downloadonly --downloaddir=/DIR PKG` | 지정 디렉터리에 패키지 다운로드 |
| `dnf clean metadata` | 저장소 메타데이터 캐시 삭제 |
| `dnf clean packages` | 다운로드된 RPM 패키지 캐시 삭제 |
| `dnf clean all` | 메타데이터·패키지·DB 캐시 등을 정리 |

### DNF 트랜잭션 이력 관리

DNF는 패키지 설치·삭제·업데이트 등의 작업을 **Transaction ID**로 관리한다.

| 명령어 | 의미 |
|---|---|
| `dnf history` | 트랜잭션 목록 확인 |
| `dnf history info ID` | 특정 트랜잭션 상세 확인 |
| `dnf history undo ID` | 특정 트랜잭션을 반대로 실행 |
| `dnf history redo ID` | 특정 트랜잭션을 다시 실행 |
| `dnf history rollback ID` | 지정한 시점 이후의 트랜잭션을 되돌림 |

트랜잭션을 되돌릴 때는 패키지 의존성이나 이후 변경 사항을 함께 고려해야 한다. 자료에서는 실제 운영 환경의 복구 수단으로 **VM Snapshot 또는 LVM Snapshot을 더 권장**하고 있다.

### 자주 사용하는 DNF 예제

`dnf list installed` — 현재 시스템에 설치된 패키지를 확인한다.

`dnf list available` — 저장소에서 설치할 수 있는 패키지를 확인한다.

`dnf info PKG` — 패키지 버전, 릴리스, 아키텍처, 저장소 등의 상세 정보를 확인한다.

`dnf search KEYWORD` — 패키지 이름이나 설명을 검색한다.

`dnf install PKG` — 패키지와 필요한 의존성을 함께 설치한다.

`dnf remove PKG` — 패키지를 삭제하고 더 이상 필요하지 않은 의존 패키지를 함께 정리할 수 있다.

`dnf provides '*/FILE'` — 특정 파일을 제공하는 패키지를 찾는다.

`dnf download PKG` — RPM 파일을 별도로 다운로드한다.

`dnf install PKG --downloadonly --downloaddir=/DIR` — 패키지를 설치하지 않고 필요한 RPM을 지정 디렉터리에 다운로드한다.

`dnf history undo ID` — 특정 패키지 트랜잭션을 취소한다.

---

# 3. DNF 저장소 관리

### DNF Repository 개념

**DNF Repository**는 RPM 패키지와 패키지 메타데이터를 제공하는 저장소이다.

자료에서는 다음과 같은 저장소 형태를 다룬다.

| 저장소 형태 | `baseurl` 예 |
|---|---|
| HTTP | `http://SERVER/PATH` |
| FTP | `ftp://SERVER/PATH` |
| 로컬 CD/DVD | `file:///mnt/cdrom/PATH` |
| 로컬 파일 시스템 | `file:///PATH` |

Repository 서버는 패키지를 제공하고, DNF 클라이언트는 `/etc/yum.repos.d/`의 `.repo` 설정 파일을 이용하여 저장소에 접근한다.

### 저장소 목록 확인

| 명령어 | 설명 |
|---|---|
| `dnf repolist` | 활성화된 저장소 목록 확인 |
| `dnf repolist all` | 활성·비활성 저장소 전체 확인 |
| `dnf repolist enabled` | 활성화된 저장소만 확인 |
| `dnf repolist disabled` | 비활성화된 저장소만 확인 |

### 저장소 활성화 및 비활성화

**DNF config-manager**를 이용하여 저장소 상태를 변경할 수 있다.

| 명령어 | 설명 |
|---|---|
| `dnf config-manager --enable REPO` | 저장소 활성화 |
| `dnf config-manager --disable REPO` | 저장소 비활성화 |
| `dnf config-manager --add-repo URL` | 저장소 설정 추가 |
| `dnf --enablerepo=REPO CMD` | 특정 명령에서만 저장소 임시 활성화 |
| `dnf --disablerepo=REPO CMD` | 특정 명령에서만 저장소 임시 비활성화 |

### Repository 설정 파일

DNF 저장소 설정은 일반적으로 다음 디렉터리의 `.repo` 파일에서 관리한다.

`/etc/yum.repos.d/*.repo`

일반적인 설정 항목은 다음과 같다.

| 항목 | 설명 |
|---|---|
| `[repo-id]` | 저장소 식별자 |
| `name` | 저장소 이름 |
| `baseurl` | 패키지 저장소 위치 |
| `enabled` | 저장소 활성화 여부 |
| `gpgcheck` | RPM 패키지 GPG 서명 검사 여부 |
| `gpgkey` | GPG 공개 키 위치 |
| `metadata_expire` | 저장소 메타데이터 만료 시간 |

### CD/DVD 기반 로컬 저장소

CentOS Stream 설치 ISO를 마운트하면 ISO 내부의 **BaseOS**와 **AppStream** 패키지를 로컬 저장소로 사용할 수 있다.

자료에서는 `/run/media/root/` 아래에 자동 마운트된 ISO를 이용하거나 `/mnt/cdrom`에 직접 마운트하는 방법을 사용한다.

대표적인 구조:

`file:///run/media/root/CentOS-Stream-10-BaseOS-x86_64/BaseOS`

`file:///run/media/root/CentOS-Stream-10-BaseOS-x86_64/AppStream`

### HTTP 기반 DNF 저장소

HTTP 기반 Repository는 다음과 같은 구조로 구성할 수 있다.

| 구성 요소 | 역할 |
|---|---|
| HTTP 서버 | RPM 패키지 제공 |
| `/var/www/html/CD` | 패키지 저장 디렉터리 |
| `createrepo` | Repository 메타데이터 생성 |
| `.repo` 파일 | 클라이언트의 Repository 설정 |
| `baseurl` | HTTP Repository 위치 |

Repository 메타데이터를 생성할 때는 **`createrepo` 또는 해당 환경에서 제공되는 `createrepo_c` 계열 도구**를 사용한다.

### 자주 사용하는 Repository 예제

`dnf repolist` — 현재 활성화된 Repository를 확인한다.

`dnf repolist all` — 모든 Repository의 활성화 상태를 확인한다.

`dnf config-manager --disable REPO` — Repository를 비활성화한다.

`dnf config-manager --enable REPO` — Repository를 활성화한다.

`dnf config-manager --add-repo URL` — 원격 Repository 설정을 추가한다.

`dnf --disablerepo=REPO install PKG` — 특정 Repository를 제외하고 패키지를 설치한다.

`dnf --enablerepo=REPO install PKG` — 특정 Repository를 임시로 활성화하여 패키지를 설치한다.

---

# 4. 그룹 패키지 관리

### 그룹 패키지 개념

**패키지 그룹(Group)**은 특정 목적에 필요한 여러 패키지를 하나의 논리적인 단위로 묶어 관리하는 기능이다.

예를 들어 개발 환경을 구성할 때 개별 컴파일러와 라이브러리를 하나씩 설치하는 대신 **Development Tools** 그룹을 설치할 수 있다.

### 그룹 목록 및 정보 확인

| 명령어 | 설명 |
|---|---|
| `dnf group list` | 사용 가능한 그룹 목록 확인 |
| `dnf group list hidden` | 숨겨진 그룹까지 확인 |
| `dnf group info GROUP` | 그룹 상세 정보 확인 |
| `dnf group list --installed` | 설치된 그룹 확인 |
| `dnf group list --available` | 설치 가능한 그룹 확인 |

그룹 정보에는 일반적으로 다음과 같은 패키지 분류가 포함된다.

| 구분 | 의미 |
|---|---|
| Mandatory Packages | 그룹 구성에 필수적인 패키지 |
| Default Packages | 기본적으로 설치되는 패키지 |
| Optional Packages | 필요할 때 선택적으로 설치하는 패키지 |

### 그룹 패키지 설치 및 삭제

| 명령어 | 설명 |
|---|---|
| `dnf group install 'GROUP'` | 그룹 설치 |
| `dnf group install @GROUP` | 그룹 ID 형식으로 그룹 설치 |
| `dnf group update 'GROUP'` | 그룹 업데이트 |
| `dnf group remove 'GROUP'` | 그룹 삭제 |

그룹 이름에 공백이 포함되어 있으면 **작은따옴표로 묶어 사용하는 것이 안전하다.**

### Development Tools

**Development Tools** 그룹은 소프트웨어 개발 및 컴파일에 필요한 기본 도구를 묶은 그룹이다.

자료에서 주요 구성 요소로 제시된 패키지는 다음과 같다.

| 패키지 | 역할 |
|---|---|
| `gcc` | C 컴파일러 |
| `gcc-c++` | C++ 컴파일러 |
| `gdb` | 디버거 |
| `make` | 빌드 자동화 도구 |
| `autoconf` | 빌드 설정 생성 도구 |
| `automake` | Makefile 생성 지원 |
| `binutils` | 바이너리 도구 모음 |
| `flex` | 어휘 분석기 생성 도구 |
| `bison` | 파서 생성 도구 |
| `glibc-devel` | 개발용 GNU C 라이브러리 |
| `libtool` | 라이브러리 빌드 지원 |
| `pkgconf` | 컴파일 및 링크 정보 관리 |
| `rpm-build` | RPM 패키지 빌드 |
| `rpm-sign` | RPM 패키지 서명 |
| `strace` | 시스템 호출 추적 |

### 자주 사용하는 그룹 패키지 예제

`dnf group list` — 설치 가능한 그룹을 확인한다.

`dnf group info 'Development Tools'` — 개발 도구 그룹의 구성 패키지를 확인한다.

`dnf group install -y 'Development Tools'` — 개발 환경을 한 번에 설치한다.

`dnf group remove -y 'Development Tools'` — 해당 그룹을 제거한다.

---

# 5. 운영체제 전체 업데이트

### 업데이트 확인

전체 업데이트 전에 **`dnf check-update`**를 이용하여 업데이트 가능한 패키지를 확인하는 것이 중요하다.

| 명령어 | 설명 |
|---|---|
| `dnf check-update` | 업데이트 가능한 패키지 확인 |
| `dnf list recent` | 최근 추가된 패키지 확인 |
| `dnf history` | 패키지 변경 이력 확인 |

업데이트 목록을 파일로 저장하려면 `tee`와 함께 사용할 수 있다.

`dnf check-update 2>&1 | tee pkg.list`

### 전체 패키지 업데이트

전체 시스템의 패키지를 업데이트할 때는 다음 명령을 사용한다.

`dnf -y update`

특정 패키지만 업데이트할 경우:

`dnf -y update PKG`

업데이트에는 의존성에 따라 여러 패키지가 함께 변경될 수 있으므로 실제 적용 전에 변경 목록을 확인하는 것이 좋다.

### 커널 업데이트와 재부팅

운영체제 전체 업데이트에는 **커널 업데이트가 포함될 수 있다.**

커널이 새 버전으로 설치되더라도 현재 실행 중인 커널은 즉시 변경되지 않는다. 새 커널을 사용하려면 일반적으로 **재부팅**이 필요하다.

| 명령어 | 설명 |
|---|---|
| `uname -a` | 현재 실행 중인 커널 및 시스템 정보 확인 |
| `uname -sr` | 커널 이름과 버전 확인 |
| `cat /etc/redhat-release` | 배포판 버전 확인 |
| `reboot` | 시스템 재부팅 |

자료에서는 **커널 또는 드라이버 업데이트가 포함된 경우 재부팅을 권장**한다.

### 업데이트 전후 확인

업데이트 전:

`uname -a`

`cat /etc/redhat-release`

`dnf check-update`

업데이트:

`dnf -y update`

필요한 경우:

`reboot`

업데이트 후:

`uname -a`

`cat /etc/redhat-release`

### 자주 사용하는 업데이트 예제

`dnf check-update` — 업데이트 가능한 패키지를 확인한다.

`dnf -y update` — 시스템의 패키지를 전체 업데이트한다.

`uname -a` — 업데이트 전후 커널 버전을 비교한다.

`reboot` — 새로 설치된 커널을 적용하기 위해 시스템을 재부팅한다.

---

# 6. Flatpak 애플리케이션 관리

### Flatpak 개요

**Flatpak**은 Linux 데스크톱 애플리케이션을 **샌드박스 환경**에서 배포하고 관리하는 시스템이다.

RPM/DNF가 시스템 구성 요소와 일반적인 시스템 패키지를 관리하는 데 적합하다면, Flatpak은 **데스크톱 애플리케이션을 운영체제 패키지와 분리하여 관리**하는 목적을 가진다.

자료에서 제시한 관리 방향은 다음과 같다.

| 대상 | 권장 관리 방식 |
|---|---|
| 시스템 구성 요소 | **RPM + DNF** |
| 서버 애플리케이션 | 컨테이너 |
| 데스크톱 애플리케이션 | **Flatpak** |
| 개발 환경 | 컨테이너 + 데스크톱 애플리케이션 |

### Flatpak 런타임

Flatpak 애플리케이션은 운영체제의 라이브러리에 직접 의존하는 대신 **Runtime**을 사용할 수 있다.

Runtime은 여러 애플리케이션이 공유할 수 있는 시스템 수준 라이브러리와 파일을 제공한다.

이를 통해 애플리케이션과 시스템 구성 요소를 분리하고 Runtime의 보안 업데이트를 독립적으로 관리할 수 있다.

### Flatpak 저장소 관리

| 명령어 | 설명 |
|---|---|
| `flatpak remotes` | 저장소 목록 확인 |
| `flatpak remotes --show-details` | 저장소 상세 정보 확인 |
| `flatpak remote-info REMOTE APP_ID` | 특정 애플리케이션의 저장소 정보 확인 |
| `flatpak remote-add --if-not-exists REMOTE URL` | 저장소 추가 |
| `flatpak remote-delete REMOTE` | 저장소 삭제 |
| `flatpak remote-ls REMOTE` | 저장소의 콘텐츠 목록 확인 |
| `flatpak remote-ls REMOTE --app` | 저장소의 애플리케이션 목록 확인 |

대표적인 저장소로 자료에서는 **RHEL, Flathub, Fedora**를 제시한다.

### 애플리케이션 관리

| 명령어 | 설명 |
|---|---|
| `flatpak list` | 설치된 전체 Flatpak 객체 확인 |
| `flatpak list --app` | 설치된 애플리케이션만 확인 |
| `flatpak list --runtime` | 설치된 Runtime만 확인 |
| `flatpak install REMOTE APP_ID` | 애플리케이션 설치 |
| `flatpak install --user REMOTE APP_ID` | 현재 사용자 영역에 설치 |
| `flatpak uninstall APP_ID` | 애플리케이션 삭제 |
| `flatpak uninstall --unused` | 사용하지 않는 Runtime 및 의존 객체 정리 |
| `flatpak run APP_ID` | 애플리케이션 실행 |

### Flatpak 식별자

Flatpak 애플리케이션은 일반적으로 다음과 같은 고유 식별자를 사용한다.

`domain.organization.Application`

보다 구체적인 객체 식별자는 다음 형식을 사용할 수 있다.

`Application-ID/architecture/branch`

| 구성 요소 | 의미 |
|---|---|
| Application ID | 애플리케이션 고유 식별자 |
| architecture | CPU 아키텍처 |
| branch | 애플리케이션의 분기 또는 버전 계열 |

### 업데이트 및 검색

| 명령어 | 설명 |
|---|---|
| `flatpak remote-ls --updates` | 업데이트 가능한 객체 확인 |
| `flatpak update` | 전체 Flatpak 객체 업데이트 |
| `flatpak update APP_ID` | 특정 애플리케이션 업데이트 |
| `flatpak history` | Flatpak 변경 이력 확인 |
| `flatpak search KEYWORD` | 애플리케이션 및 Runtime 검색 |
| `flatpak info APP_ID` | 설치 정보 확인 |
| `flatpak info --show-commit APP_ID` | Commit 정보 포함 상세 확인 |

### 권한 관리

Flatpak 애플리케이션은 샌드박스에서 실행되므로 애플리케이션이 사용할 수 있는 자원에 제한이 있다.

| 명령어 | 설명 |
|---|---|
| `flatpak info --show-permissions APP_ID` | 애플리케이션 권한 확인 |
| `flatpak override APP_ID --filesystem=host` | 전체 파일 시스템 접근 허용 |
| `flatpak override APP_ID --filesystem=/DIR` | 특정 디렉터리 접근 허용 |
| `flatpak override APP_ID --unshare=network` | 네트워크 공유 제거 |
| `flatpak override --reset APP_ID` | 애플리케이션 권한 설정 초기화 |

**샌드박스 권한을 과도하게 확대하면 Flatpak의 격리 효과가 감소하므로 필요한 범위에서만 override를 사용하는 것이 바람직하다.**

### 런타임 관리

| 명령어 | 설명 |
|---|---|
| `flatpak list --runtime` | 설치된 Runtime 목록 확인 |
| `flatpak uninstall --unused` | 사용하지 않는 Runtime 정리 |
| `flatpak info RUNTIME` | Runtime 상세 정보 확인 |

### 사용자 설치와 시스템 설치

Flatpak은 **시스템 영역**과 **사용자 영역**으로 나누어 설치할 수 있다.

| 구분 | 주요 위치 |
|---|---|
| 시스템 영역 | `/var/lib/flatpak` |
| 사용자 영역 | `~/.local/share/flatpak` |

사용자 영역 설치:

`flatpak install --user REMOTE APP_ID`

### 자주 사용하는 Flatpak 예제

`flatpak remotes` — 등록된 Flatpak 저장소를 확인한다.

`flatpak search KEYWORD` — 원하는 애플리케이션을 검색한다.

`flatpak install REMOTE APP_ID` — 애플리케이션을 설치한다.

`flatpak list --app` — 설치된 애플리케이션만 확인한다.

`flatpak update` — 설치된 Flatpak을 전체 업데이트한다.

`flatpak uninstall APP_ID` — 특정 애플리케이션을 삭제한다.

`flatpak uninstall --unused` — 사용하지 않는 Runtime을 정리한다.

`flatpak info --show-permissions APP_ID` — 애플리케이션의 샌드박스 권한을 확인한다.

---

# 7. RPM 패키지 제작

### RPM 패키지 제작 개요

소스 코드로 직접 설치한 프로그램을 RPM 패키지로 만들어 배포하면 **설치·삭제·업데이트·버전 관리**를 패키지 관리 체계에 통합할 수 있다.

자료에서 제시한 전체 제작 과정은 다음과 같다.

1. 소스 프로그램 개발
2. 소스 아카이브 생성
3. SPEC 파일 생성
4. RPM 빌드
5. GPG 키 생성
6. 패키지 서명
7. RPM Repository 구성
8. 테스트

### RPM 빌드 디렉터리

RPM 제작을 위한 기본 디렉터리 구조는 다음과 같다.

| 디렉터리 | 역할 |
|---|---|
| `BUILD` | 패키지 빌드 과정에서 사용되는 임시 작업 공간 |
| `RPMS` | 생성된 바이너리 RPM 저장 |
| `SOURCES` | 소스 아카이브 및 소스 파일 저장 |
| `SPECS` | SPEC 파일 저장 |
| `SRPMS` | Source RPM 저장 |

RPM 빌드 디렉터리 생성:

`rpmdev-setuptree`

### RPM 제작 관련 패키지

| 패키지 | 역할 |
|---|---|
| `rpmdevtools` | RPM 개발 도구 제공 |
| `rpmlint` | SPEC 및 RPM 패키지 검사 |
| `rpm-build` | RPM 패키지 빌드 |
| `rpm-sign` | RPM 패키지 서명 |

### 소스 아카이브 생성

RPM의 `SOURCES` 디렉터리에 일반적으로 `.tar.gz` 또는 `.tgz` 형태의 소스 아카이브를 저장한다.

소스 디렉터리 이름은 **SPEC 파일의 `%setup` 설정과 일치하도록 구성하는 것이 중요하다.**

### SPEC 파일

**SPEC 파일**은 RPM 패키지를 어떻게 빌드하고 어떤 파일을 패키지에 포함할지를 정의하는 핵심 파일이다.

주요 항목은 다음과 같다.

| SPEC 항목 | 설명 |
|---|---|
| `Name` | 패키지 이름 |
| `Version` | 패키지 버전 |
| `Release` | 패키지 릴리스 번호 |
| `Summary` | 패키지 요약 |
| `License` | 라이선스 |
| `URL` | 프로젝트 또는 관련 URL |
| `Source0` | 소스 아카이브 |
| `BuildRequires` | 빌드에 필요한 패키지 |
| `Requires` | 설치 시 필요한 패키지 |
| `%description` | 패키지 설명 |
| `%prep` | 빌드 준비 단계 |
| `%setup` | 소스 압축 해제 및 작업 디렉터리 설정 |
| `%build` | 실제 빌드 단계 |
| `%install` | 설치될 파일을 RPM 빌드 영역에 배치 |
| `%files` | 최종 RPM에 포함할 파일 지정 |
| `%check` | 테스트 단계 |
| `%changelog` | 변경 이력 |

### SPEC 파일 검사

`rpmlint`를 이용하여 SPEC 파일의 문법 및 패키징 문제를 검사한다.

`rpmlint ~/rpmbuild/SPECS/PKG.spec`

경고가 있다고 해서 반드시 빌드가 실패하는 것은 아니지만, 패키지 품질을 높이기 위해 가능한 경고를 확인하고 수정하는 것이 좋다.

### RPM 패키지 빌드

| 명령어 | 결과 |
|---|---|
| `rpmbuild -bs SPEC` | Source RPM 생성 |
| `rpmbuild -bb SPEC` | Binary RPM 생성 |
| `rpmbuild -ba SPEC` | Source RPM과 Binary RPM 모두 생성 |

일반적인 패키지 제작에서는 다음과 같이 사용한다.

`rpmbuild -ba ~/rpmbuild/SPECS/PKG.spec`

### 패키지 설치 및 테스트

빌드가 완료되면 생성된 RPM을 설치하여 정상적으로 동작하는지 확인한다.

주요 확인 방법:

`rpm -Uvh ~/rpmbuild/RPMS/noarch/PKG.rpm`

`rpm -qi PKG`

`rpm -ql PKG`

`rpm -qa | grep PKG`

테스트가 끝나면:

`rpm -e PKG`

### GPG 키 생성 및 패키지 서명

RPM 패키지에 GPG 서명을 추가하면 패키지의 **출처와 무결성을 검증**할 수 있다.

GPG 키 생성:

`gpg --gen-key`

키 목록 확인:

`gpg --list-keys`

RPM 서명에 사용할 키를 지정:

`echo "%_gpg_name KEY_ID" >> ~/.rpmmacros`

패키지 서명:

`rpmsign --addsign PKG.rpm`

서명 검증:

`rpm --checksig PKG.rpm`

공개 키 추출:

`gpg -a -o RPM-GPG-KEY-test --export KEY_ID`

### RPM Repository 구축

패키지를 여러 시스템에서 제공하려면 RPM 패키지를 Repository에 등록할 수 있다.

전체 구성 흐름:

**RPM 제작 → GPG 서명 → HTTP 서버에 패키지 배치 → Repository 메타데이터 생성 → `.repo` 파일 작성 → DNF 클라이언트에서 사용**

Repository 메타데이터 생성:

`createrepo /var/www/html/packages`

클라이언트 Repository 설정 예:

`/etc/yum.repos.d/PKG.repo`

일반적인 Repository 설정 항목:

| 항목 | 설명 |
|---|---|
| `name` | Repository 이름 |
| `baseurl` | Repository URL |
| `enabled=1` | Repository 활성화 |
| `gpgcheck=1` | RPM 서명 검사 활성화 |
| `gpgkey` | 공개 GPG 키 위치 |

### RPM 제작 시 중요한 파일 구조

최종적으로 다음과 같은 구조를 갖는다.

`~/rpmbuild/`

- `BUILD/`
- `RPMS/`
- `SOURCES/`
- `SPECS/`
- `SRPMS/`

### 자주 사용하는 RPM 제작 예제

`dnf install -y rpmdevtools rpmlint` — RPM 제작에 필요한 기본 도구를 설치한다.

`rpmdev-setuptree` — RPM 빌드 디렉터리 구조를 생성한다.

`rpmlint ~/rpmbuild/SPECS/PKG.spec` — SPEC 파일을 검사한다.

`rpmbuild -ba ~/rpmbuild/SPECS/PKG.spec` — Source RPM과 Binary RPM을 함께 생성한다.

`rpm -Uvh ~/rpmbuild/RPMS/noarch/PKG.rpm` — 생성된 RPM을 테스트 설치한다.

`rpm -qi PKG` — 생성된 패키지 정보를 확인한다.

`rpm -ql PKG` — 생성된 패키지가 설치한 파일을 확인한다.

`gpg --list-keys` — GPG 공개 키 목록을 확인한다.

`rpmsign --addsign PKG.rpm` — RPM 패키지에 GPG 서명을 추가한다.

`rpm --checksig PKG.rpm` — 패키지 서명을 검증한다.

`gpg -a -o RPM-GPG-KEY-test --export KEY_ID` — Repository에서 배포할 공개 키를 생성한다.

`createrepo /var/www/html/packages` — RPM Repository 메타데이터를 생성한다.

---

# 핵심 명령어 요약

## RPM

| 목적 | 명령어 |
|---|---|
| 전체 패키지 확인 | `rpm -qa` |
| 패키지 설치 확인 | `rpm -q PKG` |
| 패키지 정보 확인 | `rpm -qi PKG` |
| 패키지 파일 목록 | `rpm -ql PKG` |
| 파일이 속한 패키지 확인 | `rpm -qf FILE` |
| RPM 파일 정보 확인 | `rpm -qi -p PKG.rpm` |
| RPM 파일 목록 확인 | `rpm -ql -p PKG.rpm` |
| 패키지 설치 | `rpm -ivh PKG.rpm` |
| 패키지 업데이트 | `rpm -Uvh PKG.rpm` |
| 기존 설치 패키지만 업데이트 | `rpm -Fvh PKG.rpm` |
| 패키지 삭제 | `rpm -e PKG` |
| 패키지 검증 | `rpm -V PKG` |
| 패키지 서명 검증 | `rpm --checksig PKG.rpm` |
| RPM 내용 추출 | `rpm2cpio PKG.rpm \| cpio -idv` |

## DNF

| 목적 | 명령어 |
|---|---|
| 설치 패키지 목록 | `dnf list installed` |
| 설치 가능 패키지 | `dnf list available` |
| 패키지 검색 | `dnf search KEYWORD` |
| 패키지 정보 | `dnf info PKG` |
| 패키지 설치 | `dnf install PKG` |
| 로컬 RPM 설치 | `dnf localinstall PKG.rpm` |
| 패키지 업데이트 | `dnf update PKG` |
| 전체 업데이트 | `dnf update` |
| 패키지 삭제 | `dnf remove PKG` |
| 패키지 재설치 | `dnf reinstall PKG` |
| 의존 패키지 정리 | `dnf autoremove` |
| 파일 제공 패키지 검색 | `dnf provides '*/FILE'` |
| 패키지 다운로드 | `dnf download PKG` |
| 다운로드만 수행 | `dnf install --downloadonly PKG` |
| 업데이트 확인 | `dnf check-update` |
| 저장소 확인 | `dnf repolist` |
| 저장소 전체 확인 | `dnf repolist all` |
| 캐시 전체 삭제 | `dnf clean all` |
| 트랜잭션 확인 | `dnf history` |
| 트랜잭션 취소 | `dnf history undo ID` |
| 트랜잭션 재실행 | `dnf history redo ID` |
| 그룹 목록 | `dnf group list` |
| 그룹 정보 | `dnf group info GROUP` |
| 그룹 설치 | `dnf group install GROUP` |
| 그룹 삭제 | `dnf group remove GROUP` |

## Flatpak

| 목적 | 명령어 |
|---|---|
| 설치 목록 | `flatpak list` |
| App 목록 | `flatpak list --app` |
| Runtime 목록 | `flatpak list --runtime` |
| App 검색 | `flatpak search KEYWORD` |
| App 설치 | `flatpak install REMOTE APP_ID` |
| App 삭제 | `flatpak uninstall APP_ID` |
| App 실행 | `flatpak run APP_ID` |
| 전체 업데이트 | `flatpak update` |
| 업데이트 확인 | `flatpak remote-ls --updates` |
| 저장소 목록 | `flatpak remotes` |
| 저장소 추가 | `flatpak remote-add REMOTE URL` |
| 저장소 삭제 | `flatpak remote-delete REMOTE` |
| 권한 확인 | `flatpak info --show-permissions APP_ID` |
| 미사용 Runtime 정리 | `flatpak uninstall --unused` |

---

# 버전별 핵심 정리

| 항목 | CentOS Stream 10 기준 |
|---|---|
| 기본 저수준 패키지 관리 | **RPM** |
| 기본 고수준 패키지 관리 | **DNF** |
| YUM | DNF 호환 명령으로 유지 |
| 패키지 저장소 | DNF Repository |
| 로컬 RPM 설치 | `dnf localinstall` 또는 `rpm` |
| 의존성 자동 해결 | **DNF** |
| 그룹 패키지 | `dnf group` |
| 데스크톱 애플리케이션 | **Flatpak** |
| 패키지 제작 | `rpmbuild` |
| 패키지 품질 검사 | `rpmlint` |
| 패키지 서명 | `rpmsign` |
| Repository 메타데이터 | `createrepo` 계열 |
| 모듈 스트림 | **본 정리에서 제외** |

# 핵심 학습 포인트

1. **RPM은 개별 RPM 패키지를 직접 관리하는 도구**이고, **DNF는 Repository와 의존성을 포함하여 패키지를 관리하는 도구**이다.
2. CentOS Stream 10에서는 **DNF를 기본 패키지 관리 명령으로 이해하고 사용하는 것이 핵심**이다.
3. 패키지 설치 여부 확인에는 `rpm -q`, `rpm -qa`, `dnf list installed`를 사용한다.
4. 패키지 상세 정보 확인에는 `rpm -qi`, `dnf info`를 사용한다.
5. 특정 파일을 제공하는 패키지를 찾을 때 RPM 데이터베이스에서는 `rpm -qf`, Repository에서는 **`dnf provides`**를 사용한다.
6. 로컬 RPM 파일의 설치는 `rpm -Uvh` 또는 **`dnf localinstall`**을 사용할 수 있으며, 의존성 자동 해결이 필요한 경우 DNF가 유리하다.
7. DNF Repository는 `.repo` 파일을 통해 관리하며, 활성화 상태는 `dnf repolist`로 확인한다.
8. 여러 개발 도구를 한 번에 설치하려면 **`dnf group install`**을 사용한다.
9. 시스템 전체 업데이트 전에는 **`dnf check-update`**로 업데이트 대상과 커널 업데이트 여부를 확인한다.
10. 커널 업데이트 후 새 커널을 사용하려면 **재부팅이 필요하다.**
11. 데스크톱 애플리케이션은 시스템 RPM 패키지와 분리하여 **Flatpak**으로 관리할 수 있다.
12. RPM을 직접 제작할 때는 **소스 → SPEC → rpmbuild → GPG 서명 → Repository → 테스트**의 흐름을 이해해야 한다.
13. RPM Repository를 구축할 때는 **패키지, Repository 메타데이터, GPG 공개 키, `.repo` 설정**의 관계를 이해해야 한다.
14. 패키지 관리 작업을 되돌릴 때 DNF의 **Transaction History**를 사용할 수 있지만, 운영 환경의 전체 시스템 복구에는 **Snapshot 기반 복구가 더 안전한 경우가 많다.**
```
