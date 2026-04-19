- 172.17.0.0/16 ~ 172.31.0.0/16 대역에 속하는 경우가 많다.


```bash
docker compose up -d

docker compose ps

docker compose exec -it gitea
```


```bash
sudo docker images

sudo docker ps -a
```
- `image` : 레시피
- `container` : 레시피로 만든 요리.

<br>

```bash
sudo docker run -it -v $(pwd):/share <image>:latest
```
- `-it` : 대화형 터미널 모드 (interactive + tty)
- `-v` : 마운트

<br>

```bash
mount
```
- 컨테이너를 실행하면 `Docker`는 `OverlayFS`를 통해서 마운트.
- `mount`포인트에서 `overlay`를 캡쳐해서 컨테이너 확인.

<br>

```bash
sudo docker exec -it --privileged --user root <container id> bash

mount /dev/sda1 /mnt/
```
- exec로 실행할 경우 `privileged`옵션과 `user`옵션을 주어 루트 권한으로 획득.
- 이후 호스트의 디스크(/dev/sda1)를 /mnt/에 마운트하여 컨테이너 밖 호스트 파일시스템에 접근 가능.

<br>

```bash
docker run --name limesurvey-mysql -e MYSQL_ROOT_PASSWORD=rootpass123 -e MYSQL_DATABASE=limesurvey -e MYSQL_USER=limeuser -e MYSQL_PASSWORD=limepassword -p 3306:3306 -d mysql:latest
```
- `-d` : `mysql`이미지를 최신 버전으로 실행
- `-p` : 호스트의 포트를 컨테이너에 연결
