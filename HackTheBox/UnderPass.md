# UnderPass - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA underpass 10.10.11.48
```
<img width="1106" height="286" alt="image" src="https://github.com/user-attachments/assets/c5faf45f-8c3f-48f1-ab9e-07902faa73d8" />

- SSH(22)
- HTTP(80)

<br>

```bash
sudo nmap -p 161 -sU -sC -sV 10.10.11.48
```
<img width="1106" height="229" alt="image" src="https://github.com/user-attachments/assets/94a497e0-ba5d-4559-99a1-daa098616a99" />

- SNMP(161)

---
## SNMP
- `snmpbulkwalk`를 사용하여 MIB를 확인해보니 `daloradius`라는 출력이 보임.
```bash
snmpbulkwalk -c public -v2c 10.10.11.48 . >result
```
<img width="1106" height="166" alt="image" src="https://github.com/user-attachments/assets/ed8f511b-58e6-4785-8333-0dd25a140a7b" />

<br>
<br>

- 검색을 해보니 `web management application`라고 한다.
- https://www.daloradius.com/
<img width="566" height="264" alt="image" src="https://github.com/user-attachments/assets/d39935fd-4bc9-4d46-acdf-cc9e673ec983" />

## HTTP
- `SNMP`에서 발견한 `underpass.htb`를 `/etc/hosts`에 추가.
<img width="1103" height="65" alt="image" src="https://github.com/user-attachments/assets/e42a1d3b-edcf-4443-be84-7407cd3f5820" />

<br>
<br>

- `/daloradius` 경로로 접속 시도하였으나 접근 불가.
<img width="507" height="186" alt="image" src="https://github.com/user-attachments/assets/c2eb9a9d-846d-4b13-9cee-c685f49ca57c" />

### feroxbuster
- 디렉토리 `/daloradius`를 발견 하였기에 `gobuster` 대신에 `feroxbuster`를 사용.
```bash
feroxbuster -u http://underpass.htb/daloradius
```
<img width="1103" height="307" alt="image" src="https://github.com/user-attachments/assets/0780ac82-98be-4af7-adc6-a7e7b6aa7e01" />

<br>
<br>

- 많은 서브디렉토리 중에서 `/daloradius/app/users`로 접속하니 login 페이지로 연결됨.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/4e2b3bfc-106e-44d2-94f0-c37a7d87a68f" />

<br>
<br>

- `feroxbuster`에서 발견된 `/daloradius/doc/install/INSTALL`에서 버전 정보 획득.
<img width="524" height="184" alt="image" src="https://github.com/user-attachments/assets/4aa21b48-f74f-4cc2-9297-6b8fdc6a8c49" />

<br>
<br>

- 해당 버전으로 검색된 exploit을 시도했으나 작동하지 않음.
- https://nvd.nist.gov/vuln/detail/CVE-2009-4347
<img width="1225" height="391" alt="image" src="https://github.com/user-attachments/assets/1797eacb-250e-4ff9-9fd3-3b2d99572c54" />

<br>
<br>

- https://github.com/lirantal/daloradius/tree/master/app
- `/app` 디렉토리 내에 `/operators`경로가 있어서 서버에서 해당 경로로 접속을 시도하여 새로운 로그인 페이지 발견.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/6b19cad3-67cf-42a0-88d7-238aaf160963" />

<br>
<br>

- https://github.com/lirantal/daloradius/blob/c62f68e2e3f90f5b2f31266caf7d51cf882d7b7c/doc/install/INSTALL#L202
- 초기 설정이 `administrator:radius`로 되어 있는 것을 발견.
<img width="1060" height="273" alt="image" src="https://github.com/user-attachments/assets/86051a65-1720-45b6-80cf-eaffab3a5277" />

<br>
<br>

- 로그인 성공.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/4bb15eca-3ebc-48d0-b171-b59cc1060c0d" />

## SSH
- `Management > List Users`에서 `smvMosh`유저를 발견.
<img width="1100" height="282" alt="image" src="https://github.com/user-attachments/assets/65625e21-ee45-4003-8a79-1bca5641d1af" />

<br>
<br>

- 해당 유저 프로필에서 `Check Attributes`탭을 클릭해보니 비밀번호가 `MD5`로 해싱되어 있는 것을 확인.
<img width="495" height="50" alt="image" src="https://github.com/user-attachments/assets/1e0486da-fdc1-4391-83ac-244c69591bdb" />

<br>
<br>

- `hashcat`으로 크래킹하여 `underwaterfriends` 비밀번호 획득.
```bash
hashcat -m 0 412DD4759978ACFCC81DEAB01B382403 ~/util/rockyou.txt
```
<img width="1107" height="73" alt="image" src="https://github.com/user-attachments/assets/10f9f448-d4a3-44b6-9f19-148f23120769" />

<br>
<br>

- `svcMosh:underwaterfriends`로그인.
```bash
ssh svcMosh@10.10.11.48
```
<img width="1107" height="44" alt="image" src="https://github.com/user-attachments/assets/95a20692-c29e-4e2d-864e-0f0ee91d0dba" />

## Privesc
- `/usr/bin/mosh-server`를 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1107" height="136" alt="image" src="https://github.com/user-attachments/assets/5da9cdf4-ef0f-4b1a-80bc-63772569305d" />

<br>
<br>

- https://mosh.org/
- 모바일 환경 터미널 프로그램.
<img width="566" height="94" alt="image" src="https://github.com/user-attachments/assets/0dc9afd5-c319-4d18-aafb-eba2822c8394" />

<br>
<br>

- 링크 내용의 아래에 서버와 클라이언트 실행 방법에 대해서 나와있다.
<img width="969" height="508" alt="image" src="https://github.com/user-attachments/assets/03d5dc6d-a2f3-4c1a-9115-81db278fa3ca" />

<br>
<br>

- `mosh-server` 실행.
```bash
sudo /usr/bin/mosh-server
```
<img width="1104" height="229" alt="image" src="https://github.com/user-attachments/assets/af31b4c6-900e-4eab-9ac8-4ade6d41a3ab" />

<br>
<br>

- `mosh-client`실행하여 `root`셸 획득.
```bash
MOSH_KEY='HPl+zehM5Ea753wTd8/GYw' mosh-client 10.110.11.48 60001
```
<img width="1104" height="366" alt="image" src="https://github.com/user-attachments/assets/46795714-6e4d-47f0-b96b-1c3826bba649" />

## FLAG
- `/home/svcMosh/user.txt`
<img width="1104" height="234" alt="image" src="https://github.com/user-attachments/assets/e996501c-7cac-4e8f-9dd5-7889c8a4a86d" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="269" alt="image" src="https://github.com/user-attachments/assets/a277201a-f672-4789-b930-1619d93d8867" />
















