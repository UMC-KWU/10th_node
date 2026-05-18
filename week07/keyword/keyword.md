- 미들웨어
    - 요청(Request)과 응답(Response) 사이에서 실행되는 중간 처리 함수
    - 공통 기능을 한 곳에서 처리할 수 있음
    - 로그 출력, 인증 처리, JSON 변환, 쿠키 처리 등에 사용됨
    - 대표 예시:
        - `express.json()`
        - `cookie-parser`
        - `morgan`
- HTTP 상태 코드
    - 서버가 요청 결과를 숫자로 알려주는 코드
    - 클라이언트가 요청 성공 여부를 쉽게 판단할 수 있음
    
    대표 예시:
    
    - `200 OK` : 요청 성공
    - `201 Created` : 데이터 생성 성공
    - `400 Bad Request` : 잘못된 요청
    - `401 Unauthorized` : 인증 필요
    - `404 Not Found` : 데이터 없음
    - `409 Conflict` : 중복 데이터 충돌
    - `500 Internal Server Error` : 서버 내부 오류
- 에러 핸들링(Error Handling)
    - 서버에서 발생한 오류를 일정한 형식으로 처리하는 방식
    - 단순히 서버를 종료시키는 것이 아니라 오류 원인을 응답으로 전달함
    - 이번 실습에서는 `AppError` 기반 커스텀 에러 클래스를 사용하였다.
- Tsoa
    - TypeScript 기반 API 라우트 및 Swagger 문서를 자동 생성해주는 도구
    - 데코레이터를 사용해 API를 정의할 수 있음
    
    대표 문법:
    
    - `@Route("users")`
    → API의 기본 경로를 설정함
    - `@Get("mypage")`
    → GET 요청 API를 생성함
    - `@Post("signup")`
    → POST 요청 API를 생성함
    - `@Body()`
    → 요청 body 데이터를 받아옴
    - `@Query()`
    → Query String 값을 받아옴
    - 기존 Express보다 코드 구조를 정리하기 쉽고, Swagger 문서도 자동 생성된다는 장점이 있다.
