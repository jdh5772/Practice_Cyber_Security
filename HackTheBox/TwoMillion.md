# TwoMillion - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA twomillion 10.10.11.221
```
<img width="1104" height="267" alt="image" src="https://github.com/user-attachments/assets/3419f4a4-c79a-4237-a3e9-7085faf799ea" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `2million.htb` 추가.
<img width="1104" height="60" alt="image" src="https://github.com/user-attachments/assets/a5bc66af-5865-4831-81ad-9f1466c7ae4d" />

---
## HTTP
### banner grabbing
<img width="1104" height="336" alt="image" src="https://github.com/user-attachments/assets/ef2d5b8e-4ff3-4f4e-ab89-36e514db9dfb" />

### Command Injection
- `/invite`에서 `Invinte Code`를 사용하여 회원 가입이 진행됨.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/d9641a83-3129-4d8e-abb4-b83814a405c5" />

<br>
<br>

- `burpsuite`로 가로채서 확인해보니 `/api/v1/invite/verify`로 해당 코드가 전송.
<img width="1546" height="336" alt="image" src="https://github.com/user-attachments/assets/a986fc3a-bb21-487d-8427-78dd3cea6121" />

<br>
<br>

- `inviteapi.min.js`코드를 확인해보니 난독화가 되어 있음.
<img width="1693" height="519" alt="image" src="https://github.com/user-attachments/assets/ad3f3368-31ac-44c9-841f-ebfbb7b9afb6" />

<br>
<br>

- `eval`를 `console.log`로 교체하여 `console`탭에서 실행.
- `verifyInviteCode`함수와 `makeInviteCode`함수 발견.
<img width="1912" height="454" alt="image" src="https://github.com/user-attachments/assets/39ecf80d-31c1-4838-ab0a-e324e6cd9a7b" />

<br>
<br>

- `makeInviteCode`함수를 실행하기 위해서 `/api/v1/invite/how/to/generate`경로로 `POST`요청.
```bash
curl -X POST 'http://2million.htb/api/v1/invite/how/to/generate'
```
<img width="1104" height="111" alt="image" src="https://github.com/user-attachments/assets/a03ad0ad-ff29-4398-a519-96d872d5323b" />

<br>
<br>

- https://rot13.com/
- `data`의 내용을 `ROT13`으로 디코딩해보니 `invite code`를 생성하기 위해서 `/api/v1/invite/generate`로 `POST`요청을 시도하라고 함.
<img width="831" height="896" alt="image" src="https://github.com/user-attachments/assets/944d223c-fd75-4415-83fb-f99a7b9a7d0a" />

<br>
<br>

- `invite code` 생성.
```bash
curl -X POST 'http://2million.htb/api/v1/invite/generate'
```
<img width="1104" height="74" alt="image" src="https://github.com/user-attachments/assets/d55d6b12-d83a-4938-b9fe-e8771b5f584a" />

<br>
<br>

- 생성된 코드를 그대로 사용할시 오류가 발생.
- `base64`처럼 보여서 디코딩 진행하여 해당 코드를 입력하여 계정 생성 진행.
<img width="1104" height="74" alt="image" src="https://github.com/user-attachments/assets/51d9a597-a241-4f4f-841e-f154ad076735" />

<br>
<br>

<img width="516" height="548" alt="image" src="https://github.com/user-attachments/assets/180ff897-45f2-496e-914e-ad41e3f99d62" />

<br>
<br>

- 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/55d71a86-f12b-4007-97dc-4f71964e4388" />

<br>
<br>

- 로그인 한 상태에서 `/api/v1`경로에서 `api`에 대해서 확인.
<img width="1549" height="699" alt="image" src="https://github.com/user-attachments/assets/d2fc12bc-c134-4526-b91b-a197b534f369" />

<br>
<br>

- `/api/v1/admin/settings/update` 경로에 `PUT`요청을 실행해보니 `Content Type`이 올바르지 않다고 출력.
<img width="1549" height="288" alt="image" src="https://github.com/user-attachments/assets/b7e3d4fa-267b-4cf6-90b8-c4079b5123c3" />

<br>
<br>

- `Content-Type: application/json`을 추가해주니 `email` 파라미터가 없다고 출력.
<img width="1549" height="288" alt="image" src="https://github.com/user-attachments/assets/f056959a-ef4c-4ac5-a439-74744841f8ca" />

<br>
<br>

- `asdf@asdf.com`을 추가해주니 `is_admin`파라미터가 없다고 출력.
<img width="1549" height="349" alt="image" src="https://github.com/user-attachments/assets/06bab7b3-f39f-4b4b-9581-a9aca9a4c61c" />


<br>
<br>

- 어떤 값을 넣어야하는지 몰라 `true`를 넣어서 요청하니 0 혹은 1이 필요.
<img width="1549" height="363" alt="image" src="https://github.com/user-attachments/assets/0ab749e3-6cb5-460f-a9ad-1a446efacb87" />

<br>
<br>

- 1를 넣어서 요청해보니 완료된 것처럼 출력.
<img width="1549" height="363" alt="image" src="https://github.com/user-attachments/assets/c562d5bc-8937-4fdb-88eb-3f3d2b34a9ee" />

<br>
<br>

- `/api/v1/admin/vpn/generate`경로로 `POST`요청을 해보니 `Content Type`이 올바르지 않다고 출력.
<img width="1549" height="270" alt="image" src="https://github.com/user-attachments/assets/d52b0373-f52a-43af-95e9-4e4699739d8e" />

<br>
<br>

- `Content-Type: application/json`을 추가해보니 `username`파라미터가 필요하다고 출력.
<img width="1549" height="287" alt="image" src="https://github.com/user-attachments/assets/336f1444-8e9e-417c-89d4-d0d03d2dfe1f" />

<br>
<br>

- `asdf`를 `username`에 넣어서 요청해보니 특정 명령어가 실행되는 것처럼 출력.
<img width="1549" height="823" alt="image" src="https://github.com/user-attachments/assets/0aef2b43-2f28-49bc-8cda-019276449157" />

<br>
<br>

- ping test 시도.
<img width="772" height="366" alt="image" src="https://github.com/user-attachments/assets/4010f33c-ff37-43e7-a95a-cdf1d8c40d2c" />

<br>
<br>

- ping test 성공.
<img width="1106" height="256" alt="image" src="https://github.com/user-attachments/assets/afde4f15-56a2-4f89-870a-43fabfce9d81" />

<br>
<br>

- 리버스 셸 실행.
<img width="772" height="363" alt="image" src="https://github.com/user-attachments/assets/42d8aa1c-c854-4e7b-bd77-ca33b4f89ade" />

<br>
<br>

- 셸 획득.
<img width="1104" height="179" alt="image" src="https://github.com/user-attachments/assets/bd4a41c5-d0fb-407b-9cd1-d5f35a11233f" />

---
## Privesc
- `.env`파일에서 계정 정보 노출.
<img width="1104" height="97" alt="image" src="https://github.com/user-attachments/assets/56e47327-b8de-4ef0-8089-bdc2163ef664" />

<br>
<br>

- `admin:SuperDuperPass123` 로그인.
<img width="1104" height="136" alt="image" src="https://github.com/user-attachments/assets/856a2b5e-5568-471e-a580-6a90dd78b3d2" />

### CVE-2023-0386
- `/var/mail/admin`에서 권한 상승 힌트 발견.
<img width="1104" height="306" alt="image" src="https://github.com/user-attachments/assets/5ed3b6ab-9c14-4a2b-b7ef-41d34bb969b4" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2023-0386
- `setuid`가 설정된 실행파일이 `nosuid`마운트로부터 다른 마운트로 복사가될 때 `uid` 버그로 인해서 `root`유저로 실행이 가능해지는 취약점.
<img width="1218" height="484" alt="image" src="https://github.com/user-attachments/assets/f64cfec4-323a-4177-b42a-1b38b58fbb58" />

<br>
<br>

- https://github.com/puckiestyle/CVE-2023-0386
- 위 링크의 파일들을 압축하여 로컬로부터 다운로드 받아서 압축 해제.
<img width="1103" height="231" alt="image" src="https://github.com/user-attachments/assets/f12bf2ef-fb00-4ff3-9dc7-662719e74ab2" />

<br>
<br>

- 첫번째 셸에서 명령어 실행.
```bash
make all

./fuse ./ovlcap/lower ./gc
```
<img width="1103" height="49" alt="image" src="https://github.com/user-attachments/assets/b976fb4f-631c-45d2-91a0-718386725ee6" />

<br>
<br>

- `admin:SuperDuperPass123`로 SSH 로그인한 후 명령어 실행하여 `root` 획득.
<img width="1103" height="252" alt="image" src="https://github.com/user-attachments/assets/c94c0722-9ee1-4e36-8580-40288bcfa1ed" />

---
## FLAG
- `/home/admin/user.txt`
<img width="1103" height="231" alt="image" src="https://github.com/user-attachments/assets/cb09f659-bb98-4ee4-8add-a17f3bc68261" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="305" alt="image" src="https://github.com/user-attachments/assets/2f6af38a-780e-43d5-93d3-77e33bb3a207" />
