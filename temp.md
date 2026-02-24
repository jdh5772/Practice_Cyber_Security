<details>
  <summary><strong>CHECK</strong></summary>
  

- web : `whatweb`/`curl`/`burpsuite crawler`/`⭐html code⭐`/`JS code`/`페이지의 내용 먼저 확인`
- sql : `injection`/`connect`/`enumeration`/`update`
- ftp : `upload`/`download`/`hidden files`
- http : `robots.txt`/`gobuster`/`feroxbuster`/`ffuf`/`error page`/`cookie에 따라서 redirection`/`.git`/`configuration file location`/`link`/`서버 포트가 다르더라도 다른 포트에서 사용하는 언어에 맞는 web shell을 업로드 해볼 것.`/`burpsuite가 실패하는 경우가 있으니 curl로 시도해볼 것.`/`특별한 정보 찾을 수 없을 때 api 검색(login api)`/`CMS를 알아 낸 경우 다른 버전의 RCE를 참조해서 시도할 수 있는지 확인.`
- https : `tls 인증서 확인`
- windows : `powershell history`/`whoami`/`⭐inside files(password leaking,source code,configurations)⭐`/`public folder`/`로컬로 어떻게든 접속이 가능하다면 Responder를 사용하여 해시 캡쳐 시도`
- linux : `⭐inside files(password leaking,source code,configurations)⭐`/`env`/`open ports`/`⭐who have permissions(root or user?)⭐`/`/var/mail`/`login user's group files`/`Unkonwn port telnet or nc banner grabbing`/`process 확인`/`현재 로그인 유저가 아니더라도 다른 유저의 파일들을 확인해서 권한 상승`/`루트경로에서 리눅스에서 기본적으로 만들어진 폴더가 아닌 다른 폴더 확인.`
- Active Directory : `Privesc > return`/`ntpdate로 먼저 시간을 맞춰주고 시작.`/`유저가 속해 있는 그룹을 잘 살펴 보아야 한다.`/`유저목록 수집하여 소문자 버전도 시도`/`smbmap에서 제대로 정보가 확인되지 않을 수 있어 nxc로 한번 더 확인해봐야 한다.`
- SSH : `authorized_keys 변경 여부 확인`/`접속한 셸에서 명령어가 실행되지 않을 수 있으니 SSH 접속 시도.`

<br>

- nmap : `-sU --top-ports 100`/`-Pn`/`subdomain에서도 .git 확인`
- ffuf : `-mc all`/`http 인지 https 인지 제대로 확인`
- netexec : `AD가 아니더라도 윈도우 환경에서 테스트`/`⭐--rid-brute⭐`/`--users(description 확인)`/`mssql의 경우 로컬 인증 방식을 사용하기 때문에 --local-auth를 붙여서 테스트 해봐야 한다.`
- grep : `-r '@dog.htb'`
- strings : `raw data catch flag recover deleted files`
- gobuster : `txt,md`/`-k(tls)`/`경로를 못찾을 때는 feroxbuster 사용해보기.`
- dirbuster : `~2017년도까지 대체재.`
- wordlist : `.git`/`cgi-bin`
- input : `우회를 시도하여서 다른 공격방법 테스트(command injection만 우회가 되는 것이 아니다.)`/`⭐모든 공격을 전부 시도해봐야 한다.⭐`/`우선 순위를 먼저 테스트해봐야한다.`/`만약에 링크를 통해서 서버로 전송이 요청된다면 XSRF 공격이 가능할수도 있다.`
- searchsploit : `버전을 확인할 수 없어도 페이로드 시도.`
- Docker : `Docker 버전을 확인하여 Privesc 시도.`
- burpsuite : `요청시 파라미터 전송(위)과 body 전송(아래) 바꿔서 전달해보기. 이에 따라서 Content-Type을 바꿔야할 수도 있음.`/`요청시에 공백은 한줄만.`
- man page : `익숙하지 않은 프로그램의 경우 man page를 찾아볼 것.`
- code script : `입력 검증이 제대로 되고 있는지 확인.`
- process : `ps 명령어를 사용하여 어떤 프로그램이 실행되고 있는지 의심.(특히나 브라우저)`
- banner grabbing : `nc 혹은 telnet을 사용할 경우 버전 출력에 시간이 좀 걸릴 수 있다.`
- 날짜 : `날짜에 따라서 IDOR이 가능할 수 있다.`

</details>

---
<details>
  <summary><strong>Hex to ASCII</strong></summary>

```bash
# password : 504073737730726440313233212131323313917181922232526273031333435373839424349505154575861657475798283869091949598103106111114115119122123126130131134135
cat password | xxd -r -p
```
</details>

---
<details>
  <summary><strong>sed</strong></summary>

```bash
sed -i '1i /cgi-bin/' test.txt
```

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
  
</details>

---
<details>
  <summary><strong>BASH</strong></summary>

- `bash`에서는 `TRUE/FALSE` 값이 존재하지 않는다.

```bash
# 해당 경로 bashrc 내부의 환경변수,함수,명령어 등을 현재 셸에서 실행하는 명령.
. /opt/.bashrc

# 파일 크기가 0보다 큰지 검사.
[ -s log/photobomb.log ]

# 파일이 링크파일인지 검사.
[ -L log/photobomb.log ]

# 문자열 길이가 0인지 검사.
[ -z $CHECK_CONTENT ]
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
  <summary><strong>DOCKER</strong></summary>

```bash
sudo docker images

sudo docker ps -a
```
- `image` : 레시피
- `container` : 레시피로 만든 요리.

<br>

```bash
sudo docker run -it -v $(pwd):/share <image>:latest
```
- `-it` : 대화형 터미널 모드 (interactive + tty)
- `-v` : 마운트

<br>

```bash
mount
```
- 컨테이너를 실행하면 `Docker`는 `OverlayFS`를 통해서 마운트.
- `mount`포인트에서 `overlay`를 캡쳐해서 컨테이너 확인.
  
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
  <summary><strong>cross compile</strong></summary>

```bash
# 크로스 컴파일
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe

x86_64-w64-mingw32-gcc --shared -lws2_32 adduser.c -o adduser.dll
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
