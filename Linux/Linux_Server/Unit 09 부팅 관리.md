# 부팅 과정 및 systemd 관리

## 목차

1. [리눅스 부팅 과정](#1-리눅스-부팅-과정)
	- [Firmware 단계](#firmware-단계)
	- [Boot Loader 단계](#boot-loader-단계)
	- [GRUB2 구성 및 관리](#grub2-구성-및-관리)
	- [grubby를 이용한 커널 관리](#grubby를-이용한-커널-관리)
	- [Kernel 단계](#kernel-단계)
	- [systemd 단계](#systemd-단계)
	- [부팅 과정에서 자주 사용하는 명령어](#부팅-과정에서-자주-사용하는-명령어)

2. [systemd 데몬과 서비스 제어](#2-systemd-데몬과-서비스-제어)
	- [systemd 데몬](#systemd-데몬)
	- [systemd Unit](#systemd-unit)
	- [systemctl 기본 사용법](#systemctl-기본-사용법)
	- [서비스 제어](#서비스-제어)
	- [서비스 상태 확인](#서비스-상태-확인)
	- [서비스 Enable/Disable 원리](#서비스-enabledisable-원리)
	- [서비스 장애 처리](#서비스-장애-처리)
	- [Unit 의존성](#unit-의존성)
	- [서비스 Mask/Unmask](#서비스-maskunmask)
	- [자주 사용하는 systemctl 명령어](#자주-사용하는-systemctl-명령어)

3. [대상(Target)](#3-대상target)
	- [Target의 개념](#target의-개념)
	- [주요 Target](#주요-target)
	- [Target 확인 및 변경](#target-확인-및-변경)
	- [GRUB에서 Target 변경](#grub에서-target-변경)
	- [부팅 장애 시 디버그 쉘](#부팅-장애-시-디버그-쉘)
	- [rd.break를 이용한 파일시스템 복구](#rdbreak를-이용한-파일시스템-복구)
	- [init=/bin/bash를 이용한 root 암호 복구](#initbinbash를-이용한-root-암호-복구)
	- [emergency.target을 이용한 장애 처리](#emergencytarget을-이용한-장애-처리)
	- [rescue.target을 이용한 장애 처리](#rescuetarget을-이용한-장애-처리)
	- [SELinux 재레이블링](#selinux-재레이블링)

4. [부팅 과정과 장애 처리](#4-부팅-과정과-장애-처리)
	- [GRUB 암호 설정](#grub-암호-설정)
	- [GRUB 복구](#grub-복구)
	- [grub.cfg 복구](#grubcfg-복구)
	- [Kernel 장애 처리](#kernel-장애-처리)
	- [rc.local 활용](#rclocal-활용)
	- [systemd 서비스 등록](#systemd-서비스-등록)
	- [/etc/fstab 장애 처리](#etcfstab-장애-처리)
	- [부팅 장애 처리 핵심 정리](#부팅-장애-처리-핵심-정리)


## 1. 리눅스 부팅 과정

### 부팅 과정 개요

리눅스 시스템은 전원이 꺼진 상태에서 **로그인 가능한 실행 상태**까지 하드웨어와 소프트웨어가 순차적으로 초기화되면서 부팅된다.

| 단계 | 구성 요소 | 주요 역할 |
|---|---|---|
| Phase 0 | Power ON | 시스템에 전원을 공급 |
| Phase 1 | **Firmware** | BIOS 또는 UEFI가 하드웨어 초기화 및 부팅 장치 검색 |
| Phase 2 | **Boot Loader** | GRUB이 커널과 initramfs를 메모리에 적재 |
| Phase 3 | **Kernel** | 하드웨어 초기화, initramfs 실행 및 systemd로 제어권 전달 |
| Phase 4 | **systemd** | Target과 Unit을 기반으로 운영체제 초기화 및 서비스 실행 |

### Firmware 단계

시스템 전원이 공급되면 **BIOS 또는 UEFI Firmware**가 실행된다.

주요 작업은 다음과 같다.

- **POST(Power On Self Test)** 수행
- CPU, Memory, 그래픽 장치, 키보드, 마우스 등의 하드웨어 검사
- 하드웨어 초기화
- 부팅 가능한 장치 검색
- 검색된 장치에서 Boot Loader 실행
- BIOS/UEFI 설정 화면 진입 지원

| 구성 요소 | 설명 |
|---|---|
| BIOS | 기존 Firmware 방식 |
| UEFI | 최신 Firmware 방식 |
| POST | 전원 공급 후 하드웨어 자체 진단 과정 |
| MBR | BIOS 방식에서 디스크의 초기 부팅 영역 |
| Boot Device | 운영체제를 부팅할 장치 |

### Boot Loader 단계

Linux의 대표적인 Boot Loader는 **GRUB2**이다.

GRUB의 주요 역할은 다음과 같다.

- 부팅할 커널 선택
- 커널 이미지 로드
- **initramfs** 이미지 로드
- 커널 명령줄 전달
- 커널로 제어권 전달

CentOS Stream 계열에서는 **GRUB2**를 기본 Boot Loader로 사용한다.

| Firmware | 주요 GRUB 구성 파일 |
|---|---|
| BIOS | `/boot/grub2/grub.cfg` |
| UEFI | `/boot/efi/EFI/{redhat|centos}/grub.cfg` |

### GRUB2 구성 및 관리

GRUB2의 주요 설정은 `/etc/default/grub` 및 `/etc/grub.d/`를 기반으로 구성된다.

구성 흐름은 다음과 같다.

`/etc/default/grub` + `/etc/grub.d/*` → `grub2-mkconfig` → GRUB 설정 파일

| 항목 | 역할 |
|---|---|
| `/etc/default/grub` | GRUB 기본 환경 설정 |
| `/etc/grub.d/` | GRUB 메뉴 생성용 스크립트 |
| `/boot/grub2/grub.cfg` | BIOS 환경의 최종 GRUB 설정 |
| `/boot/efi/EFI/{redhat\|centos}/grub.cfg` | UEFI 환경의 GRUB 설정 |
| `grub2-mkconfig` | GRUB 설정 파일 생성 |
| `GRUB_TIMEOUT` | GRUB 메뉴 대기 시간 |
| `GRUB_DEFAULT` | 기본 부팅 항목 |
| `GRUB_CMDLINE_LINUX` | 커널 부팅 매개변수 |

자주 사용하는 명령어:

`grub2-mkconfig -o /boot/grub2/grub.cfg`

GRUB 메뉴 대기 시간을 변경하는 경우 `/etc/default/grub`의 `GRUB_TIMEOUT` 값을 수정한 후 `grub2-mkconfig`를 실행한다.

### GRUB과 BLS

자료에서는 `/boot/loader/entries/` 아래의 **BLS(Boot Loader Specification)** 항목도 GRUB 부팅 항목 구성에 사용되는 것으로 설명한다.

| 구성 요소 | 역할 |
|---|---|
| `/boot/loader/entries/` | 커널별 Boot Loader 항목 저장 |
| `*.conf` | 커널, initramfs, 부팅 옵션 등의 정보 저장 |
| `vmlinuz-*` | Linux Kernel 이미지 |
| `initramfs-*` | 초기 부팅에 필요한 드라이버와 초기화 환경 |

### initramfs

**initramfs**는 커널이 실제 Root 파일시스템으로 전환하기 전에 사용하는 초기 파일시스템이다.

주요 역할:

- 초기 부팅에 필요한 Kernel Module 제공
- 저장장치 및 파일시스템 초기화 지원
- 초기 Root 파일시스템을 `/sysroot`에 마운트
- 초기 systemd Unit 실행
- 실제 Root 파일시스템으로 Pivot 수행

| 항목 | 설명 |
|---|---|
| `/etc/dracut.conf.d/` | initramfs 구성 |
| `dracut` | initramfs 생성 및 갱신 |
| `lsinitrd` | initramfs 내용 확인 |
| `/boot/initramfs-*` | initramfs 이미지 |

### grubby를 이용한 커널 관리

**grubby**는 GRUB2의 커널 부팅 항목과 커널 매개변수를 CLI에서 조회하고 수정하는 도구이다.

| 명령어 | 설명 |
|---|---|
| `grubby --info=ALL` | 모든 커널 정보 확인 |
| `grubby --default-kernel` | 기본 커널 확인 |
| `grubby --set-default <KERNEL>` | 기본 커널 변경 |
| `grubby --default-index` | 기본 커널 인덱스 확인 |
| `grubby --info <KERNEL>` | 특정 커널 정보 확인 |
| `grubby --update-kernel=ALL --args="quiet splash"` | 모든 커널에 매개변수 추가 |
| `grubby --update-kernel=<KERNEL> --args="quiet splash"` | 특정 커널에 매개변수 추가 |
| `grubby --update-kernel=ALL --remove-args="quiet"` | 모든 커널에서 매개변수 삭제 |
| `cat /proc/cmdline` | 현재 실행 중인 커널의 매개변수 확인 |

### Kernel 단계

GRUB은 Kernel과 initramfs를 메모리에 적재한 후 제어권을 **Kernel**로 전달한다.

Kernel 단계의 주요 작업:

1. initramfs에서 필요한 드라이버 초기화
2. `/sbin/init` 실행
3. initramfs의 `initrd.target` 처리
4. Root 파일시스템을 `/sysroot`에 마운트
5. 실제 Root 파일시스템으로 Pivot
6. 디스크에 설치된 systemd 실행
7. systemd 단계로 제어권 전달

| 항목 | 설명 |
|---|---|
| `/boot/vmlinuz-*` | Kernel 이미지 |
| `/boot/initramfs-*` | 초기 부팅용 파일시스템 |
| `/etc/sysctl.conf` | Kernel 파라미터 설정 |
| `/etc/sysctl.d/` | Kernel 파라미터 추가 설정 |
| `/sysroot` | 초기 부팅 단계에서 실제 Root 파일시스템이 마운트되는 위치 |

### systemd 단계

Kernel은 **systemd**를 실행하고 systemd는 기본 Target을 찾아 시스템을 초기화한다.

현재 CentOS 계열에서는 과거의 `init` 및 `runlevel` 대신 **systemd와 target**을 사용한다.

| 과거 개념 | 현재 개념 |
|---|---|
| `init` | **systemd** |
| `/etc/inittab` | systemd Unit 및 Target |
| runlevel 3 | `multi-user.target` |
| runlevel 5 | `graphical.target` |

### 부팅 과정에서 자주 사용하는 명령어

| 명령어 | 용도 |
|---|---|
| `reboot` | 시스템 재부팅 |
| `dmesg` | Kernel 부팅 메시지 확인 |
| `journalctl -xn` | 최근 Journal 메시지 확인 |
| `journalctl -b` | 현재 부팅의 Journal 확인 |
| `cat /proc/cmdline` | 현재 Kernel 부팅 옵션 확인 |
| `ls /boot` | Kernel 및 initramfs 관련 파일 확인 |

### 자주 사용하는 사용 예제

`dmesg | grep -i cpu` : 부팅 과정의 CPU 관련 메시지를 검색한다.

`dmesg | grep -i mem` : 메모리 관련 부팅 메시지를 검색한다.

`dmesg | grep -i version` : Kernel 버전 관련 메시지를 검색한다.

`journalctl -b` : 현재 부팅 과정에서 기록된 Journal을 확인한다.


## 2. systemd 데몬과 서비스 제어

### systemd 데몬

**systemd**는 Linux 시스템의 초기화와 서비스 관리를 담당하는 시스템 및 서비스 관리자이다.

Kernel이 실행하는 첫 번째 사용자 공간 프로세스이며 일반적으로 **PID 1**을 가진다.

| 특징 | 설명 |
|---|---|
| PID | 일반적으로 `1` |
| 초기화 | 시스템 부팅 과정의 초기화 담당 |
| 서비스 관리 | 서비스의 시작, 중지, 재시작 관리 |
| 병렬화 | 서비스 의존성을 고려하여 여러 서비스를 병렬 실행 |
| 의존성 관리 | Unit 간 의존성 자동 처리 |
| On-demand 실행 | 필요할 때 서비스를 실행할 수 있음 |
| CGroup | 관련 프로세스를 그룹 단위로 관리 |

확인 명령어:

`pstree | head`

`ps -ef | head`

### systemd Unit

**Unit**은 systemd가 관리하는 대상에 대한 추상적인 개념이며 Unit 파일을 통해 정의된다.

| Unit 유형 | 확장자 | 역할 |
|---|---|---|
| Service | `.service` | 시스템 서비스 |
| Socket | `.socket` | IPC Socket 및 Socket Activation |
| Target | `.target` | 여러 Unit을 묶은 시스템 상태 |
| Path | `.path` | 파일 또는 디렉터리 변경 감시 |
| Device | `.device` | 장치 |
| Mount | `.mount` | 파일시스템 Mount |
| Automount | `.automount` | 자동 Mount |
| Swap | `.swap` | Swap 장치 |
| Timer | `.timer` | 예약 작업 |
| Slice | `.slice` | CGroup 계층 구성 |
| Scope | `.scope` | 외부에서 생성된 프로세스 그룹 |

### Unit 목록 확인

| 명령어 | 설명 |
|---|---|
| `systemctl` | 활성화된 Unit 확인 |
| `systemctl -a` | 활성/비활성 Unit 모두 확인 |
| `systemctl -t service` | Service Unit 확인 |
| `systemctl -t socket` | Socket Unit 확인 |
| `systemctl -t target` | Target Unit 확인 |
| `systemctl list-unit-files` | 설치된 Unit 파일 확인 |
| `systemctl list-unit-files -t service` | 설치된 Service Unit 파일 확인 |
| `systemctl -t service -a` | 모든 Service Unit 확인 |

### Unit 상태의 의미

`systemctl` 출력에서는 **UNIT, LOAD, ACTIVE, SUB, DESCRIPTION** 등의 상태를 확인할 수 있다.

| 항목 | 의미 |
|---|---|
| UNIT | Unit 이름 |
| LOAD | Unit 설정이 정상적으로 로드되었는지 여부 |
| ACTIVE | Unit의 상위 수준 활성화 상태 |
| SUB | Unit의 세부 실행 상태 |
| DESCRIPTION | Unit에 대한 설명 |

`systemctl list-units`와 `systemctl list-unit-files`는 목적이 다르다.

| 명령어 | 확인 대상 |
|---|---|
| `systemctl list-units` | **현재 실행 상태** |
| `systemctl list-unit-files` | **부팅 시 활성화 상태** |

### 서비스 제어

| 명령어 | 설명 |
|---|---|
| `systemctl start <UNIT>` | 서비스 즉시 시작 |
| `systemctl stop <UNIT>` | 서비스 즉시 중지 |
| `systemctl restart <UNIT>` | 서비스 재시작 |
| `systemctl enable <UNIT>` | 부팅 시 자동 시작 설정 |
| `systemctl disable <UNIT>` | 부팅 시 자동 시작 해제 |
| `systemctl enable --now <UNIT>` | 부팅 자동 시작 설정 + 즉시 시작 |
| `systemctl disable --now <UNIT>` | 부팅 자동 시작 해제 + 즉시 중지 |
| `systemctl status <UNIT>` | 서비스 상태 확인 |

### 서비스 상태 확인

| 명령어 | 설명 |
|---|---|
| `systemctl is-active <UNIT>` | 현재 서비스 활성화 여부 확인 |
| `systemctl is-enabled <UNIT>` | 부팅 시 자동 시작 여부 확인 |
| `systemctl is-failed <UNIT>` | 서비스 실패 여부 확인 |
| `systemctl status <UNIT>` | 상세 서비스 상태 확인 |

| 상태 | 의미 |
|---|---|
| `loaded` | Unit 설정 파일이 정상적으로 로드됨 |
| `active (running)` | 서비스가 실행 중 |
| `active (exited)` | 작업을 정상적으로 완료하고 종료된 상태 |
| `active (waiting)` | 실행 중이며 특정 이벤트를 대기하는 상태 |
| `inactive` | 실행되고 있지 않은 상태 |
| `failed` | 서비스 실행 또는 종료에 실패한 상태 |
| `enabled` | 부팅 시 자동 시작 |
| `disabled` | 부팅 시 자동 시작하지 않음 |
| `static` | 직접 enable할 수 없으며 다른 Unit의 의존성 등을 통해 실행될 수 있음 |
| `masked` | 의도적으로 실행할 수 없도록 차단된 상태 |

### Enable/Disable 원리

`systemctl enable`은 Unit 자체를 실행하는 것이 아니라 **부팅 시 해당 Unit이 시작되도록 관련 링크를 생성**한다.

`systemctl disable`은 해당 링크를 제거한다.

따라서 다음 두 개념을 구분해야 한다.

| 상태 | 의미 |
|---|---|
| `active` | 현재 실행 중인가? |
| `enabled` | 부팅할 때 자동으로 시작하도록 설정되어 있는가? |

`systemctl enable --now <UNIT>`은 **enabled + active** 상태를 동시에 구성한다.

### 서비스 장애 처리

실패한 Unit은 `--failed` 옵션으로 확인할 수 있다.

| 명령어 | 설명 |
|---|---|
| `systemctl --failed` | 실패한 Unit 확인 |
| `systemctl --failed --all` | 실패한 Unit을 포함하여 모든 관련 Unit 확인 |
| `systemctl --failed --type=service` | 실패한 Service만 확인 |
| `systemctl list-jobs` | 현재 대기 중인 Job 확인 |

`systemctl --failed`는 **실패 상태**를 확인하는 명령이고, `systemctl list-jobs`는 **대기 중인 작업**을 확인하는 명령이다.

### Unit 의존성

systemd는 서비스 간의 **의존성 관계**를 분석하여 필요한 Unit을 순서에 맞게 실행한다.

| 명령어 | 설명 |
|---|---|
| `systemctl list-dependencies <UNIT>` | 해당 Unit이 의존하는 Unit 확인 |
| `systemctl list-dependencies <UNIT> --reverse` | 해당 Unit에 의존하는 Unit 확인 |
| `systemctl list-dependencies <UNIT> --after` | 이후 관계를 고려한 의존성 확인 |

대표적인 Target 의존 관계:

`sysinit.target` → `basic.target` → `multi-user.target` → `graphical.target`

### 서비스 Mask/Unmask

**Mask**는 특정 서비스가 실수로 시작되지 않도록 실행 자체를 차단하는 기능이다.

Mask하면 해당 Unit에 대한 `/dev/null` 링크가 생성된다.

| 명령어 | 설명 |
|---|---|
| `systemctl mask <UNIT>` | Unit 실행 차단 |
| `systemctl unmask <UNIT>` | Mask 해제 |
| `systemctl disable --now <UNIT>` | 자동 시작 해제 + 현재 서비스 중지 |

Mask 상태에서는 `start`나 `enable`을 수행해도 서비스가 정상적으로 시작되지 않는다.

### 자주 사용하는 systemctl 명령어

`systemctl status <UNIT>` : 서비스의 상세 상태를 확인한다.

`systemctl restart <UNIT>` : 실행 중인 서비스를 재시작한다.

`systemctl enable --now <UNIT>` : 서비스의 부팅 자동 시작과 현재 실행을 한 번에 설정한다.

`systemctl disable --now <UNIT>` : 서비스의 부팅 자동 시작을 해제하고 현재 실행 중인 서비스도 중지한다.

`systemctl --failed --type=service` : 장애가 발생한 Service Unit을 확인한다.

`systemctl list-dependencies <UNIT>` : 특정 Unit의 의존성을 확인한다.


## 3. 대상(Target)

### Target의 개념

**Target**은 특정 시스템 상태를 만들기 위해 필요한 **Unit들의 집합**이다.

systemd는 기본 Target을 기준으로 필요한 서비스와 Unit을 실행한다.

대표적인 부팅 흐름:

`sysinit.target` → `basic.target` → `multi-user.target` → `graphical.target`

### 주요 Target

| Target | 설명 | 과거 Runlevel |
|---|---|---|
| `graphical.target` | 그래픽 기반 다중 사용자 환경 | runlevel 5 |
| `multi-user.target` | 텍스트 기반 다중 사용자 환경 | runlevel 3 |
| `rescue.target` | 단일 사용자 기반 복구 환경 | 단일 사용자 모드 |
| `emergency.target` | 최소한의 환경에서 장애를 복구하기 위한 환경 | - |

### Target 확인 및 변경

| 명령어 | 설명 |
|---|---|
| `systemctl -t target` | Target Unit 확인 |
| `systemctl get-default` | 부팅 시 기본 Target 확인 |
| `who -r` | 현재 Runlevel/Target 관련 상태 확인 |
| `systemctl isolate multi-user.target` | 현재 시스템을 텍스트 기반 환경으로 전환 |
| `systemctl isolate graphical.target` | 현재 시스템을 그래픽 기반 환경으로 전환 |
| `systemctl set-default multi-user.target` | 부팅 기본 Target을 텍스트 환경으로 설정 |
| `systemctl set-default graphical.target` | 부팅 기본 Target을 그래픽 환경으로 설정 |

**`isolate`와 `set-default`의 차이**:

| 명령 | 적용 시점 |
|---|---|
| `isolate` | **현재 실행 중인 시스템** |
| `set-default` | **다음 부팅부터 적용** |

### GRUB에서 Target 변경

GRUB 메뉴에서 Kernel 항목을 선택한 후 `e`를 눌러 Kernel 명령줄을 수정할 수 있다.

`linux`로 시작하는 Kernel 라인에 다음과 같은 매개변수를 추가한다.

| Kernel 매개변수 | 목적 |
|---|---|
| `systemd.unit=multi-user.target` | 텍스트 기반 Target으로 부팅 |
| `systemd.unit=graphical.target` | 그래픽 기반 Target으로 부팅 |
| `systemd.unit=rescue.target` | Rescue 환경으로 부팅 |
| `systemd.unit=emergency.target` | Emergency 환경으로 부팅 |

수정 후 `<Ctrl> + <X>`를 사용하여 부팅한다.

### 부팅 장애 시 디버그 쉘

부팅 과정의 서비스 문제를 분석하기 위해 **debug-shell**을 사용할 수 있다.

| 방법 | 내용 |
|---|---|
| 서비스 활성화 | `systemctl enable debug-shell.service` |
| 재부팅 | `reboot` |
| Debug Shell 접근 | `<Ctrl> + <Alt> + <F9>` |
| GRUB 일회성 설정 | `systemd.debug-shell` |
| GRUB 수정 후 부팅 | `<Ctrl> + <X>` |
| 작업 완료 후 | `systemctl disable debug-shell.service` |

**주의:** Debug Shell은 인증되지 않은 Root Shell이 로컬 콘솔에 노출될 수 있으므로 장애 처리가 끝나면 반드시 비활성화해야 한다.

### rd.break를 이용한 파일시스템 복구

`rd.break`는 **systemd가 본격적으로 실행되기 전**, initramfs 단계에서 작업할 때 사용하는 Kernel 매개변수이다.

대표적인 용도:

- Root 파일시스템 점검
- `fsck` 또는 `xfs_repair` 수행
- 초기 Root 파일시스템 장애 처리

GRUB에서 `linux` Kernel 라인 끝에 `rd.break`를 추가한 후 `<Ctrl> + <X>`로 부팅한다.

| 상태 | 특징 |
|---|---|
| Root 위치 | `/sysroot` |
| 실제 Root 전환 | 아직 완료되지 않음 |
| `/etc/fstab` | 아직 읽지 않음 |
| 주요 작업 | 파일시스템 점검 및 복구 |

파일시스템 점검 시 Root 파일시스템이 마운트되어 있다면 해제한 후 점검한다.

`umount /sysroot`

`xfs_repair /dev/<ROOT_DEVICE>`

작업 후 `exit`하여 부팅 과정을 계속 진행한다.

### init=/bin/bash를 이용한 Root 암호 복구

`init=/bin/bash`는 systemd 대신 `/bin/bash`를 PID 1로 실행하도록 Kernel 명령줄을 변경하는 방법이다.

대표적인 용도:

- **Root 암호 복구**
- systemd가 시작되기 전 시스템 관리

GRUB의 `linux` 라인에 다음을 추가한다.

`rw init=/bin/bash`

부팅 후 Root 파일시스템을 확인하고 필요하면 다시 읽기/쓰기 모드로 마운트한다.

`mount | grep root`

`mount -o remount,rw /`

Root 암호 변경:

`passwd root`

SELinux 재레이블링이 필요한 경우:

`touch /.autorelabel`

작업 완료 후:

`exec /sbin/init`

### emergency.target을 이용한 장애 처리

`emergency.target`은 **최소한의 시스템 환경**에서 장애를 처리할 때 사용한다.

특징:

- Root 파일시스템이 Pivot된 상태
- Root 파일시스템은 초기 상태에서 읽기 전용으로 사용될 수 있음
- `/etc/fstab`이 아직 정상적으로 처리되지 않은 상태
- 다른 파일시스템이 마운트되지 않은 상태
- `sulogin`을 통한 Root 접근

GRUB의 Kernel 라인에 다음을 추가한다.

`systemd.unit=emergency.target`

필요한 경우 Root 파일시스템을 읽기/쓰기 모드로 다시 마운트한다.

`mount -o remount,rw /`

### rescue.target을 이용한 장애 처리

`rescue.target`은 **일반적인 시스템 장애 복구**에 사용할 수 있는 Target이다.

특징:

- systemd가 실행됨
- Root 파일시스템이 Pivot된 상태
- `/etc/fstab`이 처리된 상태
- 필요한 파일시스템이 마운트된 상태
- `sulogin`을 통한 Root 접근

GRUB의 Kernel 라인에 다음을 추가한다.

`systemd.unit=rescue.target`

### Emergency와 Rescue 비교

| 항목 | `emergency.target` | `rescue.target` |
|---|---|---|
| 목적 | 최소 환경의 장애 복구 | 일반적인 장애 복구 |
| systemd | 실행됨 | 실행됨 |
| `/etc/fstab` | 처리 전 | 처리됨 |
| Root FS | 초기에는 읽기 전용일 수 있음 | 읽기/쓰기 환경으로 사용 |
| 기타 파일시스템 | 대부분 마운트되지 않음 | 필요한 파일시스템 마운트 |
| 활용 | 초기 부팅 장애 | 일반적인 시스템 장애 |

### SELinux 재레이블링

부팅 시 SELinux Label을 다시 적용해야 하는 경우 Kernel 명령줄에 다음을 추가할 수 있다.

`autorelabel=1`

GRUB 메뉴에서 Kernel을 선택하고 `e`를 누른 후 `linux` 라인의 끝에 추가한다.

부팅 후 재레이블링 결과 확인:

`journalctl -b | grep -i relabel`

### 자주 사용하는 Target 전환 예제

`systemctl isolate multi-user.target` : 현재 GUI 환경을 텍스트 기반 다중 사용자 환경으로 전환한다.

`systemctl isolate graphical.target` : 현재 시스템을 그래픽 환경으로 전환한다.

`systemctl set-default multi-user.target` : 다음 부팅부터 텍스트 기반 환경으로 시작하도록 설정한다.

`systemctl set-default graphical.target` : 다음 부팅부터 그래픽 환경으로 시작하도록 설정한다.


## 4. 부팅 과정과 장애 처리

### GRUB 암호 설정

GRUB 메뉴를 사용하여 Kernel 명령줄을 수정하면 일반 사용자가 부팅 과정에서 Root 권한을 획득하는 방식의 공격이 가능할 수 있다.

이를 방지하기 위해 **GRUB 메뉴 편집에 암호를 설정**할 수 있다.

| 항목 | 설명 |
|---|---|
| `/etc/grub.d/01_users` | GRUB 사용자 인증 관련 설정 생성 |
| `/boot/grub2/user.cfg` | GRUB 암호 정보 저장 |
| `grub2-setpassword` | GRUB 암호 설정 |
| `GRUB2_PASSWORD` | 암호화된 GRUB 암호 변수 |

GRUB 암호 설정:

`grub2-setpassword`

암호 파일 확인:

`cat /boot/grub2/user.cfg`

GRUB 암호 삭제:

`rm -f /boot/grub2/user.cfg`

GRUB 메뉴에서 `e`를 사용하여 편집하려면 설정된 사용자 인증이 필요하다.

### GRUB 복구

GRUB 자체에 문제가 발생하면 증상에 따라 복구 방법을 구분한다.

| 장애 유형 | 대표적인 증상 | 복구 방법 |
|---|---|---|
| `grub.cfg` 이상 | GRUB 메뉴가 정상적으로 표시되지 않음 | `grub2-mkconfig` |
| GRUB 영역 이상 | `grub2>` 등의 GRUB 오류 | `grub2-install` |
| Kernel 이상 | Kernel Panic 또는 Kernel 부팅 실패 | 이전 Kernel 선택 |
| systemd 이상 | 서비스 또는 Target 초기화 실패 | `rescue.target`, `emergency.target` 등 |

### GRUB 영역 복구

디스크의 GRUB 영역에 문제가 발생한 경우 설치 미디어의 Rescue 환경으로 부팅하여 기존 운영체제 환경으로 `chroot`한 뒤 GRUB을 복구한다.

주요 작업 흐름:

`df -hT`

`fdisk -l /dev/<DISK>`

`chroot /mnt/sysroot`

`cat /boot/grub2/grub.cfg`

`grub2-install /dev/<DISK>`

`exit`

`exit`

**핵심:** `grub2-install`은 디스크의 **GRUB 부트 영역 자체**를 복구할 때 사용한다.

### grub.cfg 복구

`/boot/grub2/grub.cfg`의 내용에 문제가 있는 경우 Rescue 환경에서 기존 Root 파일시스템으로 `chroot`한 후 설정 파일을 다시 생성한다.

주요 명령:

`chroot /mnt/sysroot`

`cat /boot/grub2/grub.cfg`

`grub2-mkconfig -o /boot/grub2/grub.cfg`

`exit`

`exit`

**구분**

| 명령어 | 주된 역할 |
|---|---|
| `grub2-mkconfig` | GRUB **설정 파일 생성** |
| `grub2-install` | 디스크의 **GRUB 부트 영역 설치/복구** |
| `grubby` | Kernel 부팅 항목 및 Kernel 매개변수 **조회/수정** |

### Kernel 장애 처리

Kernel 업데이트나 Kernel 파일 손상으로 **Kernel Panic**이 발생할 수 있다.

가장 우선적으로 고려할 방법은 GRUB 메뉴에서 **이전 Kernel을 선택하여 부팅**하는 것이다.

| 항목 | 설명 |
|---|---|
| `vmlinuz-*` | Kernel 이미지 |
| `initramfs-*` | Kernel 초기화용 이미지 |
| 이전 Kernel | 장애 발생 시 복구용 부팅 항목 |
| Rescue Kernel | 일반 Kernel 장애 시 사용할 수 있는 복구용 Kernel |

부팅에 실패한 최신 Kernel 대신 정상적으로 동작하는 이전 Kernel을 GRUB 메뉴에서 선택한다.

정상 부팅 후 손상된 Kernel 파일을 복구하거나 해당 Kernel을 재설치한다.

### rc.local 활용

`/etc/rc.d/rc.local`은 과거의 부팅 스크립트 호환성을 위해 제공된다.

CentOS 계열의 systemd 환경에서는 일반적으로 **전용 systemd Unit을 작성하는 방식이 권장**된다.

| 항목 | 설명 |
|---|---|
| `/etc/rc.local` | `/etc/rc.d/rc.local`에 연결된 경로 |
| `/etc/rc.d/rc.local` | 부팅 시 실행할 호환성 스크립트 |
| `rc-local.service` | rc.local 실행을 담당하는 systemd Unit |
| 실행 권한 | `chmod +x /etc/rc.d/rc.local` 필요 |

확인:

`ls -l /etc/rc.local`

`ls -l /etc/rc.d/rc.local`

`systemctl status rc-local`

`systemctl list-unit-files | grep rc-local`

rc.local에 부팅 시 실행할 명령을 추가한 경우 실행 권한을 확인해야 한다.

`chmod +x /etc/rc.d/rc.local`

### systemd 서비스 등록

부팅 시 특정 프로그램이나 스크립트를 자동으로 실행해야 한다면 **systemd Service Unit**을 생성할 수 있다.

일반적인 Unit 파일 위치:

`/usr/lib/systemd/system/<SERVICE>.service`

주요 구성 영역:

| Section | 주요 내용 |
|---|---|
| `[Unit]` | Unit 설명 및 의존성 |
| `[Service]` | 실행할 프로그램과 서비스 동작 방식 |
| `[Install]` | 부팅 Target과의 연결 |

대표적인 Service 설정 항목:

| 설정 | 설명 |
|---|---|
| `Description=` | 서비스 설명 |
| `Type=` | 서비스 실행 방식 |
| `ExecStart=` | 서비스 시작 명령 |
| `ExecStop=` | 서비스 종료 명령 |
| `Restart=` | 서비스 장애 시 재시작 정책 |
| `WantedBy=` | 서비스가 연결될 Target |

서비스 파일을 생성하거나 수정한 후에는 systemd가 변경 내용을 다시 읽도록 해야 한다.

`systemctl daemon-reload`

서비스 등록 및 실행:

`systemctl enable <SERVICE>`

`systemctl start <SERVICE>`

또는:

`systemctl enable --now <SERVICE>`

서비스 확인:

`systemctl status <SERVICE>`

`systemctl list-unit-files | grep <SERVICE>`

### systemd 서비스 등록 구조

서비스 파일과 부팅 Target의 관계는 다음과 같이 이해할 수 있다.

`/usr/lib/systemd/system/<SERVICE>.service`

→ `systemctl enable <SERVICE>`

→ `/etc/systemd/system/multi-user.target.wants/<SERVICE>.service`

즉, `enable`은 부팅 시 해당 Target에서 서비스를 실행할 수 있도록 링크를 구성한다.

### /etc/fstab 장애 처리

`/etc/fstab` 설정이 잘못되면 부팅 과정에서 파일시스템 Mount에 실패하여 시스템이 정상적으로 부팅되지 않을 수 있다.

장애 발생 시 기본적인 처리 순서는 다음과 같다.

1. Root 암호로 유지보수 환경에 진입
2. Root 파일시스템의 읽기/쓰기 상태 확인
3. Root 파일시스템을 읽기/쓰기로 다시 Mount
4. `/etc/fstab` 오류 수정
5. systemd 설정 다시 로드
6. 부팅 과정 계속 진행

주요 명령:

`mount | grep root`

`mount -o remount,rw /`

`vi /etc/fstab`

`systemctl daemon-reload`

`exit`

### /etc/fstab 장애 처리 시 확인 사항

| 확인 항목 | 설명 |
|---|---|
| 장치 존재 여부 | 설정한 Device가 실제로 존재하는지 확인 |
| Mount Point | Mount 대상 디렉터리가 존재하는지 확인 |
| 파일시스템 종류 | 실제 파일시스템과 `fstab`의 타입이 일치하는지 확인 |
| UUID | UUID를 사용하는 경우 실제 UUID와 일치하는지 확인 |
| Mount 옵션 | `defaults` 등 옵션의 오류 여부 확인 |
| Dump/FSCK 필드 | 마지막 두 필드의 설정 확인 |
| Root FS 상태 | 수정 전 `rw` 상태로 전환 필요 여부 확인 |

보조 확인 명령:

`ls -l /dev/<DEVICE>`

`ls -ld <MOUNT_POINT>`

`blkid`

### 부팅 장애 처리 핵심 흐름

부팅이 어느 단계에서 중단되었는지 판단한 후 해당 단계에 맞는 복구 방법을 선택한다.

| 장애 단계 | 주요 문제 | 주요 대응 |
|---|---|---|
| Firmware | 하드웨어 또는 부팅 장치 인식 문제 | BIOS/UEFI 및 POST 확인 |
| Boot Loader | GRUB 영역 또는 설정 문제 | `grub2-install`, `grub2-mkconfig` |
| Kernel | Kernel Panic, Kernel 이미지 문제 | 이전 Kernel 부팅, Kernel 복구 |
| systemd | 서비스, Target, Mount 문제 | `systemctl`, `rescue.target`, `emergency.target` |
| `/etc/fstab` | 파일시스템 Mount 실패 | Root FS `rw` 전환 후 `fstab` 수정 |
| 서비스 | 특정 서비스 시작 실패 | `systemctl --failed`, `status`, `journalctl` |

### 부팅 장애 처리 핵심 명령어

| 명령어 | 핵심 용도 |
|---|---|
| `grub2-mkconfig -o /boot/grub2/grub.cfg` | GRUB 설정 재생성 |
| `grub2-install /dev/<DISK>` | GRUB 부트 영역 복구 |
| `grubby --info=ALL` | Kernel 목록 및 설정 확인 |
| `grubby --default-kernel` | 기본 Kernel 확인 |
| `dmesg` | Kernel 부팅 메시지 확인 |
| `journalctl -b` | 현재 부팅 로그 확인 |
| `systemctl --failed` | 실패한 Unit 확인 |
| `systemctl list-jobs` | 대기 중인 Job 확인 |
| `systemctl get-default` | 기본 Target 확인 |
| `systemctl isolate <TARGET>` | 현재 Target 변경 |
| `systemctl set-default <TARGET>` | 기본 Target 변경 |
| `systemctl daemon-reload` | systemd Unit 설정 다시 읽기 |
| `mount -o remount,rw /` | Root 파일시스템을 읽기/쓰기로 재마운트 |
| `chroot /mnt/sysroot` | Rescue 환경에서 설치된 Root 환경으로 전환 |
| `xfs_repair <DEVICE>` | XFS 파일시스템 복구 |

### 자주 사용하는 장애 처리 예제

**GRUB 설정 파일이 손상된 경우**

`chroot /mnt/sysroot` → `grub2-mkconfig -o /boot/grub2/grub.cfg` → `exit` → `exit`

**GRUB 부트 영역이 손상된 경우**

`chroot /mnt/sysroot` → `grub2-install /dev/<DISK>` → `exit` → `exit`

**Kernel 부팅에 실패한 경우**

GRUB 메뉴 → 이전 Kernel 선택 → 정상 부팅 → 손상된 Kernel 복구

**Root 파일시스템을 점검해야 하는 경우**

GRUB 메뉴 → `e` → `linux` 라인에 `rd.break` 추가 → `<Ctrl> + <X>` → `umount /sysroot` → `xfs_repair /dev/<ROOT_DEVICE>`

**Root 암호를 복구해야 하는 경우**

GRUB 메뉴 → `e` → `linux` 라인에 `rw init=/bin/bash` 추가 → `<Ctrl> + <X>` → `mount -o remount,rw /` → `passwd root` → `touch /.autorelabel` → `exec /sbin/init`

**일반적인 systemd 장애를 처리하는 경우**

GRUB 메뉴 → `e` → `linux` 라인에 `systemd.unit=rescue.target` 또는 `systemd.unit=emergency.target` 추가 → `<Ctrl> + <X>`

**서비스 장애를 확인하는 경우**

`systemctl --failed --type=service` → `systemctl status <UNIT>` → `journalctl -u <UNIT>`

**부팅 시 특정 서비스를 실행해야 하는 경우**

Service Unit 작성 → `systemctl daemon-reload` → `systemctl enable --now <SERVICE>`

**/etc/fstab 오류로 부팅이 중단된 경우**

Root 암호 입력 → `mount -o remount,rw /` → `/etc/fstab` 수정 → `systemctl daemon-reload` → `exit`

### 핵심 정리

**부팅 순서**

`Firmware → GRUB → Kernel → systemd`

**GRUB 관련 명령**

`grub2-mkconfig` = **GRUB 설정 파일 생성**

`grub2-install` = **GRUB 부트 영역 복구**

`grubby` = **Kernel 부팅 항목 및 매개변수 조회/수정**

**systemd 관련 명령**

`systemctl start` = **현재 서비스 시작**

`systemctl stop` = **현재 서비스 중지**

`systemctl restart` = **현재 서비스 재시작**

`systemctl enable` = **부팅 시 자동 시작**

`systemctl disable` = **부팅 시 자동 시작 해제**

`systemctl enable --now` = **부팅 자동 시작 + 현재 시작**

`systemctl --failed` = **실패한 Unit 확인**

**Target 관련 명령**

`systemctl isolate` = **현재 Target 변경**

`systemctl set-default` = **부팅 기본 Target 변경**

**장애 처리 핵심**

`rd.break` = **initramfs 단계에서 파일시스템 장애 처리**

`init=/bin/bash` = **systemd 대신 Bash 실행, Root 암호 복구 등에 활용**

`systemd.unit=emergency.target` = **최소 환경의 장애 처리**

`systemd.unit=rescue.target` = **일반적인 장애 처리**

`systemd.debug-shell` = **부팅 중 Debug Shell 사용**

`autorelabel=1` = **SELinux 재레이블링 강제 수행**
