# Paper - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,443 -sC -sV -vv -oA paper 10.10.11.143
```
<img width="1104" height="517" alt="image" src="https://github.com/user-attachments/assets/ee9351f0-39fe-472d-a824-673f6cb6b27c" />

- SSH(22)
- HTTP(80)
- HTTPS(443)

---
## HTTP
### banner grabbing
<img width="1104" height="365" alt="image" src="https://github.com/user-attachments/assets/12559aca-1ac1-4d85-8952-01138edab69c" />

<br>
<br>

- `X-Backend`헤더에서 서버에 대한 정보가 노출되어 `/etc/hosts`에 `office.paper`를 추가.
<img width="1104" height="64" alt="image" src="https://github.com/user-attachments/assets/c096cbea-f3f4-42b3-9cdb-ec02cea9b7cc" />

<br>
<br>

- `office.paper` banner grabbing
<img width="1104" height="336" alt="image" src="https://github.com/user-attachments/assets/7536398c-5e57-4edb-8f26-1f03917e3cbf" />

---
### CVE-2019-17671
- `office.paper`에서 첫번째 글의 댓글에서 `drafts`로부터 중요한 정보를 제거해야한다고 함.
<img width="802" height="708" alt="image" src="https://github.com/user-attachments/assets/8c37e7db-397e-4477-a166-69ea5ec618de" />

- `banner grabbing`과정에서 `wordpress`를 발견하여 `wpscan`을 사용하여 취약점 발견.
```bash
wpscan --url http://office.paper --api-token <token> -e vp,vt
```
<img width="1104" height="208" alt="image" src="https://github.com/user-attachments/assets/2b92b317-ee67-483e-84aa-80568bc7c1cb" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2019-17671
<img width="1222" height="368" alt="image" src="https://github.com/user-attachments/assets/6983ca82-fa6f-4fff-bae6-d73d23ba83a3" />

<br>
<br>

- https://www.exploit-db.com/exploits/47690
- `static`파라미터에 값을 줘서 확인해보니 `chat.office.paper`에 숨겨진 계정 등록 링크를 발견.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/8c6e4c58-3749-4a1d-9d8d-7c046aaa26d6" />

<br>
<br>

- `/etc/hosts`에 `chat.office.paper`를 등록하여 해당 링크로 접속.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/15934c55-3500-4835-b991-73fc9523ef99" />

<br>
<br>

- 로그인하여 기다리면 자동으로 `general`에 초대된다.
- `recyclops`라는 봇이 만들어져 있는데 리스팅과 파일 내용을 읽을 수 있는 명령어를 실행할 수 있는 것처럼 보임.
<img width="1466" height="751" alt="image" src="https://github.com/user-attachments/assets/dc537be4-3429-4597-abf0-490e9f99855a" />

<br>
<br>

- `recyclops`에게 1:1 대화를 신청하여 `recyclops help`를 입력하면 명령어 사용 방법에 대해서 출력.
<img width="845" height="182" alt="image" src="https://github.com/user-attachments/assets/66de5518-c53e-40b0-9f0a-31c7436a81dd" />

<br>
<br>

- `Directory traversal`이 가능.
```
recyclops list ../
```
<img width="495" height="447" alt="image" src="https://github.com/user-attachments/assets/de255342-e83a-4263-ba65-65b9a67514e7" />

<br>
<br>

- `../hubot/.env`파일에서 계정 정보 노출.
```
recyclops file ../hubot/.env
```
<img width="495" height="238" alt="image" src="https://github.com/user-attachments/assets/414ebf45-4653-4ec8-9e1e-71d2130774e9" />

<br>
<br>

- `dwight:Queenofblad3s!23`로 SSH 로그인 시도하여 성공.
<img width="1104" height="160" alt="image" src="https://github.com/user-attachments/assets/c69149b1-734c-47d0-9a65-04b6e6596d59" />

---
## Privesc
- 몇가지 권한상승 방법들을 시도해보았으나 실패하여 `linpeas`를 로컬로부터 다운로드 받아서 실행.
- `CVE-2021-3560`취약점 발견.
<img width="1104" height="51" alt="image" src="https://github.com/user-attachments/assets/5c7b93b9-216b-4514-a203-b7a4192a398e" />

<br>
<br>

- https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation
- 위 스크립트를 다운받아서 실행.
- 실패하는 경우가 생겨서 여러번 시도.
<img width="1104" height="460" alt="image" src="https://github.com/user-attachments/assets/c9657bb9-2bdc-433c-a5d0-fa1234dfd209" />

<br>
<br>

- `secnigma:secnigmaftw`로 로그인하여 `root` 권한으로 `bash` 실행.
```bash
su - secnigma

sudo bash
```
<img width="1104" height="116" alt="image" src="https://github.com/user-attachments/assets/873c09f7-15cb-43a0-a1f1-d5d09cdb7789" />

---
## FLAG
- `/home/dwight/user.txt`
<img width="1104" height="406" alt="image" src="https://github.com/user-attachments/assets/2e944841-cd00-4c32-b9b6-29f95085b88a" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="422" alt="image" src="https://github.com/user-attachments/assets/001ae34c-0401-4b95-9c14-724b25fb424c" />
