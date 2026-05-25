https://github.com/Hagyeong13/umc_node/tree/feature/chapter-08

- 미션 기록
    
    우선 service.ts에서 throw 하고 있는 거 파악
    각각 어떤 api 인지 작성 @summary 포함해서
    그리고 각각 @Path, @Query등 작성
    throw 하고 있는 거 제시한대로 TsoaResponse로 작성
    
    - mission
        
        ```tsx
        @Route("")
        @Tags("Missions")
        export class MissionController extends Controller {
          /**
           * 미션 도전 API
           * @summary 사용자가 특정 미션에 도전합니다.
           */
          @Post("missions/{missionId}/challenge")
          @SuccessResponse(200, "미션 도전 성공")
          @TsoaResponse<UserMissionCreateFailedError>(500,"도전 미션 조회에 실패했습니다.")
          @TsoaResponse<MissionAlreadyChallengedError>(409,"이미 도전 중인 미션입니다.")
          @TsoaResponse<MissionNotFoundError>(404,"존재하지 않는 미션입니다.")
          public async handleCreateMission(
            /** 도전할 미션 ID */
            @Path() missionId: number,
            /** 미션 도전 요청 body */
            @Body() body: ChallengeMissionRequest
          ): Promise<ApiResponse<Awaited<ReturnType<typeof challengeMission>>>> {
            const userMission = await challengeMission(
              missionId,
              bodyToChallengeMission(body)
            );
        
            return success(userMission);
          }
        
          /**
           * 가게 미션 목록 조회 API
           * @summary 특정 가게에 등록된 미션 목록을 cursor 기반으로 조회합니다.
           */
          @Get("stores/{storeId}/missions")
          @SuccessResponse(200, "가게 미션 목록 조회 성공")
          public async handleListStoreMissions(
            /** 미션을 조회할 가게 ID */
            @Path() storeId: number,
            /** 페이지네이션 cursor 값 */
            @Query() cursor?: number
          ): Promise<ApiResponse<StoreMissionListResponse>> {
            const missions = await listStoreMissions(storeId, cursor ?? 0);
        
            return success(missions);
          }
        
          /**
           * 진행 중인 미션 목록 조회 API
           * @summary 특정 사용자가 진행 중인 미션 목록을 cursor 기반으로 조회합니다.
           */
        
          @Get("users/{userId}/missions/ongoing")
          @SuccessResponse(200, "진행 중인 미션 목록 조회 성공")
          public async handleListOngoingUserMissions(
            /** 진행 중인 미션을 조회할 사용자 ID */
            @Path() userId: number,
            /** 페이지네이션 cursor 값 */
            @Query() cursor?: number
          ): Promise<ApiResponse<OngoingMissionListResponse>> {
            const missions = await listOngoingUserMissions(userId, cursor ?? 0);
        
            return success(missions);
          }
        }
        ```
        
    - review
        
        ```tsx
        @Route("")
        @Tags("Reviews")
        export class ReviewController extends Controller {
          /**
           * 리뷰 생성 API
           * @summary 사용자가 리뷰를 작성합니다.
           */
          @Post("stores/{storeId}/reviews")
          @SuccessResponse(200, "리뷰 작성 성공")
          @TsoaResponse<ReviewCreateFailedError>(500,"리뷰 생성 후 조회 실패했습니다.")
          @TsoaResponse<ReviewStoreNotFoundError>(404,"존재하지 않는 가게입니다.")
          public async handleCreateReview (
            /** 작성할 가게 ID */
            @Path() storeId : number,
            /**리뷰 생성 요청 body */
            @Body() body : CreateReviewRequest
          ) : Promise<ApiResponse<ReviewResponse>>{
            const review = await createReview(
              storeId,
              bodyToReview(body)
            );
        
            return success(review);
          }
        
        	/**
           * 리뷰 목록 조회 API
           * @summary 사용자가 작성한 리뷰를 조회합니다.
           */
          @Get("users/{userId}/reviews")
          @SuccessResponse(200, "사용자 리뷰 목록 보기 성공")
          public async handleListMyReviews (
            /**리뷰 확인 사용자 ID */
            @Path() userId : number,
            /** 다음 페이지 조회를 위한 cursor */
            @Query() cursor? : number
          ): Promise<ApiResponse<MyReviewListResponse>> {
            const reviews = await listMyReviews(userId, cursor??0);
            return success(reviews);
          }
        }
        ```
        
    - store
        
        ```tsx
        @Route("")
        @Tags("Stores")
        export class StoreController extends Controller {
          /**
           * 가게 생성 API
           * @summary 가게를 생성합니다.
           */
          @Post("regions/{regionId}/stores")
          @SuccessResponse(200,"가게 생성 성공")
          @TsoaResponse<RegionNotFoundError>(404,"없는 지역입니다.")
          @TsoaResponse<StoreCreateFailedError>(500,"가게 생성 후 조회에 실패했습니다.")
          public async handleCreateStore(
            /** 가게 삽입 지역 */
            @Path() regionId: number,
            /**가게 생성 요청 body */
            @Body() body: CreateStoreRequest
          ): Promise<ApiResponse<StoreResponse>> {
            const store = await createStore(regionId, bodyToStore(body));
        
            return success(store);
          }
        
          /**
           * 가게 리뷰 조회 API
           * @summary 가게의 리뷰를 조회합니다.
           */
          @Get("stores/{storeId}/reviews")
          @SuccessResponse(200,"가게 목록 조회 성공")
          @TsoaResponse<StoreNotFoundError>(404,"없는 가게입니다.")
          public async handleListStoreReviews(
            @Path() storeId: number,
            @Query() cursor?: number
          ): Promise<ApiResponse<ReviewListResponse>> {
            const reviews = await listStoreReviews(storeId, cursor ?? 0);
        
            return success(reviews);
          }
        }
        ```