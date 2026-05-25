- Swagger& OpenAPI
    
    Swagger와 OpenAPI는 REST API를 설계하고 문서화하기 위한 대표적인 기술이다.
    
    OpenAPI는 API 명세를 작성하는 표준 규격이고, Swagger는 그 명세를 시각적으로 보여주고 테스트할 수 있게 해주는 도구 모음이라고 보면 된다.
    
    장점: 
    
    - API 문서를 자동화할 수 있어 유지보수가 편함
    - 신규 개발자가 프로젝트 구조를 빠르게 이해 가능
    - 클라이언트 코드 생성 등 다양한 확장 기능 지원
    
    단점:
    
    - 명세를 직접 관리하면 중복 작업이 발생할 수 있음
    - 프로젝트 규모가 커질수록 설정이 복잡해질 수 있음
- TSOA(TypeScript-first OpenAPI) (핵심적인 부분이라 한번 더 넣었어요!)
    
    TSOA는 TypeScript 기반으로 API를 개발하면서 동시에 OpenAPI 문서를 자동 생성해주는 라이브러리이다.
    
    Express 서버에서 Controller와 DTO를 TypeScript 코드로 작성하면, 이를 기반으로 Swagger(OpenAPI) 문서를 자동으로 만들어준다.
    
    장점:
    
    - 코드와 문서의 일치성을 유지하기 쉬움
    - TypeScript 타입 안정성을 활용 가능
    - 생산성과 유지보수성이 높아짐
    - DTO 기반 구조라 협업에 유리함
    
    단점:
    
    - TypeScript 환경에 의존적임
- Type-Driven-Documentation
    
    Type-Driven Documentation은 TypeScript의 타입 정보를 기반으로 API 문서를 자동 생성하는 개발 방식이다. TSOA가 대표적인 예시 
    
    장점:
    
    - 코드 중심의 문서 자동화 방식
    - 타입 안정성과 문서화를 동시에 관리 가능
    - 유지보수 효율 증가
    - 개발 속도와 협업 효율 향상
    
    단점:
    
    - 타입 구조가 복잡하면 문서도 복잡해질 수 있음
    - TypeScript 의존성이 강함
