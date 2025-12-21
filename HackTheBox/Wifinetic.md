# Wifinetic - HackTheBox
## Recon
```bash
sudo nmap -p 21,22,53 -sC -sV -vv -oA wifi 10.10.11.247
```
<img width="1104" height="557" alt="image" src="https://github.com/user-attachments/assets/4d4b9f84-c6bd-42e4-82a1-6fbabe1ef083" />

- FTP(21)
- SSH(22)
- DNS(53)

---
## FTP
- `anonymous` 로그인 성공하여 내부 파일 다운로드
```bash
ftp 10.10.11.247
```
<img width="1104" height="594" alt="image" src="https://github.com/user-attachments/assets/2d209677-2ead-4c73-8acf-525c873f8bb5" />

---
## SSH
- pdf파일들에서는 중요한 정보를 발견하지 못함.

<br>

- `backup-OpenWrt-2023-07-26.tar`를 압축해제.
```bash
tar -xvf backup-OpenWrt-2023-07-26.tar
```
<img width="1104" height="418" alt="image" src="https://github.com/user-attachments/assets/acdc4898-0778-40b2-88db-b5890b60d08f" />

<br>
<br>

- `passwd`파일에서 user 정보 획득.
<img width="1104" height="248" alt="image" src="https://github.com/user-attachments/assets/4329954e-20cb-4077-a686-85f5ca6a6bd3" />

<br>
<br>

- `/config/wireless`파일에서 비밀번호 정보 획득.
<img width="1104" height="307" alt="image" src="https://github.com/user-attachments/assets/2d316973-bfae-43de-8201-fb032237efb4" />

<br>
<br>

- `netadmin:VeRyUniUqWiFIPasswrd1!`로 로그인 시도.
```bash
ssh netadmin@10.10.11.247
```
<img width="1104" height="46" alt="image" src="https://github.com/user-attachments/assets/086bcff2-cb01-42dd-a7e6-39ea09a5b05d" />

---
## Privesc
- `getcap`을 사용하여 `reaver`프로그램을 발견.
```bash
getcap -r / 2>/dev/null
```
<img width="1104" height="119" alt="image" src="https://github.com/user-attachments/assets/456c08c2-b449-4667-b7ee-b56e64f7fe4d" />

<br>
<br>

- `reaver` : PIN번호를 브루트포스 공격을 사용해 패스워드를 알아내는 도구.
- https://airheads.hpe.com/discussion/how-dangerous-is-the-reaver-wps-attack
<img width="1078" height="142" alt="image" src="https://github.com/user-attachments/assets/515ec1fe-1624-42fe-80d3-f79da6acf222" />

<br>
<br>

- `iwconfig`을 통해서 모니터모드와 공격을 실행할 AP를 찾아냄.
- `wlan0`는 사용되지 않는 것처럼 보이며, `wlan1`을 공격 대상으로 지정.
```bash
iwconfig
```
<img width="1078" height="554" alt="image" src="https://github.com/user-attachments/assets/d5ba64f2-e967-4b7f-af1e-37cd82ec4bb4" />

<br>
<br>

- `reaver`를 사용하여 패스워드 획득.
```bash
reaver -i mon0 -b 02:00:00:00:00:00 -v
```
<img width="1078" height="374" alt="image" src="https://github.com/user-attachments/assets/eb64fa6c-c333-4151-91c2-f16ae9fe0d39" />

<br>
<br>

- `root`로그인.(root:WhatIsRealAnDWhAtIsNot51121!)
<img width="1078" height="84" alt="image" src="https://github.com/user-attachments/assets/e9958219-e091-4779-ad23-db092ddcd6c7" />

---
## FLAG
- `/home/netadmin/user.txt`
<img width="1104" height="196" alt="image" src="https://github.com/user-attachments/assets/7eaee815-a46b-4ddc-9c3a-19cb4ffbec64" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="275" alt="image" src="https://github.com/user-attachments/assets/0ddf1e0c-3a41-4cdb-8273-f39dcc832eec" />










