## Kerberos
- 패스워드를 네트워크에 노출하지 않고, 신뢰된 제3자(KDC)가 발급한 티켓으로 인증하는 프로토콜
- Active Directory는 Kerberos를 기본 인증 프로토콜로 사용
- 윈도우에서 최초 로그인을 통해서 티켓을 발급받으면 이후에 인증절차 없이 서비스를 사용 가능하게 됨.
```
[계정] --(ID/PW 인증)--> [KDC 내 AS] --> [TGT] 발급

[계정] --(TGT 제시)--> [KDC 내 TGS] --> [Service Ticket] 발급

[계정] --(Service Ticket 제시)--> [실제 서비스] --> 접근 허용
```

### SPN(Service Principal Name)
- 해당 서비스를 특정한 계정이 실행하고 있다고 AD에 등록해두는 이름표
- 관리자가 서비스를 실행할 때 최소 권한 원칙을 적용하기 위해 사용.

### Kerberoasting
- 도메인 일반 계정으로 KDC에 인증한 후, SPN이 등록된 서비스 계정의 티켓(ST)을 발급받아 오프라인에서 패스워드를 크래킹하는 공격.
- 도메인 계정으로 인증되면, AD에 등록된 모든 SPN의 티켓(ST)을 요청할 수 있다
- 티켓이 NTLM 해시로 암호화 되어서 출력이 된다.
- 사전인증 비활성화 + SPN 등록 계정의 경우 비밀번호 없이도 요청이 가능하다.

---

## LDAP(Lightweight Directory Access Protocol)
- 조직 내 모든 계정·자원 정보를 중앙에서 관리하고, 빠르게 검색·인증할 수 있게 해주는 표준 프로토콜
- 사용자, 컴퓨터, 그룹, 권한 등의 정보를 계층 구조로 저장하고 빠르게 검색할 수 있게 해준다.
- Simple : LDAP 서버에 직접 아이디/비밀번호 전송하여 인증
- Simple + TLS 사용시 암호화로 패킷 도청 어려움.
- SASL : Kerberos 서버에서 티켓 발급받아 그 티켓으로 인증
- SASL은 비밀번호 자체를 전송하지 않으므로 TLS 없이도 Simple+TLS보다 근본적으로 안전
```
dc=company,dc=com          ← 루트 (도메인)
├── ou=Users               ← 조직 단위 (부서)
│   ├── cn=Alice           ← 개별 항목 (사용자)
│   └── cn=Bob
├── ou=Groups
│   └── cn=Developers
└── ou=Computers
    └── cn=Server01
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| **인증 (Authentication)** | 사용자 ID/PW 검증 |
| **검색 (Search)** | 조건에 맞는 사용자·그룹 조회 |
| **추가/수정/삭제** | 디렉터리 데이터 관리 |
| **비교 (Compare)** | 특정 속성 값 일치 여부 확인 |

### 인증 비교
- 도메인 환경 내부에서 인증을 진행하면 Kerberos 티켓이 발급.
- 도메인 환경 내부라도 IP로 직접 접근 혹은 Kerberos 실패시 NTLM 인증 진행.
- 도메인이 아닌 로컬로 인증을 진행할 경우 NTLM 인증으로 진행.
- 도메인 환경이 아니라면 LDAP 인증으로 진행되며, 티켓 발급은 되지 않음.

---
## Backup Operators Group
-  백업 및 복원 작업을 수행하기 위한 특수 권한.
-  사실상 관리자에 준하는 권한을 가져 `NTDS.dit`에 접근을 하여 도메인 계정의 패스워드 해시 추출 가능.
-  `SAM`, `SYSTEM`, `SECURITY` 레지스트리 하이브 백업 가능.

### 핵심 파일 비교표
- `NTDS.dit` → `C:\Windows\NTDS\NTDS.dit`
- `SAM`      → `C:\Windows\System32\config\SAM`
- `SYSTEM`   → `C:\Windows\System32\config\SYSTEM`
- `SECURITY` → `C:\Windows\System32\config\SECURITY`
- `LSA (Local Security Authority)`: Windows가 자동화를 위해 의도적으로 저장하는 자격증명 모음. 사용자는 존재를 잘 모르나, 서비스 자동 실행·오프라인 로그인 등에 활용된다.

| 항목 | NTDS.dit | SAM | SYSTEM | SECURITY |
|------|----------|-----|--------|----------|
| **존재 위치** | 도메인 컨트롤러(DC)만 | 모든 Windows 시스템 | 모든 Windows 시스템 | 모든 Windows 시스템 |
| **파일 형식** | ESE DB (데이터베이스) | 레지스트리 하이브 | 레지스트리 하이브 | 레지스트리 하이브 |
| **핵심 역할** | 도메인 전체 계정 DB | 로컬 계정 해시 저장 | 복호화 키(Boot Key) 보관 | LSA(Local Security Authority) Secrets 저장 |
| **저장 정보** | 도메인 계정 NT Hash, krbtgt 해시, 그룹/OU 구조 | 로컬 계정 NT Hash | Boot Key (SysKey) | 캐시 도메인 해시(DCC2), 서비스 계정 PW |
| **단독 사용 가능 여부** | ❌ SYSTEM 필요 | ❌ SYSTEM 필요 | ❌ SAM/NTDS.dit 필요 | △ 일부 정보만 |
| **필요한 조합** | NTDS.dit + SYSTEM | SAM + SYSTEM | SAM or NTDS.dit와 조합 | SYSTEM과 조합 |
| **공격 임팩트** | 💀 도메인 전체 장악 가능 | 로컬 계정 탈취 | 해시 복호화 가능 | 서비스 계정·캐시 탈취 |
| **실행 중 잠금** | ✅ (VSS로 우회 가능) | ✅ (SeBackupPrivilege로 우회) | ✅ (SeBackupPrivilege로 우회) | ✅ (SeBackupPrivilege로 우회) |

---
## AD CS (Active Directory Certificate Services)

### CA(Certificate Authority)
- `AD CS`의 핵심 구성 요소
- 디지털 인증서를 발급하고 관리하는 신뢰할 수 있는 기관.
- `CA`자체는 인터넷에서 보편적으로 사용되는 기술이나, AD와 결합해서 사용하기도 하는 것.

#### 실제 활용 예시
- 사내 HTTPS (SSL/TLS) 인증서 발급
- 스마트카드 로그인 인증
- VPN 클라이언트 인증
- 이메일 서명/암호화 (S/MIME)
- Wi-Fi 802.1X 인증

### 인증서 템플릿
- 인증서를 발급할 때 어떤 규칙으로 만들어라고 정의한 설정 양식.
- 아래 3가지 조건이 동시에 충족되면 취약한 템플릿이 됨.
  1. 낮은 권한 사용자도 인증서 요청 가능 (Enrollment Rights = Domain Users)
  3. SAN(Subject Alternative Name)을 요청자가 직접 지정 가능(Administrator를 지정하여 요청)
  4. 발급된 인증서로 Kerberos 인증 가능 (Client Authentication)
