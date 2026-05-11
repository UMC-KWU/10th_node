- ORM
    
    ORM(Object Relational Mapping)은 객체 지향 언어의 객체와 관계형 데이터베이스(DB)를 연결해주는 기술이다. 즉, SQL을 직접 작성하지 않고 객체 형태로 데이터를 조회하고 저장할 수 있도록 도와준다.예를 들어 Prisma ORM에서는 SQL 대신 다음과 같이 코드를 작성하여 데이터를 조회할 수 있다.
    
- Prisma 문서 살펴보기
    - ex. Prisma의 Connection Pool 관리 방법
        - Prisma는 내부적으로 DB 연결(Connection)을 재사용하여 관리한다.
        - 너무 많은 연결이 생성되는 것을 방지하여 성능 저하를 줄일 수 있다.
        - PrismaClient를 여러 번 생성하지 않고 하나의 인스턴스를 재사용하는 방식이 권장된다.
    - ex. Prisma의 Migration 관리 방법
        - Prisma는 migration 기능을 통해 DB 스키마 변경 내역을 관리한다.
        - `npx prisma migrate dev` 명령어를 사용하여 migration 파일을 생성하고 DB에 반영할 수 있다.
        - migration 파일을 통해 DB 구조 변경 이력을 추적할 수 있다.
- ORM(Prisma)을 사용하여 좋은 점과 나쁜 점
    
    좋은점: 
    
    - SQL을 직접 작성하지 않아도 되어 개발 속도가 빠르다.
    - TypeScript 타입 지원이 좋아 자동완성과 타입 체크가 편리하다.
    - 코드 가독성이 좋아진다.
    
    나쁜점:
    
    - 복잡한 쿼리는 오히려 SQL보다 작성이 어려울 수 있다.
    - ORM 구조를 이해해야 해서 초반 학습 비용이 있다.
    - 추상화 때문에 실제 SQL 동작을 놓칠 수 있다.
- 다양한 ORM 라이브러리 살펴보기
    - Prisma
    - TypeORM
    - Sequelize
    - Drizzle ORM
- 페이지네이션을 사용하는 다른 API 찾아보기
    - ex. https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2022-11-28
    - ex. https://developers.notion.com/reference/intro#pagination
