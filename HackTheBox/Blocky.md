# Blocky - HackTheBox
## Recon
```bash
sudo nmap -p 21,22,80,25565 -sC -sV -vv -oA blocky 10.10.10.37
```
<img width="1204" height="381" alt="image" src="https://github.com/user-attachments/assets/f8f17380-aef8-47e8-a4aa-4f7d6a722c4f" />

- FTP(21)
- SSH(22)
- HTTP(80)

---
## FTP
- `anonymous`로그인 실패.

---
## HTTP
- `/etc/hosts`에 `blocky.htb` 등록.
<img width="1204" height="63" alt="image" src="https://github.com/user-attachments/assets/e37d7e67-b649-4455-8ce7-7eddd4d99920" />

### banner grabbing
```bash
whatweb http://blocky.htb

curl -IL http://blocky.htb
```
<img width="1204" height="263" alt="image" src="https://github.com/user-attachments/assets/812ddf72-4397-465e-abcc-c7df59479528" />

### wordpress
- `wpscan`으로 취약점을 찾아보았으나, RCE를 실행시킬 만한 취약점은 발견되지 않았음.
```bash
wpscan --url http://blocky.htb --api-token <my_token>
```

### gobuster
- `/plugins`발견
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://blocky.htb -x php,md
```
<img width="1204" height="421" alt="image" src="https://github.com/user-attachments/assets/7a1a4704-aefc-44cc-aa74-aecfe18f1da8" />

<br>
<br>

- `jar`파일 다운로드.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/732eda6d-c816-43f3-a004-a83cb12873f3" />

---
## SSH
- `BlockyCore.jar`파일에서 계정 노출.
```bash
jd-gui BlockyCore.jar
```
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/5d62250d-7870-4790-93ba-548733822bb0" />

<br>
<br>

- `root:8YsqfCTnvxAUeduzjNSXe22`로 SSH 로그인을 시도했으나 실패.
- 서버에 작성되어 글의 작성자가 `notch`로 되어 있는 것을 발견.
<img width="592" height="297" alt="image" src="https://github.com/user-attachments/assets/2ab41411-ee66-4163-9ce2-d51cae6a0a51" />

<br>
<br>

- `notch:8YsqfCTnvxAUeduzjNSXe22` 로그인.
<img width="1205" height="350" alt="image" src="https://github.com/user-attachments/assets/58a5916e-ebc6-4174-89c6-9d5d8bd6ab8e" />

---
## Privesc
- 해당 유저로 모든 명령어를 `root`권한으로 실행이 가능.
```bash
sudo -l
```
<img width="1205" height="183" alt="image" src="https://github.com/user-attachments/assets/eae57448-f363-4cd5-84f4-f1ddab900607" />

<br>
<br>

- `root`셸 획득.
```bash
sudo /bin/bash
```
<img width="1205" height="62" alt="image" src="https://github.com/user-attachments/assets/fb1de8d2-a3b5-4bb6-bb0a-d0610f7721e5" />

---
## FLAG
- `/home/notch/user.txt`
<img width="1205" height="239" alt="image" src="https://github.com/user-attachments/assets/22dafa08-55e2-48ba-b11d-fd8d833f49c2" />

<br>
<br>

- `/root/root.txt`
<img width="1205" height="168" alt="image" src="https://github.com/user-attachments/assets/8697200b-6798-48dc-b922-6021f3c6ff39" />











