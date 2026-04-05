- 홈 화면
    - 홈 화면 상단 (진행상황)
        - API Endpoint
            
            → GET /api/v1/home/summary
            
        - Query String
            
            →  RegionId: 선택 지역을 의미
            
        - Request Header
            
            → Authorization: Bearer {accessToken}
           
        
        → 홈 화면 상단의 진행률과 현재 포인트를 조회하는 API로, 선택한 지역 기준으로 전체 미션 수, 완료한 미션 수, 보유 포인트를 조회한다.
        
    - 홈 화면 하단 (미션 목록 조회)
        - API Endpoint
            
            → GET /api/v1/home/missions
            
        - Query String
            
            
            | 이름 | 설명 |
            | --- | --- |
            | RegionId | 선택 지역 |
            | page | 페이지번호 |
            | size | 페이지크기 |
        - Request Header
            
            → Authorization: Bearer {accessToken}
            
        
        → 홈 화면 하단의 도전 가능한 미션 목록을 조회하는 API이다. 
        
- 마이페이지 리뷰 작성
    - API Endpoint
        
        → POST /api/v1/reviews
        
    - Request Header
        
        →  Authorization: Bearer {accessToken} (로그인한 사용자를 인증하기 위한 헤더)
        
        Content-Type: application/json (Body에 담긴 데이터의 형식이 Json임을 알리는 헤더)
        
    - Path Variable
        
        → memberMissionId: 완료된 미션 Id
        
    - Request Body
        
        ```json
        {
        	"memberMissionId": 20,
        	"rating":5,
        	"comment": "맛있어요~~"
        	}
        ```
        
    
    → 완료한 미션에 대해 리뷰를 작성하는 API. 어떤 미션에 관한 리뷰인지 구분하기 위해 memberMissionId를 함께 전달한다.
    
- 미션 목록 조회
    - 진행중
        - API Endpoint
            
            → GET /api/v1/missions/challenging
            
        - Request Header
            
            → Authorization: {accessToken}
            
        - Query String
            
            
            | 이름 | 설명 |
            | --- | --- |
            | page | 페이지번호 |
            | size | 페이지크기 |
        
        → 사용자가 현재 수행중인 미션 목록을 조회하는 API이다. 
        
    - 진행 완료
        - API Endpoint
            
            → GET /api/v1/missions/completed
            
        - Request Header
            
            → Authorization: {accessToken}
            
        - Query String
            
            
            | 이름 | 설명 |
            | --- | --- |
            | page | 페이지번호 |
            | size | 페이지크기 |
        
        → 사용자가 완료한 미션 목록을 조회하는 API이다. 완료된 미션에 대해선 리뷰 작성이 가능하다.
        
- 미션 성공 누르기
    - API Endpoint
        
        → PATCH /api/v1/missions/{memberMissionId}/success
        
    - Request Header
        
        → Authorization: {accessToken}
        
        Content-Type: application/json
        
    - Path Variable
        
        → memberMissionId : 사용자의 성공 처리할 미션
        
    - Request Body
        
        ```json
        {
          "status": "COMPLETED"
        }
        ```
        
    
    → 진행중인 미션에 대해 성공 요청을 보내는 API. 성공 요청 시 상태가 진행중에서 Completed로 변경된다.
    
- 회원가입
    - API Endpoint
        
        → POST /api/v1/members/signup
        
    - Request Header
        
        → Content-Type: application/json
        
    - Request Body
        
        ```json
        {
          "name": "정민지",
          "gender": "FEMALE",
          "birth": "2004-05-16",
          "address": "서울 노원구",
          "preferFoodCategoryIds": [1, 2, 3]
        }
        ```
        
        → 소셜 로그인을 제외한 일반 회원가입 API. 회원가입 화면에서 이름, 성별, 생년월일, 주소, 선호하는 음식 종류를 저장한다.
