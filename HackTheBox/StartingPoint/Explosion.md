# Explosion - HackTheBox StartingPoint

## Recon
```bash
sudo nmap -p 135,139,445,3389,5985 -sC -sV -vv -oA explosion 10.129.4.38
```
<img width="768" height="279" alt="image" src="https://github.com/user-attachments/assets/fafe61a9-bffc-4e96-811d-beb2c6e808df" />

<img width="853" height="239" alt="image" src="https://github.com/user-attachments/assets/5156b02e-095c-47c3-bd14-958c280889ee" />

- SMB(445)
- RDP(3389)
- WINRM(5985)
---
## SMB
- SMB guest 로그인 시도했으나 특별한 정보 없음.
```bash
smbmap -u ‘’ -p ‘’ -H 10.129.4.38
smbmap -u ‘guest' -p ‘’ -H 10.129.4.38
```

## RDP
- windows의 최고 권한 계정은 administrator
- 어떤 유저 계정으로 로그인되는지 알 수 없어서, 최고 권한 계정인 administrator로 비밀번호 없이 로그인을 시도해봄.
```bash
xfreerdp3 /v:10.129.4.38 /u:administrator
```
<img width="1025" height="582" alt="image" src="https://github.com/user-attachments/assets/ec75f9c1-481a-433f-a430-64d053f794fe" />

---
## flag.txt
<img width="1025" height="319" alt="image" src="https://github.com/user-attachments/assets/5f38fe16-1a1c-4294-a5e2-a8221b90339a" />

