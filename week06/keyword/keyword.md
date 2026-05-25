- ORM
    
    ORM(Object-Realtional Mapping)
    
    - 객체와 관계형 데이터베이스의 테이블을 자동으로 연결해주는 기술이다. 개발자가 복잡한  SQL 쿼리문을 직접 짜지 않고, 자바스크립트나 타입스크립트 코드로 데이터베이를 다룰 수 있게 해준다.
        
        ```tsx
         // 💻 SQL을 직접 쓸 때 (Raw Query)
        SELECT * FROM user WHERE id = 1;
        
        // 🚀 ORM(Prisma)을 쓸 때
        prisma.user.findUnique({ where: { id: 1 } });
        ```
        
- Prisma 문서 살펴보기
    
    Prisma는 Node.js와 TS 생태계에서 가장 핫한 차세대 ORM이다. 기존 ORM들과 다르게 schema.prisma라는 직관적인 설계도를 중심으로 작동한다.
    
    - ex. Prisma의 Connection Pool 관리 방법
        - Connection Pool: 서버가 DB에 접근할 때마다 매번 문을 열고 닫으면 너무 느리기 때문에 미리 연결 통로를 여러 개 만들어두고 돌려쓰는 것
            - 자동 최적화: 프리즈마는 DATABASE_URL을 보고 CPU 코어 수 등에 맞춰 대략 (코어 수 * 2) + 1개의 연결(기본값)을 자동으로 만들어준다.
            - 수동 조절: 만약 동시 접속자가 많아서 pool timeout 에러가 난다면, 연결 주소 뒤에 옵션을 붙여서 통로 개수를 늘릴 수 있다.
            - `DATABASE_URL="mysql://root:pw@localhost:3306/db?connection_limit=20”`
    - ex. Prisma의 Migration 관리 방법
        - 마이그레이션: 코드로 테이블 구조(Schema)를 바꾸면, 그걸 실제 DB에 반영하고 기록하는 과정이다.
            - 설계도 중심: schema.prisma 파일을 수정하고 npx prisma migrate dev를 치면 된다.
            - 이력서 관리: 프리즈마는 수정 내역을 prisma/mirgation/ 폴더 안에 타임스탬프가 찍힌 SQL 파일로 자동 저장해준다. 깃에 이 폴더를 같이 올리면 팀원들과 똑같은  DB 구조를 공유할 수 있다.
        
- ORM(Prisma)을 사용하여 좋은 점과 나쁜 점
    - 장점
        - 생산성 향상: SQL 문법을 사용하지 않아 코드가 훨씬 짧아지고 직관적이어서, 비즈니스 로직에 더 집중할 수 있다.
        - DB 종속성 탈피: DB를 MySQL에서 MariaDB로 변경하는 것처럼 DB를 바꿔도, ORM 코드 체계는 거의 그대로 유지되어 마이그레이션이 편하다.
        - 타입 안정성: 타입스크립트와 결합하면 오타나 잘못된 필드 접근을 컴파일 단계에서 미리 잡아준다.
    - 단점
        - 성능 한계(N+1 문제): 조인 시, ORM이 내부적으로 비효율적인 쿼리를 여러번 날려서 서버가 느려질 수 있다.
        - 복잡한 쿼리의 어려움: 통계 쿼리나 대용량 데이터 배치 처리처럼 복잡한 SQL은 ORM 문법으로 표현하기 힘들어서 결국 Raw SQL을 섞어서 사용해야 한다.
        - 블랙박스 리스크: 내부적으로 SQL이 어떻게 생성되는지 모르면, 무거운 쿼리가 돌고 있어도 원인을 찾기 힘들다.
- 다양한 ORM 라이브러리 살펴보기
    1. Prisma
        - 특징: 최신 트렌드, 스키마 중심
        - 장점: 타입 지원 최고, 가독성 좋음, 공식 문서 친절
        - 단점: 프리즈마 7처럼 대대적 버전 변경 시 삽질 유발, 약간의 오버헤드
    2. TypeORM
        - 특징: 데코레이터(@Entity) 중심
        - 장점: Nest.js와 찰떡궁합,전통적인 백엔드 아키텍처에 익숙함
        - 단점: 타입 추론이 프리즈마보다 살짝 아쉽고 설정이 무거움
    3. Sequelize
        - 특징: 가장 오래된 Node.js ORM
        - 장점: 자바스크립트 환경에서 레퍼런스가 아주 많음, 안정적
        - 단점: 타입스크립트 지원이 손상된 상태에서는 다소 불편함
- 페이지네이션을 사용하는 다른 API 찾아보기
    - ex. https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2022-11-28GitHub REST API (Offset+Cursor 혼합)
        
        깃허브는 수많은 커밋과 이슈를 보여줘야 해서 페이지네이션이 정교하다.
        
        - 방식: 기본적으로 page와 per_page 쿼리를 쓰는 Offeset 방식을 지원하지만, 대량 데이터나 실시간성이 중요한 트렌드 데이터에는 Link 헤더를 통해 next, last 페이지 전체 URL(Cursor 역할)을 내려준다.
        - 특징: 다음 페이지 주소를 헤더에 담아주기 때문에 클라이언트가 페이지 계산을 직접 안 해도 되어서 편하다.
    - ex. https://developers.notion.com/reference/intro#paginationNotion API(완벽한 Cursor 방식)
        
        노션은 페이지 내부의 수많은 블록 데이터가 실시간으로 추가/삭제되기 때문에 Cursor 페이징을 강제하고 있다.
        
        - 방식: 요청할 때 start_cursor와 page_size를 보낸다.
        - 응답: 데이터를 줄 때 JSON 안에 next_cursor 값과 has_more (다음 페이지가 더 있는지 여부)를 함께 보내준다.
        - 이유: 노션 글을 여러 명이 동시에 편집할 때, Offset 방식을 쓰면 중간에 글이 추가되었을 때 중복 데이턱 노출되는 대참사가 난다.