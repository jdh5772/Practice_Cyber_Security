# Nunchucks - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,443 -sC -sV -vv -oA Nunchucks 10.10.11.122
```
<img width="1102" height="392" alt="image" src="https://github.com/user-attachments/assets/ece80e2b-0eba-4cb2-a872-47bbda6cb9ae" />
<img width="1102" height="231" alt="image" src="https://github.com/user-attachments/assets/85ddbb50-cee0-411a-bff7-289ae89a7c43" />

- SSH(22)
- HTTP(80)
- HTTPS(443)

---
## HTTPS(443)
- `/etc/hosts`에 `nunchucks.htb` 추가.
<img width="1102" height="64" alt="image" src="https://github.com/user-attachments/assets/bbda6b9f-92e6-43b4-996d-741c7e75d39b" />

<br>
<br>

- HTTP로 연결시  HTTPS로 리다이렉팅.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/062ebdc4-b0f1-44d0-897c-c9210773d7f7" />

### banner grabbing
<img width="1102" height="337" alt="image" src="https://github.com/user-attachments/assets/714fce6d-6223-4dfb-a6c1-547b42eb82ea" />

### FUFF
- `store.nunchucks.htb` 발견.
```bash
ffuf -w ~/util/subdomains.txt -u https://nunchucks.htb -H 'Host:FUZZ.nunchucks.htb' -mc all -fs 30589
```
<img width="1102" height="582" alt="image" src="https://github.com/user-attachments/assets/47d00ed8-6a6d-4279-88c9-7012477d54d6" />

<br>
<br>

- `/etc/hosts`에 추가.
<img width="1102" height="65" alt="image" src="https://github.com/user-attachments/assets/714b621f-59cd-4930-938f-a2a7696d3467" />

### SSTI
- 입력란에 입력을 한 후 `burpsuite`로 캡쳐해서 SSTI공격 시도하여 성공.
<img width="1100" height="311" alt="image" src="https://github.com/user-attachments/assets/c2cc4386-6688-4554-ae8b-171cf86495c6" />

<br>
<br>

- https://github.com/ping-0day/templates/blob/main/node-nunjucks-ssti.yaml
- 어떤 템플릿을 사용하는지 정확하게 파악되지 않으나, 머신의 이름에서 힌트를 추측.
- `Nunchucks` 템플릿을 사용한다고 가정하여 명령어 실행 성공.
<img width="1100" height="331" alt="image" src="https://github.com/user-attachments/assets/dfdc6172-4635-4102-ba1b-33cfe125e47e" />

<br>
<br>

- 로컬에 `ex.sh`생성.
<img width="1103" height="100" alt="image" src="https://github.com/user-attachments/assets/d003b0af-af3a-47fd-8011-76620ed6122d" />

<br>
<br>

- 로컬로부터 다운로드.
<img width="1100" height="331" alt="image" src="https://github.com/user-attachments/assets/bdf253c9-6ab4-477c-9039-e13fc13a0a4a" />

<br>
<br>

- 리버스 셸 실행.
<img width="772" height="466" alt="image" src="https://github.com/user-attachments/assets/ecb8f74d-7e10-4509-a434-f3df8a53a411" />

<br>
<br>

- 셸 획득.
<img width="1105" height="179" alt="image" src="https://github.com/user-attachments/assets/cc26cade-27e5-4e52-a3b9-aaf514c6e2f9" />

---
## Privesc
- `getcap`으로 `perl`스크립트를 `root`권한으로 실행할 수 있는 것을 발견.
```bash
getcap -r / 2>/dev/null
```
<img width="1105" height="116" alt="image" src="https://github.com/user-attachments/assets/d44a42bd-cb8d-49ab-983c-ec8c3808606d" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/perl/
- `GTFOBIN`을 따라서 셸을 실행해보았으나 실패.
- 다른 명령어는 실행이 가능.
```bash
/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "whoami";'
```
<img width="1105" height="39" alt="image" src="https://github.com/user-attachments/assets/62748204-fa49-4843-a823-bfb2e06086ce" />

<br>
<br>

- `/etc/apparmor.d/usr.bin.perl`에서 허용되는 프로그램 확인.
<img width="1105" height="458" alt="image" src="https://github.com/user-attachments/assets/f5d3c677-63c2-4512-b49e-dd9567d4ae82" />

<br>
<br>

- `/tmp`에 `/bin/bash`를 실행시키는 `perl`스크립트 생성.
- `perl`에 SETUID가 설정되어 있기 때문에 스크립트를 실행할 경우 권한상승으로 연결될 것이라 예상.
<img width="1105" height="64" alt="image" src="https://github.com/user-attachments/assets/82cf0ceb-fbe9-4256-b929-bd4be7857477" />

<br>
<br>

- `perl`로 실행할 경우 `Apparmor`를 거쳐서 실행되는데, `/bin/bash`가 허용되지 않기 때문에 실패.
```bash
perl ex.pl
```
<img width="1105" height="43" alt="image" src="https://github.com/user-attachments/assets/c41da5ce-23a5-4609-9617-f9a1f325bbe2" />

<br>
<br>

- `#!/usr/bin/perl`를 첫줄에 넣어서 스크립트 자체를 실행할 수 있도록 만들 경우 `Apparmor`를 우회하기 때문에 스크립트를 실행 가능.
- `root` 획득.
<img width="1105" height="60" alt="image" src="https://github.com/user-attachments/assets/5a2fc248-931b-4ade-aca2-0ce54db3ec13" />

---
## FLAG
- `/home/david/user.txt`
<img width="1105" height="290" alt="image" src="https://github.com/user-attachments/assets/4250b4b2-3666-444d-9c5e-da770b32286b" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="309" alt="image" src="https://github.com/user-attachments/assets/331f2868-c62f-42f7-b75f-4195fe3205db" />




















