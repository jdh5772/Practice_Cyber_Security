# Mirai - HackTheBox
## Recon
```bash
sudo nmap -p 22,53,80,1560,32400,32469 -sC -sV -vv -oA mirai 10.10.10.48
```
<img width="1104" height="535" alt="image" src="https://github.com/user-attachments/assets/27d8e2b3-966a-40c5-a117-67180f8229c3" />
<img width="1104" height="194" alt="image" src="https://github.com/user-attachments/assets/364820b1-2391-4e7e-8381-e6a8489148ca" />

- SSH(22)
- HTTP(80)
- HTTP(32400)

---
## HTTP(80)
### banner grabbing
- `404` error 페이지가 반환되지만 헤더에 `X-Pi-hole`이 발견됨.
- https://github.com/pi-hole/pi-hole
```bash
whatweb http://10.10.10.48

curl -IL http://10.10.10.48
```
<img width="1106" height="255" alt="image" src="https://github.com/user-attachments/assets/1f77411b-ae88-4a60-a0ef-ba174efd8cf0" />

<br>
<br>

### gobuster
- `/admin` 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.48
```
<img width="1106" height="352" alt="image" src="https://github.com/user-attachments/assets/87448a91-ff99-4c08-863b-9f8c42a18184" />

---
## SSH
- `/admin`경로에서 기본 비밀번호로 로그인 시도하였으나 실패.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/cecbaba8-362c-4b69-a9d7-14ec335eddf6" />

<br>
<br>

- `Pi-hole`은 리눅스 프로그램이고, 라즈베리 파이로 구현되는 경우가 많다.
- 라즈베리 파이의 기본 설정 계정 `pi:raspberry`로 SSH 로그인 시도.
```bash
ssh pi@10.10.10.48
```
<img width="1106" height="502" alt="image" src="https://github.com/user-attachments/assets/81b41af2-7060-47e7-a201-8d64e933de5c" />

---
## Privesc
- `root`권한으로 모든 명령어 실행 가능.
<img width="1106" height="135" alt="image" src="https://github.com/user-attachments/assets/fbc74263-63fe-4630-a2af-36983802eeec" />

<br>
<br>

- `root` 획득.
```bash
sudo /bin/bash
```
<img width="1106" height="43" alt="image" src="https://github.com/user-attachments/assets/952f9837-30bc-40e3-b47c-8fbdb3ed8045" />

---
## FLAG
- `/home/pi/Desktop/user.txt`
<img width="1106" height="117" alt="image" src="https://github.com/user-attachments/assets/1cd4bae3-894c-4e8f-bf7b-72d11e233034" />

<br>
<br>

- `/root/root.txt`
- `root.txt`에서 flag를 바로 획득 불가.
<img width="1106" height="231" alt="image" src="https://github.com/user-attachments/assets/b25e77f5-dab3-415e-9a56-a643c51ccfdd" />

<br>
<br>

- `/media/usbstick`에 `/dev/sdb`가 마운트 되어 있는 것을 확인.
```bash
mount
```
<img width="1106" height="60" alt="image" src="https://github.com/user-attachments/assets/20c2f54a-2129-4c62-8a39-9d0a6d712836" />

<br>
<br>

- `/media/usbstick`에서 `damnit.txt`을 확인하니 파일이 삭제 되었다고 한다.
<img width="1106" height="96" alt="image" src="https://github.com/user-attachments/assets/d5f9592e-4e2a-49cf-92a2-40fe3572c145" />

<br>
<br>

- 파일을 삭제하더라도 메타데이터만 삭제가 되어 raw파일이 덮어쓰기가 되지 않는한 복구가 가능.
- `strings /dev/sdb`로 `root.txt`플래그 획득.









