# LVM(Logical Volume Manager) 관리

## 목차

- [1. LVM 개요](#1-lvm-개요)
	- [LVM이란](#lvm이란)
	- [LVM의 장점](#lvm의-장점)
	- [LVM과 RAID](#lvm과-raid)
- [2. LVM 구성요소](#2-lvm-구성요소)
	- [PV](#pvphysical-volume)
	- [VG](#vgvolume-group)
	- [LV](#lvlogical-volume)
	- [PE와 LE](#pe와-le)
- [3. LVM 작동 방식과 볼륨 유형](#3-lvm-작동-방식과-볼륨-유형)
	- [LVM 데이터 매핑](#lvm-데이터-매핑)
	- [Linear 볼륨](#linear-볼륨)
	- [Striping 볼륨](#striping-볼륨)
	- [Mirroring 볼륨](#mirroring-볼륨)
- [4. LVM 기본 구성 및 관리](#4-lvm-기본-구성-및-관리)
	- [LVM 구성 순서](#lvm-구성-순서)
	- [파티션 준비](#파티션-준비)
	- [PV 관리](#pv-관리)
	- [VG 관리](#vg-관리)
	- [LV 관리](#lv-관리)
	- [파일시스템 생성](#파일시스템-생성)
	- [마운트 및 fstab](#마운트-및-fstab)
- [5. LVM 용량 관리](#5-lvm-용량-관리)
	- [LV 용량 증가](#lv-용량-증가)
	- [파일시스템 용량 증가](#파일시스템-용량-증가)
	- [파일시스템 용량 감소](#파일시스템-용량-감소)
	- [파일시스템별 용량 변경](#파일시스템별-용량-변경)
	- [VG에 PV 추가 및 제거](#vg에-pv-추가-및-제거)
	- [PV 교체](#pv-교체)
- [6. LVM 고급 기능](#6-lvm-고급-기능)
	- [Thin Provisioning](#thin-provisioning)
	- [VDO](#vdo)
	- [LVM Snapshot](#lvm-snapshot)
	- [LVM RAID](#lvm-raid)
	- [Mirroring](#mirroring)
- [7. LVM 정보 확인 및 문제 해결](#7-lvm-정보-확인-및-문제-해결)
	- [정보 확인 명령어](#정보-확인-명령어)
	- [상세 출력 옵션](#상세-출력-옵션)
	- [fstab 오류 대응](#fstab-오류-대응)
	- [LVM 메타데이터 관리](#lvm-메타데이터-관리)
- [8. LVM 핵심 실습 정리](#8-lvm-핵심-실습-정리)
	- [기본 LVM 구성](#기본-lvm-구성)
	- [LVM 삭제 순서](#lvm-삭제-순서)
	- [자주 사용하는 명령어](#자주-사용하는-명령어)
	- [핵심 확인 사항](#핵심-확인-사항)


## 1. LVM 개요

### LVM이란

**LVM(Logical Volume Manager)**은 물리적인 디스크 또는 파티션을 **PV(Physical Volume)**로 구성하고, 이를 하나 이상의 **VG(Volume Group)**로 묶은 후 필요한 크기만큼 **LV(Logical Volume)**를 생성하여 사용하는 스토리지 관리 기술이다.

기본적인 구조는 다음과 같다.

**Physical Disk/Partition → PV → VG → LV → File System → Mount Point**

LVM을 사용하면 물리 디스크의 구성과 파일시스템에 할당되는 논리적인 저장 공간을 분리하여 관리할 수 있다.

### LVM의 장점

| 장점 | 설명 |
|---|---|
| **확장성** | VG에 PV를 추가하여 저장 공간을 확장할 수 있다. |
| **유연성** | VG의 여유 공간을 원하는 크기의 LV로 나누어 사용할 수 있다. |
| **관리 편의성** | 물리 디스크와 논리적인 저장 공간을 분리하여 관리할 수 있다. |
| **볼륨 확장** | VG의 여유 공간을 이용하여 LV를 확장할 수 있다. |
| **볼륨 축소** | 파일시스템의 지원 여부와 절차를 준수하면 LV 및 파일시스템을 축소할 수 있다. |
| **데이터 보호 기능** | Mirror, RAID, Snapshot 등의 기능을 제공한다. |
| **스토리지 통합** | 여러 PV를 하나의 VG로 구성하여 하나의 저장 공간처럼 관리할 수 있다. |

### LVM과 RAID

LVM과 RAID는 목적과 동작 계층이 다르다.

| 구분 | LVM | RAID |
|---|---|---|
| 주요 목적 | 논리적인 저장 공간 관리 | 성능 향상 및 데이터 보호 |
| 기본 구성 | PV → VG → LV | 여러 디스크를 RAID 배열로 구성 |
| 확장성 | VG에 PV 추가 가능 | RAID 레벨 및 구성에 따라 다름 |
| 주요 기능 | LV 확장, 축소, Snapshot, Thin Provisioning 등 | Striping, Mirroring, Parity 등 |

LVM에서는 **Linear, Striping, Mirroring, Snapshot, Thin Provisioning, RAID** 등의 형태로 저장 공간을 구성할 수 있다.


## 2. LVM 구성요소

### PV(Physical Volume)

**PV(Physical Volume)**는 LVM에서 사용할 수 있도록 물리적인 디스크 또는 파티션에 LVM 메타데이터를 생성한 것이다.

| 항목 | 설명 |
|---|---|
| **PV** | LVM에서 사용하는 물리적 저장 공간 |
| 대상 | 물리 디스크 또는 디스크 파티션 |
| 예 | `/dev/sda1`, `/dev/nvme0n1p1`, `/dev/sdb` |
| 주요 명령어 | `pvcreate`, `pvremove`, `pvs`, `pvdisplay`, `pvscan` |

PV는 VG에 포함되어야 LV를 생성하는 데 사용할 수 있다.

### VG(Volume Group)

**VG(Volume Group)**는 하나 이상의 PV를 하나의 논리적인 저장 공간으로 묶은 영역이다.

| 항목 | 설명 |
|---|---|
| **VG** | 하나 이상의 PV를 묶은 논리적인 저장 공간 |
| 구성 | PV 1개 이상 |
| 역할 | LV를 생성할 수 있는 공간 제공 |
| 주요 명령어 | `vgcreate`, `vgremove`, `vgextend`, `vgreduce`, `vgrename`, `vgs`, `vgdisplay`, `vgscan` |

VG의 여유 공간은 **VFree**로 확인할 수 있다.

### LV(Logical Volume)

**LV(Logical Volume)**는 VG에서 필요한 크기만큼 논리적으로 할당한 저장 공간이다.

LV에는 일반 파일시스템을 생성하거나 swap, 기타 블록 장치 용도로 사용할 수 있다.

| 항목 | 설명 |
|---|---|
| **LV** | VG에서 논리적으로 할당한 저장 공간 |
| 구성 | 하나 이상의 PV에서 제공되는 VG 공간 |
| 파일시스템 | XFS, ext4 등 생성 가능 |
| 주요 명령어 | `lvcreate`, `lvremove`, `lvextend`, `lvreduce`, `lvrename`, `lvs`, `lvdisplay`, `lvscan` |

### PE와 LE

**PE(Physical Extent)**는 PV를 일정한 크기의 블록으로 나눈 단위이며, **LE(Logical Extent)**는 LV를 구성하는 논리적인 할당 단위이다.

| 구분 | PE | LE |
|---|---|---|
| 전체 이름 | Physical Extent | Logical Extent |
| 소속 | PV | LV |
| 역할 | 물리 공간의 할당 단위 | 논리 공간의 할당 단위 |
| 기본 크기 | **4 MiB** | PE와 동일한 크기 |
| 설정 | VG 생성 시 지정 | PE 크기에 따라 결정 |

자료에서는 VG 생성 시 PE 크기를 `1 MiB ~ 256 MiB` 범위에서 지정할 수 있으며 기본값은 **4 MiB**로 설명한다.

예를 들어 `vgcreate`에서 `-s 16M`을 지정하면 해당 VG의 PE/LE 크기가 16 MiB로 설정된다.

**주의:** VG 생성 후에는 PE/LE 크기를 변경할 수 없으므로 VG 생성 시 신중하게 결정해야 한다.


## 3. LVM 작동 방식과 볼륨 유형

### LVM 데이터 매핑

LVM은 LV의 **LE(Logical Extent)**와 실제 PV의 **PE(Physical Extent)**를 매핑하여 논리적인 저장 공간을 제공한다.

따라서 사용자는 실제 데이터가 어느 물리 디스크의 어느 영역에 저장되어 있는지 직접 관리하지 않고 **LV를 블록 장치처럼 사용**할 수 있다.

루트, 부트, 기본 스왑 또는 덤프 등에 사용하는 LV는 자료에서 **연속적인 PE 영역**이 필요하다고 설명한다.

반면 일반적인 데이터 LV는 여러 PV에 걸쳐 PE가 분산될 수 있다.

### Linear 볼륨

**Linear Volume**은 LV의 데이터를 PV에 순차적으로 배치하는 기본적인 방식이다.

특별한 옵션 없이 LV를 생성하면 일반적으로 Linear 방식으로 생성된다.

| 특징 | 설명 |
|---|---|
| 기본 방식 | 특별한 옵션 없이 `lvcreate` 실행 |
| 데이터 배치 | PV 공간을 순차적으로 사용 |
| 성능 특성 | Striping과 같은 병렬 I/O 목적이 아님 |
| 데이터 보호 | 별도의 미러링 기능 없음 |

### Striping 볼륨

**Striping**은 여러 PV에 데이터를 분산하여 저장하는 방식이다.

여러 물리 장치에 I/O를 분산시킬 수 있으므로 순차적인 읽기/쓰기 성능 향상을 목적으로 사용할 수 있다.

| 옵션 | 설명 |
|---|---|
| `-i` | Stripe에 사용할 장치 수를 지정 |
| 예 | `-i 3`은 3개의 장치를 사용하는 Stripe 구성 |

### Mirroring 볼륨

**Mirroring**은 데이터를 여러 PV에 복제하여 저장하는 방식이다.

자료에서는 RAID 1과 유사한 방식으로 설명하며, 하나의 미러 사본을 사용하는 경우 원본과 미러에 각각 PE가 할당된다.

| 옵션 | 설명 |
|---|---|
| `-m` | Mirror 개수를 지정 |
| `-m 1` | 원본 + 1개의 미러 사본으로 구성 |
| 특징 | 데이터 보호 목적 |
| 필요 PV | 일반적으로 최소 2개의 PV 필요 |

예: `lvcreate -L 1G -n lv1 -m 1 VG_Name`


## 4. LVM 기본 구성 및 관리

### LVM 구성 순서

일반적인 LVM 구성 순서는 다음과 같다.

**디스크 준비 → 파티션 설정 → PV 생성 → VG 생성 → LV 생성 → 파일시스템 생성 → 마운트 → `/etc/fstab` 등록**

| 단계 | 주요 명령어 | 역할 |
|---|---|---|
| 1 | `parted`, `fdisk`, `gdisk` | 디스크 및 파티션 준비 |
| 2 | `pvcreate` | PV 생성 |
| 3 | `vgcreate` | VG 생성 |
| 4 | `lvcreate` | LV 생성 |
| 5 | `mkfs` | 파일시스템 생성 |
| 6 | `mount` | 파일시스템 마운트 |
| 7 | `/etc/fstab` | 부팅 시 자동 마운트 설정 |

### 파티션 준비

GPT 파티션을 사용하는 경우 `parted`를 이용하여 파티션을 생성하고 LVM 플래그를 지정할 수 있다.

주요 명령어는 `parted -s /dev/Disk_Name mklabel gpt mkpart primary 1MiB 100% set 1 lvm on`과 같은 형태로 사용한다.

과거 MBR/fdisk 환경에서는 LVM 파티션 ID를 **8e**, GPT 환경에서는 **8e00**으로 표시하는 방식이 자료에 제시되어 있다.

| 도구 | LVM 식별자 |
|---|---|
| `fdisk` / MBR | `8e` |
| `gdisk` / GPT | `8e00` |
| `parted` / GPT | `lvm` 플래그 |

### PV 관리

#### PV 생성

| 명령어 | 설명 |
|---|---|
| `pvcreate /dev/Device` | 하나의 장치를 PV로 생성 |
| `pvcreate /dev/Device1 /dev/Device2` | 여러 장치를 동시에 PV로 생성 |
| `pvcreate /dev/sd[abc]1` | 패턴을 이용하여 여러 장치 지정 |
| `pvcreate -f /dev/Device` | 기존 시그니처 등을 초기화하고 PV 생성 |

#### PV 삭제

| 명령어 | 설명 |
|---|---|
| `pvremove /dev/Device` | PV 제거 |
| `pvremove /dev/Device1 /dev/Device2` | 여러 PV 제거 |

PV를 제거하기 전에 **VG에 포함되어 있는지와 데이터가 사용 중인지 확인**해야 한다.

#### PV 정보 확인

| 명령어 | 설명 |
|---|---|
| `pvs` | PV 요약 정보 |
| `pvdisplay` | PV 상세 정보 |
| `pvscan` | 시스템의 PV 검색 |
| `lvmdiskscan` | PV로 사용할 수 있는 장치 검색 |

### VG 관리

#### VG 생성

| 명령어 | 설명 |
|---|---|
| `vgcreate VG_Name PV_Name` | 하나의 PV로 VG 생성 |
| `vgcreate VG_Name PV1 PV2` | 여러 PV를 포함하여 VG 생성 |
| `vgcreate VG_Name -s 16M PV_Name` | PE 크기를 16 MiB로 지정 |

#### VG 삭제

`vgremove VG_Name`을 사용하여 VG를 삭제한다.

VG에 LV가 존재하는 경우에는 **LV를 먼저 제거**해야 한다.

#### VG에 PV 추가 및 제거

| 명령어 | 설명 |
|---|---|
| `vgextend VG_Name PV_Name` | VG에 PV 추가 |
| `vgreduce VG_Name PV_Name` | VG에서 PV 제거 |

PV를 제거하기 전에 해당 PV의 **PFree**를 확인하여 사용 중인 공간이 없는지 확인해야 한다.

#### VG 이름 변경

| 명령어 | 설명 |
|---|---|
| `vgrename Old_Name New_Name` | VG 이름 변경 |
| `vgrename /dev/Old_Name /dev/New_Name` | 장치 경로를 이용한 이름 변경 |

#### VG 정보 확인

| 명령어 | 설명 |
|---|---|
| `vgs` | VG 요약 정보 |
| `vgdisplay` | VG 상세 정보 |
| `vgscan` | 시스템의 VG 검색 |

### LV 관리

#### LV 생성

| 옵션 | 설명 |
|---|---|
| `-L Size` | 용량 단위로 LV 크기 지정 |
| `-l Number` | LE 개수로 LV 크기 지정 |
| `-n Name` | LV 이름 지정 |
| `-i Number` | Striping에 사용할 장치 수 지정 |
| `-m Number` | Mirror 개수 지정 |
| `-V Size` | Thin LV의 논리적 크기 지정 |
| `-T VG/ThinPool` | Thin Pool 지정 |

용량 단위를 지정하지 않은 `-L` 값은 자료에서 MB 단위로 설명한다.

`-l 100%FREE`는 VG의 남은 공간 전체를 사용하는 방식이다.

**주의:** `100%FREE`는 `-L`이 아니라 **`-l` 옵션**과 함께 사용한다.

#### LV 삭제

`lvremove /dev/VG_Name/LV_Name` 형식으로 삭제한다.

활성 상태의 LV를 삭제할 때는 삭제 확인 메시지가 발생할 수 있으며, 강제 삭제가 필요한 경우 자료에서는 `-f` 또는 `-y` 사용 예를 제시한다.

#### LV 이름 변경

| 명령어 | 설명 |
|---|---|
| `lvrename /dev/VG_Name/Old_Name /dev/VG_Name/New_Name` | 전체 경로 사용 |
| `lvrename VG_Name Old_Name New_Name` | VG와 LV 이름을 분리하여 지정 |

#### LV 용량 변경

| 명령어 | 설명 |
|---|---|
| `lvextend -L Size LV_Path` | 최종 크기를 지정하여 증가 |
| `lvextend -L +Size LV_Path` | 지정한 크기만큼 증가 |
| `lvextend -l +100%FREE LV_Path` | VG의 남은 공간을 모두 추가 |
| `lvreduce -L Size LV_Path` | 최종 크기를 지정하여 감소 |
| `lvreduce -L -Size LV_Path` | 지정한 크기만큼 감소 |
| `lvreduce -l -Number LV_Path` | 지정한 LE 수만큼 감소 |

LV의 크기를 줄일 때는 **파일시스템의 실제 크기를 먼저 줄인 후 LV 크기를 줄여야 한다.**


### 파일시스템 생성

LV 생성 후에는 원하는 파일시스템을 생성한다.

| 명령어 | 설명 |
|---|---|
| `mkfs.ext4 /dev/VG_Name/LV_Name` | ext4 파일시스템 생성 |
| `mkfs -t ext4 /dev/VG_Name/LV_Name` | 파일시스템 유형을 지정하여 생성 |
| `mkfs.xfs /dev/VG_Name/LV_Name` | XFS 파일시스템 생성 |
| `lsblk -pf` | 블록 장치와 파일시스템 정보 확인 |

**주의:** 기존 데이터가 있는 LV에 `mkfs`를 실행하면 기존 파일시스템과 데이터가 손상될 수 있다.

### 마운트 및 fstab

LV에 파일시스템을 생성한 후 빈 디렉터리를 마운트 포인트로 사용하여 마운트한다.

| 명령어 | 설명 |
|---|---|
| `mkdir -p /Mount_Point` | 마운트 포인트 생성 |
| `mount /dev/VG_Name/LV_Name /Mount_Point` | LV를 직접 마운트 |
| `mount /Mount_Point` | `/etc/fstab` 정보를 이용하여 마운트 |
| `umount /Mount_Point` | 마운트 해제 |
| `df -hT` | 파일시스템 사용량과 유형 확인 |
| `mount` | 현재 마운트 상태 확인 |

부팅 후에도 자동으로 마운트하려면 `/etc/fstab`에 등록해야 한다.

일반적인 형식은 `Device Mount_Point FileSystem_Type Mount_Options Dump Pass`이다.

예를 들어 `UUID="UUID_Value" /Mount_Point ext4 defaults 0 2`와 같이 지정할 수 있다.

XFS는 자료에서 `/etc/fstab`의 마지막 필드에 일반적으로 `0`을 사용하는 것으로 설명하며, ext4는 일반적으로 `1`, `2` 등의 값을 사용한다.

**중요:** `/etc/fstab`을 수정한 후에는 `mount -a` 등을 이용하여 설정 오류가 없는지 확인하는 것이 중요하다.


## 5. LVM 용량 관리

### LV 용량 증가

LV 용량을 증가시키려면 먼저 VG에 충분한 **VFree**가 있는지 확인한다.

일반적인 순서는 다음과 같다.

`vgs` → `VFree` 확인 → `lvextend` → 파일시스템 확장 → `df -hT` 확인

파일시스템까지 한 번에 확장하려면 `lvextend -r` 옵션을 사용할 수 있다.

### 파일시스템 용량 증가

#### ext4

ext4는 LV 확장 후 `resize2fs`를 이용하여 파일시스템을 확장할 수 있다.

대표적인 방법은 다음과 같다.

`lvextend -L +50G /dev/VG_Name/LV_Name` → `resize2fs /dev/VG_Name/LV_Name`

또는 `lvextend -L +50G -r /dev/VG_Name/LV_Name`처럼 사용할 수 있다.

#### XFS

XFS는 LV 확장 후 `xfs_growfs`를 사용하여 파일시스템을 확장한다.

대표적인 방법은 `lvextend -L +50G /dev/VG_Name/LV_Name` → `xfs_growfs /Mount_Point`이다.

또는 `lvextend -L +50G -r /dev/VG_Name/LV_Name`처럼 사용할 수 있다.

### 파일시스템 용량 감소

**파일시스템 감소는 증가보다 위험성이 높으며, 반드시 파일시스템이 지원하는지 확인해야 한다.**

자료에서는 다음과 같이 설명한다.

| 파일시스템 | 증가 | 감소 |
|---|---:|---:|
| **ext4** | 가능 | 가능 |
| **XFS** | 가능 | **불가능** |

ext4의 축소 순서는 다음과 같다.

**백업 → 언마운트 → 파일시스템 점검 → 파일시스템 축소 → LV 축소 → 마운트 → 확인**

| 단계 | 명령어 | 목적 |
|---|---|---|
| 1 | `umount /Mount_Point` | 파일시스템 언마운트 |
| 2 | `e2fsck -f /dev/VG_Name/LV_Name` | 파일시스템 검사 |
| 3 | `resize2fs /dev/VG_Name/LV_Name Final_Size` | 파일시스템 축소 |
| 4 | `lvreduce -L Final_Size /dev/VG_Name/LV_Name` | LV 축소 |
| 5 | `mount /Mount_Point` | 다시 마운트 |
| 6 | `df -hT` | 결과 확인 |

**절대로 LV를 먼저 줄인 후 파일시스템을 줄여서는 안 된다.**

LV만 먼저 축소하면 파일시스템이 LV보다 커지는 상황이 발생하여 데이터 손상이 발생할 수 있다.

### 파일시스템별 용량 변경

| 작업 | XFS | ext4 |
|---|---|---|
| LV 증가 | `lvextend` | `lvextend` |
| FS 증가 | `xfs_growfs` | `resize2fs` |
| `lvextend -r` | 지원 | 지원 |
| FS 감소 | **지원하지 않음** | 지원 |
| LV 감소 | 파일시스템 감소 불가로 일반적인 안전한 축소 불가 | FS를 먼저 축소한 후 가능 |

### VG에 PV 추가 및 제거

VG의 저장 공간이 부족하면 새로운 디스크 또는 파티션을 PV로 만든 후 VG에 추가한다.

순서는 다음과 같다.

`pvcreate /dev/New_PV` → `vgextend VG_Name /dev/New_PV` → `vgs` → `pvs`

VG에서 PV를 제거하려면 해당 PV에 저장된 데이터를 다른 PV로 이동한 후 `vgreduce`를 사용한다.

### PV 교체

VG 내부의 기존 PV를 새로운 PV로 교체할 때는 **`pvmove`**가 핵심 명령어이다.

일반적인 절차는 다음과 같다.

**새 디스크 준비 → 새 PV 생성 → VG에 추가 → `pvmove`로 데이터 이동 → 기존 PV 제거**

예를 들어 새 PV를 VG에 추가한 후 `pvmove /dev/Old_PV /dev/New_PV` 형태로 데이터를 이동할 수 있다.

데이터 이동이 완료된 후 `vgreduce VG_Name /dev/Old_PV`를 수행한다.

**pvmove는 VG 내부에서 PV를 교체하거나 데이터를 다른 PV로 이동할 때 사용하는 핵심 명령어이다.**


## 6. LVM 고급 기능

### Thin Provisioning

**Thin Provisioning**은 실제 물리 공간보다 큰 논리적 용량을 미리 할당할 수 있도록 하는 기술이다.

실제 데이터가 기록될 때 물리 공간을 동적으로 할당한다.

| 구분 | Thick LV | Thin LV |
|---|---|---|
| 공간 할당 | 실제 공간을 미리 할당 | 실제 사용량에 따라 동적 할당 |
| 논리적 크기 | 실제 공간 범위 내 | 실제 물리 공간보다 크게 설정 가능 |
| 기반 | 일반 VG 공간 | Thin Pool |
| 관리 주의점 | VG 여유 공간 | Thin Pool 공간 부족 주의 |

#### Thin Pool 생성

`lvcreate --type thin-pool -l 100%FREE -n ThinPool_Name VG_Name`

Thin Pool에는 데이터 영역과 메타데이터 영역이 함께 구성된다.

#### Thin LV 생성

`lvcreate -V 10G -T VG_Name/ThinPool_Name -n ThinLV_Name`

| 옵션 | 설명 |
|---|---|
| `-V` | Thin LV의 논리적 크기 |
| `-T` | 사용할 Thin Pool |
| `-n` | Thin LV 이름 |

Thin LV의 논리적인 크기가 Thin Pool보다 클 수 있지만, **실제 데이터 사용량이 Thin Pool의 물리 공간을 초과하면 문제가 발생할 수 있다.**

### VDO

**VDO(Virtual Data Optimizer)**는 인라인 수준에서 **중복 제거(Deduplication), 압축(Compression), Thin Provisioning** 기능을 제공하는 스토리지 기술이다.

자료에서는 LVM VDO를 다음 두 영역으로 설명한다.

| 구성요소 | 역할 |
|---|---|
| **VDO Pool LV** | 데이터 저장, 중복 제거, 압축 수행 |
| **VDO LV** | 실제 사용자에게 제공되는 논리적 가상 장치 |

VDO를 사용하려면 자료에서는 `vdo`, `kmod-kvdo` 패키지를 설치하는 과정을 제시한다.

VDO LV 생성은 `lvcreate --type vdo --name VDO_LV_Name --size 5G VG_Name` 형태를 사용한다.

### LVM Snapshot

**LVM Snapshot**은 특정 시점의 LV 상태를 보존하기 위한 기능이다.

Snapshot은 원본 LV의 전체 데이터를 즉시 복사하는 방식이 아니라 **COW(Copy-On-Write)** 방식으로 변경된 영역을 추적한다.

| 용도 | 설명 |
|---|---|
| 백업 | 특정 시점의 일관된 상태 확보 |
| 테스트 | 변경 작업 전 상태 보존 |
| 업그레이드 | 업그레이드 전 상태로 롤백 |
| 복구 | Snapshot Merge를 통한 원본 상태 복구 |

#### Thick Snapshot

Thick Snapshot은 Snapshot 생성 시 별도의 공간을 할당한다.

`lvcreate -s -L 500M -n Snapshot_Name VG_Name/Source_LV`

Snapshot의 크기가 원본 변경량보다 부족하면 Snapshot이 invalid 상태가 될 수 있으므로 **변경량을 고려하여 Snapshot 크기를 결정해야 한다.**

#### Snapshot 자동 확장

자료에서는 `/etc/lvm/lvm.conf`의 다음 설정을 제시한다.

`snapshot_autoextend_threshold = 70`

`snapshot_autoextend_percent = 20`

| 설정 | 의미 |
|---|---|
| `snapshot_autoextend_threshold` | Snapshot 사용률 기준 |
| `snapshot_autoextend_percent` | 기준 도달 시 증가시킬 비율 |

설정 변경 후 자료에서는 `systemctl restart lvm2-monitor`를 이용하여 적용한다.

#### Thin Snapshot

Thin Snapshot은 Thin Pool 기반으로 동작하며 별도의 Snapshot 크기를 지정하지 않고 Thin Pool 공간을 공유한다.

`lvcreate -s -n ThinSnapshot_Name VG_Name/ThinLV_Name`

Thin Snapshot은 일반적으로 별도의 마운트 대상이 아니라 **롤백을 위한 메커니즘**으로 사용할 수 있다.

#### Snapshot Merge

Snapshot을 원본 LV에 병합하면 Snapshot 생성 시점의 상태로 원본을 되돌릴 수 있다.

`lvconvert --merge VG_Name/Snapshot_Name`

일반적인 복구 절차는 다음과 같다.

**원본 언마운트 → Snapshot Merge → 원본 활성화 → 다시 마운트 → 데이터 확인**

Snapshot Merge가 완료되면 Snapshot LV는 삭제된다.

### LVM RAID

LVM은 RAID 형태의 논리 볼륨을 생성할 수 있다.

자료에서는 **RAID 0, 1, 4, 5, 6, 10**을 지원하는 것으로 설명한다.

| RAID | 주요 특징 | 자료의 최소 구성 |
|---|---|---:|
| **RAID 0** | Striping, 성능 중심 | 2개 이상 |
| **RAID 1** | Mirroring, 데이터 보호 | 2개 |
| **RAID 4** | 전용 Parity | 자료에서 지원 레벨로 제시 |
| **RAID 5** | 분산 Parity | **3개 이상** |
| **RAID 6** | 이중 Parity | **5개 이상** |
| **RAID 10** | Striping + Mirroring | 4개 이상 |

#### RAID 5

자료에서는 RAID 5를 **2개의 데이터 장치 + 1개의 패리티 장치** 구조로 설명한다.

예: `lvcreate --type raid5 -L 200M -n raid5 -i 2 VG_Name`

#### RAID 6

자료에서는 RAID 6을 **3개의 데이터 장치 + 2개의 패리티 장치** 구조로 설명한다.

예: `lvcreate --type raid6 -L 300M -n raid6 -i 3 VG_Name`

#### RAID 10

RAID 10은 **Mirroring과 Striping을 결합**한 구조이다.

자료의 실습에서는 `--type raid10`을 사용하여 RAID 10 LV를 생성한다.

### Mirroring

일반적인 Mirror LV는 `lvcreate -m` 옵션을 이용하여 생성할 수 있다.

예를 들어 `lvcreate -L 500M -n Mirror_LV -m 1 VG_Name`은 원본과 하나의 미러 사본을 구성하는 방식이다.

Mirror는 **데이터 보호**가 목적이며, Striping은 주로 **I/O 성능 향상**을 목적으로 한다.


## 7. LVM 정보 확인 및 문제 해결

### 정보 확인 명령어

LVM 관리에서는 생성·삭제 명령어뿐만 아니라 현재 구성을 확인하는 명령어가 중요하다.

| 명령어 | 주요 목적 |
|---|---|
| `pvs` | PV 요약 정보 |
| `pvdisplay` | PV 상세 정보 |
| `pvscan` | PV 검색 |
| `vgs` | VG 요약 정보 |
| `vgdisplay` | VG 상세 정보 |
| `vgscan` | VG 검색 |
| `lvs` | LV 요약 정보 |
| `lvdisplay` | LV 상세 정보 |
| `lvscan` | LV 검색 |
| `lsblk` | 블록 장치 구조 확인 |
| `lsblk -f` | 파일시스템 정보 포함 |
| `df -hT` | 파일시스템 사용량 확인 |
| `mount` | 현재 마운트 상태 확인 |

### 상세 출력 옵션

LVM 명령어는 `--help`, `man`, verbose 옵션 등을 이용하여 상세 정보를 확인할 수 있다.

| 방법 | 설명 |
|---|---|
| `command --help` | 명령어 옵션 확인 |
| `man command` | 매뉴얼 페이지 확인 |
| `-v` | 상세 출력 |
| `-vv` | 더 상세한 출력 |
| `-vvv` | 매우 상세한 출력 |
| `-vvvv` | 최대 수준의 디버깅 출력 |

예를 들어 `pvcreate --help`, `man pvcreate`, `lvcreate -vvv` 등의 형태로 사용할 수 있다.

### LVM 명령어 자동 완성

Bash의 Tab 자동 완성을 이용하여 LVM 관련 명령어를 탐색할 수 있다.

`pv` 입력 후 `[Tab][Tab]`, `vg` 입력 후 `[Tab][Tab]`, `lv` 입력 후 `[Tab][Tab]`과 같이 사용할 수 있다.

### fstab 오류 대응

`/etc/fstab`의 설정 오류로 부팅 과정에서 문제가 발생할 수 있다.

자료에서는 부팅 오류가 발생했을 때 다음과 같은 대응 절차를 제시한다.

**root 환경 진입 → `/etc/fstab` 수정 → 루트 파일시스템을 읽기/쓰기로 다시 마운트 → 설정 확인 → `systemctl daemon-reload` → 재부팅 과정 진행**

루트 파일시스템이 읽기 전용인 경우 `mount -o remount,rw /`를 이용하여 쓰기 가능한 상태로 다시 마운트할 수 있다.

### LVM 메타데이터 관리

LVM은 구성 정보를 메타데이터로 관리하므로 메타데이터 백업 및 복구 명령어를 알아둘 필요가 있다.

| 명령어 | 설명 |
|---|---|
| `vgcfgbackup` | VG 구성 메타데이터 백업 |
| `vgcfgrestore` | VG 구성 메타데이터 복구 |
| `vgck` | VG 메타데이터 검사 |
| `pvck` | PV 메타데이터 검사 |
| `pvmove` | PV 간 데이터 이동 |
| `pvresize` | PV 크기 변경 |

### PV 제거 후 정보가 남아 있는 경우

자료에서는 `pvremove` 후에도 `pvscan` 결과에 정보가 남는 경우를 설명한다.

이 경우 남아 있는 LVM 관련 메타데이터를 확인하고 필요한 상황에서는 장치의 LVM 시그니처를 제거하는 작업을 수행할 수 있다.

**주의:** 디스크의 특정 영역을 직접 덮어쓰는 방식은 해당 장치의 기존 데이터와 메타데이터를 파괴할 수 있으므로 반드시 대상 장치를 정확하게 확인해야 한다.


## 8. LVM 핵심 실습 정리

### 기본 LVM 구성

일반적인 LVM 구성 과정은 다음과 같이 기억하면 된다.

**① 디스크 준비**

`lsblk`

**② GPT 파티션 및 LVM 플래그 설정**

`parted -s /dev/Disk_Name mklabel gpt mkpart primary 1MiB 100% set 1 lvm on`

**③ PV 생성**

`pvcreate /dev/Partition_Name`

**④ VG 생성**

`vgcreate VG_Name /dev/Partition_Name`

**⑤ LV 생성**

`lvcreate -L Size -n LV_Name VG_Name`

**⑥ 파일시스템 생성**

`mkfs.ext4 /dev/VG_Name/LV_Name`

또는 `mkfs.xfs /dev/VG_Name/LV_Name`

**⑦ 마운트 포인트 생성**

`mkdir -p /Mount_Point`

**⑧ 마운트**

`mount /dev/VG_Name/LV_Name /Mount_Point`

**⑨ `/etc/fstab` 등록**

`vi /etc/fstab`

**⑩ 상태 확인**

`df -hT`, `lsblk -f`, `pvs`, `vgs`, `lvs`

### LVM 삭제 순서

LVM을 제거할 때는 구성 계층의 반대 순서로 삭제한다.

**마운트 해제 → LV 삭제 → VG 삭제 → PV 삭제**

| 순서 | 작업 | 명령어 |
|---|---|---|
| 1 | fstab 등록 제거 | `vi /etc/fstab` |
| 2 | 마운트 해제 | `umount /Mount_Point` |
| 3 | LV 삭제 | `lvremove /dev/VG_Name/LV_Name` |
| 4 | VG 삭제 | `vgremove VG_Name` |
| 5 | PV 삭제 | `pvremove /dev/PV_Name` |

**주의:** LV를 삭제하면 해당 LV의 파일시스템과 데이터에 접근할 수 없으므로 삭제 전에 반드시 백업 여부를 확인한다.

### 자주 사용하는 명령어

#### 디스크 및 파일시스템 확인

`lsblk`

`lsblk -f`

`lsblk -pf`

`df -hT`

`mount`

#### PV

`pvcreate /dev/PV_Name`

`pvremove /dev/PV_Name`

`pvs`

`pvdisplay`

`pvscan`

`pvmove /dev/Old_PV /dev/New_PV`

#### VG

`vgcreate VG_Name /dev/PV_Name`

`vgremove VG_Name`

`vgextend VG_Name /dev/New_PV`

`vgreduce VG_Name /dev/Old_PV`

`vgrename Old_Name New_Name`

`vgs`

`vgdisplay`

`vgscan`

#### LV

`lvcreate -L Size -n LV_Name VG_Name`

`lvcreate -l 100%FREE -n LV_Name VG_Name`

`lvremove /dev/VG_Name/LV_Name`

`lvrename VG_Name Old_Name New_Name`

`lvextend -L +Size /dev/VG_Name/LV_Name`

`lvextend -l +100%FREE /dev/VG_Name/LV_Name`

`lvreduce -L -Size /dev/VG_Name/LV_Name`

`lvs`

`lvdisplay`

`lvscan`

#### 파일시스템

`mkfs.ext4 /dev/VG_Name/LV_Name`

`mkfs.xfs /dev/VG_Name/LV_Name`

`resize2fs /dev/VG_Name/LV_Name`

`xfs_growfs /Mount_Point`

#### 마운트

`mkdir -p /Mount_Point`

`mount /dev/VG_Name/LV_Name /Mount_Point`

`umount /Mount_Point`

`mount -a`

### 자주 사용하는 사용 예제

#### VG의 남은 공간을 모두 사용하여 LV 생성

`lvcreate -l 100%FREE -n LV_Name VG_Name`

#### LV 용량을 1GB 증가

`lvextend -L +1G /dev/VG_Name/LV_Name`

#### LV와 파일시스템을 동시에 증가

`lvextend -L +1G -r /dev/VG_Name/LV_Name`

#### ext4 파일시스템 확장

`resize2fs /dev/VG_Name/LV_Name`

#### XFS 파일시스템 확장

`xfs_growfs /Mount_Point`

#### VG에 새로운 PV 추가

`vgextend VG_Name /dev/New_PV`

#### VG에서 PV 제거

`vgreduce VG_Name /dev/Old_PV`

#### PV의 데이터 이동

`pvmove /dev/Old_PV /dev/New_PV`

#### Thick Snapshot 생성

`lvcreate -s -L 500M -n Snapshot_Name VG_Name/Source_LV`

#### Snapshot Merge

`lvconvert --merge VG_Name/Snapshot_Name`

#### Thin Pool 생성

`lvcreate --type thin-pool -l 100%FREE -n ThinPool_Name VG_Name`

#### Thin LV 생성

`lvcreate -V 10G -T VG_Name/ThinPool_Name -n ThinLV_Name`

#### LVM RAID 5 생성

`lvcreate --type raid5 -L 200M -n RAID5_Name -i 2 VG_Name`

#### LVM RAID 6 생성

`lvcreate --type raid6 -L 300M -n RAID6_Name -i 3 VG_Name`

### 핵심 확인 사항

LVM 관리에서 다음 관계를 반드시 기억한다.

**PV → VG → LV → File System → Mount**

용량 확장에서는 다음 관계를 기억한다.

**VG 여유 공간 확인 → LV 확장 → 파일시스템 확장**

용량 축소에서는 다음 순서를 반드시 지킨다.

**백업 → 언마운트 → 파일시스템 점검 → 파일시스템 축소 → LV 축소 → 마운트 → 확인**

특히 **XFS는 파일시스템 자체의 축소를 지원하지 않으므로**, ext4와 동일한 방식으로 `lvreduce`를 적용해서는 안 된다.

LVM의 핵심 관리 명령어는 다음과 같이 묶어서 기억하면 효율적이다.

| 대상 | 생성 | 삭제 | 정보 확인 | 확장/축소 |
|---|---|---|---|---|
| **PV** | `pvcreate` | `pvremove` | `pvs`, `pvdisplay`, `pvscan` | `pvresize` |
| **VG** | `vgcreate` | `vgremove` | `vgs`, `vgdisplay`, `vgscan` | `vgextend`, `vgreduce` |
| **LV** | `lvcreate` | `lvremove` | `lvs`, `lvdisplay`, `lvscan` | `lvextend`, `lvreduce` |

**최종 핵심:** LVM은 물리적인 저장 공간인 **PV**를 **VG**로 묶고, VG에서 필요한 크기의 **LV**를 논리적으로 할당하여 파일시스템과 마운트 포인트를 구성하는 스토리지 관리 체계이다.