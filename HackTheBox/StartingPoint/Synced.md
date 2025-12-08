# Synced - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 873 -sC -sV -vv -oA synced 10.129.228.37
```
<img width="1102" height="67" alt="image" src="https://github.com/user-attachments/assets/1dc230c4-6f25-429c-ae70-bad683d19727" />

- rsync(873)

## rsync(remote sync)
- unix 및 linux 시스템에서 파일을 효율적으로 전송하고 동기화 하기 위한 유틸리티
- nmap에서 공유된 폴더를 찾을 수 없었음.
- rsync를 통해서 탐색을 할 수 있는지 시도.
```bash
rsync -av --list-only rsync://10.129.228.37
```
<img width="1102" height="67" alt="image" src="https://github.com/user-attachments/assets/09737868-f42b-4274-87b8-b17a98f3a3a7" />

- 공유폴더(public, Anonymous Share)를 발견했고, public폴더를 추가적으로 탐색.
- `flag.txt` 발견
```bash
rsync -av --list-only rsync://10.129.228.37/public
```
<img width="1102" height="117" alt="image" src="https://github.com/user-attachments/assets/3d530c28-f0d6-4199-9cc4-b6a224b94482" />

- public 폴더를 local에 복사하여 flag 획득
```bash
rsync -av rsync://10.129.228.37/public ./rsync
```
<img width="1102" height="440" alt="image" src="https://github.com/user-attachments/assets/60326b9f-d6c4-4041-a1b4-edda9afc78a4" />

