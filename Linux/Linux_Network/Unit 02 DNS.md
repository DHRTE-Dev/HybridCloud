# Linux CentOS 10 BIND DNS Server 정리

## 목차

1. [DNS 개요 및 주요 용어](#1-dns-개요-및-주요-용어)
	- [DNS의 역할](#dns의-역할)
	- [도메인과 FQDN](#도메인과-fqdn)
	- [DNS 서버 종류](#dns-서버-종류)
2. [DNS 이름 해석 구조](#2-dns-이름-해석-구조)
	- [/etc/hosts와 DNS](#etchosts와-dns)
	- [정방향 조회와 역방향 조회](#정방향-조회와-역방향-조회)
	- [도메인 위임](#도메인-위임)
3. [BIND DNS 서버 기본 구성](#3-bind-dns-서버-기본-구성)
	- [패키지 및 서비스](#패키지-및-서비스)
	- [주요 설정 파일](#주요-설정-파일)
	- [포트와 프로토콜](#포트와-프로토콜)
4. [CentOS DNS 서버 선수 설정](#4-centos-dns-서버-선수-설정)
	- [SELinux](#selinux)
	- [firewalld](#firewalld)
	- [/etc/hosts 및 NetworkManager](#etchosts-및-networkmanager)
	- [공개키 인증](#공개키-인증)
5. [BIND named.conf 설정](#5-bind-namedconf-설정)
	- [options 설정](#options-설정)
	- [zone 설정](#zone-설정)
	- [주석 및 include](#주석-및-include)
	- [설정 문법 검사](#설정-문법-검사)
6. [Zone 파일과 Resource Record](#6-zone-파일과-resource-record)
	- [Zone 파일 기본 형식](#zone-파일-기본-형식)
	- [SOA Record](#soa-record)
	- [주요 Resource Record](#주요-resource-record)
	- [정방향 Zone 파일](#정방향-zone-파일)
	- [역방향 Zone 파일](#역방향-zone-파일)
7. [DNS 서버 구축 절차](#7-dns-서버-구축-절차)
	- [Root 및 TLD DNS 구성](#root-및-tld-dns-구성)
	- [Authoritative DNS 구성](#authoritative-dns-구성)
	- [DNS 클라이언트 설정](#dns-클라이언트-설정)
8. [DNS 질의 및 검증 명령어](#8-dns-질의-및-검증-명령어)
	- [nslookup](#nslookup)
	- [dig](#dig)
	- [host](#host)
	- [서비스 및 네트워크 확인](#서비스-및-네트워크-확인)
9. [DNS 고급 구성](#9-dns-고급-구성)
	- [DNS Load Balancing](#dns-load-balancing)
	- [Domain Delegation](#domain-delegation)
	- [Master/Slave DNS](#masterslave-dns)
	- [Zone Transfer](#zone-transfer)
10. [DNS 동적 업데이트 및 rndc](#10-dns-동적-업데이트-및-rndc)
	- [SOA Serial과 Slave 동기화](#soa-serial과-slave-동기화)
	- [rndc](#rndc)
	- [Dynamic DNS와 nsupdate](#dynamic-dns와-nsupdate)
11. [Forwarding DNS Server](#11-forwarding-dns-server)
	- [Forwarders](#forwarders)
	- [Forward Only](#forward-only)
	- [DNS Client 구성](#dns-client-구성-1)
12. [DNS 보안 및 장애 처리](#12-dns-보안-및-장애-처리)
	- [allow-query](#allow-query)
	- [allow-transfer](#allow-transfer)
	- [allow-update](#allow-update)
	- [chroot 및 패키지 보안](#chroot-및-패키지-보안)
	- [DNSSEC과 TSIG](#dnssec과-tsig)
	- [장애 처리 순서](#장애-처리-순서)
13. [핵심 명령어 및 설정 요약](#13-핵심-명령어-및-설정-요약)

## 1. DNS 개요 및 주요 용어

### DNS의 역할

**DNS(Domain Name System)**는 도메인 이름과 IP 주소를 매핑하여 사람이 기억하기 쉬운 이름으로 네트워크 자원을 찾을 수 있도록 하는 시스템이다.

DNS의 핵심 기능은 다음과 같다.

| 개념 | 설명 |
|---|---|
| DNS | 도메인 이름과 IP 주소를 매핑하는 분산형 이름 해석 시스템 |
| Name Server | DNS 서비스를 제공하는 서버 |
| Resolver | DNS 서버에 질의하여 이름을 해석하는 클라이언트 측 구성요소 |
| DDNS | 외부 프로그램 또는 에이전트를 통해 DNS 정보를 동적으로 갱신하는 기능 |
| DNS Query | DNS 서버에 이름 해석 정보를 요청하는 작업 |
| Authoritative DNS | 특정 Zone에 대해 공식적인 DNS 정보를 보유하고 응답하는 서버 |

DNS 정보는 하나의 중앙 서버에 모두 저장하지 않고 **도메인 계층 구조에 따라 분산 관리**된다.

### 도메인과 FQDN

도메인은 여러 계층으로 구성되며, 가장 오른쪽부터 상위 계층으로 해석한다.

| 용어 | 설명 |
|---|---|
| Host Name | 특정 시스템 또는 서비스의 이름 |
| Domain Name | 시스템을 소속시키는 도메인 이름 |
| FQDN | Host Name과 Domain Name을 합친 완전한 이름 |
| TLD | 최상위 도메인. `.com`, `.net`, `.org`, 국가 코드 등이 해당 |
| gTLD | 일반 최상위 도메인 |
| ccTLD | 국가 코드 최상위 도메인 |
| sTLD | 특정 기관이나 목적을 위한 스폰서형 최상위 도메인 |

도메인 이름 구성 예시는 다음과 같이 일반화할 수 있다.

| 표현 | 의미 |
|---|---|
| `www` | Host Name |
| `example.test` | Domain Name |
| `www.example.test` | FQDN |

### DNS 서버 종류

| 종류 | 역할 |
|---|---|
| **Master / Primary** | Zone의 원본 정보를 관리하고 권한 있는 응답을 제공 |
| **Slave / Secondary** | Master로부터 Zone 정보를 전달받아 보조 DNS 서버 역할 수행 |
| **Caching-only** | 자체 Zone을 관리하지 않고 질의 결과를 캐시하여 응답 |
| **Forwarding DNS** | 다른 DNS 서버로 질의를 전달하여 이름을 해석 |

Master와 Slave는 서로 다른 서버에 배치하여 장애 대응과 부하 분산에 활용하는 것이 일반적이다.

Caching-only 서버는 질의 결과를 일정 시간 동안 저장하여 반복적인 외부 DNS 질의를 줄인다.

Forwarding DNS는 내부 클라이언트의 질의를 지정된 DNS 서버로 전달한다.

## 2. DNS 이름 해석 구조

### /etc/hosts와 DNS

DNS가 일반화되기 전에는 `/etc/hosts`에 Host Name과 IP 주소를 직접 등록하여 이름을 해석할 수 있었다.

| 파일 | 용도 |
|---|---|
| `/etc/hosts` | 로컬 Host Name ↔ IP 주소 매핑 |
| `/etc/host.conf` | 로컬 이름 해석 순서 등의 설정 |
| `/etc/resolv.conf` | 사용할 DNS 서버와 검색 도메인 설정 |

`/etc/hosts`는 소규모 환경에는 사용할 수 있지만 대규모 환경에서는 관리가 어렵기 때문에 DNS 서버를 사용하는 것이 적절하다.

NetworkManager가 네트워크 설정을 관리하는 환경에서는 `/etc/resolv.conf`를 직접 수정하기보다 **NetworkManager 또는 `nmcli`를 이용해 DNS 설정을 변경**하는 방식을 사용한다.

### 정방향 조회와 역방향 조회

| 조회 방식 | 방향 | Zone |
|---|---|---|
| **Forward Lookup** | Domain → IP | 일반 도메인 Zone |
| **Reverse Lookup** | IP → Domain | `in-addr.arpa` Zone |
| IPv6 Reverse Lookup | IPv6 → Domain | `ip6.arpa` Zone |

정방향 조회에서는 주로 **A, AAAA** 등의 Record를 사용한다.

역방향 조회에서는 **PTR** Record를 사용한다.

### 도메인 위임

**Domain Delegation**은 상위 DNS 서버가 특정 하위 도메인의 관리를 다른 DNS 서버에 위임하는 방식이다.

예를 들어 상위 Zone이 `<TLD>`를 관리하고, 하위 `<DOMAIN>` Zone을 별도의 DNS 서버가 관리하도록 구성할 수 있다.

위임 과정의 핵심은 상위 Zone에 하위 Zone의 **NS Record와 필요한 Glue A/AAAA Record**를 등록하는 것이다.

| 구성 | 역할 |
|---|---|
| Root Zone | TLD DNS 서버 정보 제공 |
| TLD Zone | 하위 도메인 DNS 서버 정보 제공 |
| Authoritative Zone | 실제 호스트 및 서비스의 DNS Record 관리 |

## 3. BIND DNS 서버 기본 구성

### 패키지 및 서비스

CentOS 환경의 BIND DNS 서버 구성에는 주로 다음 패키지를 사용한다.

| 패키지 | 설명 |
|---|---|
| `bind` | BIND DNS 서버 |
| `bind-utils` | `dig`, `host`, `nslookup`, `nsupdate` 등의 DNS 유틸리티 |
| `bind-chroot` | BIND 실행 환경을 chroot 방식으로 분리할 때 사용 |

패키지 설치:

`dnf -y install bind bind-utils`

필요한 경우:

`dnf -y install bind bind-utils bind-chroot`

서비스 이름은 **`named.service`**이다.

| 항목 | 값 |
|---|---|
| 데몬 | `named` |
| 서비스 | `named.service` |
| DNS 포트 | `53` |
| 일반 DNS 프로토콜 | UDP/TCP |
| RNDC 제어 포트 | `953/TCP` |
| 주 설정 디렉터리 | `/etc` |
| Zone 파일 디렉터리 | `/var/named` |

### 주요 설정 파일

| 파일/디렉터리 | 용도 |
|---|---|
| `/etc/named.conf` | BIND 주 설정 파일 |
| `/etc/named.rfc1912.zones` | Zone 설정을 분리하여 관리 |
| `/etc/named.root.key` | Root DNS 관련 키 정보 |
| `/etc/rndc.conf` | RNDC 클라이언트 설정 |
| `/etc/rndc.key` | 기본 RNDC 인증 키 |
| `/var/named/` | Zone 및 DNS 데이터 저장 |
| `/var/named/named.ca` | Root Hint 정보 |
| `/var/named/named.localhost` | localhost Zone |
| `/var/named/named.loopback` | Loopback Reverse Zone |
| `/var/named/slaves/` | Slave DNS가 전달받은 Zone 저장 |

### 포트와 프로토콜

| 통신 | 프로토콜 | 포트 | 용도 |
|---|---|---:|---|
| 일반 DNS Query | UDP | 53 | 일반적인 이름 질의 |
| DNS TCP | TCP | 53 | Zone Transfer 등 |
| RNDC | TCP | 953 | named 원격 제어 |
| DNS-over-TLS | TCP/UDP | 853 | 자료에서 참고 항목으로 소개 |

## 4. CentOS DNS 서버 선수 설정

### SELinux

강의 실습 환경에서는 SELinux를 **Permissive** 상태로 구성한다.

확인:

`sestatus`

일시적으로 Permissive 설정:

`setenforce 0`

설정 파일 변경:

`sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config`

실습 자료에서는 DNS 구축 편의를 위해 Permissive를 사용하지만, 실제 운영 환경에서는 **SELinux를 비활성화하는 것보다 적절한 정책을 사용하여 Enforcing 상태를 유지하는 것이 바람직하다.**

### firewalld

상태 확인:

`systemctl status firewalld`

서비스 시작 및 부팅 시 자동 실행:

`systemctl enable --now firewalld`

DNS 서비스 등록:

`firewall-cmd --permanent --add-service=dns`

설정 적용:

`firewall-cmd --reload`

확인:

`firewall-cmd --list-all`

DNS 서비스는 기본적으로 **53/TCP와 53/UDP**를 사용한다.

### /etc/hosts 및 NetworkManager

실습 서버 간 기본적인 이름 해석을 위해 `/etc/hosts`에 각 서버의 Host Name과 IP 주소를 등록한다.

| 항목 | 예시 형식 |
|---|---|
| DNS Master | `<DNS_MASTER_IP> <DNS_MASTER_FQDN> <DNS_MASTER_NAME>` |
| DNS Server 1 | `<DNS_SERVER1_IP> <DNS_SERVER1_FQDN> <DNS_SERVER1_NAME>` |
| DNS Server 2 | `<DNS_SERVER2_IP> <DNS_SERVER2_FQDN> <DNS_SERVER2_NAME>` |

NetworkManager의 Connection Profile 이름을 일정하게 관리하려면 `nmcli`를 이용한다.

확인:

`nmcli connection`

Connection 이름 변경:

`nmcli connection modify <CONNECTION> connection.id eth0`

Connection 활성화:

`nmcli connection up eth0`

기본 Target 확인:

`systemctl get-default`

GUI Target 설정:

`systemctl set-default graphical.target`

### 공개키 인증

실습 환경의 여러 서버를 편리하게 관리하기 위해 SSH 공개키 인증을 구성한다.

| 명령어 | 용도 |
|---|---|
| `ssh-keygen` | SSH 키 쌍 생성 |
| `ssh-copy-id` | 공개키를 원격 서버에 등록 |
| `sshpass` | 실습 환경에서 자동화된 암호 입력에 사용 |

예시는 특정 계정명이나 암호 대신 **UserName**과 같은 일반화된 계정을 사용한다.

## 5. BIND named.conf 설정

### options 설정

`/etc/named.conf`의 `options` 블록은 named의 전역 동작을 정의한다.

| 지시자 | 설명 |
|---|---|
| `listen-on` | IPv4에서 DNS 요청을 수신할 주소 지정 |
| `listen-on-v6` | IPv6에서 DNS 요청을 수신할 주소 지정 |
| `directory` | Zone 파일의 기본 디렉터리 |
| `dump-file` | Cache Dump 파일 위치 |
| `statistics-file` | 통계 정보 파일 위치 |
| `memstatistics-file` | 메모리 통계 정보 |
| `secroots-file` | Security Root 정보 |
| `recursing-file` | Recursive Query 관련 정보 |
| `allow-query` | DNS 질의를 허용할 클라이언트 범위 |
| `recursion` | Recursive DNS 기능 사용 여부 |
| `dnssec-validation` | DNSSEC 검증 관련 설정 |
| `managed-keys-directory` | 관리되는 키 저장 위치 |
| `pid-file` | named PID 파일 |

권한 있는 **Authoritative DNS 서버**를 구성할 때에는 불필요한 Recursive 기능을 활성화하지 않도록 주의한다.

### zone 설정

Zone은 특정 DNS Namespace에 대한 관리 단위이다.

기본 형식:

`zone "<ZONE_NAME>" IN { type <master|slave>; file "<ZONE_FILE>"; };`

주요 설정:

| 설정 | 설명 |
|---|---|
| `type master` | 원본 Zone 데이터를 관리 |
| `type slave` | Master에서 Zone 데이터를 전달받음 |
| `file` | Zone 파일 위치 지정 |
| `masters` | Slave가 데이터를 가져올 Master DNS 지정 |
| `also-notify` | Zone 변경 시 지정 DNS 서버에 변경 사실 통보 |
| `allow-transfer` | Zone Transfer를 허용할 DNS 서버 지정 |
| `allow-update` | Dynamic Update를 허용할 클라이언트 지정 |

### 주석 및 include

BIND 설정 파일에서 사용하는 주요 주석 형식:

| 형식 | 의미 |
|---|---|
| `;` | BIND 설정에서 일반적으로 사용되는 문장 종료 및 주석 처리 |
| `//` | C++ 스타일 주석 |
| `/* ... */` | 여러 줄 주석 |

`include`는 다른 설정 파일을 읽어 들인다.

대표적인 설정:

`include "/etc/named.rfc1912.zones";`

### 설정 문법 검사

**named-checkconf**는 BIND 주 설정 파일의 문법을 검사한다.

기본 형식:

`named-checkconf <설정파일>`

예:

`named-checkconf /etc/named.conf`

전체 Include 설정을 병합하여 확인:

`named-checkconf -p`

정상이라면 일반적으로 별도의 오류 메시지가 출력되지 않는다.

## 6. Zone 파일과 Resource Record

### Zone 파일 기본 형식

Zone 파일의 기본 구조는 다음과 같다.

`[Domain] [TTL] IN [RecordType] [Data]`

| 필드 | 설명 |
|---|---|
| Domain | Record가 적용되는 도메인 |
| TTL | Cache에 보관할 수 있는 시간 |
| Class | 일반적으로 `IN(Internet)` |
| Record Type | A, NS, MX, CNAME, PTR 등 |
| Data | Record에 해당하는 실제 값 |

Zone의 최상위 영역은 일반적으로 `@`로 표현할 수 있다.

### SOA Record

**SOA(Start of Authority)**는 Zone의 권한 및 동기화 정보를 정의한다.

형식:

`@ IN SOA <Primary DNS> <Responsible Email> ( <Serial> <Refresh> <Retry> <Expire> <Minimum> )`

| 필드 | 의미 |
|---|---|
| Serial | Zone 데이터 버전 번호 |
| Refresh | Slave가 Master 변경 여부를 확인하는 주기 |
| Retry | 동기화 실패 시 재시도 주기 |
| Expire | Master와 통신하지 못할 때 Slave가 기존 정보를 유지할 수 있는 기간 |
| Minimum | 자료에서는 TTL 성격의 값으로 설명 |

특히 **Serial 번호는 Master/Slave Zone 동기화에서 핵심**이다.

Zone 파일을 수정했다면 반드시 기존 Serial보다 **높은 값**으로 변경해야 한다.

### 주요 Resource Record

| Record | 용도 | 주요 Zone |
|---|---|---|
| **A** | Host Name → IPv4 주소 | Forward |
| **AAAA** | Host Name → IPv6 주소 | Forward |
| **NS** | 해당 Zone의 Name Server 지정 | Forward/Reverse |
| **MX** | Mail Server 지정 | Forward |
| **CNAME** | 별칭을 Canonical Name으로 연결 | Forward |
| **PTR** | IP 주소 → Host Name | Reverse |
| **SOA** | Zone 권한 및 동기화 정보 | Zone |
| **TXT** | 문자열 기반 정보 저장 | Forward |
| **DNSKEY** | DNSSEC 공개 키 | DNSSEC |
| **AXFR** | 전체 Zone Transfer | Master → Slave |
| **IXFR** | 변경된 Zone 정보만 Transfer | Master → Slave |

### 정방향 Zone 파일

정방향 Zone은 **도메인에서 IP 주소를 찾는 정보**를 저장한다.

자주 사용하는 구성:

| 요구사항 | Record |
|---|---|
| 웹 서버 등록 | `www` → A |
| FTP 서버 등록 | `ftp` → A |
| 메일 서버 등록 | Zone → MX |
| 서비스 별칭 | `cafe` → CNAME |
| IPv6 주소 등록 | AAAA |
| 텍스트 정보 | TXT |

### 역방향 Zone 파일

Reverse Zone은 **IP 주소에서 도메인 이름을 찾기 위한 Zone**이다.

IPv4 Reverse Zone은 `in-addr.arpa` 형식을 사용한다.

Reverse Zone에서는 주로 **PTR Record**를 사용한다.

실습 자료에서는 일반적인 Web Host를 추가할 때 Reverse Zone 등록은 필수가 아니며, DNS/메일/SSH 등 운영 정책에 따라 Reverse DNS를 구성하는 경우를 강조한다.

## 7. DNS 서버 구축 절차

### Root 및 TLD DNS 구성

강의 실습에서는 가상의 계층형 DNS 구조를 구성한다.

| 서버 역할 | 관리 대상 |
|---|---|
| Root DNS | Root 및 상위 영역 |
| TLD DNS | `<TLD>` Zone |
| Authoritative DNS | `<DOMAIN>` Zone |

일반적인 구축 순서:

1. BIND 패키지 설치
2. `named.conf` 설정
3. `named.rfc1912.zones`에 Zone 등록
4. Hint 파일 확인
5. Forward Zone 작성
6. Reverse Zone 작성
7. `named-checkconf` 실행
8. `named-checkzone` 실행
9. `named` 서비스 시작
10. 방화벽 DNS 서비스 허용
11. 클라이언트의 Resolver 설정
12. `nslookup`, `dig`, `host`로 검증

### Authoritative DNS 구성

Authoritative DNS는 관리하는 Zone의 공식 정보를 직접 제공한다.

기본 구성 요소:

| 구성요소 | 파일 |
|---|---|
| 주 설정 | `/etc/named.conf` |
| Zone 선언 | `/etc/named.rfc1912.zones` |
| Forward Zone | `/var/named/<DOMAIN>.zone` |
| Reverse Zone | `/var/named/<REVERSE_ZONE>.rev` |
| Root Hint | `/var/named/named.ca` |

서비스 관리:

`systemctl enable --now named`

상태 확인:

`systemctl status named`

프로세스 확인:

`pgrep -a named`

### DNS 클라이언트 설정

NetworkManager를 이용한 DNS 설정:

`nmcli connection modify eth0 ipv4.dns <DNS_SERVER_IP> +ipv4.dns <BACKUP_DNS_IP> ipv4.dns-search <DOMAIN>`

설정 적용:

`nmcli connection up eth0`

결과 확인:

`cat /etc/resolv.conf`

`/etc/resolv.conf`에는 일반적으로 다음 정보가 반영된다.

| 항목 | 의미 |
|---|---|
| `search` | 짧은 Host Name 질의에 사용할 검색 도메인 |
| `nameserver` | 사용할 DNS 서버 주소 |

## 8. DNS 질의 및 검증 명령어

### nslookup

`nslookup`은 DNS 질의를 빠르게 확인하기 위한 대표적인 도구이다.

형식:

`nslookup [-option] [name | -] [server]`

| 옵션 | 설명 |
|---|---|
| `-type=A` | A Record 조회 |
| `-type=NS` | NS Record 조회 |
| `-type=MX` | MX Record 조회 |
| `-type=PTR` | Reverse Lookup |
| `server` | 특정 DNS 서버 지정 |

`-q=` 형식도 자료에 소개되어 있으나 최근 사용 형식으로는 **`-type=` 사용을 중심으로 정리한다.**

대화형 모드에서는 `nslookup` 실행 후 질의 대상을 입력한다.

### dig

`dig`는 DNS 질의를 상세하게 확인할 수 있는 강력한 도구이다.

형식:

`dig [옵션] 도메인 [레코드]`

| 옵션/Record | 설명 |
|---|---|
| `@<DNS_SERVER>` | 특정 DNS 서버 지정 |
| `+short` | 결과를 간단하게 출력 |
| `-x` | Reverse Lookup |
| `A` | IPv4 주소 조회 |
| `AAAA` | IPv6 주소 조회 |
| `MX` | 메일 서버 조회 |
| `NS` | Name Server 조회 |
| `TXT` | TXT Record 조회 |
| `ANY` | 여러 Record를 요청하는 방식 |

Zone Transfer 점검:

`dig @<MASTER_DNS> <DOMAIN> AXFR`

IXFR 점검:

`dig @<MASTER_DNS> <DOMAIN> IXFR=<SERIAL>`

### host

`host`는 간단한 DNS 조회에 유용하다.

| 명령어 | 설명 |
|---|---|
| `host <DOMAIN>` | 일반 DNS 조회 |
| `host -v <DOMAIN>` | 상세 조회 |
| `host <IP>` | Reverse Lookup |
| `host -l <DOMAIN> <DNS_SERVER>` | AXFR 기반 Zone 전체 조회 |

`host -l`은 **Zone Transfer 정책에 따라 실패할 수 있다.**

### 서비스 및 네트워크 확인

| 명령어 | 용도 |
|---|---|
| `systemctl status named` | named 서비스 상태 |
| `pgrep -a named` | named 프로세스 확인 |
| `ss -lntup \| grep :53` | 53번 포트 Listen 여부 확인 |
| `firewall-cmd --list-all` | 방화벽 상태 |
| `journalctl -u named -f` | named 실시간 로그 |
| `tail -f /var/log/messages` | 시스템 로그 확인 |
| `named-checkconf` | 설정 파일 문법 검사 |
| `named-checkzone` | Zone 파일 검사 |

## 9. DNS 고급 구성

### DNS Load Balancing

하나의 Host Name에 여러 개의 A Record를 등록하면 DNS 응답에서 여러 IP를 반환할 수 있다.

예를 들어:

| Host | IP |
|---|---|
| `www.<DOMAIN>` | `<WEB_IP_1>` |
| `www.<DOMAIN>` | `<WEB_IP_2>` |
| `www.<DOMAIN>` | `<WEB_IP_3>` |

강의 자료에서는 반복적인 `nslookup` 요청을 통해 여러 IP가 응답되는 형태를 확인한다.

주의할 점은 **DNS 기반 부하 분산은 L4/L7 Load Balancer와 동일한 기능이 아니며**, DNS 응답에 여러 주소를 제공하는 방식이다.

### Domain Delegation

상위 Zone에서는 하위 Zone의 Name Server를 NS Record로 지정한다.

일반적인 구조:

`<TLD>` → `<SUBDOMAIN>` → `<HOST>`

위임을 위해 필요한 핵심 정보:

| 정보 | 역할 |
|---|---|
| NS Record | 하위 Zone을 관리하는 DNS 서버 지정 |
| A/AAAA Glue | Name Server의 실제 IP 정보 제공 |
| 하위 Zone | 위임받은 DNS 서버에서 직접 관리 |

### Master/Slave DNS

Master는 원본 Zone 데이터를 관리하고 Slave는 Master의 Zone 데이터를 전달받는다.

| 역할 | 핵심 설정 |
|---|---|
| Master | `type master` |
| Slave | `type slave` |
| Slave의 Master 지정 | `masters { <MASTER_IP>; };` |
| Slave Zone 저장 | `/var/named/slaves/` |
| 변경 통보 | `also-notify` |
| Zone Transfer 제한 | `allow-transfer` |

Slave 설정 예:

`zone "<DOMAIN>" IN { type slave; masters { <MASTER_IP>; }; file "slaves/<DOMAIN>.zone"; };`

### Zone Transfer

Zone Transfer는 Master와 Slave 사이에서 Zone 정보를 복제하는 기능이다.

| 방식 | 의미 |
|---|---|
| **AXFR** | Zone 전체를 전송 |
| **IXFR** | 변경된 부분을 중심으로 증분 전송 |

Master에서 Slave의 IP만 허용하는 형태:

`allow-transfer { <SLAVE_IP>; };`

Zone 변경 사실을 알리는 설정:

`also-notify { <SLAVE_IP>; };`

Slave가 여러 대인 경우 여러 주소를 등록할 수 있다.

## 10. DNS 동적 업데이트 및 rndc

### SOA Serial과 Slave 동기화

Master의 Zone 파일을 변경한 뒤 Slave에 변경 내용이 반영되려면 **SOA Serial Number를 증가시켜야 한다.**

| 항목 | 설명 |
|---|---|
| Serial | Zone 데이터 변경 버전 |
| Refresh | Slave가 Master 변경 여부를 확인하는 간격 |
| Retry | 확인 실패 시 재시도 간격 |
| Expire | Master와 장기간 통신하지 못할 때 기존 데이터 유지 한계 |
| Minimum | Zone 파일 SOA의 최소 TTL 관련 값 |

Serial을 변경하지 않고 파일만 수정한 경우 Slave가 변경 사항을 가져오지 않을 수 있다.

### rndc

**RNDC(Remote Name Daemon Control)**는 `named` 데몬을 제어하는 도구이다.

| 명령어 | 설명 |
|---|---|
| `rndc reload` | 설정 또는 Zone 정보를 다시 로드 |
| `rndc status` | DNS 서버 상태 확인 |
| `rndc stat` | 통계 정보 생성 |
| `rndc flush` | DNS 캐시 삭제 |
| `rndc dumpdb` | DNS 데이터 Dump |
| `rndc start` | named 시작 |
| `rndc stop` | named 종료 |

`rndc-confgen`은 RNDC 인증 키와 설정에 필요한 내용을 생성한다.

RNDC 기본 제어 포트는 **953/TCP**이다.

원격 RNDC를 사용할 경우 다음을 함께 구성해야 한다.

| 구성 | 설명 |
|---|---|
| `key` | RNDC 인증 키 |
| `controls` | RNDC 제어 주소와 포트 |
| `allow` | 접속 가능한 Remote Client |
| `keys` | 허용할 인증 키 |
| firewalld | 953/TCP 접근 허용 |

### Dynamic DNS와 nsupdate

**DDNS**는 DNS Zone 데이터를 동적으로 변경하는 기능이다.

`allow-update`는 Dynamic Update가 가능한 클라이언트 범위를 지정한다.

예:

`allow-update { <CLIENT_NETWORK>; 127.0.0.1; };`

`nsupdate`를 통한 동적 갱신 과정:

1. DNS 서버 지정
2. 대상 Zone 지정
3. Record 추가/삭제
4. 업데이트 전송
5. DNS 질의로 결과 확인

### 자주 사용하는 사용 예제

`nsupdate` 실행 후 다음과 같은 작업 흐름을 사용한다.

`server <DNS_SERVER_IP>`

`zone <DOMAIN>`

`update add <HOST>.<DOMAIN>. <TTL> A <IP>`

`update delete <HOST>.<DOMAIN>. A`

`update add <ALIAS>.<DOMAIN>. <TTL> CNAME <TARGET>.<DOMAIN>.`

`quit`

동적 업데이트가 발생하면 BIND에서는 **`.jnl` Journal 파일**을 생성하여 변경 정보를 기록할 수 있다.

## 11. Forwarding DNS Server

### Forwarders

Forwarding DNS는 자체적으로 처리할 수 없는 질의를 지정된 상위 DNS 서버로 전달한다.

`named.conf`의 `options`에서 설정한다.

`forwarders { <UPSTREAM_DNS>; };`

### Forward Only

`forward only;`를 설정하면 질의를 반드시 Forwarder로 전달한다.

자료에서는 `forward only` 사용 시 Root Hint Zone 설정을 비활성화하는 구성도 함께 소개한다.

핵심 설정:

| 설정 | 설명 |
|---|---|
| `forwarders` | 질의를 전달할 상위 DNS 서버 |
| `forward only` | 지정된 Forwarder만 사용 |
| Root Hint | 직접 Root부터 질의를 시작할 때 사용 |

### DNS Client 구성

Forwarding DNS를 사용할 클라이언트는 `/etc/resolv.conf`에서 Forwarding DNS 서버를 바라보도록 설정한다.

예:

`nmcli connection modify eth0 ipv4.dns <FORWARDING_DNS_IP> +ipv4.dns <BACKUP_DNS_IP> ipv4.dns-search <DOMAIN>`

그 후:

`nmcli connection up eth0`

확인:

`nslookup`

Forwarding DNS가 내부 Zone은 자체적으로 응답하고 외부 Domain은 Forwarder를 통해 해결하는지 확인한다.

### 자주 사용하는 사용 예제

내부 Zone 조회:

`nslookup <INTERNAL_HOST>`

외부 Zone 조회:

`nslookup <PUBLIC_HOST>`

DNS 서버를 직접 지정하여 비교:

`dig @<FORWARDING_DNS_IP> <DOMAIN>`

## 12. DNS 보안 및 장애 처리

### allow-query

`allow-query`는 DNS 질의를 요청할 수 있는 클라이언트 범위를 제한한다.

권장 방향:

`allow-query { <INTERNAL_NETWORK>; };`

특별한 목적이 없다면 모든 외부 클라이언트가 무제한으로 질의할 수 있도록 구성하지 않는다.

### allow-transfer

`allow-transfer`는 **Zone Transfer가 허용되는 대상**을 제한한다.

| 설정 | 의미 |
|---|---|
| `allow-transfer { <SLAVE_IP>; };` | 지정된 Slave만 Zone Transfer 가능 |
| `allow-transfer { none; };` | Zone Transfer 차단 |
| 네트워크 대역 지정 | 특정 네트워크에서만 Transfer 허용 |

Master → Slave 영역 전송을 보안적으로 제한하려면 Slave의 IP를 명시하는 것이 중요하다.

### allow-update

`allow-update`는 Dynamic DNS 업데이트를 허용할 클라이언트 범위를 제한한다.

예:

`allow-update { <DDNS_CLIENT_NETWORK>; };`

공개된 환경에서 광범위하게 허용하면 임의의 클라이언트가 Zone 데이터를 변경할 수 있으므로 주의해야 한다.

### chroot 및 패키지 보안

강의 자료에서는 `bind-chroot`를 이용한 격리 환경을 소개한다.

패키지 설치:

`dnf install bind bind-utils bind-chroot`

자료에서는 과거 chroot 방식과 최근 VM/Container 기반 격리를 비교해서 설명한다.

실제 운영에서는 **BIND 패키지를 최신 보안 패치 상태로 유지하고 SELinux 및 접근 제어 정책을 함께 적용**하는 것이 중요하다.

BIND 패키지 업데이트:

`dnf -y update bind bind-utils`

### DNSSEC과 TSIG

| 기술 | 목적 | 방식 |
|---|---|---|
| **DNSSEC** | DNS 데이터 위조 및 변조 방지 | 공개키 기반 서명 |
| **TSIG** | DNS 서버 간 또는 동적 업데이트 인증 | 공유 비밀키 기반 인증 |

DNSSEC은 공개키 서명 구조를 이용하며, DNS 데이터의 **무결성 및 진위 검증**에 사용한다.

TSIG는 Master/Slave 간 Zone Transfer나 Dynamic Update 등의 인증에 활용할 수 있다.

### 장애 처리 순서

DNS 문제가 발생하면 다음 순서로 확인한다.

| 순서 | 확인 내용 | 명령어 |
|---:|---|---|
| 1 | DNS 패키지 설치 여부 | `rpm -qa \| grep bind` |
| 2 | named 서비스 상태 | `systemctl status named` |
| 3 | 설정 문법 | `named-checkconf /etc/named.conf` |
| 4 | Zone 문법 | `named-checkzone <DOMAIN> <ZONE_FILE>` |
| 5 | 53번 포트 Listen | `ss -lntup \| grep :53` |
| 6 | 방화벽 | `firewall-cmd --list-all` |
| 7 | Resolver 설정 | `cat /etc/resolv.conf` |
| 8 | 로컬 DNS 질의 | `nslookup <DOMAIN>` |
| 9 | 특정 서버 질의 | `dig @<DNS_IP> <DOMAIN>` |
| 10 | 로그 확인 | `journalctl -u named -f` |
| 11 | Master/Slave Serial 확인 | `dig <DOMAIN> SOA` |
| 12 | Zone Transfer 확인 | `dig @<MASTER_IP> <DOMAIN> AXFR` |

### 자주 발생하는 문제

| 증상 | 주요 원인 |
|---|---|
| `named` 시작 실패 | `named.conf` 문법 오류 또는 Zone 오류 |
| DNS 응답 없음 | 53번 포트, firewalld, listen-on 문제 |
| 내부 Domain만 해석 안 됨 | Zone 선언 또는 Zone 파일 오류 |
| Reverse Lookup 실패 | Reverse Zone 또는 PTR 오류 |
| Slave에 변경 미반영 | SOA Serial 증가 누락 |
| AXFR 실패 | `allow-transfer` 설정 문제 |
| DDNS 실패 | `allow-update`, 인증 또는 파일 권한 문제 |
| 외부 Domain 해석 실패 | `forwarders`, `recursion`, 네트워크 문제 |
| RNDC 접속 실패 | key, `controls`, 953/TCP 방화벽 설정 문제 |
| IPv6 관련 오류 증가 | IPv6 수신 설정 또는 네트워크 환경 확인 필요 |

## 13. 핵심 명령어 및 설정 요약

### 패키지 및 서비스 명령어

| 명령어 | 용도 |
|---|---|
| `dnf -y install bind bind-utils` | BIND 서버 및 DNS 유틸리티 설치 |
| `dnf -y install bind-chroot` | chroot 패키지 설치 |
| `systemctl enable --now named` | named 활성화 및 즉시 시작 |
| `systemctl status named` | 서비스 상태 확인 |
| `systemctl restart named` | named 재시작 |
| `systemctl disable --now named` | 서비스 비활성화 및 중지 |

### 설정 검사 명령어

| 명령어 | 용도 |
|---|---|
| `named-checkconf /etc/named.conf` | 주 설정 문법 검사 |
| `named-checkconf -p` | Include 파일을 포함한 전체 설정 확인 |
| `named-checkzone <DOMAIN> <ZONE_FILE>` | Forward Zone 검사 |
| `named-checkzone <REVERSE_ZONE> <ZONE_FILE>` | Reverse Zone 검사 |

### 방화벽 명령어

| 명령어 | 용도 |
|---|---|
| `firewall-cmd --info-service=dns` | DNS 방화벽 서비스 정보 |
| `firewall-cmd --permanent --add-service=dns` | DNS 서비스 허용 |
| `firewall-cmd --reload` | 방화벽 설정 적용 |
| `firewall-cmd --list-all` | 현재 방화벽 정책 확인 |
| `firewall-cmd --permanent --add-port=953/tcp` | RNDC 원격 제어 포트 허용 |

### DNS 조회 명령어

| 명령어 | 용도 |
|---|---|
| `nslookup <DOMAIN>` | 기본 DNS 조회 |
| `nslookup <IP>` | Reverse Lookup |
| `nslookup -type=A <DOMAIN>` | A Record 조회 |
| `nslookup -type=NS <DOMAIN>` | NS Record 조회 |
| `nslookup -type=MX <DOMAIN>` | MX Record 조회 |
| `dig <DOMAIN>` | 상세 DNS 조회 |
| `dig +short <DOMAIN>` | 간단한 결과 출력 |
| `dig @<DNS_IP> <DOMAIN>` | 특정 DNS 서버 조회 |
| `dig -x <IP>` | Reverse Lookup |
| `dig <DOMAIN> MX` | MX 조회 |
| `dig <DOMAIN> NS` | NS 조회 |
| `dig <DOMAIN> TXT` | TXT 조회 |
| `host <DOMAIN>` | 간단한 DNS 조회 |
| `host -v <DOMAIN>` | 상세 조회 |
| `host -l <DOMAIN> <DNS_IP>` | AXFR 기반 Zone 목록 조회 |

### Master/Slave 및 Zone Transfer 명령어

| 명령어 | 용도 |
|---|---|
| `dig @<MASTER_IP> <DOMAIN> AXFR` | 전체 Zone Transfer 요청 |
| `dig @<MASTER_IP> <DOMAIN> IXFR=<SERIAL>` | 증분 Zone Transfer 요청 |
| `watch ls -l /var/named/slaves/` | Slave Zone 파일 변경 감시 |
| `dig <DOMAIN> SOA` | SOA Serial 확인 |

### rndc 명령어

| 명령어 | 설명 |
|---|---|
| `rndc reload` | named 설정 및 Zone Reload |
| `rndc status` | named 상태 확인 |
| `rndc stat` | 통계 생성 |
| `rndc dumpdb` | DNS 데이터 Dump |
| `rndc flush` | DNS Cache 삭제 |
| `rndc start` | named 시작 |
| `rndc stop` | named 종료 |
| `rndc-confgen` | RNDC 인증 설정 생성 |

### 자주 사용하는 사용 예제

DNS 설정을 수정한 뒤 가장 일반적인 검증 순서는 다음과 같다.

`named-checkconf /etc/named.conf`

`named-checkzone <DOMAIN> /var/named/<DOMAIN>.zone`

`systemctl restart named`

`systemctl status named`

`firewall-cmd --list-all`

`ss -lntup | grep :53`

`dig @<DNS_SERVER_IP> <HOST>.<DOMAIN>`

Reverse DNS 확인:

`dig -x <IP_ADDRESS>`

Master/Slave 동기화 확인:

`dig <DOMAIN> SOA`

Zone Transfer 확인:

`dig @<MASTER_IP> <DOMAIN> AXFR`

로그 실시간 확인:

`journalctl -u named -f`

### 최종 암기 포인트

| 핵심 | 기억할 내용 |
|---|---|
| DNS 기본 역할 | **Domain ↔ IP 매핑** |
| DNS 서비스 | **named** |
| DNS 포트 | **53/TCP, 53/UDP** |
| RNDC 포트 | **953/TCP** |
| 주 설정 | `/etc/named.conf` |
| Zone 설정 | `/etc/named.rfc1912.zones` |
| Zone 파일 | `/var/named/` |
| Forward | **Domain → IP** |
| Reverse | **IP → Domain** |
| Forward 대표 Record | **A, AAAA, NS, MX, CNAME, TXT** |
| Reverse 대표 Record | **PTR** |
| Zone 권한 정보 | **SOA** |
| Master/Slave | **type master / type slave** |
| Slave Master 지정 | **masters** |
| Zone 변경 통보 | **also-notify** |
| Zone Transfer 제한 | **allow-transfer** |
| Query 제한 | **allow-query** |
| Dynamic Update 제한 | **allow-update** |
| 전체 Zone Transfer | **AXFR** |
| 증분 Zone Transfer | **IXFR** |
| DNS 테스트 | **nslookup / dig / host** |
| 설정 검사 | **named-checkconf / named-checkzone** |
| 동적 변경 | **nsupdate** |
| DNS 데몬 제어 | **rndc** |
| Slave 동기화 핵심 | **SOA Serial 증가** |
| 보안 핵심 | **allow-query / allow-transfer / allow-update / DNSSEC / TSIG / 패치** |