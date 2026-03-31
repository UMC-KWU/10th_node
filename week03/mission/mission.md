필요하다고 생각되는 API
- 홈 화면 : 

- 선택한 지역의 남은 미션 목록을 조회하는 API
- 사용자가 특정 지역에서 달성한 미션 수를 조회하는 API

- 마이 페이지 리뷰 작성 : 

- 사용자가 특정 가게에 리뷰를 등록하는 API

- 미션 목록 조회 : 

- 사용자의 진행중/완료 미션 목록을 상태별로 조회하는 API

- 미션 성공 누르기 : 

- 사용자가 수행 중인 미션을 성공 처리하는 API

- 회원 가입 하기 : 

- 사용자의 회원 정보를 저장하는 API

1. Endpoint, Method 정하기
- `GET /home/progress`
- `GET /home/todo-missions`
- `GET /missions`
- `PATCH /missions/{missionId}/success`
- `POST /reviews`
- `POST /users/signup`
- HTTP Method
    - 홈 화면 조회 → `GET`
    - 미션 목록 조회 → `GET`
    - 리뷰 작성 → `POST`
    - 미션 성공 처리 → `PATCH`
    - 회원가입 → `POST`

1. Request Header
- `GET /home/todo-missions`
    
    : Authorization - 로그인 한 사용자의 남은 미션
    
- `GET /home/progress`
    
    : Authorization - 로그인 한 사용자가 미션을 얼마나 한 지
    
- `GET /missions`
    
    : Authorization - 로그인 한 사용자의 미션 진행 여부
    
- `PATCH /missions/{missionId}/success`
    
    : Authorization - 로그인 한 사용자의 미션
    
- `POST /reviews`
    
    : Authorization  - 로그인 한 사용자가 리뷰 작성
    
- `POST /users/signup`
    
    : Content-Type - 데이터가 어떤 형식인지
    

1. Path Variable
- `PATCH /missions/{missionId}/success`
    
    : missionId - 어떤 미션이 성공한 건지에 대해
    

1. Query String
- `GET /home/todo-missions`
    
    : regionId - 선택한 지역 내의 미션만
    
- `GET /home/progress`
    
    : regionId - 선택한 지역 내의 미션만
    
- `GET /missions`
    
    : status - 진행 중인지 진행 완료인지
    

1. Request Body
- `PATCH /missions/{missionId}/success`
    
    : ownerCode
    
- `POST /reviews`
    
    : storeId, rating, content, images
    
- `POST /users/signup`
    
    : name, gender, birth, address, preferFoodCategoryIds
    

1. Response
- `GET /home/todo-missions`
    
    : missionId, storeName, missionContent, regionName, point, deadline
    
- `GET /home/progress`
    
    : regionName, completedMissionCount, currentProgressCount, goalCount, rewardPoint
    
- `GET /missions`
    
    : missionId, storeName, missionContent, point, status
    
- `PATCH /missions/{missionId}/success`
    
    : missionId, status, completedAt
    
- `POST /reviews`
    
    : reviewId
    
- `POST /users/signup`
    
    : userId, name

[API 명세서](https://www.notion.so/makeus-challenge/333b57f4596b80edbdb9f51b511616d0?v=333b57f4596b81de852f000c89b2bef4&source=copy_link)