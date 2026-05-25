- OAuth 2.0
    
    사용자 비밀번호를 직접 공유하지 않고도, 외부 애플리케이션이 사용자 정보에 제한적으로 접근할 수 있도록 해주는 인증 및 권한 부여 프레임워크
    
    OAuth 2.0에서는 Google과 같은 인증 제공자(Provider)가 사용자를 대신 인증해줌
    
    구성요소:
    
    - Resource Owner: 사용자
    - Client: 서비스를 제공하는 애플리케이션
    - Authorization Server: 인증을 담당하는 서버 (Google 등)
    - Resource Server: 사용자 정보를 제공하는 서버
- JWT
    
    사용자 인증 정보를 JSON 형태로 저장하여 안전하게 전달하기 위한 토큰 기반 인증 방식으로 서버는 로그인 성공 시 JWT를 발급하고, 이후 사용자는 매 요청마다 JWT를 함께 전송하여 자신의 인증 상태를 증명함
    
    구조:
    
    - Header: 토큰 타입, 암호화 알고리즘 정보
    - Payload: 사용자 ID, 이메일 등의 데이터
    - Signature: 위변조 방지를 위한 서명값
- Bearer Token
    
    Bearer Token은 HTTP 요청 헤더에 담아 보내는 인증 토큰 방식으로 주로 JWT와 함께 사용되며, 서버는 해당 토큰을 검증하여 사용자를 인증함
    
    Authorization: Bearer {TOKEN} 이런식으로 전송됨