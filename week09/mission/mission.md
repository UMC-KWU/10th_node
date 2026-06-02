https://github.com/minji-jeong516/umc_node/pull/18
## 💪 미션 기록

<aside>
🍀 미션 기록의 경우, 아래 미션 기록 토글 속에 작성하시거나, 페이지를 새로 생성하여 해당 페이지에 기록하여도 좋습니다!

하지만, 결과물만 올리는 것이 아닌, **중간 과정 모두 기록하셔야 한다는 점!** 잊지 말아주세요.

</aside>

[과제 제출 폼](https://www.notion.so/311b57f4596b81ac9518f7f801f0c5ac?pvs=21)

- 
    - 하드코딩 제거
        
        기존 코드에서는 사용자 id를 고정값인 1로 사용하고 있었다. 이를 JWT 인증 기반 구조에 맞게 수정하여 현재 로그인한 사용자의 정보를 사용하도록 변경하였다.
        
        const userId = 1; → const userId = (req as any).user.id;
        
        ex) review.controller.ts
        
        ```jsx
        
            /** 리뷰를 작성할 가게 ID */
            @Path() storeId: number,
        
            @Body() body: CreateReviewRequest,
        
            @Request() req: ExpressRequest
          ): Promise<ApiResponse<any>> {
            const userId = (req as any).user.id;
        
            const result = await addReview(storeId, userId, body);
        
            return success(result);
          }
        ```
        
    - 내 정보 수정 API 구현
        
        기존 회원가입 API는 이미 존재하는 이메일에 대해 회원가입을 막는 구조였다.
        
        Google 로그인 사용자처럼 추가 정보를 나중에 입력해야 하는 상황을 고려하여, 내 정보를 수정하는  API를 새로 구현했다.
        
        구현 내용
        
        - `UpdateMyInfoRequest` DTO 추가
        - 사용자 정보 수정 Repository 함수 구현
        - 기존 선호 카테고리 삭제 후 재등록 로직 추가
        - JWT 기반 현재 사용자 정보 수정 처리
    - JWT 인증 시스템 적용
        
        기존 API들에 JWT 인증 미들웨어를 적용하여, 로그인한 사용자만 접근 가능하도록 수정하였다.
        
        구현방식:
        
        `authenticateJWT()` 미들웨어를 구현하여 Authorization 헤더의 JWT 토큰을 검증하도록 하였다.
        
        ```jsx
        const authorization = req.headers.authorization;
        ```
        
        토큰 검증 성공 시:
        
        ```jsx
        (req as any).user = decoded;
        ```
        
        형태로 사용자 정보를 저장하고, 이후 Controller에서 로그인 사용자 정보를 사용할 수 있도록 구현하였다.
        
        적용한 API
        
        - 리뷰 작성 API
        - 내 리뷰 조회 API
        - 가게 추가 API
        - 미션 도전 API
        - 도전 중 미션 조회 API
        - 미션 완료 처리 API
        - 내 정보 수정 API
            
            <img width="1045" height="1242" alt="스크린샷 2026-05-27 124715" src="https://github.com/user-attachments/assets/316d0da6-5a7c-49e3-b337-3459c9f58e1d" />
<img width="1044" height="1284" alt="스크린샷 2026-05-27 124457" src="https://github.com/user-attachments/assets/92156d75-feac-4bf3-b0f4-03feefae49c6" />

