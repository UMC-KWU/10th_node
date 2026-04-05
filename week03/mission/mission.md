- API 설계 사고 과정
    1. 엔드포인트(Endpoint) 명명 규칙
    - 누가(Actor) 무엇을(Resource) 하는가?를 기준으로 계층을 나눈다.
    - 모든 API 앞에 /api/v1을 붙여 버전을 관리하고, 도메인별로 /users, /mission, /stores로 그룹화한다.
    1. HTTP 메소드의 선택
    - 데이터를 새로 만드는 것인가(POST)? 일부만 수정하는 것인가(PATCH)?
    - 회원가입, 리뷰 작성은 새로운 데이터를 생성하니까 POST, 미션 완료는 기존 ‘도전 중’ 상태를 ‘완료’로 바꾸는 것이니까 PATCH
    1. 인증(Authorization)의 유무
    - 이 정보를 아무나 봐도 되는 것인가? 아니면 개인정보인가?
    - 회원가입을 제외한 모든 마이페이지, 미션 성공, 리뷰 작성에는 Authorization 헤더를 필수값으로 넣는다.(보안을 위해)
- 회원가입
    - 설명: 서비스에 처음 가입하는 단계입니다. 휴대폰 번호는 선택 사항이며, 가입 직후 선호 카테고리 설정(온보딩)으로 넘어가는 흐름을 고려했습니다.
    - **Endpoint**: `/api/v1/users/signup`
    - **Method**: `POST`
    - **Request Header**: `Content-Type: application/json`
    - **Query String**: 필요 없음
    - **Request Body**
    
    ```json
    {
      "email": "jihyun0909@kw.ac.kr",
      "password": "securePassword123!",
      "name": "변지현",
      "phoneNum": "010-1234-5678"
    }
    ```
    
    - **Response Body**
    
    ```json
    {
      "success": true,
      "code": "S200",
      "message": "회원가입이 완료되었습니다.",
      "data": { "id": 1, "name": "변지현" }
    }
    ```
    
    - 에러 처리
        - 이메일 중복[E4001]
        
        ```json
        {"success": false, "code": "E4001", "message": "이미 존재하는 이메일입니다.", "data": null}
        ```
        
        - 형식 오류[E4002]
        
        ```json
        {"success": false, "code": "E4002", "message": "비밀번호는 8자 이상, 특수문자를 포함해야 합니다.", "data": null}
        ```
        
- 홈 화면(도전 가능 미션 목록)
    - **설명**: 사용자가 선택한 지역에서 아직 도전하지 않은 미션들을 보여줍니다. 무한 스크롤을 위해 **커서 기반 페이징**을 적용했습니다.
    - **Endpoint**: `/api/v1/missions`
    - **Method**: `GET`
    - **Request Header**: `Authorization: Bearer {accessToken}` (나의 도전 여부를 체크하기 위해 필요)
    - **Query String**: `regionId=1`, `lastDeadline=2026-04-10T12:00:00`, `lastId=15`, `size=10`
    - **Request Body**: 필요 없음
    - **Response Body**
    
    ```json
    {
      "success": true,
      "code": "S200",
      "data": {
        "missions": [
          { "id": 16, "storeName": "광운대카츠", "category": "일식", "content": "치즈카츠 인증", "reward": 500, "deadline": "2026-04-11T12:00:00" }
        ],
        "nextCursor": { "lastDeadline": "2026-04-11T12:00:00", "lastId": 16 },
        "hasNext": true
      }
    }
    ```
    
    - 에러 처리
        - 지역 없음[E4004]
        
        ```json
        {"success": false, "code": "E4004", "message": "존재하지 않는 지역 ID입니다.", "data": null}
        ```
        
- 미션 목록 조회(진행 중, 진행 완료)
    - **설명**: 마이페이지 또는 미션 탭에서 내가 현재 '수행 중'인 미션과 '진행 완료'한 미션을 구분해서 조회합니다.
    - **Endpoint**: `/api/v1/users/me/missions`
    - **Method**: `GET`
    - **Request Header**: `Authorization: Bearer {accessToken}`
    - **Query String**: `status=CHALLENGING` (또는 `COMPLETE`)
    - **Request Body**: 필요 없음
    - **Response Body**
    
    ```json
    {
      "success": true,
      "code": "S200",
      "data": {
        "status": "CHALLENGING",
        "count": 3,
        "missions": [
          { "missionId": 10, "storeName": "광운대카페", "content": "아메리카노 1잔", "reward": 100 }
        ]
      }
    }
    ```
    
    - 에러 처리
        - 잘못된 상태 값 [E4008]
        
        ```json
        {"success": false, "code": "E4008", "message": "유효하지 않은 상태 값입니다.", "data": null}
        ```
        
- 미션 성공 누르기
    - **설명**: 수행 중인 미션을 완료 처리하고 포인트를 획득합니다. 데이터의 상태를 변경하므로 `PATCH`를 사용합니다.
    - **Endpoint**: `/api/v1/missions/{missionId}/complete`
    - **Method**: `PATCH`
    - **Request Header**: `Authorization: Bearer {accessToken}`
    - **Query String**: 필요 없음
    - **Path Variable**: `missionId`
    - **Request Body**: 필요 없음 (인증 헤더로 유저 식별)
    - **Response Body**
    
    ```json
    {
      "success": true,
      "code": "S200",
      "message": "미션 성공! 포인트가 적립되었습니다.",
      "data": { "addedPoint": 500, "totalPoint": 3000 }
    }
    ```
    
    - 에러 처리
        - 중복 완료[E4003]
        
        ```json
        {"success": false, "code": "E4003", "message": "이미 완료된 미션입니다.", "data": null}
        ```
        
        - 도전 중 아님[E4005]
        
        ```json
        {"success": false, "code": "E4005", "message": "도전 중인 미션 리스트에 없습니다.", "data": null}
        ```
        
- 마이페이지 리뷰 작성
    - **설명**: 특정 가게에 대해 별점과 내용을 포함한 리뷰를 작성합니다.
    - **Endpoint**: `/api/v1/stores/{storeId}/reviews`
    - **Method**: `POST`
    - **Request Header**: `Authorization: Bearer {accessToken}`
    - **Query String**: 필요 없음
    - **Path Variable**: `storeId`
    - **Request Body**
    
    ```json
    {
      "score": 5,
      "content": "최고의 맛집이에요! 사장님도 친절하시네요."
    }
    ```
    
    - **Response Body**
    
    ```json
    {
      "success": true,
      "code": "S200",
      "message": "리뷰가 성공적으로 등록되었습니다.",
      "data": { "reviewId": 101, "createdAt": "2026-04-01T23:00:00" }
    }
    ```
    
    - **에러 처리**
        - 내용 누락[E4006]
        
        ```json
        
        {"success": false, "code": "E4006", "message": "리뷰 내용은 최소 10자 이상이어야 합니다.", "data": null}
        
        ```