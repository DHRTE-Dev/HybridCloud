# SWAP 관리

## 목차

- [1. 스왑(SWAP)의 개념](#1-스왑swap의-개념)
	- [스왑이란?](#스왑이란)
	- [페이징(Paging)과 스왑](#페이징paging과-스왑)
	- [스왑 사용 시 고려사항](#스왑-사용-시-고려사항)
	- [스왑 공간 크기 권장사항](#스왑-공간-크기-권장사항)
- [2. 스왑(SWAP) 확인](#2-스왑swap-확인)
	- [스왑 관련 정보 확인](#스왑-관련-정보-확인)
	- [free 명령어](#free-명령어)
	- [top 명령어](#top-명령어)
	- [/proc/swaps 파일](#proc_swaps-파일)
- [3. 스왑(SWAP) 관리](#3-스왑swap-관리)
	- [스왑 관리 명령어](#스왑-관리-명령어)
	- [스왑 파일 추가](#스왑-파일-추가)
	- [스왑 파티션 추가](#스왑-파티션-추가)
	- [스왑 LV 추가](#스왑-lv-추가)
- [4. 스왑(SWAP) 삭제](#4-스왑swap-삭제)
	- [스왑 파일 삭제](#스왑-파일-삭제)
	- [스왑 파티션 삭제](#스왑-파티션-삭제)
	- [스왑 LV 삭제](#스왑-lv-삭제)
- [5. 스왑 우선순위(Priority)](#5-스왑-우선순위priority)
	- [스왑 우선순위 개념](#스왑-우선순위-개념)
	- [pri 옵션](#pri-옵션)
	- [swapon의 우선순위 옵션](#swapon의-우선순위-옵션)
	- [동일 우선순위의 스왑 영역](#동일-우선순위의-스왑-영역)
- [6. 스왑 LV 확장 및 축소](#6-스왑-lv-확장-및-축소)
	- [스왑 LV 확장](#스왑-lv-확장)
	- [스왑 LV 축소](#스왑-lv-축소)
- [7. 실무에서의 SWAP 관리](#7-실무에서의-swap-관리)
	- [스왑 추가가 필요한 경우](#스왑-추가가-필요한-경우)
	- [스왑 방식 선택](#스왑-방식-선택)
- [8. 자주 사용하는 SWAP 명령 예제](#8-자주-사용하는-swap-명령-예제)

---

## 1. 스왑(SWAP)의 개념

### 스왑이란?

**SWAP**은 디스크에 존재하는 가상적인 메모리 공간으로, 물리 메모리인 **RAM의 부족분을 보완하는 공간**이다.

Linux에서는 **SWAP**, Windows에서는 일반적으로 **페이징 파일(Page File)**이라는 용어를 사용한다.

RAM이 부족해지면 운영체제는 RAM에 있는 **비활성 메모리 페이지**를 SWAP 영역으로 이동시킬 수 있다.

| 구분 | 설명 |
|---|---|
| RAM | 실제 물리 메모리 |
| SWAP | 디스크에 마련된 가상 메모리 영역 |
| File System | 일반적인 파일 저장 영역 |
| Paging | 메모리 페이지를 RAM과 디스크 사이에서 이동시키는 작업 |

SWAP은 RAM의 **대체재가 아니라 보조적인 메모리 공간**이다.

디스크의 접근 속도는 RAM보다 느리므로 SWAP 사용량이 지속적으로 증가하면 시스템 성능이 저하될 수 있다.

### 페이징(Paging)과 스왑

메모리가 부족하면 다음과 같은 과정이 발생할 수 있다.

**RAM 부족 → 비활성 페이지를 SWAP으로 이동 → Paging 증가 → 디스크 I/O 증가 → 시스템 성능 저하**

따라서 SWAP 사용량이 지속적으로 증가한다면 단순히 SWAP 공간을 늘리는 것뿐만 아니라 **RAM 사용량과 프로세스의 메모리 사용 상태를 함께 확인**해야 한다.

### 스왑 사용 시 고려사항

- SWAP은 **물리 메모리의 확장 기능**으로 사용할 수 있다.
- RAM보다 디스크 접근 속도가 느리므로 SWAP에 과도하게 의존하면 성능이 저하될 수 있다.
- SWAP은 다음과 같은 형태로 구성할 수 있다.
  - **스왑 파티션**
  - **스왑 파일**
  - **스왑 LV(Logical Volume)**
- 일반적인 시스템 운영에서는 충분한 RAM을 확보하는 것이 우선이다.
- SWAP 추가 여부는 단순한 RAM 크기뿐 아니라 **시스템의 실제 메모리 사용량과 워크로드**를 고려해야 한다.

### 스왑 공간 크기 권장사항

자료에서는 RHEL 가이드 기준의 SWAP 크기 권장사항을 다음과 같이 제시한다.

| RAM 크기 | 일반적인 SWAP 권장 크기 | 최대 절전 모드 사용 시 |
|---|---:|---:|
| 2GB 미만 | RAM × 2 | RAM × 3 |
| 2GB ~ 8GB | RAM과 동일 | RAM × 2 |
| 8GB ~ 64GB | 최소 4GB 이상 | RAM × 1.5 |
| 64GB 초과 | 워크로드에 따라 결정 | 워크로드에 따라 결정 |

최신 시스템에서는 RAM 용량만으로 SWAP 크기를 결정하기보다 **시스템의 워크로드와 메모리 사용 패턴**을 기준으로 결정하는 것이 중요하다.

---

## 2. 스왑(SWAP) 확인

### 스왑 관련 정보 확인

SWAP의 전체 크기와 사용량을 확인할 때는 다음 명령어를 사용할 수 있다.

| 명령어 | 설명 |
|---|---|
| `free` | RAM과 SWAP의 전체 용량 및 사용량 확인 |
| `free -h` | 사람이 읽기 쉬운 단위로 메모리 정보 표시 |
| `free -ht` | RAM과 SWAP 정보를 사람이 읽기 쉬운 단위로 표시 |
| `top` | 실행 중인 프로세스와 RAM/SWAP 사용량을 실시간으로 확인 |
| `cat /proc/swaps` | 현재 활성화된 SWAP 영역 확인 |
| `swapon --show` | 활성화된 SWAP 영역을 상세하게 표시 |
| `swapon -s` | 활성화된 SWAP 영역의 요약 정보 표시 |
| `cat /proc/meminfo` | 커널이 관리하는 메모리 관련 상세 정보 확인 |
| `lsblk` | 블록 장치와 파티션, LVM 등의 구조 확인 |

### free 명령어

`free` 명령어는 시스템의 **RAM 및 SWAP 사용량**을 확인하는 데 사용한다.

| 명령어 | 설명 |
|---|---|
| `free` | 메모리 정보를 기본 단위로 표시 |
| `free -h` | 사람이 읽기 쉬운 단위로 표시 |
| `free -t` | RAM과 SWAP을 합산한 Total 표시 |
| `free -ht` | `-h`와 `-t` 옵션을 함께 사용 |

주요 출력 항목은 다음과 같다.

| 항목 | 설명 |
|---|---|
| total | 전체 메모리 또는 SWAP 용량 |
| used | 사용 중인 용량 |
| free | 사용되지 않는 용량 |
| shared | 공유 메모리 |
| buff/cache | 버퍼 및 캐시 영역 |
| available | 실제로 사용할 수 있는 메모리 추정량 |

### top 명령어

`top`은 시스템의 프로세스와 메모리 사용 상태를 **실시간으로 모니터링**할 때 사용한다.

특히 SWAP 사용량이 증가하고 있는지 확인하거나 메모리를 많이 사용하는 프로세스를 찾을 때 유용하다.

### /proc/swaps 파일

`/proc/swaps`는 현재 시스템에서 **활성화된 SWAP 영역**의 정보를 제공한다.

| 항목 | 설명 |
|---|---|
| Filename | SWAP 영역의 장치 또는 파일 |
| Type | SWAP 영역의 유형 |
| Size | SWAP 전체 크기 |
| Used | 현재 사용 중인 SWAP 크기 |
| Priority | SWAP 우선순위 |

### 자주 사용하는 확인 예제

`free -h`를 사용하여 RAM과 SWAP의 현재 사용량을 확인한다.

`cat /proc/swaps`를 사용하여 현재 활성화된 SWAP 영역과 우선순위를 확인한다.

`swapon --show`를 사용하여 활성화된 SWAP 영역의 상세 정보를 확인한다.

`lsblk`를 사용하여 디스크, 파티션 및 LVM 구조를 확인한다.

---

## 3. 스왑(SWAP) 관리

### 스왑 관리 명령어

| 명령어 | 설명 |
|---|---|
| `mkswap` | 장치 또는 파일을 SWAP 영역으로 초기화 |
| `swapon` | SWAP 영역 활성화 |
| `swapoff` | SWAP 영역 비활성화 |
| `swapon --show` | 활성화된 SWAP 영역 표시 |
| `swapon -s` | 활성화된 SWAP 영역의 요약 정보 표시 |
| `dd` | SWAP 파일을 생성할 때 파일 크기를 확보 |
| `rm` | SWAP 파일 삭제 |
| `fdisk` | 파티션 생성 및 관리 |
| `parted` | 파티션 생성 및 관리 |
| `pvcreate` | 파티션을 LVM Physical Volume으로 초기화 |
| `vgextend` | 기존 Volume Group에 PV 추가 |
| `lvcreate` | Logical Volume 생성 |
| `lvremove` | Logical Volume 삭제 |
| `lvextend` | Logical Volume 확장 |
| `lvreduce` | Logical Volume 축소 |

### 스왑 파일 추가

SWAP 파일을 추가하는 기본 절차는 다음과 같다.

**파일 생성 → SWAP 영역 초기화 → SWAP 활성화 → `/etc/fstab` 등록**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `mkdir /swap` | SWAP 파일을 저장할 디렉터리 생성 |
| 2 | `dd if=/dev/zero of=/swap/swapfile bs=1M count=1024` | 지정된 크기의 파일 생성 |
| 3 | `chmod 600 /swap/swapfile` | SWAP 파일 접근 권한 제한 |
| 4 | `mkswap /swap/swapfile` | 파일을 SWAP 영역으로 초기화 |
| 5 | `swapon /swap/swapfile` | SWAP 활성화 |
| 6 | `/etc/fstab` 등록 | 시스템 부팅 시 자동 활성화 |

SWAP 파일은 보안상 **`600(rw-------)` 권한**으로 설정하는 것이 적절하다.

`dd` 명령어의 주요 옵션은 다음과 같다.

| 옵션 | 의미 |
|---|---|
| `if=` | 입력 파일 |
| `of=` | 출력 파일 |
| `bs=` | 한 번에 처리할 블록 크기 |
| `count=` | 처리할 블록 수 |

SWAP 파일의 크기는 일반적으로 `bs × count`를 기준으로 결정한다.

### 스왑 파일을 `/etc/fstab`에 등록

부팅 시 SWAP 파일을 자동으로 활성화하려면 `/etc/fstab`에 등록한다.

| 필드 | 값 |
|---|---|
| 장치 또는 파일 | `/swap/swapfile` |
| 마운트 위치 | `none` |
| 파일시스템 유형 | `swap` |
| 옵션 | `defaults` |
| dump | `0` |
| fsck | `0` |

등록 후에는 `swapon -a`를 사용하여 `/etc/fstab`에 정의된 SWAP을 활성화할 수 있다.

### 스왑 파티션 추가

SWAP 파티션은 다음 순서로 구성한다.

**파티션 생성 → SWAP 영역 초기화 → SWAP 활성화 → `/etc/fstab` 등록**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `fdisk` 또는 `parted` | SWAP용 파티션 생성 |
| 2 | `mkswap /dev/디바이스명` | 파티션을 SWAP 영역으로 초기화 |
| 3 | `swapon /dev/디바이스명` | SWAP 활성화 |
| 4 | `/etc/fstab` 등록 | 부팅 시 자동 활성화 |

GPT 파티션을 사용하는 경우에는 `parted`를 이용하여 **swap 플래그**를 설정할 수 있다.

SWAP 파티션은 `/etc/fstab`에서 **UUID를 사용하는 방법을 권장**한다.

### 스왑 LV 추가

LVM 환경에서는 기존 VG에 PV를 추가한 후 SWAP용 LV를 생성할 수 있다.

**파티션 생성 → PV 생성 → VG 확장 → LV 생성 → SWAP 초기화 → SWAP 활성화 → `/etc/fstab` 등록**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `parted` | LVM용 파티션 생성 |
| 2 | `pvcreate /dev/디바이스명` | PV 생성 |
| 3 | `vgextend vg0 /dev/디바이스명` | 기존 VG에 PV 추가 |
| 4 | `lvcreate -L 500M -n swaplv vg0` | SWAP용 LV 생성 |
| 5 | `mkswap /dev/vg0/swaplv` | LV를 SWAP 영역으로 초기화 |
| 6 | `swapon /dev/vg0/swaplv` | SWAP 활성화 |
| 7 | `/etc/fstab` 등록 | 부팅 시 자동 활성화 |

### 자주 사용하는 추가 예제

SWAP 파일을 추가하는 경우 `dd`로 파일을 생성한 후 `chmod 600`, `mkswap`, `swapon` 순서로 처리한다.

SWAP 파티션을 추가하는 경우 파티션을 준비한 후 `mkswap`과 `swapon`을 실행한다.

LVM 환경에서 SWAP을 추가하는 경우 PV → VG → LV 순서로 LVM 구성을 준비한 후 `mkswap`과 `swapon`을 실행한다.

---

## 4. 스왑(SWAP) 삭제

### 스왑 파일 삭제

SWAP 파일은 **반드시 비활성화한 후 삭제**해야 한다.

**SWAP 비활성화 → `/etc/fstab` 항목 제거 → 파일 삭제**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `swapoff /swap/swapfile` | SWAP 비활성화 |
| 2 | `/etc/fstab` 수정 | 부팅 시 자동 활성화 설정 제거 |
| 3 | `rm -f /swap/swapfile` | SWAP 파일 삭제 |

사용 중인 SWAP 영역은 바로 삭제할 수 없으므로 먼저 `swapoff`를 수행해야 한다.

### 스왑 파티션 삭제

SWAP 파티션도 먼저 비활성화한 후 `/etc/fstab` 설정을 제거한다.

**SWAP 비활성화 → `/etc/fstab` 항목 제거 → 파티션 재사용 또는 삭제**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `swapoff /dev/디바이스명` | SWAP 비활성화 |
| 2 | `/etc/fstab` 수정 | SWAP 자동 활성화 설정 제거 |
| 3 | 파티션 관리 도구 사용 | 파티션 삭제 또는 다른 용도로 재구성 |

### 스왑 LV 삭제

SWAP LV를 삭제하는 경우 먼저 SWAP을 비활성화해야 한다.

**SWAP 비활성화 → `/etc/fstab` 항목 제거 → LV 삭제**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `swapoff /dev/vg0/swaplv` | SWAP 비활성화 |
| 2 | `/etc/fstab` 수정 | SWAP 설정 제거 |
| 3 | `lvremove -f /dev/vg0/swaplv` | LV 삭제 |

### 자주 사용하는 삭제 예제

SWAP 파일을 삭제할 때는 `swapoff`를 실행한 후 `/etc/fstab`에서 해당 항목을 제거하고 `rm`으로 파일을 삭제한다.

SWAP 파티션을 제거하거나 재사용할 때도 먼저 `swapoff`를 실행한다.

SWAP LV를 삭제할 때는 `swapoff` 후 `lvremove`를 실행한다.

---

## 5. 스왑 우선순위(Priority)

### 스왑 우선순위 개념

시스템에 여러 개의 SWAP 영역이 존재하면 각 SWAP 영역에 **Priority**를 지정할 수 있다.

**숫자가 클수록 우선순위가 높다.**

서로 다른 우선순위를 가진 SWAP 영역이 있으면 **높은 우선순위의 영역을 먼저 사용**한다.

| Priority | 의미 |
|---:|---|
| 높은 값 | 높은 우선순위 |
| 낮은 값 | 낮은 우선순위 |
| 동일한 Priority | 사용 가능한 영역 사이에서 라운드 로빈 방식으로 할당 |

우선순위는 `/etc/fstab`의 `pri=` 옵션이나 `swapon` 명령어의 `-p`, `--priority` 옵션으로 지정할 수 있다.

### pri 옵션

`/etc/fstab`에서 SWAP 우선순위를 설정할 때 `pri=` 옵션을 사용한다.

| 설정 | 의미 |
|---|---|
| `pri=10` | 우선순위 10 |
| `pri=4` | 우선순위 4 |
| `pri=0` | 우선순위 0 |

예를 들어 우선순위가 **10, 4, 기본값**인 세 개의 SWAP 영역이 있다면 우선순위 10인 영역을 먼저 사용하고, 이후 낮은 우선순위의 영역을 사용한다.

### swapon의 우선순위 옵션

`swapon`에서는 `-p` 또는 `--priority` 옵션으로 우선순위를 지정할 수 있다.

| 옵션 | 설명 |
|---|---|
| `-p <priority>` | SWAP 우선순위 지정 |
| `--priority <priority>` | SWAP 우선순위 지정 |
| 높은 값 | 높은 우선순위 |

SWAP이 이미 활성화되어 사용 중인 상태에서는 해당 영역의 우선순위를 임의로 변경할 수 없다.

우선순위를 변경하려면 **SWAP을 비활성화한 후 원하는 우선순위로 다시 활성화**하는 방식으로 처리한다.

### 동일 우선순위의 스왑 영역

두 개 이상의 SWAP 영역이 동일한 최고 우선순위를 가지고 있으면 커널은 해당 영역들 사이에서 **라운드 로빈 방식**으로 페이지를 할당할 수 있다.

### 자주 사용하는 우선순위 설정 예제

SWAP 파일을 특정 우선순위로 활성화하려면 `swapon --priority 10 /swap/swapfile`과 같이 우선순위를 지정할 수 있다.

현재 SWAP 영역의 우선순위는 `swapon --show` 또는 `cat /proc/swaps`로 확인할 수 있다.

---

## 6. 스왑 LV 확장 및 축소

### 스왑 LV 확장

SWAP으로 사용 중인 LV의 크기를 변경할 때는 일반적인 파일시스템 확장과 다르게 처리해야 한다.

SWAP LV는 **오프라인 상태**에서 크기를 변경해야 한다.

**SWAP 비활성화 → LV 확장 → SWAP 영역 재생성 → SWAP 활성화**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `swapoff /dev/vg0/swaplv` | SWAP 비활성화 |
| 2 | `lvextend -L +300M /dev/vg0/swaplv` | LV 크기 증가 |
| 3 | `mkswap /dev/vg0/swaplv` | 변경된 영역을 SWAP으로 재초기화 |
| 4 | `swapon /dev/vg0/swaplv` | SWAP 재활성화 |

### 스왑 LV 축소

SWAP LV를 축소할 때도 반드시 오프라인 상태에서 작업한다.

**SWAP 비활성화 → LV 축소 → SWAP 영역 재생성 → SWAP 활성화**

| 단계 | 명령어 | 설명 |
|---|---|---|
| 1 | `swapoff /dev/vg0/swaplv` | SWAP 비활성화 |
| 2 | `lvreduce -L -300M /dev/vg0/swaplv` | LV 크기 감소 |
| 3 | `mkswap /dev/vg0/swaplv` | 변경된 영역을 SWAP으로 재초기화 |
| 4 | `swapon /dev/vg0/swaplv` | SWAP 재활성화 |

SWAP LV의 크기를 변경할 때는 `swapoff`를 먼저 수행하는 것이 중요하다.

### 자주 사용하는 LV 크기 변경 예제

SWAP LV를 확장할 때는 `swapoff → lvextend → mkswap → swapon` 순서로 작업한다.

SWAP LV를 축소할 때는 `swapoff → lvreduce → mkswap → swapon` 순서로 작업한다.

---

## 7. 실무에서의 SWAP 관리

### 스왑 추가가 필요한 경우

서버 운영 중 다음과 같은 상황이 발생하면 SWAP 추가를 검토할 수 있다.

- **메모리 사용량이 지속적으로 증가**
- **메모리 사용량이 높은 수준으로 유지**
- 메모리 부족으로 인해 시스템의 안정적인 운영이 어려움
- 특정 워크로드에서 추가적인 메모리 공간이 필요함

다만 SWAP 사용량이 증가했다고 해서 무조건 SWAP 크기만 늘리는 것은 적절하지 않다.

먼저 다음 정보를 함께 확인한다.

| 확인 항목 | 확인 방법 |
|---|---|
| RAM 전체/사용량 | `free -h` |
| SWAP 전체/사용량 | `free -h` |
| 활성 SWAP 목록 | `swapon --show` |
| SWAP 우선순위 | `swapon --show` |
| 메모리를 많이 사용하는 프로세스 | `top` |
| 전체 메모리 상세 정보 | `cat /proc/meminfo` |
| 디스크/LVM 구조 | `lsblk` |

### 스왑 방식 선택

SWAP을 추가하는 방법은 크게 **스왑 파일, 스왑 파티션, 스왑 LV**로 구분할 수 있다.

| 방식 | 특징 |
|---|---|
| SWAP 파일 | 기존 파일시스템의 공간을 이용하여 비교적 유연하게 추가 가능 |
| SWAP 파티션 | 별도의 파티션을 SWAP 전용 영역으로 사용 |
| SWAP LV | LVM 환경에서 LV를 SWAP 영역으로 사용하여 관리 가능 |

실무에서는 시스템 구성과 남은 디스크 공간, LVM 사용 여부, 관리 편의성 등을 고려하여 적절한 방식을 선택한다.

---

## 8. 자주 사용하는 SWAP 명령 예제

### SWAP 상태 확인

`free -h`로 RAM과 SWAP의 전체 용량 및 사용량을 확인한다.

`cat /proc/swaps`로 현재 활성화된 SWAP 영역을 확인한다.

`swapon --show`로 활성화된 SWAP 영역과 Priority를 확인한다.

### SWAP 파일 생성

`dd if=/dev/zero of=/swap/swapfile bs=1M count=1024`로 SWAP 파일을 생성한다.

`chmod 600 /swap/swapfile`로 SWAP 파일의 접근 권한을 제한한다.

`mkswap /swap/swapfile`로 SWAP 영역을 초기화한다.

`swapon /swap/swapfile`로 SWAP을 활성화한다.

### SWAP 파일 비활성화 및 삭제

`swapoff /swap/swapfile`로 SWAP을 비활성화한다.

`/etc/fstab`에서 해당 SWAP 파일의 자동 활성화 설정을 제거한다.

`rm -f /swap/swapfile`로 SWAP 파일을 삭제한다.

### SWAP 파티션 생성 및 활성화

파티션 관리 도구로 SWAP용 파티션을 생성한다.

`mkswap /dev/디바이스명`으로 SWAP 영역을 초기화한다.

`swapon /dev/디바이스명`으로 SWAP을 활성화한다.

`/etc/fstab`에 UUID 기반으로 SWAP 영역을 등록하여 부팅 시 자동 활성화되도록 구성한다.

### SWAP LV 생성 및 활성화

`pvcreate /dev/디바이스명`으로 PV를 생성한다.

`vgextend vg0 /dev/디바이스명`으로 기존 VG에 PV를 추가한다.

`lvcreate -L 500M -n swaplv vg0`으로 SWAP용 LV를 생성한다.

`mkswap /dev/vg0/swaplv`로 LV를 SWAP 영역으로 초기화한다.

`swapon /dev/vg0/swaplv`로 SWAP을 활성화한다.

`/etc/fstab`에 SWAP LV를 등록하여 부팅 시 자동 활성화되도록 구성한다.

### SWAP LV 삭제

`swapoff /dev/vg0/swaplv`로 SWAP을 비활성화한다.

`/etc/fstab`에서 해당 SWAP LV 설정을 제거한다.

`lvremove -f /dev/vg0/swaplv`로 SWAP LV를 삭제한다.

### SWAP 우선순위 지정

`swapon --priority 10 /swap/swapfile`과 같이 `swapon` 실행 시 우선순위를 지정할 수 있다.

`/etc/fstab`에서는 `pri=10`과 같은 형태로 SWAP 우선순위를 설정할 수 있다.

`swapon --show` 또는 `cat /proc/swaps`를 사용하여 설정된 Priority를 확인한다.

### SWAP LV 크기 변경

SWAP LV 확장은 `swapoff → lvextend → mkswap → swapon` 순서로 수행한다.

SWAP LV 축소는 `swapoff → lvreduce → mkswap → swapon` 순서로 수행한다.

### 핵심 정리

**SWAP 관리의 핵심 순서**

**확인 → 생성 → `mkswap` → `swapon` → `/etc/fstab` 등록**

SWAP을 제거할 때는

**`swapoff` → `/etc/fstab` 설정 제거 → 실제 파일/LV/파티션 제거**

SWAP LV의 크기를 변경할 때는

**`swapoff` → LV 크기 변경 → `mkswap` → `swapon`**

순서를 기억하는 것이 중요하다.