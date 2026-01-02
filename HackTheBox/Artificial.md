# Artificial - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA artificial 10.10.11.74
```
<img width="1104" height="421" alt="image" src="https://github.com/user-attachments/assets/b2ff1134-490a-4dca-9714-57347e310eae" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `artificial.htb`추가.
<img width="1104" height="67" alt="image" src="https://github.com/user-attachments/assets/64ced308-f657-4cd3-8d77-17bcdc515c6a" />

### banner grabbing
<img width="1104" height="259" alt="image" src="https://github.com/user-attachments/assets/276f35f6-f361-412e-a6a5-ca129a5b2c2f" />

### CVE-2024-3660
- 메인 페이지에서 계정을 생성하여 로그인하면 `Dockerfile`을 다운받을 수 있다.
- `tensorflow 2.13.1`
<img width="1103" height="293" alt="image" src="https://github.com/user-attachments/assets/a7223ee4-0687-4f8d-ad0d-c4d5c24c2e4f" />

<br>
<br>

- 업로드할 수 있는 파일확장자가 `h5`로 되어있다.
- `tensorflow 2.13.1`으로 `h5`파일을 생성하여 업로드하여 해당 서버가 실행하도록 할 수 있는지 가정.
<img width="1100" height="620" alt="image" src="https://github.com/user-attachments/assets/cea07485-0d63-4886-90cf-065b60247149" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2024-3660
<img width="1234" height="414" alt="image" src="https://github.com/user-attachments/assets/dbeb7cda-6e49-4cc3-8b51-8d4e5bbd657b" />

<br>
<br>

- https://github.com/aaryanbhujang/CVE-2024-3660-PoC
- 위 링크의 `Dockerfile`에서 같은 `tensorflow`버전을 사용하기에 위 `Dockerfile`에서 ping test 코드로 수정.
<img width="1104" height="98" alt="image" src="https://github.com/user-attachments/assets/1e04a16c-9e1a-414b-8da8-abd201ac61ec" />

<br>
<br>

- `Docker`를 빌드해서 생성한 `h5`파일 복사.
```bash
sudo docker buildx build -t tfimg .

container_id=$(sudo docker create tfimg)

sudo docker cp $container_id:/CVE20243660/CVE20243660.h5 ./CVE20243660.h5 && docker rm $container_id
```
<img width="1104" height="600" alt="image" src="https://github.com/user-attachments/assets/69c3e94e-cb14-470c-9a48-973bde0a7ab8" />

<br>
<br>

- `h5`파일 업로드하여 `View Predictions` 클릭.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/1c62f4d9-1b05-43ee-b7c0-fa265f12e465" />

<br>
<br>

- ping test 성공.
<img width="1104" height="238" alt="image" src="https://github.com/user-attachments/assets/044785ed-e11f-4083-bd4b-1ab576ef8791" />

<br>
<br>

- `Dockerfile`에서 리버스 셸 코드로 수정.
<img width="1104" height="104" alt="image" src="https://github.com/user-attachments/assets/7062c9dc-1318-4969-b0f1-f153b5801d2a" />

<br>
<br>

- 같은 방식으로 `h5`파일을 업로드한 후 실행하여 셸 획득.
<img width="1104" height="182" alt="image" src="https://github.com/user-attachments/assets/67d40b54-363d-4b2e-9424-ec03e44d055e" />

---
## Privesc
- `/home/app/app/instance`에서 `users.db`발견.
<img width="1104" height="102" alt="image" src="https://github.com/user-attachments/assets/49e0c808-fe2e-4bdc-b8dc-7df3946faef5" />

<br>
<br>

- 저장된 계정 발견.
```bash
sqlite3 users.db 'select username,password from user;'
```
<img width="1104" height="134" alt="image" src="https://github.com/user-attachments/assets/d6146786-5df2-4833-b484-04d06df3900d" />

<br>
<br>

- `/home/app/app/app.py`에서 비밀번호가 `md5`로 해싱되어 있는 것 확인.
<img width="1104" height="85" alt="image" src="https://github.com/user-attachments/assets/1c77bdfd-899c-49c6-8518-e6b615613273" />

<br>
<br>

- `hashcat`으로 크래킹.
<img width="1104" height="107" alt="image" src="https://github.com/user-attachments/assets/ccd624e3-6327-49b4-99de-82da65022f5c" />

<br>
<br>

- `gael:mattp005numbertwo` 로그인.
<img width="1104" height="79" alt="image" src="https://github.com/user-attachments/assets/3dceab88-d24b-4c17-a958-93f689a74ff6" />

<br>
<br>

- SSH 로그인.(gael:mattp005numbertwo)
<img width="1104" height="42" alt="image" src="https://github.com/user-attachments/assets/554322a0-1fb5-4a60-ade4-01a3b1978185" />

<br>
<br>

- `gael`유저가 `sysadm`그룹에 속해있어 해당 그룹의 파일 확인.
<img width="1104" height="82" alt="image" src="https://github.com/user-attachments/assets/ab1108e6-cdff-4d27-b7f1-e9ab80c58132" />

<br>
<br>

- `backrest_backup.tar.gz`파일을 로컬로 옮겨서 압축 해제.
- `/backrest/.config/backrest/config.json`에서 로그인 계정 발견.
<img width="1104" height="318" alt="image" src="https://github.com/user-attachments/assets/24f688e5-8b4f-42ad-b9eb-6fe853f5674e" />

<br>
<br>

- `Bcrypt`로 해싱이 되어있다하나 형식이 달라서 `base64`로 디코딩해보니 올바른 형식이 출력.
```bash
echo 'JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP' |base64 -d
```
<img width="1104" height="70" alt="image" src="https://github.com/user-attachments/assets/ade8dca1-e9b3-44e5-a03a-ec04fb8caef6" />

<br>
<br>

- `john`으로 크래킹.
```bash
john hash2 --wordlist=~/util/rockyou.txt

john hash2 --show
```
<img width="1104" height="70" alt="image" src="https://github.com/user-attachments/assets/09855a20-ec36-4bd3-8d04-4e6a4dfa7a4c" />

<br>
<br>

- 9898번포트가 열려있는 것을 확인.
```bash
ss -tnlp

curl -v http://127.0.0.1:9898
```
<img width="1104" height="498" alt="image" src="https://github.com/user-attachments/assets/e16c7d88-c742-4a85-8556-53ae9bcfe9ee" />

<br>
<br>

- SSH 9898번 포트포워딩하여 로컬에서 접속.
- `backrest_root:!@#$%^`로 로그인시도하여 성공.
```bash
ssh -L 9898:localhost:9898 gael@10.10.11.74
```
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/fcbd3b53-63d4-41d4-894b-813c8d8d4309" />

<br>
<br>

- 아래와 같이 입력을 해 Repository를 만든 후, Command execution이 되는지 확인.
<img width="1104" height="280" alt="image" src="https://github.com/user-attachments/assets/f7331558-bd5b-4d28-abe5-292070ccd8ee" />
<img width="1104" height="311" alt="image" src="https://github.com/user-attachments/assets/08fcca85-2fae-4463-b76f-9968a79af85e" />


<br>
<br>

- `Check Now`를 클릭하여 Command execution 성공.
<img width="1104" height="107" alt="image" src="https://github.com/user-attachments/assets/b618b162-7c35-4e2b-b3be-6a2f78e52d73" />

<br>
<br>

- 리버스 셸 명령어 실행.
<img width="1104" height="324" alt="image" src="https://github.com/user-attachments/assets/043f9567-33bd-4a8d-bdef-64b23b3f181c" />

<br>
<br>

- 셸 획득.
<img width="1104" height="186" alt="image" src="https://github.com/user-attachments/assets/2caafce4-e08a-4353-8646-1db73994ac7d" />

---
## FLAG
- `/home/gael/user.txt`
<img width="1104" height="269" alt="image" src="https://github.com/user-attachments/assets/0cba4b69-edbd-4129-b1a6-d93e145e2c04" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="271" alt="image" src="https://github.com/user-attachments/assets/3282a445-97ff-43de-bd68-21f23f3df8af" />



























