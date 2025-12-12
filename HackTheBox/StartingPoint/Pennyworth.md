# Pennyworth -HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 8080 -sC -sV -oA penny -vv 10.129.8.207
```
<img width="709" height="153" alt="image" src="https://github.com/user-attachments/assets/551e3606-1d89-40e5-a8db-b89861e16577" />

- HTTP(8080)
---
## HTTP(8080)
- banner grabbing
```bash
whatweb http://10.129.8.207:8080

curl -IL http://10.129.8.207:8080
```
<img width="1054" height="509" alt="image" src="https://github.com/user-attachments/assets/b69e4c42-c8e4-4706-8125-056f59d8738f" />

<br>
<br>

- Jenkins(2.289.1)
<img width="463" height="520" alt="image" src="https://github.com/user-attachments/assets/ea1069d0-e739-4c98-9743-fa0c13cbb5e2" />

<br>
<br>

- 로그인 계정에 대한 정보를 알 수 없어서 시도를 해보다, `root:password`로 로그인이 가능 함을 확인.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/5701c1ff-cf7e-4348-b793-9f3dd11b1080" />

---
## RCE
- https://www.hackingarticles.in/jenkins-penetration-testing
- `Manage Jenkins` -> `Script Console`
<img width="1200" height="873" alt="image" src="https://github.com/user-attachments/assets/d6c67645-6049-433a-bdea-f3c534195066" />

<br>
<br>

- https://www.revshells.com/
- `Groovy` script 생성
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/3292185a-c645-4f21-ace4-9d7577fac5d1" />

<br>
<br>

- script 실행
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/56fd5a7b-f127-4152-ac56-e6342fc90ff2" />

<br>
<br>

- shell 및 flag 획득
<img width="633" height="153" alt="image" src="https://github.com/user-attachments/assets/8bb1e755-a99a-4b3b-97ee-bc3ccb136338" />

<br>
<br>

<img width="690" height="281" alt="image" src="https://github.com/user-attachments/assets/a0d91e05-0818-4162-92db-7e936abd524e" />








