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

## LLMNR/NBT-NS Poisoning
- DNS 실패 시 브로드캐스트로 물어보는 특성을 이용해서 중간에서 가짜 응답을 날리는 공격
- NTLMv2 해시 획득이 크랙 or NTLM Relay 공격으로 이어짐

### 예방법
- LLMNR/NBT-NS 완전 비활성화 (GPO로 설정)
- SMB 서명 강제 적용 (Relay 공격 방어)
- 네트워크 접근 제어 (NAC) : 물리 장비 혹은 소프트웨어로 해당 네트워크에 접속하기 위한 인증서를 요구하는 시스템 구축.
