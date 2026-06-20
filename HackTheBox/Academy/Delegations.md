## Delegation
- 특정 서비스(A)를 이용하려면 다른 서비스(B)와도 상호작용이 필요한 경우가 있다. 원래대로라면 유저가 B와도 직접 인증/통신을 해야 한다.
- Delegation은 유저가 B와 직접 상호작용하는 대신, A가 유저의 권한(identity)을 대리하여 B에 접근하는 것을 말한다.
- 이때 B 입장에서는 A가 아니라 "유저 본인"이 요청한 것처럼 보인다.

### Unconstrained Delegation
- 유저가 서비스에 인증하면 그 유저의 TGT 복사본이 서비스에 저장되고, 서비스는 이 TGT로 어떤 서비스든 ST를 요청해 유저 행세를 할 수 있다.

### Constrained Delegation
- 서비스(A)가 KDC에 User가 서비스(A)에 접근할 때 썼던 티켓을 그대로 `additional tickets`로 첨부 제출하면서, "이 티켓 속 유저(cname) 자격으로 서비스(B)용 새 ST를 발급해달라"고 요청하는 것(S4U2Proxy)
- `cname-in-addl-tkt` 플래그를 통해 KDC는 서비스가 아니라 첨부 티켓 속 cname(User) 기준으로 발급 대상을 판단
- 단, 서비스(A)의 `msDS-AllowedToDelegateTo` 속성에 서비스(B)가 명시되어 있어야만 발급 가능

### Resource-Based Constrained Delegation
- 위임 가능 여부를 자원을 가진 쪽(B)이 자기 객체 속성(msDS-AllowedToActOnBehalfOfOtherIdentity)에 "A를 신뢰한다"고 설정하는 방식,
- Domain Admin 권한 없이도 B에 대한 쓰기 권한만 있으면 수정 가능하다.
