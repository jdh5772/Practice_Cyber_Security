# Knife - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA knife 10.10.10.242
```
<img width="1209" height="355" alt="image" src="https://github.com/user-attachments/assets/ee1db9ce-57ef-4c5c-a4c2-2f174ced0258" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.242

curl -IL http://10.10.10.242
```
<img width="1209" height="242" alt="image" src="https://github.com/user-attachments/assets/98532f65-8027-4147-9d1c-f78df9d2707f" />

### gobuster
- `gobuster`로 directory를 탐색해봐도 특정한 파일을 발견할 수 없었음.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.242/ -x php,md
```

### PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution
- 해당 서버가 `PHP 8.1.0-dev`버전을 사용하고 있어 구글에 검색을 시도.
- https://www.exploit-db.com/exploits/49933
- `User-Agentt`헤더로 RCE가 실행되는 취약점 확인.
<img width="1209" height="312" alt="image" src="https://github.com/user-attachments/assets/09de3541-b733-4d43-b1a9-e5fad7820674" />

<br>
<br>

- `burpsuite`로 헤더를 변경하여 ping test 시도.
<img width="773" height="249" alt="image" src="https://github.com/user-attachments/assets/7c7b00d3-6d1a-46c0-a201-8f79e3865bcf" />

<br>
<br>

- ping test 성공.
<img width="773" height="238" alt="image" src="https://github.com/user-attachments/assets/998d4fc8-3e92-45c0-9618-8a238baaca0d" />

<br>
<br>

- 리버스 셸 연결 시도.
<img width="772" height="247" alt="image" src="https://github.com/user-attachments/assets/86e32886-ebb2-4d98-9946-b51de12c3af8" />

<br>
<br>

- 셸 획득.
<img width="1206" height="176" alt="image" src="https://github.com/user-attachments/assets/69b7c37b-9481-4dd9-860d-2531d0c63fae" />

---
## Privesc
- `sudo`를 시용하여 `knife`를 `root`권한으로 실행할 수 있는것을 확인.
```bash
sudo -l
```
<img width="1206" height="114" alt="image" src="https://github.com/user-attachments/assets/e5a1a9f6-f03f-40c7-aff0-59b5ce1dffc2" />

<br>
<br>

- `GTFOBIN`을 참조하여 `root`권한 획득.
- https://gtfobins.github.io/gtfobins/knife/
```bash
sudo /usr/bin/knife exec -E 'exec "/bin/bash"'
```
<img width="1206" height="60" alt="image" src="https://github.com/user-attachments/assets/60727a26-cc7f-43bb-b4d6-7f0aa25f3e8c" />

---
## FLAG
- `/home/james/user.txt`
<img width="1206" height="242" alt="image" src="https://github.com/user-attachments/assets/fe9bc7e8-16da-4e74-b3d0-c00b9056b29e" />

<br>
<br>

- `/root/root.txt`
<img width="1206" height="309" alt="image" src="https://github.com/user-attachments/assets/bb58619c-bc33-4ac2-9546-e67e9545f27e" />










