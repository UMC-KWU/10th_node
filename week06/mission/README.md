GitHub 저장소 주소
https://github.com/Hagyeong13/umc_node

- 미션 기록
    - 내가 작성한 리뷰 목록
        1. API 설계
        
        GET /api/v1/users/:userId/reviews?cursor=0
        
        커서를 마지막 조회한 reviewId로 두고 한 번에 5개씩, 다음 요청 때 커서보다 큰 reviewId만
        
        1. dto
        
        리뷰 보여주기에 필요한 것들을 response로
        
        ```jsx
        export interface MyReviewListResponse {
          data: {
            reviewId: number;
            userId: number | null;
            storeId: number | null;
            storeName: string | null;
            star: number | null;
            content: string | null;
            picture: string | null;
            createdAt: Date | null;
          }[];
          pagination: {
            cursor: number | null;
          };
        }
        
        export const responseFromMyReviews = (
          reviews: any[]
        ): MyReviewListResponse => {
          const data = reviews.map((review) => ({
            reviewId: review.reviewId,
            userId: review.userId,
            storeId: review.storeId,
            storeName: review.store?.name ?? null,
            star: review.star,
            content: review.content,
            picture: review.picture,
            createdAt: review.createdAt,
          }));
        
          const lastReview = data[data.length - 1];
        
          return {
            data,
            pagination: {
              cursor: lastReview ? lastReview.reviewId : null,
            },
          };
        };
        ```
        
        3. findMany 사용하기
        
        ```jsx
        export const getMyReviews = async (
          userId: number,
          cursor: number
        ) => {
          return await prisma.review.findMany({
            where: {
              userId,
              reviewId: {
                gt: cursor,
              },
            },
            include: {
              store: true,
            },
            orderBy: {
              reviewId: "asc",
            },
            take: 5,
          });
        };
        ```
        
        4. getMyReviews를 services에서 호출
        
        ```jsx
        export const listMyReviews = async (
          userId: number,
          cursor: number
        ): Promise<MyReviewListResponse> => {
          const reviews = await getMyReviews(userId, cursor);
        
          return responseFromMyReviews(reviews);
        };
        ```
        
        1. 컨트롤러
        
        ```jsx
        export const handleListMyReviews = async (
          req: Request,
          res: Response,
          next: NextFunction
        ): Promise<void> => {
          try {
            const userId = parseInt(req.params.userId as string, 10);
        
            const cursor =
              typeof req.query.cursor === "string"
                ? parseInt(req.query.cursor, 10)
                : 0;
        
            const reviews = await listMyReviews(userId, cursor);
        
            res.status(StatusCodes.OK).json(reviews);
          } catch (err) {
            next(err);
          }
        };
        ```
        
    - 특정 가게의 미션 목록, 내가 진행 중인 미션 목록
        
        ### 특정 가게의 미션 목록 조회
        
        ```
        GET /api/v1/stores/:storeId/missions?cursor=0
        ```
        
        ### 내가 진행 중인 미션 목록 조회
        
        ```
        GET /api/v1/users/:userId/missions/ongoing?cursor=0
        ```
        
        1. response 형태 위한 dto
        
        ```jsx
        export interface StoreMissionListResponse {
          data: {
            missionId: number;
            storeId: number | null;
            content: string | null;
            point: number | null;
            deadline: Date | null;
          }[];
          pagination: {
            cursor: number | null;
          };
        }
        
        export interface OngoingMissionListResponse {
          data: {
            userMissionId: number;
            userId: number | null;
            missionId: number | null;
            storeId: number | null;
            storeName: string | null;
            content: string | null;
            point: number | null;
            deadline: Date | null;
            status: "READY" | "DONE" | null;
          }[];
          pagination: {
            cursor: number | null;
          };
        }
        
        export const responseFromStoreMissions = (
          missions: any[]
        ): StoreMissionListResponse => {
          const data = missions.map((mission) => ({
            missionId: mission.missionId,
            storeId: mission.storeId,
            content: mission.content,
            point: mission.point,
            deadline: mission.deadline,
          }));
        
          const lastMission = data[data.length - 1];
        
          return {
            data,
            pagination: {
              cursor: lastMission ? lastMission.missionId : null,
            },
          };
        };
        
        export const responseFromOngoingMissions = (
          userMissions: any[]
        ): OngoingMissionListResponse => {
          const data = userMissions.map((userMission) => ({
            userMissionId: userMission.userMissionId,
            userId: userMission.userId,
            missionId: userMission.missionId,
            storeId: userMission.mission?.storeId ?? null,
            storeName: userMission.mission?.store?.name ?? null,
            content: userMission.mission?.content ?? null,
            point: userMission.mission?.point ?? null,
            deadline: userMission.mission?.deadline ?? null,
            status: userMission.status,
          }));
        
          const lastUserMission = data[data.length - 1];
        
          return {
            data,
            pagination: {
              cursor: lastUserMission ? lastUserMission.userMissionId : null,
            },
          };
        };
        ```
        
        2. findMany로 조회하기
        
        ```jsx
        export const getStoreMissions = async (
          storeId: number,
          cursor: number
        ) => {
          return await prisma.mission.findMany({
            where: {
              storeId,
              missionId: {
                gt: cursor,
              },
            },
            orderBy: {
              missionId: "asc",
            },
            take: 5,
          });
        };
        
        export const getOngoingUserMissions = async (
          userId: number,
          cursor: number
        ) => {
          return await prisma.userMission.findMany({
            where: {
              userId,
              status: "READY",
              userMissionId: {
                gt: cursor,
              },
            },
            include: {
              mission: {
                include: {
                  store: true,
                },
              },
            },
            orderBy: {
              userMissionId: "asc",
            },
            take: 5,
          });
        };
        ```
        
        3. 서비스로 get함수들 호출
        
        ```jsx
        export const listStoreMissions = async (
          storeId: number,
          cursor: number
        ): Promise<StoreMissionListResponse> => {
          const missions = await getStoreMissions(storeId, cursor);
        
          return responseFromStoreMissions(missions);
        };
        
        export const listOngoingUserMissions = async (
          userId: number,
          cursor: number
        ): Promise<OngoingMissionListResponse> => {
          const userMissions = await getOngoingUserMissions(userId, cursor);
        
          return responseFromOngoingMissions(userMissions);
        };
        ```
        
        1. 컨트롤러
        
        ```jsx
        export const handleListStoreMissions = async (
          req: Request,
          res: Response,
          next: NextFunction
        ): Promise<void> => {
          try {
            const storeId = parseInt(req.params.storeId as string, 10);
        
            const cursor =
              typeof req.query.cursor === "string"
                ? parseInt(req.query.cursor, 10)
                : 0;
        
            const missions = await listStoreMissions(storeId, cursor);
        
            res.status(StatusCodes.OK).json(missions);
          } catch (err) {
            next(err);
          }
        };
        
        export const handleListOngoingUserMissions = async (
          req: Request,
          res: Response,
          next: NextFunction
        ): Promise<void> => {
          try {
            const userId = parseInt(req.params.userId as string, 10);
        
            const cursor =
              typeof req.query.cursor === "string"
                ? parseInt(req.query.cursor, 10)
                : 0;
        
            const missions = await listOngoingUserMissions(userId, cursor);
        
            res.status(StatusCodes.OK).json(missions);
          } catch (err) {
            next(err);
          }
        };
        ```

https://www.notion.so/makeus-challenge/Chapter-6-ORM-350b57f4596b8142b7bbfa985c78927c
ㄴ 노션 링크