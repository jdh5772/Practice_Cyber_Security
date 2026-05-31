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
