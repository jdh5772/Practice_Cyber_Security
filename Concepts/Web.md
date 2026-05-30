# Apache / Apache Tomcat
- 정적 파일 서빙만 필요하다면 Apache/Nginx만으로 충분.
- 동적 코드 실행이 필요할 때 언어별 실행 환경이 추가로 필요.
  - Java → Tomcat
  - Python → gunicorn
  - PHP → php-fpm (Apache 내장 가능)
