# Nocturnal - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA nocturnal 10.10.11.64
```
<img width="1103" height="413" alt="image" src="https://github.com/user-attachments/assets/eff73c51-e7b9-4893-8f5c-61adf817937c" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `nocturnal.htb`추가.
<img width="1103" height="63" alt="image" src="https://github.com/user-attachments/assets/72e3a02d-2b0a-422b-ae34-dc01e72ddf13" />

### banner grabbing
<img width="1103" height="320" alt="image" src="https://github.com/user-attachments/assets/d109de10-31d2-4812-8566-bffdf93db985" />

### IDOR
- 메인페이지에서 계정을 생성하여 로그인하면 파일 업로드가 가능.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/20ff7193-f15c-4d12-a19f-dbc9bc308411" />

<br>
<br>

- 허용되는 타입이 아니면 업로드가 불가능.
<img width="466" height="44" alt="image" src="https://github.com/user-attachments/assets/616237db-6c2a-4ae5-8f07-80ab7362f0e8" />

<br>
<br>

- `burpsuite`로 다운로드를 가로채서 같은 파일에 대해서 `username`을 `admin`과 `asdfasdf`로 바꿔보았을 때 다른 응답이 출력.
- `admin`의 경우 파일을 다운로드 할 수 없다고 하나, `asdfasdf`의 경우 유저를 찾을 수 없다는 응답.
<img width="1100" height="485" alt="image" src="https://github.com/user-attachments/assets/b5e91568-17ea-4a05-a3e0-4383b78eed84" />

<br>
<br>

<img width="1100" height="506" alt="image" src="https://github.com/user-attachments/assets/952d0ed5-8097-4d74-8250-6d2c4bb12f1c" />

<br>
<br>

- `file`파라미터에서 파일 이름을 변경했을 때는 업로드 된 파일을 보여주지만 파일이 존재하지 않는다는 메시지가 출력.
<img width="1100" height="516" alt="image" src="https://github.com/user-attachments/assets/2c33a02c-7a94-47cb-a267-6cb5b265562d" />

<br>
<br>

- `file`의 확장자는 `pdf`로 고정을 한 상태에서 `username`을 바꾸면 해당하는 유저의 파일들을 볼 수 있을 것이라 예상.
- `ffuf`를 사용하여 존재하는 유저 수집.
- `admin`, `amanda`, `tobias` 발견.
```bash
ffuf -H 'Cookie: PHPSESSID=ap7lth2tmbhlr686uppgjdsjkk' -u 'http://nocturnal.htb/view.php?username=FUZZ&file=.pdf' -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -fs 2985
```
<img width="1105" height="576" alt="image" src="https://github.com/user-attachments/assets/e1ec3bbf-5fe7-4b1e-b47b-6e5ba7a036f6" />

<br>
<br>

- `amanda`유저에게서 `privacy.odt`파일을 발견.
<img width="1100" height="493" alt="image" src="https://github.com/user-attachments/assets/8d54b909-3832-4a11-bdf3-8ef334d19a43" />

<br>
<br>

- 로컬로 다운로드받아 내용 확인.
- 비밀번호 발견.
<img width="670" height="231" alt="image" src="https://github.com/user-attachments/assets/66d990a8-4b2c-4d21-86e3-e87c1dde1789" />

<br>
<br>

- 메인페이지에서 `amanda:arHkG7HAI68X8s1J` 로그인.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/a7e033e6-0e8c-41f6-badc-006395aa9fda" />

<br>
<br>

- `Admin Panel`로 이동.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/af39c6da-2f5d-430c-9862-a9fdb715bdb5" />

<br>
<br>

- `backup password`에 `arHkG7HAI68X8s1J`를 입력하여 백업파일 다운로드.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/50998086-8ef7-42d7-9730-df785058b591" />

### Command Injection
- `arHkG7HAI68X8s1J` 비밀번호로 압축 해제.
<img width="1104" height="299" alt="image" src="https://github.com/user-attachments/assets/b3975c97-c3bf-4294-a7ce-d5dfb09d9c5e" />

<br>
<br>

- `admin.php`파일 아래에 백업파일을 생성하는 명령어가 실행되는 코드 발견.
- `password` 파라미터에서 제대로 검증이 되지 않는다면 원하는 명령어 실행이 가능할 것이라 예상.
<img width="1104" height="486" alt="image" src="https://github.com/user-attachments/assets/8b19659d-9854-44ff-a713-6827fe40f861" />

<br>
<br>

- `cleanEntry`함수에서 허용되지 않는 문자들이 보이나, 개행문자는 지정되어 있지 않음.(`\n`)
<img width="1104" height="231" alt="image" src="https://github.com/user-attachments/assets/ec4375fe-c1b6-49aa-97a6-b359b150521b" />

<br>
<br>

- `Create Backup` 버튼을 눌러 백업을 만드는 과정을 가로채서 확인.
<img width="797" height="356" alt="image" src="https://github.com/user-attachments/assets/00b6e9b6-d95c-4fad-8b20-f574e8da3b6b" />

<br>
<br>

- 개행문자 바로 뒤에 명령어를 입력해보았으나 실패.(`\nwhoami#`)
<img width="1100" height="506" alt="image" src="https://github.com/user-attachments/assets/89a8b948-3339-4cbe-9e97-1bf1b1a6996a" />

<br>
<br>

- `bash -c “whoami”#`를 입력해보았으나 실패.
- 다른 오류 메시지.
<img width="1100" height="506" alt="image" src="https://github.com/user-attachments/assets/1e7650ad-592f-41df-9592-150b9c340da5" />

<br>
<br>

- 공백을 `tab`으로 바꿔서 실행 가능.(%20 > %09)
<img width="1100" height="506" alt="image" src="https://github.com/user-attachments/assets/1926ff8e-2d5c-4dfa-a393-e512dbf830ce" />

<br>
<br>

- `ex.sh` 생성.
<img width="1106" height="94" alt="image" src="https://github.com/user-attachments/assets/4a1249a6-7ee6-4e09-a043-27982e2e6857" />

<br>
<br>

- 로컬로부터 `ex.sh` 다운로드.(공백은 모두 탭으로 치환)
<img width="1100" height="506" alt="image" src="https://github.com/user-attachments/assets/a3370a86-b24e-4ba9-a7bc-a163055ce586" />

<br>
<br>

- 리버스 셸 실행.(공백은 모두 탭으로 치환)
<img width="796" height="352" alt="image" src="https://github.com/user-attachments/assets/19c93b12-8586-4861-848d-ab3073bc3895" />

<br>
<br>

- 셸 획득.
<img width="1104" height="178" alt="image" src="https://github.com/user-attachments/assets/ed67aa0a-dd9a-4dc9-a0af-e1ade54985e7" />

---
## Privesc
- `/var/www/nocturnal_database`에서 `nocturnal_database.db`발견.
<img width="1104" height="99" alt="image" src="https://github.com/user-attachments/assets/95f84a98-7ac7-4986-a676-2f3fedbc7bc8" />

<br>
<br>

- 계정 획득.
```bash
sqlite3 nocturnal_database.db 'select username,password from users;'
```
<img width="1104" height="136" alt="image" src="https://github.com/user-attachments/assets/7940e6f3-812c-409b-b653-05898b95a15f" />

<br>
<br>

- `hashcat`으로 크래킹하여 `tobias:slowmotionapocalypse` 획득.
<img width="1104" height="77" alt="image" src="https://github.com/user-attachments/assets/3274793a-24d1-4f54-8ae6-7b7d07e80fc2" />

<br>
<br>

- SSH 로그인(tobias:slowmotionapocalypse)
```bash
ssh tobias@10.10.11.64
```
<img width="1104" height="40" alt="image" src="https://github.com/user-attachments/assets/98dd99f4-3847-41a2-840a-981e0975daf5" />

### CVE-2023-46818 
- 8080번포트 발견.
```bash
curl -v http://127.0.0.1:8080
```
<img width="1104" height="458" alt="image" src="https://github.com/user-attachments/assets/8a919dfc-b9a9-481f-ab0e-70eacf732824" />

<br>
<br>

- SSH로 8080포트 포워딩 하여 접속.
- 서버로 접속이 안되어서 쿠키 초기화를 진행하였더니 접속 가능.
- `ispconfig`
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/81fe7a36-e6c4-4514-8025-6ccfcb5950de" />

<br>
<br>

- `admin:slowmotionapocalypse` 로그인.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/0ceb7267-a8b4-44d4-8fa9-d20a14ce431d" />

<br>
<br>

- 버전 확인(ISPConfig 3.2.10p1)
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/e5435b42-53c2-42dc-b166-a6e901761b91" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2023-46818
<img width="1100" height="409" alt="image" src="https://github.com/user-attachments/assets/c8b7e6f3-63d9-456b-ba5b-85f2fe32afb9" />

<br>
<br>

- https://github.com/bipbopbup/CVE-2023-46818-python-exploit
- `burpsuite` 프록시를 설정하여 `POST`와 `GET`요청에 넣어서 실행.
<img width="1105" height="49" alt="image" src="https://github.com/user-attachments/assets/a5c19d03-ecb5-46d0-9407-29d784dd016f" />

<br>
<br>

- 로그인하여 `ISPCSESS`획득.
<img width="1100" height="281" alt="image" src="https://github.com/user-attachments/assets/80580ca0-602b-453c-951d-08715140e83d" />

<br>
<br>

- `/admin/language_edit.php`로 데이터를 전달하여 `_csrf_id`와 `_csrf_key`값을 획득.
<img width="1100" height="281" alt="image" src="https://github.com/user-attachments/assets/66887309-9b1d-4b2c-9b98-cb6ce768489d" />

<br>
<br>

- `PHP`코드를 `base64`로 인코딩한 후 페이로드에 넣어서 전달.
<img width="1104" height="202" alt="image" src="https://github.com/user-attachments/assets/0eb6162e-5156-40f5-8b8a-e94f7408d802" />

<br>
<br>

<img width="1100" height="282" alt="image" src="https://github.com/user-attachments/assets/09a071ba-97cd-460a-8f4e-d1d40a2eb21e" />

<br>
<br>

- 헤더에 명령어를 `base64`로 인코딩한 후 전달하여 명령어 실행.
```bash
# d2hvYW1p
echo -n 'whoami' |base64 -w0

curl -H 'C:d2hvYW1p' http://127.0.0.1:9001/admin/sh.php
```
<img width="1106" height="92" alt="image" src="https://github.com/user-attachments/assets/c44ce889-29c9-4b8b-9641-cf00d3f38248" />

<br>
<br>

- 이전에 로컬로부터 받은 `/tmp/ex.sh`를 실행.
```bash
# L2Jpbi9iYXNoIC90bXAvZXguc2g=
echo -n '/bin/bash /tmp/ex.sh'|base64 -w0

curl -H 'C:L2Jpbi9iYXNoIC90bXAvZXguc2g=' http://127.0.0.1:9001/admin/sh.php
```
<img width="1106" height="124" alt="image" src="https://github.com/user-attachments/assets/7d20c6a6-a02d-44ba-bef6-429412494a91" />

<br>
<br>

- 셸 획득.
<img width="1106" height="172" alt="image" src="https://github.com/user-attachments/assets/b42d21a2-64f1-41a1-859a-ed1816eb9ca8" />

---
## FLAG
- `/home/tobias/user.txt`
<img width="1106" height="289" alt="image" src="https://github.com/user-attachments/assets/9e789337-91ea-45d4-a7a6-ddd258fe7578" />

<br>
<br>

- `/root/root.txt`
<img width="1106" height="268" alt="image" src="https://github.com/user-attachments/assets/f0f15879-08a4-48a0-8beb-98acf5e6b076" />













