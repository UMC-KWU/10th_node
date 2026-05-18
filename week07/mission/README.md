https://github.com/Hagyeong13/umc_node/tree/feature/chapter-07

큰 틀로 기존에는 controller에서 응답을 직접 만든 걸 공통 응답 함수로 바꿈

과제에서 커스텀 `Error` 객체 사용하라 해서 상황별 에러 클래스 추가함

- review
    
    우선 기존 api
    
    ```
    app.post("/api/v1/stores/:storeId/reviews", handleCreateReview);
    app.get("/api/v1/users/:userId/reviews", handleListMyReviews);
    ```
    
    이렇게 2개 있었기 때문에
    
    tsoa 큰 틀은
    
    @Route("")
    @Tags("Reviews")
    export class ReviewController extends Controller {
    @Post("stores/{storeId}/reviews")
    ~~
    
    @Get("users/{userId}/reviews")
    ~~
    }
    
    1) handleCreateReview
    우선 기존 코드에서 params는 @Path로 body는 @Body로 수정하면
    
    ```
    public async handleCreateReview (
      @Path() storeId : number,
      @Body() body : CreateReviewRequest
    ) : Promise<ApiResponse<ReviewResponse>>{
    ~~
    }
    ```
    
    여기에 기존 내부 코드 그대로 값 변경 해주면 된다.
    
    추가로 @Query() cursor? : number는 이렇게 변경하고 cursor??0으로 해서 값을 넘기면 된다.
    
    아래는 최종 결과
    
    ```tsx
    @Route("")
    @Tags("Reviews")
    export class ReviewController extends Controller {
      @Post("stores/{storeId}/reviews")
      public async handleCreateReview (
        @Path() storeId : number,
        @Body() body : CreateReviewRequest
      ) : Promise<ApiResponse<ReviewResponse>>{
        const review = await createReview(
          storeId,
          bodyToReview(body)
        );
    
        return success(review);
      }
    
      @Get("users/{userId}/reviews")
      public async handleListMyReviews (
        @Path() userId : number,
        @Query() cursor? : number
      ): Promise<ApiResponse<MyReviewListResponse>> {
        const reviews = await listMyReviews(userId, cursor??0);
        return success(reviews);
      }
    }
    ```
    
    - error 객체 추가
        
        export class ReviewStoreNotFoundError extends AppError {
        constructor(data?: unknown) {
        super({
        errorCode: "R001",
        statusCode: 404,
        message: "존재하지 않는 가게입니다.",
        data,
        });
        }
        }
        
        export class ReviewCreateFailedError extends AppError {
        constructor(data?: unknown) {
        super({
        errorCode: "R002",
        statusCode: 500,
        message: "리뷰 생성 후 조회 실패했습니다.",
        data,
        });
        }
        }
        
    
    이후 services 수정
    
    기존:
    
    ```
    if(store===null)
    thrownewError("존재하지 않는 가게입니다.");
    ```
    
    변경:
    
    ```
    if (store===null) {
    thrownewReviewStoreNotFoundError({ storeId });
    }
    ```
    
    기존:
    
    ```
    if(review===null)
    thrownewError("리뷰 생성 후 조회 실패했습니다.");
    ```
    
    변경:
    
    ```
    if (review===null) {
    thrownewReviewCreateFailedError({ reviewId });
    }
    ```
    review에 한 그대로 수행한다.
    
- store
    - 추가한 error 객체
        
        export class RegionNotFoundError extends AppError {
        
        constructor(data?: unknown) {
        
        super({
        
        errorCode: "R001",
        
        statusCode: 404,
        
        message: "없는 지역입니다.",
        
        data,
        
        });
        
        }
        
        }
        
        export class StoreNotFoundError extends AppError {
        
        constructor(data?: unknown) {
        
        super({
        
        errorCode: "S001",
        
        statusCode: 404,
        
        message: "없는 가게입니다.",
        
        data,
        
        });
        
        }
        
        }
        
        export class StoreCreateFailedError extends AppError {
        
        constructor(data?: unknown) {
        
        super({
        
        errorCode: "S002",
        
        statusCode: 500,
        
        message: "가게 생성 후 조회에 실패했습니다.",
        
        data,
        
        });
        
        }
        
        }
        
    
    기존 
    
    res.status(StatusCodes.OK).json({
    
        result: store,
    
    });
    res.status(StatusCodes.OK).json(reviews);
    해당 응답을
    res.status(StatusCodes.OK).json(success(store));
    res.status(StatusCodes.OK).json(success(reviews));
    로 변경함
    
    error는
    
    if (region === null) {
    throw new Error("없는 지역");
    }
    이렇게 사용했는데
    
    if (region === null) {
    throw new RegionNotFoundError({ regionId });
    }
    커스텀 에러로 수정함
    
    - controllers
        
        ```tsx
        @Route("")
        @Tags("Stores")
        export class StoreController extends Controller {
          @Post("regions/{regionId}/stores")
          public async handleCreateStore(
            @Path() regionId: number,
            @Body() body: CreateStoreRequest
          ): Promise<ApiResponse<StoreResponse>> {
            const store = await createStore(regionId, bodyToStore(body));
        
            return success(store);
          }
        
          @Get("stores/{storeId}/reviews")
          public async handleListStoreReviews(
            @Path() storeId: number,
            @Query() cursor?: number
          ): Promise<ApiResponse<ReviewListResponse>> {
            const reviews = await listStoreReviews(storeId, cursor ?? 0);
        
            return success(reviews);
          }
        }
        ```
        
- mission
    
    우선 throw 하던 거 다 error 객체 만들어서 처리하기
    
    기존 함수들을 형태 바꾸기
    
    - services
        
        ```tsx
        if (mission === null) {
            throw new MissionNotFoundError({ missionId });
          }
          
        if (userMission !== null) {
            throw new MissionAlreadyChallengedError({
              userId: data.userId,
              missionId,
            });
          }
          
        if (createdUserMission === null) {
            throw new UserMissionCreateFailedError({ userMissionId });
          }
        ```
        
    - controllers
        
        ```tsx
        @Route("")
        @Tags("Missions")
        export class MissionController extends Controller {
          @Post("missions/{missionId}/challenge")
          public async handleCreateMission(
            @Path() missionId: number,
            @Body() body: ChallengeMissionRequest
          ): Promise<ApiResponse<Awaited<ReturnType<typeof challengeMission>>>> {
            const userMission = await challengeMission(
              missionId,
              bodyToChallengeMission(body)
            );
        
            return success(userMission);
          }
        
          @Get("stores/{storeId}/missions")
          public async handleListStoreMissions(
            @Path() storeId: number,
            @Query() cursor?: number
          ): Promise<ApiResponse<StoreMissionListResponse>> {
            const missions = await listStoreMissions(storeId, cursor ?? 0);
        
            return success(missions);
          }
        
          @Get("users/{userId}/missions/ongoing")
          public async handleListOngoingUserMissions(
            @Path() userId: number,
            @Query() cursor?: number
          ): Promise<ApiResponse<OngoingMissionListResponse>> {
            const missions = await listOngoingUserMissions(userId, cursor ?? 0);
        
            return success(missions);
          }
        ```