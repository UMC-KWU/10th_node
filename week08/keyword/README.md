- Swagger& OpenAPI
    
    OpenAPI는 REST API의 구조를 표준 형식으로 정의하기 위한 명세(Specification)이며, Swagger는 이러한 OpenAPI 명세를 기반으로 API 문서를 시각화하고 테스트할 수 있도록 도와주는 도구
    
    주요 기능:
    
    - API 요청/응답 구조 문서화
    - Swagger UI를 통한 API 테스트
    - 프론트엔드와 백엔드 간 명세 공유
    - API 유지보수 및 협업 효율 향상
    
    장점:
    
    - API 구조를 한눈에 파악 가능
    - 협업 시 API 명세 공유가 쉬움
    - 테스트 환경을 따로 만들지 않아도 됨
    - 문서와 실제 코드의 일관성을 유지하기 쉬움
- TSOA(TypeScript-first OpenAPI) (핵심적인 부분이라 한번 더 넣었어요!)
    
    TSOA는 TypeScript 기반으로 OpenAPI 문서를 자동 생성해주는 라이브러리
    TypeScript 코드와 Swagger(OpenAPI) 문서를 연결해주는 역할을 수행함
    
    기존에는 API 문서를 직접 작성해야 했지만, TSOA를 사용하면 TypeScript 타입과 데코레이터를 기반으로 자동으로 OpenAPI 문서를 생성할 수 있음
    
    주요 데코레이터:
    
    - `@Route`: API 기본 경로 설정
    - `@Get`, `@Post`: HTTP 메소드 정의
    - `@Body`, `@Path`, `@Query`: 요청 데이터 정의
    - `@SuccessResponse`: 성공 응답 정의
    - `@Tags`: Swagger 그룹화
- Type-Driven-Documentation
    
    Type-Driven-Documentation은 TypeScript의 타입 정보를 기반으로 API 문서를 자동 생성하고 유지하는 방식으로 타입 자체가 문서 역할을 수행하는 방식이라고 볼 수 있음
    
    예를 들어 Request DTO와 Response DTO를 TypeScript 인터페이스로 정의하면, 해당 타입 정보를 기반으로 API 요청 형식과 응답 구조가 자동으로 문서화된다.