# 마운트 관리

## 목차

1. [마운트 개요](#1-마운트-개요)
	- [마운트의 개념](#마운트의-개념)
	- [마운트 관련 주요 명령어](#마운트-관련-주요-명령어)
	- [마운트 상태 확인](#마운트-상태-확인)
2. [마운트 명령어](#2-마운트-명령어)
	- [mount 명령어](#mount-명령어)
	- [umount 명령어](#umount-명령어)
	- [mount 옵션](#mount-옵션)
	- [장치 지정 방법](#장치-지정-방법)
	- [마운트 포인트](#마운트-포인트)
	- [다중 마운트와 오버마운트](#다중-마운트와-오버마운트)
	- [Bind Mount](#bind-mount)
3. [/etc/fstab](#3-etcfstab)
	- [/etc/fstab의 개념](#etcfstab의-개념)
	- [/etc/fstab 필드](#etcfstab-필드)
	- [주요 마운트 옵션](#주요-마운트-옵션)
	- [파일 시스템 점검 필드](#파일-시스템-점검-필드)
	- [/etc/fstab 적용 및 테스트](#etcfstab-적용-및-테스트)
	- [/etc/fstab 오류 확인](#etcfstab-오류-확인)
4. [마운트 옵션 활용](#4-마운트-옵션-활용)
	- [읽기 전용 ro](#읽기-전용-ro)
	- [remount](#remount)
	- [noatime](#noatime)
	- [nosuid](#nosuid)
	- [noexec와 nodev](#noexec와-nodev)
	- [mount -a와 umount -a](#mount--a와-umount--a)
	- [사용 중인 파일 시스템 언마운트](#사용-중인-파일-시스템-언마운트)
5. [디스크 추가 및 마운트](#5-디스크-추가-및-마운트)
	- [디스크 추가 작업 순서](#디스크-추가-작업-순서)
	- [디스크 인식 확인](#디스크-인식-확인)
	- [파티션 생성](#파티션-생성)
	- [파일 시스템 생성](#파일-시스템-생성)
	- [영구 마운트 설정](#영구-마운트-설정)
6. [CD/DVD 및 ISO 이미지 마운트](#6-cddvd-및-iso-이미지-마운트)
	- [CD/DVD 자동 마운트](#cddvd-자동-마운트)
	- [CD/DVD 수동 마운트](#cddvd-수동-마운트)
	- [ISO 이미지 마운트](#iso-이미지-마운트)
7. [USB 저장 장치 마운트](#7-usb-저장-장치-마운트)
	- [FAT 계열 파일 시스템](#fat-계열-파일-시스템)
	- [NTFS 파일 시스템](#ntfs-파일-시스템)
8. [tmpfs와 RAM 디스크](#8-tmpfs와-ram-디스크)
	- [tmpfs의 개념](#tmpfs의-개념)
	- [tmpfs 마운트](#tmpfs-마운트)
	- [RAM 디스크 사용 시 주의사항](#ram-디스크-사용-시-주의사항)
9. [NFS 원격 마운트](#9-nfs-원격-마운트)
	- [NFS의 개념](#nfs의-개념)
	- [NFS 서버 설정](#nfs-서버-설정)
	- [NFS 클라이언트 마운트](#nfs-클라이언트-마운트)
10. [CIFS/SMB 원격 마운트](#10-cifssmb-원격-마운트)
	- [CIFS의 개념](#cifs의-개념)
	- [SMB 공유 확인](#smb-공유-확인)
	- [CIFS 마운트](#cifs-마운트)
11. [Loop 장치 마운트](#11-loop-장치-마운트)
	- [Loop 장치의 개념](#loop-장치의-개념)
	- [이미지 파일을 Loop 장치로 연결](#이미지-파일을-loop-장치로-연결)
	- [Loop 장치에 파일 시스템 생성](#loop-장치에-파일-시스템-생성)
	- [Loop 장치 해제](#loop-장치-해제)
12. [마운트 문제 해결](#12-마운트-문제-해결)
	- [마운트 상태 확인](#마운트-상태-확인-1)
	- [target is busy 오류](#target-is-busy-오류)
	- [/etc/fstab 오류](#etcfstab-오류)
	- [Rescue 환경에서의 점검](#rescue-환경에서의-점검)
13. [디스크 추가 작업 자동화](#13-디스크-추가-작업-자동화)
	- [자동화 작업 순서](#자동화-작업-순서)
	- [스크립트 작성 시 주의사항](#스크립트-작성-시-주의사항)
14. [핵심 명령어 및 옵션 정리](#14-핵심-명령어-및-옵션-정리)

---

## 1. 마운트 개요

### 마운트의 개념

**마운트(Mount)**는 파일 시스템을 Linux의 디렉토리 트리에 연결하여 사용할 수 있도록 하는 작업이다.

Linux에서는 모든 파일과 디렉토리가 **하나의 디렉토리 트리**에 포함되며, 최상위 디렉토리는 `/`이다.

일반적인 저장 장치 사용 과정은 다음과 같다.

**디스크 → 파티션 → 파일 시스템 → 마운트 포인트 → 파일 및 디렉토리 사용**

| 구성 요소 | 설명 |
|---|---|
| 디스크 | 물리적 또는 가상 저장 장치 |
| 파티션 | 디스크를 논리적으로 분할한 영역 |
| 파일 시스템 | 파일과 디렉토리를 저장하기 위한 구조 |
| 마운트 포인트 | 파일 시스템이 연결되는 디렉토리 |
| 마운트 | 파일 시스템을 특정 디렉토리에 연결하는 작업 |
| 언마운트 | 연결된 파일 시스템을 해제하는 작업 |

### 마운트 관련 주요 명령어

| 명령어 | 설명 |
|---|---|
| `mount` | 파일 시스템을 마운트하거나 현재 마운트 상태를 확인 |
| `umount` | 파일 시스템의 마운트를 해제 |
| `mount -a` | `/etc/fstab`에 정의된 파일 시스템 중 마운트 가능한 항목을 마운트 |
| `umount -a` | 언마운트 가능한 파일 시스템을 일괄적으로 언마운트 |
| `findmnt` | 현재 마운트된 파일 시스템을 계층적으로 확인 |
| `df` | 파일 시스템의 용량 및 사용량 확인 |
| `lsblk` | 블록 장치 및 마운트 상태 확인 |
| `blkid` | 장치의 UUID, LABEL, 파일 시스템 타입 확인 |

### 마운트 상태 확인

현재 마운트된 파일 시스템은 여러 명령어로 확인할 수 있다.

| 명령어 | 주요 용도 |
|---|---|
| `mount` | 마운트된 장치와 마운트 옵션 확인 |
| `findmnt` | 마운트 구조를 계층적으로 확인 |
| `df -hT` | 파일 시스템 타입 및 사용량 확인 |
| `lsblk -f` | 장치, 파일 시스템, UUID, 마운트 포인트 확인 |
| `blkid` | UUID 및 파일 시스템 타입 확인 |

### 자주 사용하는 사용 예제

전체 마운트 상태를 확인한다.

`findmnt`

파일 시스템 타입과 사용량을 함께 확인한다.

`df -hT`

블록 장치와 파일 시스템 정보를 확인한다.

`lsblk -f`

특정 파일 시스템의 마운트 상태를 확인한다.

`mount | grep /mnt/data`

---

## 2. 마운트 명령어

### mount 명령어

`mount`는 파일 시스템을 특정 디렉토리에 연결하는 명령어이다.

| 형식 | 설명 |
|---|---|
| `mount DEVICE MOUNT_POINT` | 장치를 지정하여 마운트 |
| `mount -t TYPE DEVICE MOUNT_POINT` | 파일 시스템 타입을 지정하여 마운트 |
| `mount -o OPTIONS DEVICE MOUNT_POINT` | 마운트 옵션을 지정 |
| `mount MOUNT_POINT` | `/etc/fstab`에 정의된 정보를 이용하여 마운트 |
| `mount` | 현재 마운트된 파일 시스템 확인 |

| 옵션 | 설명 |
|---|---|
| `-t TYPE` | 파일 시스템 타입 지정 |
| `-o OPTIONS` | 마운트 옵션 지정 |
| `-a` | `/etc/fstab`의 마운트 가능한 항목을 모두 마운트 |
| `-r` | 읽기 전용으로 마운트 |
| `-w` | 읽기/쓰기 모드로 마운트 |
| `-v` | 마운트 과정을 자세하게 출력 |

### umount 명령어

`umount`는 마운트된 파일 시스템을 해제한다.

| 형식 | 설명 |
|---|---|
| `umount DEVICE` | 장치를 기준으로 언마운트 |
| `umount MOUNT_POINT` | 마운트 포인트를 기준으로 언마운트 |
| `umount -a` | 언마운트 가능한 파일 시스템을 일괄 해제 |

언마운트하기 전에 해당 마운트 포인트를 현재 작업 디렉토리로 사용하고 있지 않은지 확인해야 한다.

### mount 옵션

마운트 옵션은 `-o` 뒤에 지정하며 여러 옵션은 **쉼표(`,`)**로 구분한다.

| 옵션 | 설명 |
|---|---|
| `defaults` | 기본 옵션 묶음 |
| `rw` | 읽기/쓰기 허용 |
| `ro` | 읽기 전용 |
| `suid` | SetUID/SetGID 비트의 동작 허용 |
| `nosuid` | SetUID/SetGID 비트의 동작 제한 |
| `dev` | 장치 파일을 해석 |
| `nodev` | 장치 파일을 해석하지 않음 |
| `exec` | 실행 파일 실행 허용 |
| `noexec` | 실행 파일의 직접 실행 제한 |
| `auto` | `mount -a`를 통한 자동 마운트 대상 |
| `noauto` | `mount -a`로 자동 마운트하지 않음 |
| `user` | 일반 사용자의 마운트를 허용 |
| `nouser` | 일반 사용자의 마운트를 제한 |
| `async` | 비동기 I/O 사용 |
| `sync` | 동기 I/O 사용 |
| `atime` | 파일 접근 시간 갱신 |
| `noatime` | 파일 접근 시간 갱신을 하지 않음 |
| `remount` | 기존 마운트의 옵션을 변경하여 다시 마운트 |

**`defaults`는 일반적으로 `rw,suid,dev,exec,auto,nouser,async`를 의미한다.**

### 장치 지정 방법

마운트할 장치는 일반적으로 **장치 파일명** 또는 **UUID**로 지정한다.

| 방식 | 설명 |
|---|---|
| `/dev/nvme0n1p1` | 장치 파일명을 이용 |
| `UUID="..."` | 파일 시스템 UUID를 이용 |
| `LABEL="..."` | 파일 시스템 LABEL을 이용 |

장치명은 하드웨어 구성이나 부팅 순서에 따라 변경될 수 있으므로 **영구 마운트에는 UUID 또는 LABEL 사용을 권장**한다.

### 자주 사용하는 사용 예제

장치명을 사용하여 마운트한다.

`mount /dev/nvme0n3p1 /mnt/data`

UUID를 사용하여 마운트한다.

`mount UUID="UUID_VALUE" /mnt/data`

읽기 전용으로 마운트한다.

`mount -o ro /dev/nvme0n3p1 /mnt/data`

여러 옵션을 함께 지정한다.

`mount -o ro,nosuid,nodev /dev/nvme0n3p1 /mnt/data`

### 마운트 포인트

**마운트 포인트(Mount Point)**는 파일 시스템이 연결되는 디렉토리이다.

마운트 포인트로 사용할 디렉토리는 일반적으로 **비어 있는 디렉토리를 사용하는 것이 안전하다.**

비어 있지 않은 디렉토리에 다른 파일 시스템을 마운트하면 기존 파일은 삭제되지 않지만 **마운트된 파일 시스템에 의해 가려져 접근할 수 없게 된다.**

언마운트하면 기존 파일에 다시 접근할 수 있다.

| 주의사항 | 설명 |
|---|---|
| 빈 디렉토리 사용 | 기존 파일이 가려지는 문제 방지 |
| 절대 경로 사용 | 마운트 포인트를 명확하게 지정 |
| 사용 중인 디렉토리 확인 | 언마운트 실패 방지 |
| 중복 마운트 확인 | 다중 마운트 및 오버마운트 방지 |

### 다중 마운트와 오버마운트

**다중 마운트(Multi-Mount)**는 하나의 장치를 여러 마운트 포인트에 연결하는 상황이다.

**오버마운트(Over-Mount)**는 하나의 마운트 포인트에 여러 파일 시스템을 연속적으로 마운트하는 상황이다.

일반적인 운영 환경에서는 의도하지 않은 다중 마운트와 오버마운트를 피해야 한다.

| 형태 | 설명 |
|---|---|
| 장치 1개 → 마운트 포인트 2개 | Multi-Mount |
| 장치 2개 → 마운트 포인트 1개 | Over-Mount |
| 하나의 파일 시스템을 여러 경로에서 사용 | 필요하면 Bind Mount 고려 |

### Bind Mount

**Bind Mount**는 기존 디렉토리를 다른 디렉토리에 다시 연결하는 방식이다.

일반적인 장치의 다중 마운트와 달리 **기존 파일 시스템의 특정 디렉토리를 다른 경로에서 접근**할 때 사용한다.

| 명령어 | 설명 |
|---|---|
| `mount --bind SOURCE TARGET` | SOURCE 디렉토리를 TARGET에 연결 |
| `mount --rbind SOURCE TARGET` | 하위 마운트까지 포함하여 재귀적으로 연결 |

### 자주 사용하는 사용 예제

기존 디렉토리를 다른 경로에서 접근할 수 있도록 Bind Mount한다.

`mount --bind /var/log/app /srv/logs`

Bind Mount를 해제한다.

`umount /srv/logs`

---

## 3. /etc/fstab

### /etc/fstab의 개념

**`/etc/fstab`**은 시스템 부팅 과정에서 사용할 파일 시스템의 마운트 정보를 정의하는 설정 파일이다.

일반적인 마운트 정보는 다음과 같이 관리된다.

**현재 마운트 → `mount` 명령어**

**부팅 시 마운트 → `/etc/fstab`**

`mount` 명령으로만 마운트한 파일 시스템은 별도의 영구 설정이 없으면 재부팅 후 자동으로 마운트되지 않는다.

### /etc/fstab의 필드

일반적인 `/etc/fstab` 항목은 다음과 같은 **6개 필드**로 구성된다.

| 순서 | 필드 | 설명 |
|---:|---|---|
| 1 | 파일 시스템 장치 | `/dev/...`, `UUID=...`, `LABEL=...` 등 |
| 2 | 마운트 포인트 | `/`, `/home`, `/mnt/data` 등 |
| 3 | 파일 시스템 타입 | `xfs`, `ext4`, `vfat`, `nfs`, `cifs` 등 |
| 4 | 마운트 옵션 | `defaults`, `ro`, `noatime` 등 |
| 5 | dump | 전통적인 `dump` 백업 관련 설정 |
| 6 | fsck 순서 | 부팅 시 파일 시스템 검사 순서 |

### 주요 마운트 옵션

| 옵션 | 설명 |
|---|---|
| `defaults` | 기본 마운트 옵션 사용 |
| `auto` | `mount -a` 대상에 포함 |
| `noauto` | `mount -a` 대상에서 제외 |
| `rw` | 읽기/쓰기 |
| `ro` | 읽기 전용 |
| `exec` | 실행 파일 실행 허용 |
| `noexec` | 실행 파일 직접 실행 제한 |
| `suid` | SetUID/SetGID 허용 |
| `nosuid` | SetUID/SetGID 제한 |
| `dev` | 장치 파일 해석 |
| `nodev` | 장치 파일 해석 제한 |
| `user` | 일반 사용자 마운트 허용 |
| `nouser` | 일반 사용자 마운트 제한 |
| `noatime` | 접근 시간 갱신 방지 |
| `sync` | 동기식 I/O |
| `async` | 비동기식 I/O |

### 파일 시스템 점검 필드

`/etc/fstab`의 마지막 필드는 부팅 시 파일 시스템 검사 순서를 지정하는 데 사용된다.

| 값 | 의미 |
|---:|---|
| `0` | 자동 파일 시스템 검사 대상에서 제외 |
| `1` | 일반적으로 루트 파일 시스템에 사용 |
| `2` | 루트 이외의 파일 시스템 검사에 사용 |

현대 Linux 시스템에서는 **XFS의 부팅 시 파일 시스템 점검 방식이 EXT 계열과 다르므로**, 단순히 모든 파일 시스템에 동일한 fsck 순서를 적용해서는 안 된다.

### /etc/fstab 적용 및 테스트

`/etc/fstab`을 수정한 후에는 재부팅하기 전에 **`mount -a`로 설정 오류를 확인하는 것이 안전하다.**

systemd 환경에서는 `/etc/fstab` 변경 후 다음 명령을 사용하여 생성된 mount unit 정보를 다시 반영할 수 있다.

| 명령어 | 설명 |
|---|---|
| `systemctl daemon-reload` | systemd가 변경된 설정을 다시 읽도록 요청 |
| `mount -a` | `/etc/fstab`의 마운트 가능한 항목을 테스트 |
| `findmnt --verify` | `/etc/fstab`의 마운트 설정 검증 |
| `df -hT` | 실제 마운트 결과 확인 |

### 자주 사용하는 사용 예제

`/etc/fstab`을 수정한 후 systemd 설정을 다시 읽는다.

`systemctl daemon-reload`

`/etc/fstab`의 마운트 설정을 적용한다.

`mount -a`

마운트 결과를 확인한다.

`df -hT`

`/etc/fstab`의 구문과 마운트 정보를 검증한다.

`findmnt --verify`

### /etc/fstab 오류 확인

`/etc/fstab`에 잘못된 장치, 파일 시스템 타입, 마운트 포인트 또는 옵션을 입력하면 부팅 과정에서 문제가 발생할 수 있다.

설정 변경 후에는 다음 순서로 확인한다.

1. `/etc/fstab` 수정
2. `systemctl daemon-reload`
3. `mount -a`
4. 오류 발생 여부 확인
5. `df -hT` 또는 `findmnt`로 실제 마운트 확인

### 자주 사용하는 사용 예제

잘못된 `/etc/fstab` 설정을 재부팅 전에 확인한다.

`vi /etc/fstab`

`systemctl daemon-reload`

`mount -a`

`df -hT`

---

## 4. 마운트 옵션 활용

### 읽기 전용 ro

**`ro(Read Only)`**는 파일 시스템을 읽기 전용으로 마운트하는 옵션이다.

읽기 전용으로 마운트된 파일 시스템에서는 일반적인 파일 생성이나 수정이 제한된다.

| 명령어 | 설명 |
|---|---|
| `mount -o ro DEVICE MOUNT_POINT` | 읽기 전용 마운트 |
| `mount -o rw DEVICE MOUNT_POINT` | 읽기/쓰기 마운트 |

### remount

**`remount`**는 이미 마운트된 파일 시스템의 마운트 옵션을 변경할 때 사용한다.

| 명령어 | 설명 |
|---|---|
| `mount -o remount,ro MOUNT_POINT` | 읽기 전용으로 변경 |
| `mount -o remount,rw MOUNT_POINT` | 읽기/쓰기 모드로 변경 |

### 자주 사용하는 사용 예제

마운트된 파일 시스템을 읽기 전용으로 변경한다.

`mount -o remount,ro /mnt/data`

다시 읽기/쓰기 모드로 변경한다.

`mount -o remount,rw /mnt/data`

현재 적용된 옵션을 확인한다.

`mount | grep /mnt/data`

### noatime

**`noatime`**은 파일에 접근할 때 inode의 access time을 갱신하지 않도록 하는 옵션이다.

파일의 시간 정보는 다음과 같이 구분된다.

| 시간 | 의미 | 확인 방법 |
|---|---|---|
| **mtime** | 파일 내용 수정 시간 | `ls -l` |
| **atime** | 파일 접근 시간 | `ls -lu` |
| **ctime** | inode 속성 변경 시간 | `ls -lc` |

`noatime`은 파일 접근 시 발생하는 atime 갱신을 줄여 **불필요한 메타데이터 쓰기 작업을 감소**시킬 수 있다.

### 자주 사용하는 사용 예제

파일 시스템을 `noatime` 옵션으로 마운트한다.

`mount -o noatime DEVICE /mnt/data`

현재 적용된 마운트 옵션을 확인한다.

`mount | grep /mnt/data`

파일의 시간 정보를 확인한다.

`stat FILE`

### nosuid

**`nosuid`**는 해당 파일 시스템에서 SetUID 및 SetGID 비트의 효과를 제한하는 보안 옵션이다.

사용자에게 쓰기 권한이 있는 파일 시스템에서 SetUID 프로그램이 실행되어 권한 상승에 악용되는 위험을 줄이는 데 활용할 수 있다.

| 옵션 | 설명 |
|---|---|
| `suid` | SetUID/SetGID 효과 허용 |
| `nosuid` | SetUID/SetGID 효과 제한 |

### 자주 사용하는 사용 예제

사용자 데이터 영역 등에 `nosuid`를 적용한다.

`mount -o nosuid DEVICE /home`

현재 적용된 옵션을 확인한다.

`mount | grep /home`

### noexec와 nodev

| 옵션 | 설명 |
|---|---|
| `noexec` | 마운트된 파일 시스템의 실행 파일 직접 실행 제한 |
| `nodev` | 마운트된 파일 시스템에서 장치 파일 해석 제한 |

보안 강화가 필요한 파일 시스템에서 사용하지만, **애플리케이션이나 시스템 서비스가 정상적으로 동작하지 않을 수 있으므로 적용 전에 호환성을 확인**해야 한다.

### mount -a와 umount -a

| 명령어 | 기준 | 설명 |
|---|---|---|
| `mount -a` | `/etc/fstab` | 마운트 가능한 항목을 모두 마운트 |
| `umount -a` | 현재 마운트 상태 | 언마운트 가능한 파일 시스템을 일괄 해제 |

`umount -a`는 시스템의 모든 파일 시스템을 무조건 해제하는 명령어가 아니다. 루트 파일 시스템이나 사용 중인 가상 파일 시스템 등은 언마운트할 수 없다.

### 사용 중인 파일 시스템 언마운트

마운트 포인트를 프로세스가 사용하고 있으면 `umount`가 실패하면서 **`target is busy`** 오류가 발생할 수 있다.

먼저 어떤 프로세스가 해당 파일 시스템을 사용하는지 확인해야 한다.

| 명령어 | 설명 |
|---|---|
| `fuser -m MOUNT_POINT` | 해당 파일 시스템을 사용하는 프로세스 확인 |
| `fuser -vm MOUNT_POINT` | 프로세스 정보를 자세하게 확인 |
| `lsof +D MOUNT_POINT` | 해당 디렉토리 아래의 열린 파일 확인 |
| `umount MOUNT_POINT` | 정상적인 언마운트 |
| `fuser -km MOUNT_POINT` | 해당 파일 시스템을 사용하는 프로세스를 종료할 수 있음 |

**`fuser -k`는 프로세스를 강제로 종료할 수 있으므로 운영 환경에서는 프로세스를 확인한 후 신중하게 사용해야 한다.**

### 자주 사용하는 사용 예제

사용 중인 프로세스를 확인한다.

`fuser -vm /mnt/data`

열린 파일을 확인한다.

`lsof +D /mnt/data`

프로세스를 정리한 후 언마운트한다.

`umount /mnt/data`

---

## 5. 디스크 추가 및 마운트

### 디스크 추가 작업 순서

새로운 디스크를 추가하여 파일 시스템으로 사용하는 일반적인 순서는 다음과 같다.

**디스크 인식 → 파티션 설정 → 파일 시스템 생성 → 마운트 포인트 생성 → 마운트 → `/etc/fstab` 설정 → 검증**

| 단계 | 작업 | 주요 명령 |
|---|---|---|
| 1 | 디스크 인식 | `lsblk` |
| 2 | 파티션 설정 | `fdisk`, `parted` |
| 3 | 파일 시스템 생성 | `mkfs.ext4`, `mkfs.xfs` |
| 4 | 마운트 포인트 생성 | `mkdir -p` |
| 5 | 임시 마운트 | `mount` |
| 6 | 영구 마운트 설정 | `/etc/fstab` |
| 7 | 설정 적용 | `systemctl daemon-reload`, `mount -a` |
| 8 | 결과 확인 | `df -hT`, `lsblk -f`, `findmnt` |

### 디스크 인식 확인

새 디스크를 시스템에 연결한 후 다음 명령어로 인식 여부를 확인한다.

| 명령어 | 설명 |
|---|---|
| `lsblk` | 디스크 및 파티션 구조 확인 |
| `lsblk -f` | 파일 시스템 및 UUID 확인 |
| `fdisk -l` | 디스크와 파티션 상세 정보 확인 |
| `udevadm settle` | udev 장치 이벤트 처리가 완료될 때까지 대기 |

### 파티션 생성

`parted`를 이용하면 GPT 파티션 테이블과 파티션을 비대화형으로 생성할 수 있다.

| 명령어 | 설명 |
|---|---|
| `parted DEVICE print` | 파티션 정보 확인 |
| `parted -s DEVICE mklabel gpt` | GPT 파티션 테이블 생성 |
| `parted -s DEVICE mkpart primary 1MiB 100%` | 전체 영역을 사용하는 파티션 생성 |

파티션 작업은 기존 데이터를 삭제할 수 있으므로 **대상 디스크를 반드시 확인한 후 실행**한다.

### 파일 시스템 생성

| 명령어 | 설명 |
|---|---|
| `mkfs.ext4 DEVICE` | EXT4 생성 |
| `mkfs.xfs DEVICE` | XFS 생성 |
| `mkfs -t ext4 DEVICE` | EXT4 생성 |
| `mkfs -t xfs DEVICE` | XFS 생성 |

XFS를 이미 파일 시스템이 존재하는 장치에 다시 생성할 때는 `-f` 옵션이 사용될 수 있지만, **기존 데이터가 삭제되므로 반드시 대상 장치를 확인해야 한다.**

### 영구 마운트 설정

예를 들어 특정 파일 시스템을 `/mnt/data`에 영구적으로 마운트하려면 다음과 같은 정보를 `/etc/fstab`에 등록한다.

| 필드 | 예시 |
|---|---|
| 장치 | `UUID=UUID_VALUE` |
| 마운트 포인트 | `/mnt/data` |
| 파일 시스템 | `xfs` |
| 옵션 | `defaults` |
| dump | `0` |
| fsck | `0` |

### 자주 사용하는 사용 예제

새 디스크를 확인한다.

`lsblk`

파티션 정보를 확인한다.

`parted /dev/nvme0n3 print`

GPT 파티션을 생성한다.

`parted -s /dev/nvme0n3 mklabel gpt`

파티션을 생성한다.

`parted -s /dev/nvme0n3 mkpart primary 1MiB 100%`

파일 시스템을 생성한다.

`mkfs.xfs /dev/nvme0n3p1`

마운트 포인트를 생성한다.

`mkdir -p /mnt/data`

임시 마운트한다.

`mount /dev/nvme0n3p1 /mnt/data`

UUID를 확인한다.

`blkid /dev/nvme0n3p1`

---

## 6. CD/DVD 및 ISO 이미지 마운트

### CD/DVD 자동 마운트

데스크톱 환경에서는 CD/DVD 또는 ISO 미디어가 **자동 마운트**될 수 있다.

일반적으로 사용자 세션에서 이동식 미디어가 연결되면 `/run/media/UserName/LABEL` 형태의 경로가 사용될 수 있다.

| 명령어 | 설명 |
|---|---|
| `lsblk -f` | CD/DVD 장치 및 파일 시스템 확인 |
| `df -hT` | 자동 마운트 상태 확인 |
| `ls /run/media/UserName/LABEL` | 자동 마운트된 미디어 확인 |
| `umount /run/media/UserName/LABEL` | 마운트 해제 |

### CD/DVD 수동 마운트

CD/DVD는 일반적으로 **ISO9660** 파일 시스템으로 마운트한다.

| 명령어 | 설명 |
|---|---|
| `mkdir -p /mnt/cdrom` | 마운트 포인트 생성 |
| `mount -t iso9660 -o ro /dev/cdrom /mnt/cdrom` | CD/DVD를 읽기 전용으로 마운트 |
| `umount /mnt/cdrom` | 마운트 해제 |

### 자주 사용하는 사용 예제

CD/DVD를 수동으로 마운트한다.

`mkdir -p /mnt/cdrom`

`mount -t iso9660 -o ro /dev/cdrom /mnt/cdrom`

마운트된 내용을 확인한다.

`ls /mnt/cdrom`

마운트 해제한다.

`umount /mnt/cdrom`

### ISO 이미지 마운트

ISO 파일은 일반 블록 장치가 아니므로 **Loop 장치**를 이용하여 마운트할 수 있다.

`mount` 명령에서 `loop` 옵션을 사용하면 ISO 파일을 Loop 장치와 연결하여 마운트할 수 있다.

| 옵션 | 설명 |
|---|---|
| `loop` | 파일을 Loop 장치로 연결 |
| `ro` | 읽기 전용으로 마운트 |
| `iso9660` | ISO 9660 파일 시스템 타입 지정 |

### 자주 사용하는 사용 예제

ISO 파일을 읽기 전용으로 마운트한다.

`mkdir -p /mnt/iso`

`mount -t iso9660 -o loop,ro /path/to/image.iso /mnt/iso`

마운트 상태를 확인한다.

`df -hT /mnt/iso`

ISO를 사용한 후 마운트 해제한다.

`umount /mnt/iso`

---

## 7. USB 저장 장치 마운트

### FAT 계열 파일 시스템

FAT32 파일 시스템은 Linux에서 일반적으로 **`vfat`** 타입으로 마운트한다.

| 명령어 | 설명 |
|---|---|
| `lsblk -f` | USB 장치 및 파일 시스템 확인 |
| `mount -t vfat DEVICE MOUNT_POINT` | FAT 계열 파일 시스템 마운트 |
| `umount MOUNT_POINT` | 마운트 해제 |

### 자주 사용하는 사용 예제

USB 장치를 확인한다.

`lsblk -f`

FAT32 장치를 수동으로 마운트한다.

`mkdir -p /mnt/usb`

`mount -t vfat /dev/sdX1 /mnt/usb`

사용 후 해제한다.

`umount /mnt/usb`

### NTFS 파일 시스템

NTFS는 Windows에서 주로 사용하는 파일 시스템이다.

Linux에서 NTFS를 사용하려면 시스템에서 해당 파일 시스템을 지원하는 **NTFS 드라이버/도구**가 필요하다.

자료에서는 `ntfs-3g`를 이용한 방식이 제시되어 있다.

| 명령어 | 설명 |
|---|---|
| `mount -t ntfs DEVICE MOUNT_POINT` | NTFS 마운트 |
| `mount.ntfs-3g DEVICE MOUNT_POINT` | ntfs-3g 기반 마운트 |
| `mount.ntfs-3g -o windows_names DEVICE MOUNT_POINT` | Windows에서 문제가 될 수 있는 파일명 제한 |
| `umount MOUNT_POINT` | 마운트 해제 |

`ntfs-3g` 패키지 설치 방법은 사용하는 CentOS 10.x 저장소 구성에 따라 달라질 수 있으므로 **현재 활성화된 저장소에서 제공되는 NTFS 지원 패키지를 확인한 후 설치**한다.

### 자주 사용하는 사용 예제

NTFS 장치를 확인한다.

`lsblk -f`

NTFS 장치를 마운트한다.

`mkdir -p /mnt/ntfs`

`mount -t ntfs /dev/sdX1 /mnt/ntfs`

마운트 상태를 확인한다.

`df -hT /mnt/ntfs`

사용 후 마운트 해제한다.

`umount /mnt/ntfs`

---

## 8. tmpfs와 RAM 디스크

### tmpfs의 개념

**tmpfs**는 메모리 및 필요에 따라 스왑 영역을 이용하여 데이터를 저장하는 가상 파일 시스템이다.

RAM 디스크처럼 사용할 수 있으며, 일반 디스크보다 빠른 임시 저장 공간이 필요한 경우 활용할 수 있다.

| 특징 | 설명 |
|---|---|
| 저장 위치 | 메모리 중심 |
| 성능 | 일반 디스크보다 빠를 수 있음 |
| 크기 | 마운트 옵션으로 제한 가능 |
| 영속성 | 일반적으로 재부팅 시 내용이 유지되지 않음 |
| 용도 | 임시 데이터, 캐시, 성능 테스트 등 |

### tmpfs 마운트

| 명령어 | 설명 |
|---|---|
| `mount -t tmpfs none MOUNT_POINT` | 기본 tmpfs 마운트 |
| `mount -t tmpfs -o size=10M none MOUNT_POINT` | 최대 크기를 지정하여 마운트 |
| `df -hT` | tmpfs 상태 확인 |
| `umount MOUNT_POINT` | tmpfs 해제 |

### 자주 사용하는 사용 예제

10MiB 크기의 tmpfs를 생성한다.

`mkdir -p /mnt/ramdisk`

`mount -t tmpfs -o size=10M none /mnt/ramdisk`

상태를 확인한다.

`df -hT /mnt/ramdisk`

사용 후 해제한다.

`umount /mnt/ramdisk`

### RAM 디스크 사용 시 주의사항

- **재부팅하면 일반적으로 데이터가 유지되지 않는다.**
- 시스템 메모리를 사용하므로 크기를 과도하게 지정하면 메모리 부족을 유발할 수 있다.
- 중요한 데이터를 저장하는 영구 저장소로 사용해서는 안 된다.

---

## 9. NFS 원격 마운트

### NFS의 개념

**NFS(Network File System)**는 Linux/Unix 시스템 사이에서 원격 디렉토리를 공유하는 네트워크 파일 시스템이다.

구조는 다음과 같다.

**NFS 서버 → 네트워크 공유 → NFS 클라이언트**

| 구성 | 역할 |
|---|---|
| NFS Server | 디렉토리를 네트워크로 공유 |
| NFS Client | 원격 디렉토리를 자신의 파일 시스템에 마운트 |

### NFS 서버 설정

NFS 서버에서는 일반적으로 **`nfs-utils`** 패키지를 사용한다.

| 명령어 | 설명 |
|---|---|
| `dnf install nfs-utils` | NFS 관련 패키지 설치 |
| `/etc/exports.d/*.exports` | NFS 공유 설정 |
| `exportfs -v` | 현재 export 정보 확인 |
| `exportfs -rav` | export 설정을 다시 읽고 공유 |
| `systemctl enable --now nfs-server` | NFS 서버 서비스 활성화 및 시작 |

공유 설정은 일반적으로 다음과 같은 형태를 사용한다.

`/share CLIENT(rw)`

보안과 접근 제어를 고려하여 **`*`로 모든 클라이언트를 허용하는 설정은 실제 운영 환경에서는 피하는 것이 좋다.**

### NFS 클라이언트 마운트

클라이언트에서는 NFS 서버의 공유 디렉토리를 마운트한다.

| 명령어 | 설명 |
|---|---|
| `dnf install nfs-utils` | NFS 클라이언트 도구 설치 |
| `showmount -e SERVER` | 서버에서 export한 공유 목록 확인 |
| `mount -t nfs SERVER:/share /mnt/nfs` | NFS 공유 마운트 |
| `umount /mnt/nfs` | NFS 마운트 해제 |

### 자주 사용하는 사용 예제

NFS 서버의 공유 목록을 확인한다.

`showmount -e SERVER_IP`

NFS 공유를 마운트한다.

`mkdir -p /mnt/nfs`

`mount -t nfs SERVER_IP:/share /mnt/nfs`

마운트 상태를 확인한다.

`df -hT /mnt/nfs`

사용 후 해제한다.

`umount /mnt/nfs`

---

## 10. CIFS/SMB 원격 마운트

### CIFS의 개념

**CIFS(Common Internet File System)**는 SMB 계열 프로토콜을 이용하여 Windows 등의 네트워크 공유 폴더를 Linux에서 사용할 수 있도록 하는 방식이다.

구조는 다음과 같다.

**Windows/SMB 서버 → 네트워크 공유 → Linux 클라이언트**

### SMB 공유 확인

Linux에서 Windows 또는 SMB 서버의 공유 목록을 확인할 때 **`smbclient`**를 사용할 수 있다.

| 명령어 | 설명 |
|---|---|
| `dnf install cifs-utils samba-client` | CIFS/SMB 클라이언트 도구 설치 |
| `smbclient -L SERVER -U UserName` | 서버의 공유 목록 확인 |
| `mount.cifs` | CIFS 파일 시스템 마운트 도구 |

### CIFS 마운트

| 명령어 | 설명 |
|---|---|
| `mount -t cifs //SERVER/SHARE /mnt/cifs` | SMB 공유 마운트 |
| `mount -t cifs //SERVER/SHARE /mnt/cifs -o username=UserName` | 사용자 계정 지정 |
| `umount /mnt/cifs` | CIFS 마운트 해제 |

암호를 명령행에 직접 입력하는 방식은 보안상 권장하지 않는다. 운영 환경에서는 **credentials 파일**을 사용하고 파일 권한을 제한하는 방법을 고려한다.

### 자주 사용하는 사용 예제

SMB 서버의 공유 목록을 확인한다.

`smbclient -L SERVER_IP -U UserName`

공유 디렉토리를 마운트한다.

`mkdir -p /mnt/cifs`

`mount -t cifs //SERVER_IP/ShareName /mnt/cifs -o username=UserName`

마운트 상태를 확인한다.

`df -hT /mnt/cifs`

사용 후 해제한다.

`umount /mnt/cifs`

---

## 11. Loop 장치 마운트

### Loop 장치의 개념

**Loop Device**는 일반 파일을 블록 장치처럼 사용할 수 있도록 연결하는 가상 장치이다.

예를 들어 `/tmp/disk.img`와 같은 일반 이미지 파일에 파일 시스템을 생성한 후 `/dev/loop0`과 연결하면 일반 디스크처럼 마운트할 수 있다.

구조는 다음과 같다.

**이미지 파일 → Loop 장치 → 파일 시스템 → 마운트 포인트**

### 이미지 파일을 Loop 장치로 연결

`dd`로 이미지 파일을 생성하고 `losetup`으로 Loop 장치에 연결한다.

| 명령어 | 설명 |
|---|---|
| `dd if=/dev/zero of=/tmp/disk.img bs=1M count=100` | 100MiB 이미지 파일 생성 |
| `losetup -f /tmp/disk.img` | 사용 가능한 Loop 장치에 연결 |
| `losetup -fP /tmp/disk.img` | Loop 장치 연결 및 파티션 스캔 |
| `losetup -a` | 연결된 Loop 장치 확인 |
| `lsblk` | Loop 장치 및 파티션 확인 |

### 자주 사용하는 사용 예제

100MiB 이미지 파일을 생성한다.

`dd if=/dev/zero of=/tmp/disk.img bs=1M count=100`

사용 가능한 Loop 장치에 연결한다.

`losetup -fP /tmp/disk.img`

연결 상태를 확인한다.

`losetup -a`

### Loop 장치에 파일 시스템 생성

Loop 장치에 직접 파일 시스템을 생성한 후 마운트할 수 있다.

| 명령어 | 설명 |
|---|---|
| `mkfs.ext4 /dev/loop0` | Loop 장치에 EXT4 생성 |
| `mkdir -p /mnt/testdisk` | 마운트 포인트 생성 |
| `mount /dev/loop0 /mnt/testdisk` | Loop 장치 마운트 |
| `df -hT /mnt/testdisk` | 마운트 상태 확인 |

### 자주 사용하는 사용 예제

Loop 장치에 EXT4 파일 시스템을 생성한다.

`mkfs.ext4 /dev/loop0`

마운트한다.

`mkdir -p /mnt/testdisk`

`mount /dev/loop0 /mnt/testdisk`

사용량을 확인한다.

`df -hT /mnt/testdisk`

### Loop 장치 해제

Loop 장치를 해제하려면 먼저 해당 파일 시스템을 언마운트해야 한다.

| 순서 | 명령어 | 설명 |
|---:|---|---|
| 1 | `umount /mnt/testdisk` | 파일 시스템 언마운트 |
| 2 | `losetup -d /dev/loop0` | Loop 장치 연결 해제 |
| 3 | `rm -f /tmp/disk.img` | 이미지 파일 삭제 |

### 자주 사용하는 사용 예제

Loop 파일 시스템을 정리한다.

`umount /mnt/testdisk`

`losetup -d /dev/loop0`

`rm -f /tmp/disk.img`

---

## 12. 마운트 문제 해결

### 마운트 상태 확인

마운트 문제가 발생하면 다음 순서로 상태를 확인한다.

| 명령어 | 확인 내용 |
|---|---|
| `lsblk -f` | 장치, 파일 시스템, UUID |
| `blkid` | UUID 및 파일 시스템 타입 |
| `findmnt` | 현재 마운트 구조 |
| `mount` | 마운트 옵션 |
| `df -hT` | 파일 시스템 사용량 |
| `cat /etc/fstab` | 영구 마운트 설정 |
| `findmnt --verify` | fstab 설정 검증 |

### target is busy 오류

`umount` 실행 시 다음과 같은 오류가 발생할 수 있다.

**`target is busy`**

주요 원인은 다음과 같다.

- 현재 셸이 마운트 포인트 안에 있음
- 프로세스가 해당 디렉토리의 파일을 사용 중
- 서비스가 해당 파일 시스템을 사용 중
- 열린 파일 또는 작업 디렉토리가 존재

### 자주 사용하는 사용 예제

현재 작업 디렉토리를 빠져나온다.

`cd`

사용 중인 프로세스를 확인한다.

`fuser -vm /mnt/data`

열린 파일을 확인한다.

`lsof +D /mnt/data`

프로세스를 정리한 후 언마운트한다.

`umount /mnt/data`

### /etc/fstab 오류

부팅 과정에서 `/etc/fstab`의 오류가 발생하면 시스템이 정상적으로 부팅되지 않거나 **긴급 셸(sulogin)** 환경으로 진입할 수 있다.

주요 원인은 다음과 같다.

| 원인 | 예 |
|---|---|
| 잘못된 장치명 | 존재하지 않는 `/dev/...` |
| 잘못된 UUID | 실제 UUID와 불일치 |
| 잘못된 파일 시스템 타입 | XFS 장치에 EXT4 지정 |
| 존재하지 않는 마운트 포인트 | 디렉토리 미생성 |
| 잘못된 옵션 | 지원하지 않는 마운트 옵션 |
| 잘못된 필드 수 | `/etc/fstab` 문법 오류 |

### Rescue 환경에서의 점검

`/etc/fstab` 오류로 정상 부팅되지 않는 경우 긴급 환경에서 파일을 수정해야 할 수 있다.

일반적인 처리 과정은 다음과 같다.

1. 긴급 셸로 진입
2. 루트 파일 시스템을 쓰기 가능 상태로 확인
3. `/etc/fstab` 확인
4. 잘못된 항목 수정 또는 주석 처리
5. `systemctl daemon-reload`
6. `mount -a`로 검증
7. 정상 부팅

루트 파일 시스템이 읽기 전용으로 마운트된 경우 환경에 따라 다음과 같이 다시 읽기/쓰기로 변경할 수 있다.

`mount -o remount,rw /`

### 자주 사용하는 사용 예제

`/etc/fstab`을 확인한다.

`vi /etc/fstab`

systemd가 변경된 내용을 다시 읽도록 한다.

`systemctl daemon-reload`

fstab 설정을 테스트한다.

`mount -a`

마운트 상태를 확인한다.

`df -hT`

---

## 13. 디스크 추가 작업 자동화

### 자동화 작업 순서

반복적인 디스크 추가 작업은 Shell Script로 자동화할 수 있다.

일반적인 작업 순서는 다음과 같다.

**파티션 생성 → 파일 시스템 생성 → 마운트 포인트 생성 → `/etc/fstab` 등록 → 마운트 → 검증**

| 단계 | 주요 명령 |
|---|---|
| 파티션 | `parted` |
| 파일 시스템 | `mkfs.xfs`, `mkfs.ext4` |
| 디렉토리 | `mkdir -p` |
| 영구 설정 | `/etc/fstab` |
| 적용 | `mount -a` |
| systemd 반영 | `systemctl daemon-reload` |
| 검증 | `df -hT`, `lsblk -f`, `findmnt` |

### 스크립트 작성 시 주의사항

스크립트에서 작업 실패를 감지하기 위해 다음과 같은 설정을 사용할 수 있다.

| 구문 | 설명 |
|---|---|
| `set -e` | 명령 실패 시 스크립트 종료 |
| `echo` | 작업 상태 메시지 출력 |
| `mkdir -p` | 필요한 상위 디렉토리를 함께 생성 |
| `>>` | 파일 끝에 내용 추가 |
| `&&` | 앞 명령이 성공한 경우 다음 명령 실행 |

자동화 스크립트에서 `/etc/fstab`에 동일한 항목을 반복 추가하면 **중복 마운트 설정**이 발생할 수 있으므로 기존 설정을 확인한 후 추가해야 한다.

또한 실제 운영 환경에서는 장치명을 하드코딩하기보다 **UUID 또는 LABEL 기반 설정**을 사용하는 것이 안전하다.

### 자주 사용하는 사용 예제

스크립트에서 오류 발생 시 즉시 종료한다.

`set -e`

마운트 포인트를 생성한다.

`mkdir -p /mnt/data`

fstab 설정을 추가한 후 적용한다.

`systemctl daemon-reload`

`mount -a`

작업 결과를 확인한다.

`df -hT /mnt/data`

---

## 14. 핵심 명령어 및 옵션 정리

| 명령어 | 주요 용도 | 핵심 옵션 |
|---|---|---|
| `mount` | 파일 시스템 마운트 및 상태 확인 | `-t`, `-o`, `-a`, `-r`, `-w` |
| `umount` | 파일 시스템 언마운트 | `-a` |
| `findmnt` | 마운트 구조 확인 | `--verify` |
| `df` | 파일 시스템 사용량 확인 | `-h`, `-T`, `-t`, `-x` |
| `lsblk` | 블록 장치 확인 | `-f` |
| `blkid` | UUID 및 파일 시스템 정보 확인 | - |
| `parted` | 파티션 생성 및 관리 | `-s`, `mklabel`, `mkpart` |
| `mkfs.ext4` | EXT4 생성 | - |
| `mkfs.xfs` | XFS 생성 | `-f` |
| `mount --bind` | Bind Mount | `--bind`, `--rbind` |
| `fuser` | 파일 시스템 사용 프로세스 확인 | `-m`, `-v`, `-k` |
| `lsof` | 열린 파일 확인 | `+D` |
| `systemctl` | systemd 설정 반영 | `daemon-reload` |
| `showmount` | NFS export 확인 | `-e` |
| `exportfs` | NFS export 관리 | `-v`, `-r`, `-a` |
| `smbclient` | SMB 공유 확인 | `-L`, `-U` |
| `mount.cifs` | CIFS 마운트 | `-o` |
| `losetup` | Loop 장치 연결/해제 | `-f`, `-P`, `-a`, `-d` |
| `dd` | 이미지 파일 생성 | `if`, `of`, `bs`, `count` |

### 마운트 옵션 핵심 정리

| 옵션 | 핵심 의미 | 주요 활용 |
|---|---|---|
| `defaults` | 기본 마운트 옵션 | 일반적인 파일 시스템 |
| `rw` | 읽기/쓰기 | 일반 데이터 저장 |
| `ro` | 읽기 전용 | CD/DVD, 보호가 필요한 데이터 |
| `remount` | 기존 마운트 옵션 변경 | `ro`/`rw` 전환 |
| `noatime` | atime 갱신 방지 | 메타데이터 쓰기 감소 |
| `nosuid` | SetUID/SetGID 제한 | 보안 강화 |
| `noexec` | 실행 파일 직접 실행 제한 | 보안 강화 |
| `nodev` | 장치 파일 해석 제한 | 보안 강화 |
| `auto` | `mount -a` 대상 | 자동 마운트 |
| `noauto` | `mount -a` 제외 | 수동 마운트 |
| `user` | 일반 사용자 마운트 허용 | 사용자별 이동식 미디어 |
| `nouser` | 일반 사용자 마운트 제한 | 기본적인 관리 환경 |
| `sync` | 동기 I/O | 데이터 즉시 반영이 필요한 환경 |
| `async` | 비동기 I/O | 일반적인 기본 동작 |

### 파일 시스템별 주요 마운트 방법

| 파일 시스템 | 주요 방식 |
|---|---|
| **XFS** | `mount -t xfs DEVICE MOUNT_POINT` |
| **EXT4** | `mount -t ext4 DEVICE MOUNT_POINT` |
| **ISO9660** | `mount -t iso9660 -o ro DEVICE MOUNT_POINT` |
| **FAT32** | `mount -t vfat DEVICE MOUNT_POINT` |
| **NTFS** | `mount -t ntfs DEVICE MOUNT_POINT` 또는 NTFS 지원 도구 사용 |
| **tmpfs** | `mount -t tmpfs -o size=SIZE none MOUNT_POINT` |
| **NFS** | `mount -t nfs SERVER:/SHARE MOUNT_POINT` |
| **CIFS** | `mount -t cifs //SERVER/SHARE MOUNT_POINT` |
| **Loop** | `losetup`으로 장치 연결 후 일반 장치처럼 마운트 |

### 디스크 추가 작업 핵심 순서

**① 디스크 인식**

`lsblk`

**② 파티션 생성**

`parted`

**③ 파일 시스템 생성**

`mkfs.xfs DEVICE`

**④ 마운트 포인트 생성**

`mkdir -p /mnt/data`

**⑤ 임시 마운트**

`mount DEVICE /mnt/data`

**⑥ UUID 확인**

`blkid DEVICE`

**⑦ `/etc/fstab` 등록**

`UUID=UUID_VALUE /mnt/data xfs defaults 0 0`

**⑧ systemd 설정 반영**

`systemctl daemon-reload`

**⑨ fstab 테스트**

`mount -a`

**⑩ 최종 확인**

`df -hT`

`lsblk -f`

`findmnt`

### 문제 발생 시 점검 순서

**마운트 실패**

`lsblk -f`

`blkid`

`mount`

`findmnt`

`df -hT`

**언마운트 실패**

`fuser -vm /mnt/data`

`lsof +D /mnt/data`

`umount /mnt/data`

**fstab 오류**

`vi /etc/fstab`

`systemctl daemon-reload`

`findmnt --verify`

`mount -a`

**부팅 실패**

`/etc/fstab` 오류 확인 → 긴급/Rescue 환경 진입 → 잘못된 설정 수정 → `mount -a`로 검증

### 최종 핵심 정리

- **`mount` → 현재 파일 시스템 마운트**
- **`umount` → 파일 시스템 마운트 해제**
- **`/etc/fstab` → 부팅 시 사용할 영구 마운트 설정**
- **`mount -a` → `/etc/fstab` 기반 마운트 테스트 및 적용**
- **`systemctl daemon-reload` → 변경된 `/etc/fstab`을 systemd에 다시 반영**
- **`findmnt --verify` → `/etc/fstab` 설정 검증**
- **`lsblk -f`, `blkid` → 장치 및 UUID 확인**
- **`df -hT` → 파일 시스템 사용량과 타입 확인**
- **`ro` → 읽기 전용**
- **`remount` → 기존 마운트 옵션 변경**
- **`noatime` → 접근 시간 갱신 방지**
- **`nosuid` → SetUID/SetGID 제한**
- **`noexec` → 실행 파일 직접 실행 제한**
- **`nodev` → 장치 파일 해석 제한**
- **XFS/EXT4 → 로컬 파일 시스템**
- **NFS → Linux/Unix 네트워크 파일 공유**
- **CIFS/SMB → Windows/SMB 네트워크 파일 공유**
- **tmpfs → 메모리 기반 임시 파일 시스템**
- **Loop Device → 이미지 파일을 블록 장치처럼 사용**
- **`fuser`, `lsof` → 사용 중인 파일 시스템의 프로세스 확인**
- **마운트 포인트는 일반적으로 빈 디렉토리를 사용하는 것이 안전**
- **운영 환경의 `/etc/fstab`은 UUID 또는 LABEL 기반으로 구성하는 것이 안전**
