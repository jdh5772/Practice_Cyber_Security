## SUID / CAP_*
- SUID / Capabilities / CAP_SETUID
- SUID : 파일 소유자의 권한으로 실행되게 해주는 속성. python3를 포함한 인터프리터에 걸더라도 보안상 커널에서 무시함.
- CAP_* : root나 SUID를 주기엔 권한이 너무 많으니, 필요한 권한만 딱 잘라서 부여할 때 사용. (최소 권한 원칙)

### CAP_SETUID
- CAP_SETUID : 프로세스가 실행 중에 스스로 UID를 변경할 수 있는 권한.
- SUID가 "실행 시 자동으로 소유자 권한"이라면, CAP_SETUID는 "실행 중에 능동적으로 root로 전환 가능".
- 결과적으로 root 탈취가 가능하므로 사실상 root와 동일한 위험도를 가짐.

---
## MOUNT
- 파일을 복사하는 게 아니라, 어딘가에 있는 저장소를 내 파일 시스템의 특정 경로로 접근할 수 있게 연결하는 개념(USB를 꽂으면 파일을 사용할 수 있는 것)
- 권한 설정에 따라서 마운트 된 파일에 수정작업이 가능할 수 있다.

---
## UDISKS
- `udisks2` : 일반 사용자가 디스크/저장장치를 다룰 수 있게 해주는 서비스
- 마운트의 경우 원래 관리자 권한으로 실행되어야 하나 `udisks2`가 이를 관리자 권한으로 대신 처리해준다.
- 권한 허용 여부는 Polkit(pkcheck)이 판단한다.

### PAM(Pluggable Authentication Modules)
- 리눅스의 인증 처리 시스템 (기본 내장)
- SSH, sudo, su 등 모든 로그인/인증 과정에 관여
- 모듈 방식이라 필요한 인증 단계를 추가/제거 가능
- ~/.pam_environment : 로그인 시 PAM이 읽는 환경변수 설정 파일

### Polkit
- 특정 작업의 허용 여부를 판단하는 권한 관리 시스템
- PAM이 "누구인가"를 담당한다면, Polkit은 "무엇을 할 수 있는가"를 담당
- pkcheck : Polkit에 권한 여부를 조회하는 명령어 도구

### CVE-2025-6018
- 해당 취약점으로 인해서 사용자가 직접 `~/.pam_environment`을 수정하고, PAM이 해당 파일을 검증 없이 사용함으로써 `allow_active`권한을 부여할 수 있게 됨.
- `allow_active` : 실제로 키보드와 마우스 앞에 앉아 있는 사람이라고 생각하는 권한.
```bash
# allow_active 확인
pkcheck --action-id org.freedesktop.udisks2.loop-setup --process $$ && echo OK
```
```
# ~/.pam_environment
XDG_SEAT OVERRIDE=seat0
XDG_VTNR OVERRIDE=1
```

### CVE-2025-6019
- https://github.com/MaxKappa/opensuse-leap-privesc-exploit
- `udisks`의 `resize`를 호출하기 위해서 `allow_active` 권한이 필요.(CVE-2025-6018 필요)
- 일반 마운트 → udisks가 nosuid 플래그를 붙임 → SUID 비트 무효화 → 안전
- resize용 임시 마운트 → libblockdev가 플래그를 빠뜨림 → SUID 비트 그대로 살아있음 → 공격자가 심어둔 SUID-root 바이너리 실행 가능
