# Appointment - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA Appointment 10.129.5.225
```
<img width="720" height="167" alt="image" src="https://github.com/user-attachments/assets/b6f761f8-45d4-4d87-bcb4-9fca623492ce" />

- HTTP(80)

## Banner Grabbing and Enumerate Directory
- `whatweb`, `curl`, `gobuster` 모두 특별한 정보를 찾아낼 수 없었음.
```bash
whatweb http://10.129.5.225

curl -I -L http://10.129.5.225

gobuster dir -u http://10.129.5.225 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```

## HTTP
- 페이지의 소스코드를 확인해보았으나 특별한 정보를 찾을 수 없었음.
<img width="1103" height="906" alt="image" src="https://github.com/user-attachments/assets/dcf1c515-8462-46e6-8067-576d64179717" />

## SQL INJECTION
- SQL INJECTION 기본 테스트가 작동되는지 해보았고, 성공하여 플래그 얻을 수 있었음.
```SQL
admin'-- -
```
<img width="1103" height="888" alt="image" src="https://github.com/user-attachments/assets/1fc589ff-f016-4c97-b67b-8884e686de05" />
