- OAuth 2.0(Open Authorization 2.0)
    - 정의: 나의 서비스가 유저의 비밀번호를 직접 알지 못해도, 구글/네이버/카카오 같은 대형 플랫폼(IdP)의 인증을 빌려와 유저를 안전하게 가입 및 로그인시키는 개방형 표준 프로토콜입니다.
    - 핵심 속성(주요 역할군)
        - Resource Owner(사용자): 구글 계정을 가진 유저 (예: 변지현님)
        - Client(우리 서비스): Node.js 백엔드 서버
        - Authorization Server: 구글의 인증 서버(`/oauth2/login/google` 요청을 받아 처리하는 구글 측 서버)
        - Resource Server: 구글의 유저 프로필 데이터 서버(이메일, 이름 등을 들고 있는 서버)
    - 장점
        - 유저가 회원가입 시 비밀번호를 새로 만들 필요가 없어 이탈률이 크게 감소한다.
        - 백엔드 서버가 유저의 민감한 구글 비밀번호를 저장하지 않으므로 보안 리스크가 매우 낮다.
    - 단점
        - 구글 서버가 장애를 일으키면 우리 서비스의 소셜 로그인도 함께 먹통이 된다.
        - 플랫폼마다 응답해 주는 프로필 데이터 규격이 달라서(네이버, 카카오 등) 추가될 때마다 백엔드 코드가 분기되어 복잡해진다.
            - 하지만 비즈니스적 가치가 너무나 강력하기 때문에, 대다수의 현대 서비스들은 백엔드의 복잡성을 기꺼이 감수(하고 아키텍처로 극복)하면서 소셜 로그인을 적극적으로 도입한다. → 어댑터 패턴 구조 사용
            
            ```tsx
            // 1. 우리 서비스가 사용할 규격을 딱 하나 정의해 둡니다.
            interface UnifiedUser {
              email: string;
              name: string;
              provider: "GOOGLE" | "KAKAO" | "NAVER";
            }
            
            // 2. 플랫폼별로 다르게 들어오는 데이터를 우리 규격으로 바꿔주는 '변환기(어댑터)'를 만듭니다.
            const googleAdapter = (profile: any): UnifiedUser => ({
              email: profile.emails[0].value,
              name: profile.displayName,
              provider: "GOOGLE",
            });
            
            const kakaoAdapter = (profile: any): UnifiedUser => ({
              email: profile.kakao_account.email,
              name: profile.properties.nickname,
              provider: "KAKAO",
            });
            ```
            
- JWT(JSON Web Token)
    - 정의: 인증에 필요한 정보들을 JSON 객체 안에 담은 뒤, 이를 비밀키로 암호화(서명)하여 클라이언트와 서버 간에 안전하게 주고받는 문자열 형태의 웹 토큰이다.
    - 구조적 속성: 구글 로그인이 성공했을 때 화면에 나오는 긴 문자열은 점(.)을 기준으로 세 부분으로 나뉜다.
        - Header(헤더): 토큰의 타입(JWT)과 알고리즘(HS256 등) 정보가 담김.
        - Payload(페이로드): 실제 유저 정보(id: 8, email: “…”, exp: 만료시간 등)
            - 누구나 디코딩해서 볼 수 있으므로 비밀번호 같은 민감 정보는 넣으면 안 됨!
        - Signature(서명): .env에 적은 JWT_SECRET 키를 이용해 생성한 고유 값. 토큰이 중간에 위조 되었는지 검증하는 핵심 장치.
    - 장점
        - Stateless(무상태성): 서버가 유저의 로그인 상태를 DB나 메모리에 들고 있을 필요가 없다. 서버는 토큰이 진짜인지 위조 되었는 지만 검증하면 되므로 서버 확장성에 매우 유리하다.
    - 단점
        - 한 번 발급된 토큰은 유효기간이 끝날 때까지 서버가 강제로 만료시키기 어렵다. 이를 보완하기 위해 유효기간이 짧은 Access Token과 긴 Refresh Token을 나누어 발급한다.
    
- Bearer Token
    - 정의: OAuth 2.0 및 JWT 기반 시스템에서 가장 널리 쓰이는 토큰 통량 규격이다. 직역하면 “이 토큰을 소지한(Bearer) 사람에게 권한을 부여하겠다)라는 뜻의 인증 타입이다.
    - 동작 속성
        - HTTP 요청 헤더의 Authorization 키에 값으로 Bearer <JWT 토큰 문자열> 형식을 맞춰서 보낸다.
        - 백엔드의 passport-jwt 미들웨어는 헤더를 읽어와 앞글자 Bearer를 잘라내고 뒤의 순수한 JWT 문자열만 파싱하여 검증을 진행한다.
    - 장점
        - 별도의 복잡한 암호화 프로토콜 없이 헤더에 문자열만 실어 보내면 되므로 구현이 매우 간단하고 직관적이다.
    - 단점
        - 토큰 자체가 그냥 ‘입장권’이기 때문에, 해커가 이 문자열을 중간에 탈취하기만 하면 제3자라도 아무런 제약 없이 유저의 권한을 행사할 수 있다. (따라서 Bearer 토큰을 쓸 때는 무조건 HTTPS 보안 통신이 강제된다.)