https://github.com/Hagyeong13/umc_node/tree/feature/chapter-09

- 1. 하드코딩된 사용자 정보 수정
    
    기존에는 리뷰 작성, 미션 도전, 진행 중인 미션 조회 등에서 userId를 request body나 path parameter로 받아 사용했다.
    수정 후에는 클라이언트가 임의로 userId를 전달하지 않도록 하고, JWT 인증 후 req.user.userId에서 로그인한 사용자의 ID를 가져와 사용하도록 변경했다.
    
    예시:
    
    ```jsx
    // 기존
    userId: body.userId
    
    // 수정
    userId: req.user.userId
    ```
    
    추가로 `users/{userId}/reviews`, `users/{userId}/missions/ongoing`처럼 특정 사용자 ID를 URL로 받던 API는 `users/me/reviews`, `users/me/missions/ongoing`으로 변경하여 로그인한 사용자 본인의 데이터만 조회하도록 수정했다.
    
- 2. 기존 사용자 정보 갱신
    
    기존 회원가입 API는 이미 존재하는 이메일이면 중복 이메일 에러를 반환했다.
    하지만 Google 로그인으로 자동 생성된 사용자는 전화번호, 생일, 주소 등의 정보가 임시값이므로, 이후 정보를 채울 수 있어야 한다.
    
    예시:
    
    ```jsx
    // 기존
    이미 존재하는 이메일이면 회원가입 차단
    
    // 수정
    이미 존재하는 이메일이면 사용자 정보 update
    존재하지 않으면 새 사용자 create
    ```
    
    기존 addUser 로직을 upsertUser 방식으로 변경했다.
    추가로 선호 카테고리는 기존 값이 중복으로 쌓이지 않도록, 기존 선호 정보를 삭제한 뒤 새로 받은 선호 카테고리를 다시 저장하도록 수정했다.
    
- 3. JWT 인증 시스템 기존 API에 적용
    
    9주차에서 구현한 Passport JWT 인증 방식을 기존 API에 적용했다.
    
    `passport.authenticate("jwt", { session: false })`를 `isLogin` 미들웨어로 정의하고, 로그인이 필요한 API 라우터에 적용했다.
    
    적용한 API:
    
    ```
    app.post("/api/v1/stores/:storeId/reviews",isLogin);
    app.post("/api/v1/regions/:regionId/stores",isLogin);
    app.post("/api/v1/missions/:missionId/challenge",isLogin);
    app.get("/api/v1/users/me/reviews",isLogin);
    app.get("/api/v1/users/me/missions/ongoing",isLogin);
    ```