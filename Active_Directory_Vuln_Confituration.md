<details>
  <summary><strong>AD CS</strong></summary>

- https://raulhwang.tistory.com/27
- https://raulhwang.tistory.com/32
- https://raulhwang.tistory.com/33

</details>

---
<details>
  <summary><strong>WSUS</strong></summary>

- https://blog.naver.com/chance0432/221857790015
- https://blog.naver.com/chance0432/221859882241
- https://blog.naver.com/chance0432/221861859036
- https://blog.false.kr/posts/Computing/OS/Windows/Windows-Server-Update-Services-Client-Set-up.html
- `https`로 설정해주면 `8531`로 포트도 변경해줘야 한다.
  
</details>

---
<details>
  <summary><strong>ESC17</strong></summary>

## CA Server
```
1. certtmpl.msc 실행
2. 인증서 템플릿 > 웹서버 템플릿 복제(서버인증 EKU만 포함)
3. 주체 이름(Subject Name) > 요청에서 제공 선택
4. 확장 > 응용 프로그램 정책 > 서버 인증 확인
5. 발급 요구 사항 > CA 인증서 관리자 승인 해제
6. 보안 > 읽기, 등록 허용
7. certsrv.msc 실행
8. 인증서 템플릿 > 새로만들기 > 발급할 인증서 템플릿 선택
```

- 구성이 된 후 certipy가 인식하는데 시간이 좀 걸릴 수 있다.
- 클라이언트가 WSUS를 HTTPS로 설정했을 때 wsuks를 통해서 통신하여 성공함.
- 수동으로 업데이트를 진행했을 때는 다운로드 후 클라이언트가 실행까지 해줘야 공격이 성공했음.
- 아마 일정시간마다 자동으로 업데이트를 다운로드 후 설치하는 것이 정책으로 걸려있을 것 같다.
- HTTPS가 되는데, HTTP가 안될까?
  
</details>

