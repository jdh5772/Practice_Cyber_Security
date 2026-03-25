# SwagShop - HackTheBox
- [Recon](#recon)
- [HTTP](#http)
- [Privesc](#privesc)
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA swagshop 10.129.229.138
```
<img width="1203" height="490" alt="image" src="https://github.com/user-attachments/assets/82f60448-0772-4513-9292-65f5ffaca12c" />

<br>
<br>

- SSH
- HTTP

---
## HTTP
- `/etc/hosts`에 `swagshop.htb` 추가.
<img width="1203" height="76" alt="image" src="https://github.com/user-attachments/assets/a751c789-270f-4c9e-806e-8c17999c1686" />

<br>
<br>

- `Magento`가 실행 중.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/71ddc3c8-732d-4f1f-9d54-83ef6fa7e260" />

<br>
<br>

- https://github.com/steverobbins/magescan
- `magescan`을 사용하여 1.9.0.0 버전 확인
```bash
php magescan.phar scan:all swagshop.htb
```
<img width="1203" height="350" alt="image" src="https://github.com/user-attachments/assets/4dfb3d60-d57d-4d81-a498-505504e8078f" />

<br>
<br>

- 서버에서 특별한 정보를 찾을 수 없어 `searchsploit`을 사용.
- 인증이 필요한 RCE 취약점과 또 다른 RCE 발견.
<img width="1203" height="75" alt="image" src="https://github.com/user-attachments/assets/5c3e5112-2662-4cd8-8666-bbd358733eee" />

<br>
<br>

- 버전이 적혀 있지 않은 취약점을 확인.
- RCE라기보다는 SQL Injection을 사용하여 관리자 권한의 계정을 생성하는것으로 보여짐.
```bash
searchsploit -m 37977
```
<img width="1203" height="583" alt="image" src="https://github.com/user-attachments/assets/afce9ab6-49f6-457c-a2bc-90877accc6bf" />

<br>
<br>

- `/admin/Cms_Wysiwyg/directive/index/`로 접속이 가능한지 시도해보았을 때 실패.
<img width="574" height="183" alt="image" src="https://github.com/user-attachments/assets/abe7b58f-5a0a-4e31-8a04-ca36e910ed7c" />

<br>
<br>

- 다른 경로를 접속하였을 때 `/index.php`로 시작하는 것으로 보임.
<img width="1234" height="437" alt="image" src="https://github.com/user-attachments/assets/93d86711-e359-433b-a112-5d25a10f35fa" />

<br>
<br>

- `/index.php/admin/Cms_Wysiwyg/directive/index/`로 접속하니 해당 경로로 직접 연결은 불가능 하지만 `/admin`으로 연결되는 것으로 확인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/dc7fd4c7-37d0-48a7-91c2-1521f4cc5db4" />

<br>
<br>

- 스크립트의 target을 변경하여 실행.
```bash
python2 37977.py
```
<img width="1204" height="48" alt="image" src="https://github.com/user-attachments/assets/efbf3919-1999-4b16-8d89-d690398de0ae" />
<img width="1204" height="103" alt="image" src="https://github.com/user-attachments/assets/4506be23-2563-460e-86f3-045295570ebe" />

<br>
<br>

- `forme:forme`로 관리자 페이지 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/11ee4e5c-353c-4ce4-9a62-d038555a5874" />

<br>
<br>

- https://github.com/0xBruno/MagentoCE_1.9.0.1_Authenticated_RCE
- https://websec.wordpress.com/2014/12/08/magento-1-9-0-1-poi/
- POC를 확인해보면 설치날짜를 알 수 있게 되면 PHP object injection을 통해서 코드를 실행할 수 있게 됨.
<img width="1282" height="685" alt="image" src="https://github.com/user-attachments/assets/fd138c9d-1035-4faf-9698-76825426ab3a" />

<br>
<br>

- `http://swagshop.htb/app/etc/local.xml` 경로에서 날짜 획득.
<img width="755" height="562" alt="image" src="https://github.com/user-attachments/assets/ebdf9b5c-7c2b-4812-a8c0-5095d814ed25" />

<br>
<br>

- 날짜와 계정 정보를 변경.
<img width="1203" height="155" alt="image" src="https://github.com/user-attachments/assets/5c27e774-6773-4247-b7f7-a4cb8589488f" />

<br>
<br>

- 스크립트를 실행하여 명령어 실행 성공.
```bash
python3 exploit.py http://swagshop.htb/index.php/admin
```
<img width="1203" height="440" alt="image" src="https://github.com/user-attachments/assets/85b10e16-600c-44a1-890e-f6a34d3ceef5" />

<br>
<br>

- `ex.sh` 생성.
<img width="1203" height="113" alt="image" src="https://github.com/user-attachments/assets/350fd087-41f0-49af-90da-db5de9f4146b" />

<br>
<br>

- 로컬로부터 다운로드.
<img width="1203" height="49" alt="image" src="https://github.com/user-attachments/assets/37128af1-d90b-40c7-a147-25e983f9ef23" />

<br>
<br>

- 리버스 셸 실행하여 셸 획득.
<img width="1203" height="219" alt="image" src="https://github.com/user-attachments/assets/f4e1854a-e1b3-483d-84e0-8af9fd85954f" />

---
## Privesc
- `/usr/bin/vi /var/www/html/*`를 `root`권한으로 실행 가능.
<img width="1203" height="145" alt="image" src="https://github.com/user-attachments/assets/60e59ca6-f219-4a40-9ac2-bf49701a7909" />

<br>
<br>

- https://gtfobins.org/gtfobins/vi/
- `GTFObin`을 참조하여 `root` 셸 획득.
```bash
sudo /usr/bin/vi /var/www/html/* -c ':!/bin/sh' /dev/null
```
<img width="1203" height="124" alt="image" src="https://github.com/user-attachments/assets/a0b92951-58e8-47f7-8895-ef642faf05da" />

---
## FLAG
- `/home/haris/user.txt`
<img width="1203" height="313" alt="image" src="https://github.com/user-attachments/assets/9febd705-41cd-4ded-88f1-8e02f3305cf7" />

<br>
<br>

- `/root/root.txt`
<img width="1203" height="268" alt="image" src="https://github.com/user-attachments/assets/f7e51c85-2cac-4843-bd86-9c952e2b7719" />
