# Apache / Apache Tomcat
- 정적 파일 서빙만 필요하다면 Apache/Nginx만으로 충분.
- 동적 코드 실행이 필요할 때 언어별 실행 환경이 추가로 필요.
  - Java → Tomcat
  - Python → gunicorn
  - PHP → php-fpm (Apache 내장 가능)

---
## SSRF(Server-Side Request Forgery)
- 서버가 공격자가 지정한 내부/외부 URL로 요청을 보내도록 유도하는 공격.
- 서버가 외부 리소스를 가져오는 기능을 악용합니다.
- `nc`를 사용하여 서버에서 보내오는 헤더를 먼저 분석.
```bash
nc -vnlp 80
```
- [CURL SSRF](https://github.com/jdh5772/Practice_Cyber_Security/blob/main/09-Exploitation.md#curl-ssrf)
---
## XSS(Cross Site Scripting)
- 신뢰되지 않은 외부 데이터가 검증 없이 HTML에 삽입되어, 브라우저가 이를 실행하는 공격
- 개발자가 HTTP 요청 헤더를 로그나 디버그 목적으로 이스케이프 없이 브라우저에 렌더링한다면 XSS 공격이 가능해진다.
- 입력란에 대해서 검증 없이 DB에 저장이 된다면 해당 DB가 사용되는 페이지에서 XSS가 발생.

### 입력 경로
- 입력란만이 XSS의 경로가 아니다. 서버가 HTML에 반영하는 **모든 외부 데이터**가 대상이다.

| 경로 | 예시 |
|---|---|
| 입력란 (폼) | 게시판 댓글, 검색창 |
| URL 파라미터 | `?search=<script>...` |
| HTTP 헤더 | `Referer`, `User-Agent`, `X-Forwarded-For` |
| 쿠키 값 | 서버가 쿠키 내용을 페이지에 출력할 때 |
| 파일명 | 업로드 파일명을 HTML에 표시할 때 |
| DB에서 읽어온 값 | 외부에서 이미 오염된 데이터 |

### 예방
| 방법 | 설명 |
|---|---|
| 화이트리스트 검증 | 허용된 값만 통과, 나머지 차단 |
| HTML 이스케이프 | `<` → `&lt;`, `>` → `&gt;` 등으로 치환 |
| CSP 헤더 | `Content-Security-Policy`로 허용 스크립트 출처 제한 |
| HttpOnly 쿠키 | JS에서 쿠키 접근 차단 |

---
## XSLT(eXtensible Stylesheet Language Transformations)
- XML 문서를 다른 형식(HTML, 다른 XML, 텍스트 등)으로 변환하기 위한 언어

### 동작원리
```
XML 문서
    +
XSLT 스타일시트  →  XSLT 프로세서  →  출력 결과 (HTML/XML/Text)
    +
XPath 표현식
```

### XSLT Injection
- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSLT%20Injection/README.md
```
# 서버에서 사용자 입력을 XSLT에 직접 삽입하는 경우
<?xml version="1.0" encoding="UTF-8"?>
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
<body>
<br />Version: <xsl:value-of select="system-property('xsl:version')" />
<br />Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br />Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
</body>
</html>
```

### 예방법
```
✅ 입력값 검증 및 이스케이핑
   → < > & ' " 등 특수문자 이스케이프 처리

✅ 사용자 입력을 XSLT에 직접 삽입 금지
   → XPath 파라미터로 안전하게 전달

✅ XSLT 프로세서 보안 설정
   → 외부 엔티티, document() 함수 비활성화
   → 확장 기능(Java/C# 스크립트) 비활성화

✅ 최소 권한 원칙
   → XSLT 프로세서가 실행되는 프로세스 권한 최소화

✅ 화이트리스트 방식으로 허용된 XSLT만 사용
```
