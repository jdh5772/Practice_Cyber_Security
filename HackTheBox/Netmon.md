# Netmon - HackTheBox
## Recon
```bash
sudo nmap -p 21,80,135,139,445,5985 -sC -sV -vv -oA netmon 10.10.10.152
```
<img width="1031" height="540" alt="image" src="https://github.com/user-attachments/assets/97bd90e2-3711-4cb7-ad68-d0f95676f85b" />

<br>
<br>

- FTP(21)
- HTTP(80)
- SMB(445)
- WINRM(5985)

---
## banner grabbing
- `PRTG-Network-Monitor 18.1.37.13946` 발견
```bash
whatweb http://10.10.10.152
```
<img width="1103" height="134" alt="image" src="https://github.com/user-attachments/assets/958ddfd2-fda5-49da-b037-ec5b48f38719" />

<br>
<br>

```bash
curl -IL http://10.10.10.152
```
<img width="1103" height="499" alt="image" src="https://github.com/user-attachments/assets/d1b739c4-1216-4e68-8157-05f6140f43f8" />

---
## FTP
- `anonymous`로 접속 가능하여 확인해보니 `C:\`에 위치한 것처럼 파악됨.
```bash
ftp 10.10.10.152
```
<img width="1103" height="409" alt="image" src="https://github.com/user-attachments/assets/e3aa8a37-d2ca-41af-827b-b6e6a0e9eca2" />

<br>
<br>

- `PRTG` configuration 파일의 위치를 구글에서 찾아보니 `C:\ProgramData\Paessler\PRTG Network Monitor`에 위치해 있다고 함.
- https://helpdesk.paessler.com/en/support/solutions/articles/76000073744-how-to-copy-data-files-and-custom-files-from-prtg-data-directory
<img width="973" height="73" alt="image" src="https://github.com/user-attachments/assets/ee9592dc-8f2d-44b5-9a92-91946ff134bd" />

<br>
<br>

- 해당 폴더로 이동하여 configuration파일 3개를 발견.
<img width="973" height="365" alt="image" src="https://github.com/user-attachments/assets/47afcc43-18d5-497b-98a1-f9f7c8597267" />

<br>
<br>

- 다운로드 받아 확인해보니 `PRTG Configuration.old.bak`파일에서 로그인 계정을 발견.
```bash
cat PRTG\ Configuration.old.bak|grep pass -A 3 -B 3
```
<img width="1102" height="183" alt="image" src="https://github.com/user-attachments/assets/662d1253-b07d-4a2f-b05f-586ff5aba888" />

---
## HTTP
- `prtgadmin:PrTg@dmin2018`로 로그인 시도하였으나 실패.
- 비밀번호에서 연도만 2019로 바꿔서 시도하여 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/2ed2b6e0-3077-4183-a11c-76817fb9389c" />

---
### RCE
- https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/
- `Setup > Account Settings > Notifications`
<img width="1100" height="260" alt="image" src="https://github.com/user-attachments/assets/37b56f90-e971-4b35-97e5-0310317227f1" />

<br>
<br>

- `Add new notification`
<img width="1100" height="260" alt="image" src="https://github.com/user-attachments/assets/61512a40-2d7d-44e1-8f93-591212e3f604" />

<br>
<br>

- `curl`로 내 서버에 접속이 가능한지 테스트
<img width="1100" height="225" alt="image" src="https://github.com/user-attachments/assets/c9b1c7a2-eff2-4291-875b-c815f09abdcf" />

<br>
<br>

- `Send test notification`
<img width="1100" height="219" alt="image" src="https://github.com/user-attachments/assets/c68fdf92-b8b4-4153-a5e0-e65b171f65dd" />

<br>
<br>

- `curl` 테스트 성공.
<img width="648" height="153" alt="image" src="https://github.com/user-attachments/assets/615b829a-8cb6-4354-86e9-197c23f82de7" />

<br>
<br>

- `nc.exe`를 서버에 다운로드 받아서 리버스 셸 연결 시도.
<img width="1100" height="196" alt="image" src="https://github.com/user-attachments/assets/7109e2df-3294-406f-8b05-ed719e5b928f" />

<br>
<br>

- 셸 획득.
<img width="1104" height="205" alt="image" src="https://github.com/user-attachments/assets/f7298fcb-0107-4d2b-879e-a050f1a08ff9" />

---
## FLAG
- `c:\users\public\desktop\user.txt`
<img width="1104" height="284" alt="image" src="https://github.com/user-attachments/assets/11ea344c-4910-4415-8104-9946a6f12450" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="249" alt="image" src="https://github.com/user-attachments/assets/dde3948f-8183-4160-8de1-fece45faf323" />
