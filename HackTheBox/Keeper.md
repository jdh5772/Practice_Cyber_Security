# Keeper - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA keeper 10.10.11.227
```
<img width="1104" height="282" alt="image" src="https://github.com/user-attachments/assets/6d893897-3fc0-442d-a3a8-596a05e9288e" />

- SSH(22)
- HTTP(80)

---
## HTTP
- 서버에 접속하면 `tickets.keeper.htb`로 연결을 하라고 한다.
<img width="530" height="87" alt="image" src="https://github.com/user-attachments/assets/065f9747-a253-40d9-bc56-18d71af8b9a1" />

<br>
<br>

- `/etc/hosts`에 등록.
<img width="607" height="63" alt="image" src="https://github.com/user-attachments/assets/3b7d45ac-f1f7-4af4-bfdc-4a9981574041" />

### banner grabbing
```bash
whatweb http://tickets.keeper.htb

curl -IL http://tickets.keeper.htb
```
<img width="1104" height="355" alt="image" src="https://github.com/user-attachments/assets/37d13704-4b50-48f4-8973-58d6a7f543a4" />

### 로그인
- `root:password`로 로그인 시도하여 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/0a022905-bb6c-49be-9fc3-e83ebcb5e497" />

---
## SSH
- 사이트를 둘러보던 중 user들을 발견.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/7311d709-125a-437d-9552-ad245d71591b" />

<br>
<br>

- `lnorgaard`유저를 살펴보니 패스워드가 노출되어 있었음.
<img width="668" height="134" alt="image" src="https://github.com/user-attachments/assets/cf8f94f2-8216-4f92-af08-7ad4efeaefe9" />

<br>
<br>

- `lnorgaard:Welcome2023!`으로 SSH 로그인.
<img width="695" height="240" alt="image" src="https://github.com/user-attachments/assets/41e8dcee-adb8-4836-8e8b-dd266f550fb6" />

---
## KeePass
- 폴더를 살펴보니 `RT30000.zip`이라는 파일을 발견.
<img width="837" height="251" alt="image" src="https://github.com/user-attachments/assets/3e2c5786-277b-49fc-8dbe-c6d6bf48ec67" />

<br>
<br>

- 해당 파일을 로컬로 옮겨서 압축 해제.
- `KeePassDumpFull.dmp`파일과 `passcodes.kdbx`파일이 압축 되어 있었음.
```bash
nc -vnlp 80 > RT30000.zip

7z e RT30000.zip
```
<img width="922" height="71" alt="image" src="https://github.com/user-attachments/assets/d21d873a-14a0-40e1-8320-c13fef83c63c" />

<br>
<br>

- `keepassxc`로 `passcodes.kdbx`파일을 열어보았으나 비밀번호가 걸려있는 것을 확인.
```bash
keepassxc passcodes.kdbx
```
<img width="707" height="387" alt="image" src="https://github.com/user-attachments/assets/326597a4-2cf3-4d0a-acc9-ec40acd728fd" />

<br>
<br>

- `kdbx`파일을 `keepass2john`을 사용하여 크래킹을 시도했으나 실패.
```bash
keepass2john passcodes.kdbx >hash

john hash --wordlist=~/util/rockyou.txt
```

### CVE-2023-32784
- `keepass`를 어떤 버전을 정확하게 사용하는지는 확인 불가능하나, `2.x`버전이라는 것은 확인.
<img width="1104" height="84" alt="image" src="https://github.com/user-attachments/assets/6133132d-e1bf-4403-bf16-9a52b8080bb2" />

<br>
<br>

- `dmp`파일을 통해서 마스터키를 복원할 수 있는 취약점 발견.
- https://nvd.nist.gov/vuln/detail/cve-2023-32784
<img width="1215" height="160" alt="image" src="https://github.com/user-attachments/assets/ba67383a-7135-4903-a962-b10c758f1dd1" />

<br>
<br>

- `dmp`파일을 `keepass-dump-masterkey`스크립트를 사용하여 크래킹 시도.
- https://github.com/matro7sh/keepass-dump-masterkey
```bash
python3 poc.py ../KeePassDumpFull.dmp
```
<img width="701" height="316" alt="image" src="https://github.com/user-attachments/assets/96dd8579-5993-463c-835d-edde9cd0746c" />

<br>
<br>

- 알아볼 수 없는 글자가 나와서 구글에 검색.
<img width="701" height="166" alt="image" src="https://github.com/user-attachments/assets/1928af31-131e-443c-ac50-9c0cda5b12a8" />

<br>
<br>

- `rødgrød med fløde`를 비밀번호로 `kdbx`파일을 확인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/24c4de4a-5d4c-4ec4-ad71-2e4bc60eed05" />

---
## Privesc
- `Network`탭에서 `root`유저의 `Putty  Private Key File(PPK)`파일 발견.
<img width="717" height="461" alt="image" src="https://github.com/user-attachments/assets/f0ad1091-7d1b-4239-a88e-031be3898bf6" />

<br>
<br>

- `ppk`파일을 `pem`파일로 바꿔서 ssh 로그인 시도.
<img width="1104" height="289" alt="image" src="https://github.com/user-attachments/assets/cde6ff24-5b79-4005-bbc2-1a73ba835ea6" />

---
## FLAG
- `/home/lnorgaard/user.txt`
<img width="1104" height="253" alt="image" src="https://github.com/user-attachments/assets/2d13bca8-1bdd-408c-9825-bc317cbd8c98" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="288" alt="image" src="https://github.com/user-attachments/assets/f602fba3-ebf3-463d-9697-f60fb233ad81" />

















