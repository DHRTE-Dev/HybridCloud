# RAID 관리

## 목차

- [1. RAID 종류 및 기본 개념](#1-raid-종류-및-기본-개념)
	- [RAID의 개념](#raid의-개념)
	- [RAID 구현 방식](#raid-구현-방식)
	- [스토리지 분류](#스토리지-분류)
	- [RAID 레벨 비교](#raid-레벨-비교)
- [2. RAID 레벨별 특징](#2-raid-레벨별-특징)
	- [RAID 0](#raid-0)
	- [RAID 1](#raid-1)
	- [RAID 0+1](#raid-01)
	- [RAID 10](#raid-10)
	- [RAID 2](#raid-2)
	- [RAID 3](#raid-3)
	- [RAID 4](#raid-4)
	- [RAID 5](#raid-5)
	- [RAID 6](#raid-6)
	- [RAID 7](#raid-7)
	- [RAID 53](#raid-53)
	- [JBOD](#jbod)
- [3. CentOS Software RAID 구성 방법](#3-centos-software-raid-구성-방법)
	- [Software RAID 구성 방식](#software-raid-구성-방식)
	- [RAID 작업 기본 절차](#raid-작업-기본-절차)
	- [LVM RAID 구성](#lvm-raid-구성)
- [4. mdadm 명령어](#4-mdadm-명령어)
	- [mdadm 개요](#mdadm-개요)
	- [mdadm 기본 형식](#mdadm-기본-형식)
	- [주요 옵션](#주요-옵션)
	- [RAID 생성 및 확인](#raid-생성-및-확인)
	- [/etc/mdadm.conf 설정](#etcmdadmconf-설정)
	- [RAID 삭제](#raid-삭제)
	- [mdadm 도움말 확인](#mdadm-도움말-확인)
- [5. RAID 0 구성 및 관리](#5-raid-0-구성-및-관리)
	- [RAID 0 구성](#raid-0-구성)
	- [RAID 0 파일시스템 및 마운트](#raid-0-파일시스템-및-마운트)
	- [RAID 0 해제](#raid-0-해제)
- [6. RAID 1 구성 및 관리](#6-raid-1-구성-및-관리)
	- [RAID 1 구성](#raid-1-구성)
	- [RAID 1 상태 확인](#raid-1-상태-확인)
	- [RAID 1 해제](#raid-1-해제)
- [7. RAID 5 구성 및 관리](#7-raid-5-구성-및-관리)
	- [RAID 5 구성](#raid-5-구성)
	- [RAID 5 파일시스템 및 마운트](#raid-5-파일시스템-및-마운트)
	- [RAID 5 해제](#raid-5-해제)
- [8. RAID 장애 디스크 교체](#8-raid-장애-디스크-교체)
	- [장애 디스크 교체 개요](#장애-디스크-교체-개요)
	- [RAID 5 장애 처리](#raid-5-장애-처리)
	- [RAID 1 장애 처리](#raid-1-장애-처리)
- [9. RAID 통합 실습 및 성능 확인](#9-raid-통합-실습-및-성능-확인)
	- [RAID 0, RAID 1, RAID 5 구성](#raid-0-raid-1-raid-5-구성)
	- [파일 생성 성능 측정](#파일-생성-성능-측정)
	- [전체 RAID 삭제](#전체-raid-삭제)
- [10. RAID와 LVM 비교 및 운영 고려사항](#10-raid와-lvm-비교-및-운영-고려사항)
	- [RAID와 LVM 비교](#raid와-lvm-비교)
	- [물리 디스크 작업 비교](#물리-디스크-작업-비교)
	- [운영 환경의 RAID 구성](#운영-환경의-raid-구성)

---

## 1. RAID 종류 및 기본 개념

### RAID의 개념

**RAID(Redundant Array of Independent Disks)**는 여러 개의 디스크를 하나의 논리적인 저장 장치처럼 구성하여 **성능 향상**, **저장 공간 활용**, **장애 대응 능력 향상** 등을 구현하는 기술이다.

RAID는 구현 방식에 따라 **Hardware RAID**, **Firmware RAID**, **Software RAID**로 구분할 수 있다.

### RAID 구현 방식

| 방식 | 지원 위치 | 구성 단위 | 특징 |
|---|---|---|---|
| **Firmware RAID** | 시스템 펌웨어 또는 보드 | 디스크 | 보드에 RAID 기능이 내장될 수 있음 |
| **Hardware RAID** | RAID Controller | 디스크 | 성능이 우수하지만 비용이 높고 구성 유연성이 상대적으로 낮음 |
| **Software RAID** | 운영체제 | 파티션 또는 블록 장치 | 비용이 낮고 유연하지만 CPU 및 OS의 영향을 받음 |

일반적인 성능 순서는 자료 기준으로 **Hardware RAID > Firmware RAID > Software RAID**이다.

### 스토리지 분류

스토리지는 **연결 방식**과 **저장 방식**에 따라 구분할 수 있다.

| 분류 기준 | 종류 | 설명 |
|---|---|---|
| 연결 방식 | **DAS** | 서버 또는 컴퓨터에 직접 연결하는 저장 장치 |
| 연결 방식 | **NAS** | 네트워크를 통해 파일 서비스를 제공하는 저장 장치 |
| 연결 방식 | **SAN** | 저장 장치 전용 네트워크를 사용하는 저장 시스템 |
| 저장 방식 | **File Storage** | 파일 단위로 데이터를 저장 |
| 저장 방식 | **Block Storage** | 블록 단위로 데이터를 제공 |
| 저장 방식 | **Object Storage** | 객체 단위로 데이터를 저장 |

### DAS

**DAS(Direct Attached Storage)**는 서버나 컴퓨터에 저장 장치를 직접 연결하는 방식이다.

- 내장 디스크와 외장 디스크 형태가 있음
- 일반적으로 다른 서버가 직접 접근하기 어려움
- 구성과 관리가 비교적 단순함

### NAS

**NAS(Network Attached Storage)**는 네트워크를 통해 파일 서비스를 제공하는 저장 장치이다.

- **NFS**, **SMB/CIFS** 등의 파일 서비스 사용
- 일반적인 TCP/IP 네트워크를 통해 접근
- 파일 단위의 저장 서비스를 제공

### SAN

**SAN(Storage Area Network)**은 저장 장치를 서버와 연결하는 별도의 고속 저장 네트워크이다.

- **Block Storage** 방식으로 사용하는 경우가 많음
- 서버와 스토리지 사이에 SAN Switch 등의 구성 요소가 사용될 수 있음
- 디스크 미러링, 백업 및 복원, 데이터 이동, 서버 간 저장 장치 공유 등에 활용

### RAID 레벨 비교

| RAID | 기본 방식 | 최소 디스크 | 사용 가능 용량 | 장애 대응 | 성능 특징 |
|---|---|---:|---:|---|---|
| **RAID 0** | Striping | 2 | N × 디스크 용량 | 없음 | 읽기/쓰기 성능 향상 |
| **RAID 1** | Mirroring | 2 | N/2 × 디스크 용량 | 1개 장애 대응 가능 | 읽기 성능 향상 |
| **RAID 0+1** | Stripe 후 Mirror | 4 | N/2 × 디스크 용량 | 구성에 따라 장애 대응 | 성능과 안정성 절충 |
| **RAID 10** | Mirror 후 Stripe | 4 | N/2 × 디스크 용량 | 미러 그룹별 장애 대응 | 높은 성능과 안정성 |
| **RAID 2** | Bit/Word Striping + Hamming Code | - | - | ECC 기반 | 현재 거의 사용하지 않음 |
| **RAID 3** | Byte Striping + Dedicated Parity | 3 | (N-1) × 용량 | 1개 장애 대응 | 동기화 필요 |
| **RAID 4** | Block Striping + Dedicated Parity | 3 | (N-1) × 용량 | 1개 장애 대응 | 패리티 디스크 병목 가능 |
| **RAID 5** | Block Striping + Distributed Parity | 3 | (N-1) × 용량 | 1개 장애 대응 | 읽기 성능과 용량의 절충 |
| **RAID 6** | Block Striping + Double Parity | 4 이상 | (N-2) × 용량 | 2개 장애 대응 | RAID 5보다 안정성 높음 |
| **JBOD** | Spanning | 2 이상 | 디스크 용량 합계 | 없음 | 성능 향상 없음 |

---

## 2. RAID 레벨별 특징

### RAID 0

**RAID 0**은 여러 디스크에 데이터를 **Striping**하여 분산 저장한다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **2개** |
| 최대 용량 | 디스크 수 × 디스크 용량 |
| 데이터 보호 | **없음** |
| 읽기 성능 | 향상 |
| 쓰기 성능 | 향상 |
| 주요 목적 | 성능 향상 및 전체 디스크 공간 활용 |

하나의 디스크라도 장애가 발생하면 RAID 0 전체 데이터에 접근할 수 없게 되므로 **장애 복구가 필요한 시스템에는 적합하지 않다**.

RAID 0에는 자료에서 **Concatenate(Linear)**와 **Stripe** 개념이 함께 제시된다.

| 방식 | 특징 |
|---|---|
| **Linear** | 디스크를 순차적으로 연결하여 공간을 확장 |
| **Stripe** | 데이터를 여러 디스크에 분산하여 입출력 성능 향상 |

### RAID 1

**RAID 1**은 동일한 데이터를 여러 디스크에 복제하는 **Mirroring** 방식이다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **2개** |
| 사용 가능 용량 | 전체 용량의 약 50% |
| 데이터 보호 | 1개 디스크 장애 대응 가능 |
| 읽기 성능 | 향상 가능 |
| 쓰기 성능 | 일반적으로 단일 디스크와 유사 |
| 주요 목적 | **데이터 안정성 및 장애 대응** |

한쪽 디스크가 장애가 발생하더라도 미러링된 다른 디스크를 통해 데이터를 유지할 수 있다.

### RAID 0+1

**RAID 0+1**은 먼저 RAID 0으로 Striping한 다음 RAID 1로 Mirroring하는 방식이다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **4개** |
| 사용 가능 용량 | 약 50% |
| 구성 | RAID 0 → RAID 1 |
| 특징 | Striping 성능과 Mirroring 안정성 결합 |

### RAID 10

**RAID 10(1+0)**은 먼저 RAID 1로 Mirroring한 다음 RAID 0으로 Striping하는 방식이다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **4개** |
| 사용 가능 용량 | 약 50% |
| 구성 | RAID 1 → RAID 0 |
| 성능 | 높음 |
| 장애 대응 | 미러 그룹별 장애 대응 |
| 특징 | 성능과 안정성을 동시에 요구하는 환경에 적합 |

**RAID 0+1과 RAID 10은 구성 순서가 반대**이며, 여러 디스크에서 장애가 발생하는 경우 RAID 10이 일반적으로 더 유리한 구조를 가진다.

### RAID 2

**RAID 2**는 데이터를 스트라이핑하고 **Hamming Code**를 이용하여 오류 검출 및 수정을 수행하는 방식이다.

현재 디스크 자체에 ECC 등의 오류 처리 기능이 포함되어 있기 때문에 **실제 환경에서는 거의 사용되지 않는다**.

### RAID 3

**RAID 3**은 데이터를 **Byte 단위**로 스트라이핑하고 별도의 디스크에 패리티를 저장한다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **3개** |
| 데이터 단위 | Byte |
| 패리티 | 별도 디스크 |
| 장애 대응 | 1개 디스크 |
| 특징 | 디스크 동기화 필요 |

디스크 동기화가 필요하기 때문에 현재 일반적인 시스템에서는 많이 사용되지 않는다.

### RAID 4

**RAID 4**는 데이터를 **Block 단위**로 스트라이핑하고 하나의 별도 디스크에 패리티를 저장한다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **3개** |
| 데이터 단위 | Block |
| 패리티 | 전용 디스크 |
| 장애 대응 | 1개 디스크 |
| 문제점 | 패리티 디스크에 쓰기 부하 집중 |

패리티 디스크가 모든 쓰기 작업에 관여하기 때문에 병목이 발생할 수 있다.

### RAID 5

**RAID 5**는 데이터를 Block 단위로 스트라이핑하면서 **패리티 정보를 여러 디스크에 분산 저장**한다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | **3개** |
| 사용 가능 용량 | (N-1) × 디스크 용량 |
| 패리티 | 분산 패리티 |
| 장애 대응 | **1개 디스크 장애** |
| 읽기 성능 | 우수 |
| 쓰기 성능 | 패리티 연산으로 상대적으로 낮음 |
| 특징 | 용량, 성능, 안정성의 절충 |

RAID 3/4와 달리 별도의 고정된 패리티 디스크가 없기 때문에 **패리티 디스크 집중에 따른 병목을 줄일 수 있다**.

### RAID 6

**RAID 6**은 RAID 5와 유사하지만 **2개의 패리티 정보**를 사용한다.

| 항목 | 내용 |
|---|---|
| 최소 디스크 | 일반적인 구성에서 **4개 이상** |
| 사용 가능 용량 | (N-2) × 디스크 용량 |
| 패리티 | 이중 패리티 |
| 장애 대응 | **최대 2개 디스크 장애** |
| RAID 5 대비 | 안정성 향상, 용량 및 쓰기 성능 감소 |

자료에서는 이론적인 최소 구성과 실제 구성의 차이를 설명하고 있으므로, 실제 운영에서는 충분한 디스크 수를 확보하여 구성하는 것이 적절하다.

### RAID 7

**RAID 7**은 컨트롤러에 실시간 운영체제, 캐시, 전용 처리 능력 등을 포함하는 특수한 RAID 방식으로 설명된다.

일반적인 Linux Software RAID 실습의 주요 대상은 아니다.

### RAID 53

**RAID 53**은 RAID 3 기반의 스트라이프 어레이를 다시 스트라이핑하는 형태로 설명된다.

- RAID 3보다 높은 성능을 목표로 함
- 구성 비용이 높음
- 일반적인 Linux Software RAID 실습에서는 거의 사용하지 않음

### JBOD

**JBOD(Just a Bunch of Disks)**는 엄밀한 의미의 RAID 방식은 아니다.

| 항목 | 내용 |
|---|---|
| 다른 명칭 | Spanning |
| 데이터 분산 | 하지 않음 |
| 사용 용량 | 여러 디스크의 용량 합계 |
| 성능 향상 | 없음 |
| 장애 대응 | 없음 |
| 장점 | 서로 다른 용량의 디스크를 구성할 수 있음 |

RAID 0과 달리 데이터를 디스크에 분산 저장하지 않고 디스크 순서대로 사용한다.

---

## 3. CentOS Software RAID 구성 방법

### Software RAID 구성 방식

CentOS에서 Software RAID는 대표적으로 **mdadm**을 이용하여 구성할 수 있다.

자료에서는 Software RAID 구성 방법으로 다음 두 가지를 제시한다.

| 방식 | 설명 |
|---|---|
| **mdadm** | Linux MD 장치를 직접 구성하고 관리 |
| **LVM RAID** | LVM의 RAID 기능을 이용하여 RAID LV 구성 |

### RAID 작업 기본 절차

RAID를 파일시스템으로 사용하기 위한 기본적인 흐름은 다음과 같다.

**파티션 준비 → RAID 구성 → RAID 설정 저장 → 파일시스템 생성 → 마운트 → `/etc/fstab` 등록**

| 단계 | 주요 명령어 또는 파일 | 주요 작업 |
|---|---|---|
| 1 | `parted`, `fdisk` | RAID용 파티션 생성 |
| 2 | `mdadm --create` | RAID 배열 생성 |
| 3 | `/etc/mdadm.conf` | RAID 배열 정보 저장 |
| 4 | `mkfs.xfs`, `mkfs.ext4` | 파일시스템 생성 |
| 5 | `mount` | 파일시스템 마운트 |
| 6 | `/etc/fstab` | 부팅 시 자동 마운트 설정 |
| 7 | `systemctl daemon-reload` | fstab 변경사항 반영 필요 시 사용 |

### LVM RAID 구성

자료에서 제시한 LVM RAID 구성 방식은 다음과 같다.

| RAID | 명령 형식 | 최소 PV 수 |
|---|---|---:|
| **RAID 0** | `lvcreate --type raid0 -L 1G -n lv1 -i 3 vg1` | 2개 이상 |
| **RAID 1** | `lvcreate --type raid1 -L 1G -n lv1 -m 1 vg1` | 2개 이상 |
| **RAID 5** | `lvcreate --type raid5 -L 1G -n lv1 -i 2 vg1` | 3개 이상 |
| **RAID 6** | `lvcreate --type raid6 -L 1G -n lv1 -i 3 vg1` | 자료 기준 5개 이상 |
| **RAID 10** | `lvcreate --type raid10 -L 1G -n lv1 -m 1 -i 2 vg1` | 4개 이상 |

`-i`는 스트라이프에 사용되는 데이터 영역의 수와 관련되고, `-m`은 미러 복제 수와 관련된다.

---

## 4. mdadm 명령어

### mdadm 개요

**mdadm**은 Linux Software RAID의 **MD(Multiple Devices)** 장치를 관리하는 명령어이다.

기본 형식은 `mdadm [mode] <raiddevice> [options] <component-devices>`이다.

Linux Software RAID에서 자료에 제시된 주요 RAID 유형은 **LINEAR, RAID 0, RAID 1, RAID 4, RAID 5, RAID 6, RAID 10** 등이다.

### mdadm 기본 형식

| 용도 | 명령 형식 |
|---|---|
| RAID 생성 | `mdadm --create <RAID장치> --level=<레벨> --raid-devices=<개수> <장치> ...` |
| RAID 확인 | `mdadm --detail <RAID장치>` |
| 배열 검색 | `mdadm --detail --scan` |
| RAID 중지 | `mdadm --stop <RAID장치>` |
| 장치 추가 | `mdadm <RAID장치> --add <장치>` |
| 장치 제거 | `mdadm <RAID장치> --remove <장치>` |
| 장치 장애 처리 | `mdadm <RAID장치> --fail <장치>` |
| Superblock 삭제 | `mdadm --zero-superblock <장치> ...` |

### 주요 옵션

| 옵션 | 긴 옵션 | 설명 |
|---|---|---|
| `-C` | `--create` | 새로운 RAID 배열 생성 |
| `-V` | `--version` | mdadm 버전 확인 |
| `-s` | `--scan` | 설정 파일 또는 `/proc/mdstat`에서 RAID 정보 검색 |
| `-f` | `--force` | 특정 작업을 강제로 수행 |
| `-n` | `--raid-devices=` | 활성 RAID 장치 수 지정 |
| `-l` | `--level=` | RAID 레벨 지정 |
| `-a` | `--add` | 장치 추가 |
| `-r` | `--remove` | 장애 또는 Spare 장치 제거 |
| `-f` | `--fail` | 장치를 장애 상태로 지정 |
| `-D` | `--detail` | RAID 상세 정보 표시 |
| `-S` | `--stop` | RAID 배열 비활성화 |
|  | `--zero-superblock` | MD Superblock 정보 삭제 |

### RAID 생성 및 확인

RAID 1 생성의 기본 형식은 다음과 같다.

`mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/device1 /dev/device2`

축약형은 다음과 같이 사용할 수 있다.

`mdadm -C /dev/md0 -l 1 -n 2 /dev/device1 /dev/device2`

여기서 `--create`는 `-C`, `--level`은 `-l`, `--raid-devices`는 `-n`에 해당한다.

RAID 상태는 다음 명령으로 확인할 수 있다.

`cat /proc/mdstat`

`mdadm --detail /dev/md0`

`mdadm --detail --scan`

### /etc/mdadm.conf 설정

`mdadm`으로 구성한 RAID 정보가 시스템 재부팅 후에도 인식되도록 RAID 배열 정보를 `/etc/mdadm.conf`에 저장한다.

가장 일반적인 방법은 다음과 같다.

`mdadm --detail --scan > /etc/mdadm.conf`

파일의 내용을 확인하려면 다음 명령을 사용한다.

`cat /etc/mdadm.conf`

`/etc/mdadm.conf`에는 RAID 배열의 장치명, metadata, UUID 등의 정보가 기록될 수 있다.

### RAID 삭제

RAID 삭제의 일반적인 순서는 다음과 같다.

**언마운트 → fstab 설정 제거 → RAID 중지 → Superblock 삭제 → mdadm.conf 정리**

주요 명령은 다음과 같다.

`umount /raidX`

`mdadm --stop /dev/mdX`

`mdadm --zero-superblock /dev/device1 /dev/device2`

`rm -f /etc/mdadm.conf`

RAID 장치를 중지하기 전에 반드시 해당 RAID 장치가 사용 중이지 않은지 확인해야 한다.

### mdadm 도움말 확인

mdadm의 세부 사용법은 다음 명령으로 확인할 수 있다.

`mdadm --help`

`mdadm --create --help`

`mdadm --stop --help`

`mdadm --zero-superblock --help`

`mdadm --config --help`

`mdadm --manage --help`

---

## 5. RAID 0 구성 및 관리

### RAID 0 구성

RAID용 파티션을 준비한 후 `mdadm --create`를 이용하여 RAID 0 배열을 생성한다.

`mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/device1p1 /dev/device2p1`

RAID 상태를 확인한다.

`cat /proc/mdstat`

`mdadm --detail /dev/md0`

### RAID 0 파일시스템 및 마운트

RAID 장치에 파일시스템을 생성한다.

`mkfs.xfs /dev/md0`

마운트 포인트를 생성한다.

`mkdir -p /raid0`

RAID 장치를 마운트한다.

`mount /dev/md0 /raid0`

마운트 상태는 다음과 같이 확인할 수 있다.

`df -hT`

영구 마운트를 위해 `/etc/fstab`에 RAID 장치와 마운트 포인트를 등록한다.

`/dev/md0 /raid0 xfs defaults 0 0`

fstab 변경 후 systemd가 이전 설정을 사용하는 경우 다음 명령으로 설정을 다시 로드한다.

`systemctl daemon-reload`

### RAID 0 해제

먼저 마운트를 해제하고 `/etc/fstab`의 RAID 관련 설정을 제거하거나 주석 처리한다.

`umount /raid0`

그 다음 RAID 배열을 중지한다.

`mdadm --stop /dev/md0`

RAID 구성 정보를 제거하기 위해 각 구성 장치의 Superblock을 삭제한다.

`mdadm --zero-superblock /dev/device1p1 /dev/device2p1`

필요한 경우 `/etc/mdadm.conf`의 RAID 정보를 삭제한다.

---

## 6. RAID 1 구성 및 관리

### RAID 1 구성

RAID용 파티션을 준비한 후 RAID 1을 생성한다.

`mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/device1p1 /dev/device2p1`

RAID 생성 과정과 동기화 상태는 `/proc/mdstat`에서 확인할 수 있다.

`watch cat /proc/mdstat`

### RAID 1 상태 확인

상세 상태는 다음 명령으로 확인한다.

`mdadm --detail /dev/md0`

정상적으로 동기화된 RAID 1은 `/proc/mdstat`에서 두 장치가 모두 정상 상태임을 나타내는 형태로 확인할 수 있다.

RAID 1은 내부 Bitmap을 사용할 수 있으며, 이를 통해 변경된 영역을 추적하여 복구 작업에 활용할 수 있다.

### RAID 1 해제

마운트된 파일시스템을 먼저 해제한다.

`umount /raid1`

`/etc/fstab`의 RAID 관련 항목을 제거하거나 주석 처리한다.

RAID 배열을 중지한다.

`mdadm --stop /dev/md0`

구성 디스크의 Superblock 정보를 삭제한다.

`mdadm --zero-superblock /dev/device1p1 /dev/device2p1`

필요하면 `/etc/mdadm.conf`를 정리한다.

`rm -f /etc/mdadm.conf`

---

## 7. RAID 5 구성 및 관리

### RAID 5 구성

RAID 5는 최소 3개의 디스크가 필요하다.

`mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/device1p1 /dev/device2p1 /dev/device3p1`

RAID 생성 후 초기 동기화 또는 복구 과정은 다음 명령으로 확인할 수 있다.

`cat /proc/mdstat`

`watch cat /proc/mdstat`

상세 RAID 상태는 다음과 같이 확인한다.

`mdadm --detail /dev/md0`

### RAID 5 파일시스템 및 마운트

파일시스템을 생성한다.

`mkfs.xfs /dev/md0`

마운트 포인트를 생성한다.

`mkdir -p /raid5`

RAID 장치를 마운트한다.

`mount /dev/md0 /raid5`

마운트 상태를 확인한다.

`df -hT`

영구 마운트를 위해 `/etc/fstab`에 다음과 같은 형식으로 등록한다.

`/dev/md0 /raid5 xfs defaults 0 0`

### RAID 5 해제

먼저 마운트를 해제한다.

`umount /raid5`

`/etc/fstab`의 RAID 5 항목을 제거하거나 주석 처리한다.

RAID 배열을 중지한다.

`mdadm --stop /dev/md0`

구성 디스크의 Superblock을 제거한다.

`mdadm --zero-superblock /dev/device1p1 /dev/device2p1 /dev/device3p1`

필요하면 `/etc/mdadm.conf`를 삭제한다.

`rm -f /etc/mdadm.conf`

---

## 8. RAID 장애 디스크 교체

### 장애 디스크 교체 개요

**RAID 1과 RAID 5**는 정상적인 구성 상태에서 장애 디스크를 제거하고 새로운 디스크를 추가하여 **온라인 상태에서 복구**할 수 있다.

기본적인 장애 처리 절차는 다음과 같다.

**장애 확인 → 장애 디스크 Fail 처리 → 장애 디스크 Remove → 물리 디스크 교체 → 새 디스크 파티션 준비 → Add → Rebuild 확인**

### RAID 5 장애 처리

장애가 발생한 디스크를 의도적으로 장애 상태로 변경하는 경우 다음 명령을 사용할 수 있다.

`mdadm /dev/md0 --fail /dev/device2p1`

축약형은 다음과 같다.

`mdadm /dev/md0 -f /dev/device2p1`

장애 상태를 확인한다.

`cat /proc/mdstat`

`mdadm --detail /dev/md0`

장애 상태가 된 장치를 RAID 배열에서 제거한다.

`mdadm /dev/md0 --remove /dev/device2p1`

축약형은 다음과 같다.

`mdadm /dev/md0 -r /dev/device2p1`

새로운 디스크를 장착하고 RAID용 파티션을 준비한 다음 RAID 배열에 추가한다.

`mdadm /dev/md0 --add /dev/device2p1`

축약형은 다음과 같다.

`mdadm /dev/md0 -a /dev/device2p1`

RAID 5에 새로운 장치를 추가하면 기존 데이터를 이용하여 **Rebuild**가 진행된다.

Rebuild 상태는 다음 명령으로 확인한다.

`watch cat /proc/mdstat`

`mdadm --detail /dev/md0`

### RAID 1 장애 처리

RAID 1에서 장애 디스크를 Fail 상태로 지정한다.

`mdadm /dev/md0 --fail /dev/device2p1`

장애 상태를 확인한다.

`mdadm --detail /dev/md0`

장애 디스크를 제거한다.

`mdadm /dev/md0 --remove /dev/device2p1`

새로운 디스크가 준비된 후 RAID 배열에 추가한다.

`mdadm /dev/md0 --add /dev/device2p1`

추가된 디스크에는 기존 정상 디스크의 데이터가 동기화된다.

동기화 상태는 다음 명령으로 확인할 수 있다.

`cat /proc/mdstat`

`mdadm --detail /dev/md0`

장애 처리 과정에서 RAID가 Degraded 상태가 되더라도 정상 디스크가 남아 있는 RAID 1은 서비스를 유지하면서 복구 작업을 수행할 수 있다.

---

## 9. RAID 통합 실습 및 성능 확인

### RAID 0, RAID 1, RAID 5 구성

추가 실습에서는 여러 개의 RAID용 파티션을 준비한 후 다음과 같이 구성한다.

| RAID | RAID 장치 | 구성 디스크 | 마운트 지점 |
|---|---|---|---|
| **RAID 0** | `/dev/md0` | 2개 | `/raid0` |
| **RAID 1** | `/dev/md1` | 2개 | `/raid1` |
| **RAID 5** | `/dev/md5` | 3개 | `/raid5` |

RAID 구성 예시는 다음과 같은 형태로 일반화할 수 있다.

`mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/device1p1 /dev/device2p1`

`mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/device3p1 /dev/device4p1`

`mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/device5p1 /dev/device6p1 /dev/device7p1`

RAID 상태를 확인한다.

`cat /proc/mdstat`

RAID 정보를 `/etc/mdadm.conf`에 저장한다.

`mdadm --detail --scan > /etc/mdadm.conf`

각 RAID 장치에 파일시스템을 생성한다.

`mkfs.xfs /dev/md0`

`mkfs.xfs /dev/md1`

`mkfs.xfs /dev/md5`

파일시스템 정보를 확인한다.

`lsblk --fs`

### 파일 생성 성능 측정

자료에서는 `dd` 명령과 `time` 명령을 이용하여 일반 파일시스템과 RAID 0, RAID 1, RAID 5의 파일 생성 시간을 비교한다.

일반적인 테스트 형식은 다음과 같다.

`time dd if=/dev/zero of=/test/file1 bs=500M count=1`

`time dd if=/dev/zero of=/raid0/file1 bs=500M count=1`

`time dd if=/dev/zero of=/raid1/file1 bs=500M count=1`

`time dd if=/dev/zero of=/raid5/file1 bs=500M count=1`

자료의 측정 결과는 특정 환경의 실습 결과이므로 **RAID 종류의 일반적인 성능 순위를 절대적인 수치로 판단하기보다는 동일한 시스템에서의 비교 결과**로 보는 것이 적절하다.

### 전체 RAID 삭제

여러 RAID를 동시에 실습한 경우 다음과 같이 마운트를 해제한다.

`umount /raid0`

`umount /raid1`

`umount /raid5`

`/etc/fstab`의 RAID 관련 항목을 제거하거나 주석 처리한다.

RAID 배열을 중지한다.

`mdadm --stop /dev/md0`

`mdadm --stop /dev/md1`

`mdadm --stop /dev/md5`

RAID 상태를 확인한다.

`cat /proc/mdstat`

필요하면 `/etc/mdadm.conf`를 삭제한다.

`rm -f /etc/mdadm.conf`

각 구성 파티션의 MD Superblock을 삭제한다.

`mdadm --zero-superblock /dev/device1p1 /dev/device2p1`

`mdadm --zero-superblock /dev/device3p1 /dev/device4p1`

`mdadm --zero-superblock /dev/device5p1 /dev/device6p1 /dev/device7p1`

---

## 10. RAID와 LVM 비교 및 운영 고려사항

### RAID와 LVM 비교

RAID와 LVM은 목적과 구현 방식이 다르지만 일부 기능은 유사한 형태로 구성할 수 있다.

| RAID | 대응되는 LVM 구성 | 기본 개념 |
|---|---|---|
| **RAID 0 Concatenate/Linear** | LVM Linear Volume | 여러 저장 공간을 연결 |
| **RAID 0 Stripe** | LVM Stripe Volume | 데이터를 여러 장치에 분산 |
| **RAID 1 Mirror** | LVM Mirror Volume | 데이터를 복제 |
| **RAID 5** | 일반 LVM에서 직접 대응 기능 없음 | 분산 패리티 기반 장애 대응 |

**RAID는 디스크 장애 대응과 데이터 분산을 주요 목적으로 하고, LVM은 저장 공간의 논리적 관리와 확장·축소 등의 유연한 볼륨 관리를 주요 목적으로 한다.**

### 물리 디스크 작업 비교

동일한 디스크를 사용하는 경우에도 일반 디스크, LVM, RAID는 구성 절차가 다르다.

| 단계 | 일반 디스크 | LVM | Software RAID |
|---|---|---|---|
| 파티션 유형 | Linux 파일시스템 | LVM | RAID |
| 파티션 준비 | `fdisk`, `parted` | `fdisk`, `parted` | `fdisk`, `parted` |
| 저장 장치 구성 | 없음 | `pvcreate` → `vgcreate` → `lvcreate` | `mdadm --create` |
| 파일시스템 | 실제 파티션 | LV | RAID 장치 |
| 파일시스템 생성 | `mkfs` | `mkfs` | `mkfs` |
| 마운트 | `mount` | `mount` | `mount` |
| 영구 설정 | `/etc/fstab` | `/etc/fstab` | `/etc/fstab` + `/etc/mdadm.conf` |

RAID 파티션은 GPT 환경에서 `parted`를 이용하여 RAID 플래그를 지정할 수 있다.

`parted -s /dev/device mklabel gpt mkpart primary 1MiB 100% set 1 raid on`

### 운영 환경의 RAID 구성

자료에서는 운영 환경을 크게 **OS 디스크**와 **DATA 디스크**로 구분하여 RAID 구성을 설명한다.

#### OS 디스크

OS 디스크는 Hardware RAID Controller 등을 이용하여 **RAID 1 또는 RAID 10과 같은 미러 기반 구성**을 사용할 수 있다.

목적은 운영체제 디스크 장애에 대한 대응이다.

#### DATA 디스크

데이터 디스크는 외부 Storage 또는 Array에서 제공하는 **RAID 10, RAID 5, RAID 6 등의 LUN**을 사용할 수 있다.

자료에서는 스토리지에서 RAID가 구성된 LUN을 제공하는 환경에서는 해당 RAID 위에 다시 LVM Stripe 등을 중복 구성하기보다는 **스토리지에서 제공하는 RAID 구성을 그대로 사용하는 방식**을 권장하는 방향으로 설명한다.

| 구성 | 주요 특성 |
|---|---|
| **RAID 10** | 성능 우수 |
| **RAID 5** | 일반적인 성능과 용량의 절충 |
| **RAID 6** | 저장 공간 효율은 낮지만 장애 대응 능력이 높음 |

### 자주 사용하는 RAID 관리 명령

| 목적 | 명령어 |
|---|---|
| 디스크 확인 | `lsblk` |
| RAID 파티션 확인 | `fdisk -l \| grep RAID` |
| RAID 상태 확인 | `cat /proc/mdstat` |
| RAID 상태 실시간 확인 | `watch cat /proc/mdstat` |
| RAID 상세 정보 | `mdadm --detail /dev/mdX` |
| RAID 생성 | `mdadm --create /dev/mdX --level=X --raid-devices=N <devices>` |
| RAID 설정 저장 | `mdadm --detail --scan > /etc/mdadm.conf` |
| RAID 중지 | `mdadm --stop /dev/mdX` |
| RAID 장치 장애 처리 | `mdadm /dev/mdX --fail <device>` |
| 장애 장치 제거 | `mdadm /dev/mdX --remove <device>` |
| 새 장치 추가 | `mdadm /dev/mdX --add <device>` |
| Superblock 삭제 | `mdadm --zero-superblock <devices>` |
| 파일시스템 확인 | `lsblk --fs` |
| 파일시스템 사용량 확인 | `df -hT` |
| 마운트 | `mount /dev/mdX /raidX` |
| 마운트 해제 | `umount /raidX` |
| fstab 변경 후 systemd 반영 | `systemctl daemon-reload` |

### 자주 사용하는 RAID 구성 절차

RAID를 실제 시스템에 구성할 때는 다음 순서를 기본 절차로 기억한다.

**① 디스크 확인 → ② RAID 파티션 생성 → ③ RAID 배열 생성 → ④ `/etc/mdadm.conf` 설정 → ⑤ 파일시스템 생성 → ⑥ 마운트 → ⑦ `/etc/fstab` 등록 → ⑧ 상태 확인**

장애가 발생한 경우에는 다음 순서로 처리한다.

**① 장애 확인 → ② `--fail` → ③ `--remove` → ④ 물리 디스크 교체 → ⑤ RAID 파티션 준비 → ⑥ `--add` → ⑦ Rebuild 확인**

RAID를 완전히 제거할 때는 다음 순서를 따른다.

**① 언마운트 → ② `/etc/fstab` 정리 → ③ `--stop` → ④ `--zero-superblock` → ⑤ `/etc/mdadm.conf` 정리**