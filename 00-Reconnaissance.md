# Reconnaissance

## 🗺️ Nmap Scanning

### Host Discovery

네트워크 내 활성 호스트 식별. 포트 스캔 없이 빠르게 타겟 범위 파악.

```bash
# 파일 목록에서 호스트 스캔
sudo nmap -sn -oA tnet -iL ip.list

# ICMP Echo 기반 호스트 발견 (라우터 너머 타겟용)
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

**주요 옵션**
- `-sn`: 포트 스캔 생략, 호스트 발견만 수행
- `-PE`: ICMP Echo Request 사용
- `--disable-arp-ping`: ARP 비활성화 (Layer 3 라우팅 환경)
- `-oA`: Normal/XML/Grepable 형식으로 저장
- `-iL`: 타겟 리스트 파일 입력

> **Note**: 라우터 너머 호스트는 ARP로 도달 불가. ICMP 또는 TCP 필수.

---

## 🔍 Banner Grabbing

서비스 식별 및 버전 정보 수집. 취약점 매칭의 기초 단계.

```bash
# TCP 배너 수집
nc -nv <ip> <port>

# HTTP 헤더 조회 (리다이렉션 추적)
curl -IL https://www.inlanefreight.com

# 웹 기술 스택 파악 (멀티 타겟)
whatweb --no-errors 10.10.10.0/24
```

**추가 정보원**
- **SSL Certificate**: CN, SAN 필드에서 서브도메인 발견
- **robots.txt**: `http://target.com/robots.txt` - 크롤러 제한 경로 = 잠재적 공격 벡터
- **JavaScript 소스**: API 엔드포인트, 하드코딩된 키/토큰 탐색

---

## 🔎 Footprinting

### SSL Certificate 기반 서브도메인 열거

Certificate Transparency 로그 활용. 공개 CA가 발급한 모든 인증서 검색 가능.

```bash
# crt.sh API로 서브도메인 추출 (dev 환경 필터링)
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | sort -u
```

> **Tip**: 와일드카드 인증서 `*.example.com` 발견 시 서브도메인 브루트포싱 수행

---

## 🔍 WHOIS

도메인 등록자 정보 조회. Social Engineering 및 ASN 추적에 활용.

```bash
whois <domain>
```

**수집 데이터**
- Registrar, Registrant 정보
- Name Server (권한 있는 DNS 서버)
- 등록/갱신/만료 날짜
- Admin/Tech Contact (GDPR로 인해 종종 비공개)

---

## 🌐 DNS Enumeration

### 기본 레코드 조회

DNS는 UDP 53번 포트 사용. 512바이트 초과 시 TCP 전환.

```bash
# A, AAAA 레코드 조회
dig inlanefreight.htb

# PTR 레코드 (역방향 DNS)
dig -x <ip>

# NS 레코드 (특정 DNS 서버 지정)
dig ns inlanefreight.htb @10.129.14.128

# DNS 서버 버전 (CHAOS 클래스)
dig CH TXT version.bind @10.129.120.85

# 모든 레코드 타입
dig any inlanefreight.htb @10.129.14.128

# 해당 웹서버가 아닌 DNS서버에 요청
dig any inlanefreight.htb @<DNS SERVER>
```

### Zone Transfer (AXFR)

DNS 서버 간 Zone 파일 복제 기능. 잘못된 ACL 설정 시 전체 도메인 구조 노출.

```bash
# Zone Transfer 시도
dig axfr <domain> @<dns server>

# 공개 테스트 서버
dig axfr @nsztm1.digi.ninja zonetransfer.me

# 실전 타겟
dig axfr inlanefreight.htb @10.129.14.128
```

> **Impact**: AXFR 성공 시 모든 A, CNAME, MX, TXT 레코드 획득

### DNS 브루트포싱

서브도메인 열거. SecLists 워드리스트 활용.

```bash
# dnsenum (멀티스레드 지원)
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r

# 특정 DNS 서버 타겟팅
dnsenum --dnsserver 10.129.167.221 --enum -p 0 -s 0 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt inlanefreight.htb
```

**핵심 용어**
- **Zone**: DNS 관리 단위 (도메인과 서브도메인 집합)
- **CNAME**: Canonical Name, 도메인 별칭 (CDN에서 자주 사용)

---

## 🔥 Firewall Evasion

### DNS 포트 우회

방화벽이 DNS 트래픽(53번 포트)을 허용한다고 가정하고 공격.

```bash
# UDP 53번 스캔
sudo nmap -sV 10.129.22.22 -Pn -p53 -sU

# TCP 53번 스캔 (Zone Transfer용)
sudo nmap -sV 10.129.22.22 -Pn -p53

# Source Port 53번 지정 (방화벽 규칙 우회)
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

# Netcat 연결 (동일 기법)
ncat -nv -p 53 10.129.2.28 50000
```

**우회 원리**
- 출발지 포트를 53번으로 설정하면 DNS 응답으로 오인
- Stateful 방화벽의 "DNS 요청 → 응답" 세션 허용 규칙 악용
- 관리자가 `--sport 53` 트래픽만 허용하도록 설정한 경우 효과적

**DNS 프로토콜 특징**
- **UDP 53**: 일반 DNS 쿼리 (512 바이트 이하)
- **TCP 53**: 큰 응답이나 Zone Transfer (512 바이트 초과)

---

## 🌍 Virtual Host Discovery

하나의 IP에서 여러 도메인 호스팅. `Host` 헤더 기반 라우팅.

### Gobuster

```bash
# 기본 VHOST 열거
gobuster vhost -u http://<target_IP> -w <wordlist> --append-domain

# 비표준 포트 지정
gobuster vhost -u http://94.237.120.112:44025 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain --domain inlanefreight.htb
```

### FFUF

```bash
# Host 헤더 퍼징 (응답 크기로 필터링)
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' -u http://83.136.253.132:32685

# 404 코드를 수집할 필요가 있음.
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' -u http://83.136.253.132:32685 -mc all
```

> **Important**: 비표준 포트 사용 시 VHOST URL에도 포트 명시 필수 (예: `http://dev.example.com:8443`)

---

## 🎯 Web Fingerprinting

웹 스택 식별. 버전 특정 취약점(CVE) 매칭에 필수.

### HTTP 헤더 분석

```bash
# 기본 헤더 확인
curl -I inlanefreight.com

# HTTPS 헤더
curl -I https://inlanefreight.com

# www 서브도메인 (CDN 설정 차이 확인)
curl -I https://www.inlanefreight.com
```

**핵심 헤더**
- `Server`: 웹 서버 종류/버전 (nginx, Apache, IIS)
- `X-Powered-By`: 백엔드 언어/프레임워크 (PHP, ASP.NET)
- `X-AspNet-Version`: .NET 버전 (IIS 환경)

### WAF 탐지 및 취약점 스캔

```bash
# WAF/IPS 식별
wafw00f inlanefreight.com

# Nikto (소프트웨어 버전 튜닝)
nikto -h inlanefreight.com -Tuning b
```

**Fingerprinting 기법**
1. **Banner Grabbing**: 서버 응답 헤더 분석
2. **HTTP Headers 분석**: 사용 기술 파악
3. **Specific Responses 프로빙**: 특정 요청에 대한 응답 패턴 분석
4. **Page Content 분석**: HTML, JavaScript 분석

---

## 🔗 Well-Known URIs

RFC 8615 표준 경로. 서비스 메타데이터 및 정책 정보 제공.

```bash
# 보안 연락처/버그 바운티 프로그램
https://example.com/.well-known/security.txt

# 패스워드 변경 엔드포인트
https://example.com/.well-known/change-password

# OAuth/OIDC 설정
https://example.com/.well-known/openid-configuration
```

> **Use Case**: `security.txt` 존재 시 책임 있는 공개(Responsible Disclosure) 가능

---

## 🕷️ Web Crawlers

사이트맵 자동 생성. `robots.txt`로 차단된 경로도 발견 가능.

### Burpsuite
- `Target-Site map`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/486c7a0f-7019-47de-8e56-6f653ea67bb0" />


### Scrapy

```bash
# Scrapy 설치
pip3 install scrapy

# ReconSpider (재귀 크롤링)
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:41954
```

> **Note**: JavaScript 렌더링 필요 시 Selenium/Puppeteer 사용

---

## 🕰️ Wayback Machine

과거 웹사이트 스냅샷 조회. 삭제된 페이지 및 설정 파일 복구.

**URL**: https://web.archive.org/

**활용 시나리오**
- `.git`, `.env` 등 민감 파일의 과거 버전 복구
- 삭제된 관리자 페이지 발견
- 도메인 소유권 변경 이력 추적
- 과거 코드나 설정 파일 분석

---

## 📊 Information Gathering - Web

### 디렉토리 슬래시 차이

```
/admin  → 리다이렉션 (301/302)
/admin/ → /admin/index 파일 직접 반환 (200)
```

> **Tip**: 슬래시 유무에 따라 서버 응답이 다를 수 있으며, 접근 제어 우회 가능성 존재

---

## 🔧 FinalRecon

> **⚠️ OSCP 시험에서 사용 불가**

통합 정보 수집 도구. 헤더, WHOIS, 서브도메인 등 자동 열거.

```bash
# 헤더 및 WHOIS 정보 수집
./finalrecon.py --headers --whois --url http://inlanefreight.com
```
