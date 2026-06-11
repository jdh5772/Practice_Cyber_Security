## fping
- 단순히 호스트가 살아있는지만 확인하는데 있어서 `nmap`보다 ping만으로 확인하는 `fping`이 나을 수 있다.
```bash
fping -asgq 172.16.5.0/23
#      -a  : alive 호스트만 출력
#      -s  : 마지막에 통계 요약 출력
#      -g  : CIDR 입력 시 IP 목록 자동 생성
#      -q  : 개별 ICMP 결과 숨김 (조용히)
```

## Nmap
```bash
sudo nmap -v -A -iL hosts.txt -oA /home/htb-student/Documents/host-enum
```

## Kerbrute
```bash
sudo git clone https://github.com/ropnop/kerbrute.git

make help

sudo make all

kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```
