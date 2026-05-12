GitHub 저장소 주소
https://github.com/Hagyeong13/umc_node

- 미션 기록
    - 1-1. 특정 지역에 가게 추가하기 API
        
        이전에 erd 기반으로 만들었던 table
        
        ![image.png](attachment:cab16363-4bda-4468-9432-c6fe390cc7c6:image.png)
        
        일단 api 설계부터
        
        POST /api/vi/regions/:regionId/stores
        
        ⇒ 특정 regionId 지역에 store를 추가한다
        
        ```jsx
        - Headers
        Content-Type: application/json
        (우선 인증 없는 구조라 Authorization: Bearer accessToken 제외 시킴)
        
        - Path Variable
        regionId: number
        
        - Request Body
        {
          "name": "제리식당",
          "ceoNumber": 12345678,
          "category": "한식"
          //이미 URL에 지역 정보가 있어서 body에는 가게 정보만 넣으면 됨
        }
        
        - Response
        {
          "result": {
            "storeId": 1,
            "regionId": 1,
            "name": "제리식당",
            "ceoNumber": 12345678,
            "category": "한식",
            "createdAt": store.created_at,
            "updatedAt": store.updated_at,
          }
        }
        ```
        
        이후 진행해야하는거
        
        - route
        POST /regions/:regionId/stores
        - controller
        req.params.regionId 가져오기
        req.body 가져오기
        service 호출
        - dto
        bodyToStore()로 body 정리
        - service
        regionId가 존재하는 지역인지 확인
        store 추가
        추가된 store 다시 조회
        responseFromStore()로 응답 형태 만들기
        - repository
        getRegion(regionId)
        addStore(regionId, data)
        getStore(storeId)
        
        1. store.dto.ts로 request, response 형태 맞추는 게 우선이라고 생각하여 아래의 코드 구현
        
        ```jsx
        export interface CreateStoreRequest {
          name: string;
          ceoNumber: number;
          category: string;
        }
        
        export interface StoreRow {
          store_id: number;
          region_id: number;
          name: string;
          ceo_number: number;
          category: string;
          created_at: string;
          updated_at: string;
        }
        
        export const bodyToStore = (body: CreateStoreRequest) => {
          return {
            name: body.name,
            ceoNumber: body.ceoNumber,
            category: body.category,
          };
        };
        
        export const responseFromStore = (store: StoreRow) => {
          return {
            storeId: store.store_id,
            regionId: store.region_id,
            name: store.name,
            ceoNumber: store.ceo_number,
            category: store.category,
            createdAt: store.created_at,
            updatedAt: store.updated_at,
          };
        };
        ```
        
        bodyToStore를 통해서 다듬어주기
        
        1. repository로 db
        
        // 1. 지역이 존재하는지 확인
        // 2. 가게 추가
        // 3. 추가된 가게 정보 응답
        
        이 세개가 필요할 거라고 생각해 아래와 같이 구현
        
        ```jsx
        import { ResultSetHeader, RowDataPacket } from "mysql2";
        import { pool } from "../../../db.config.js";
        import { CreateStoreRequest } from "../dtos/store.dto.js";
        
        // 1. 지역이 존재하는지 확인
        export const getRegion = async (regionId:number) : Promise<any | null> => {
            const [regions] = await pool.query<RowDataPacket[]>(
                `SELECT * FROM region WHERE region_id = ?;`,
                [regionId]
            );
        
            if(regions.length === 0)
                return null;
        
            return regions[0];
        };
        
        // 2. 가게 추가
        export const addStore = async (regionId : number, data:CreateStoreRequest) => {
            const [stores] = await pool.query<ResultSetHeader>(
                `INSERT INTO store (region_id, name, ceo_number, category, created_at, updated_at) VALUES (?,?,?,?);`,
                [regionId, data.name, data.ceoNumber, data.category]
            );
        
            return stores.insertId;
        }
        
        // 3. 추가된 가게 정보 응답
        export const getStore = async (storeId: number): Promise<any | null> => {
          const [stores] = await pool.query<RowDataPacket[]>(
            `SELECT * FROM store WHERE store_id = ?;`,
            [storeId]
          );
        
          if (stores.length === 0) {
            return null;
          }
        
          return stores[0];
        };
        ```
        
        1. 이제 가게 생성 - service
        
        // 가게 추가
        // 1) 지역 id 받은 거 이미 존재하는지
        // 2) 없음 에러 있음 addStore
        // 3) 응답으로 추가된 가게 정보 전달
        
        위 순서로 진행
        
        ```jsx
        // 가게 추가
        // 1) 지역 id 받은 거 이미 존재하는지
        // 2) 없음 에러 있음 addStore
        // 3) 응답으로 추가된 가게 정보 전달
        
        import { CreateStoreRequest, responseFromStore } from "../dtos/store.dto.js";
        import { addStore, getRegion, getStore } from "../repositories/store.repository.js";
        
        export const createStore = async (regionId : number, data : CreateStoreRequest) => {
            //1) 지역 id 받은 거 이미 존재하는지
            const region = await getRegion(regionId);
        
            //2) 없음 에러
            if(region===null) {
                throw new Error("없는 지역");
            }
        
            //2) 있음 addStore
            const storeId = await addStore(regionId, data);
        
            //3) 응답으로 추가된 가게 정보 전달
            const store = await getStore(storeId);
        
            if (store === null) {
                throw new Error("가게 생성 후 조회에 실패했습니다.");
            }
        
            return responseFromStore(store);
        };
        ```
        
        1. controller
        
        // 핸들러 만들기
        // 1) regionId 꺼내기
        // 2) body를 DTO로 정리해서 service에 넘기기
        // 3) 성공 응답 보내기
        
        ```jsx
        import { Request, Response, NextFunction } from "express";
        import { StatusCodes } from "http-status-codes";
        import { createStore } from "../services/store.service.js";
        import { bodyToStore, CreateStoreRequest } from "../dtos/store.dto.js";
        
        // 핸들러 만들기
        // 1) regionId 꺼내기
        // 2) body를 DTO로 정리해서 service에 넘기기
        // 3) 성공 응답 보내기
        
        export const handleCreateStore= async (req: Request, res: Response, next: NextFunction ) => {
            try {
            // 1) regionId 꺼내기
            const regionId = Number(req.params.regionId);
        
            // 2) body를 DTO로 정리해서 service에 넘기기
            const store = await createStore(
              regionId,
              bodyToStore(req.body as CreateStoreRequest)
            );
        
            // 3) 성공 응답 보내기
            res.status(StatusCodes.OK).json({
              result: store,
            });
        
          } catch (err) {
            next(err);
          }
        }
        
        ```
        
        1. 마지막 router
        
        app.post("/api/v1/regions/:regionId/stores", handleCreateStore);
        를 index.ts 에 추가하기!
        
        테스트
        
        ![image.png](attachment:90325c73-0c69-4745-b37f-41a5ebb4b46c:image.png)
        
        우선 region 하나 추가해서 등록함!
        
        ![image.png](attachment:b9dbbd59-9fe1-42b0-9d90-561457abf8bf:image.png)
        
        성공한 결과
        
        ![image.png](attachment:cc106ca8-a2d8-4867-a249-a1e7daa7feb4:image.png)
        
        없는 지역을 넣을 경우
        
    - **1-2. 가게에 리뷰 추가하기 API**
        - 리뷰를 추가하려는 가게가 존재하는지 검증이 필요합니다.
        
        1-1와 같은 흐름으로 진행할 예정
        
        ![image.png](attachment:02cb2cfe-758a-4051-b9e5-826f81a5ab61:image.png)
        
        일단 api 설계부터
        
        POST /api/v1/stores/:storeId/reviews
        
        ⇒ 특정 storeId 에 reviews를 추가한다
        
        ```jsx
        - Headers
        Content-Type: application/json
        (우선 인증 없는 구조라 Authorization: Bearer accessToken 제외 시킴)
        
        - Path Variable
        storeId: number
        
        - Request Body
        {
          "userId": 1,
          "star": 5,
          "content": "맛있어요",
          "picture": "https://example.com/review.jpg"
        }
        
        - Response
        {
          "result": {
            "reviewId": 1,
            "userId": 1,
            "storeId": 3,
            "star": 5,
            "content": "맛있어요",
            "picture": "https://example.com/review.jpg",
            "createdAt": "2026-05-12T10:00:00.000Z",
            "updatedAt": "2026-05-12T10:00:00.000Z"
          }
        }
        ```
        
        1. review.dto.ts에서 request body, response body 구조잡기
        
        ```jsx
        export interface CreateReviewRequest {
          userId: number;
          star: number;
          content: string;
          picture?: string;
        }
        
        export interface ReviewRow {
          review_id: number;
          user_id: number;
          store_id: number;
          star: number;
          content: string;
          picture: string | null;
          created_at: string;
          updated_at: string;
        }
        
        export const bodyToReview = (body: CreateReviewRequest) => {
          return {
            userId: body.userId,
            star: body.star,
            content: body.content,
            picture: body.picture || null,
          };
        };
        
        export const responseFromReview = (review: ReviewRow) => {
          return {
            reviewId: review.review_id,
            userId: review.user_id,
            storeId: review.store_id,
            star: review.star,
            content: review.content,
            picture: review.picture,
            createdAt: review.created_at,
            updatedAt: review.updated_at,
          };
        };
        ```
        
         2. 쿼리 작성
        
        ```jsx
        // 리뷰 달기
        
        import { ResultSetHeader, RowDataPacket } from "mysql2"
        import { pool } from "../../../db.config.js"
        import { CreateReviewRequest } from "../dtos/review.dto.js";
        
        //1) 리뷰 추가 전 가게 존재 확인
        export const getStore = async (storeId:number) : Promise<any|null> => {
            const [stores] = await pool.query<RowDataPacket[]>(
                `SELECT * FROM store WHERE store_id = ?;`,
                [storeId]
            );
        
            if(stores.length===0)
                return null;
        
            return stores[0];
        }
        
        //2) 리뷰 저장
        export const addReview = async ( storeId: number, data: CreateReviewRequest ): Promise<number> => {
          const [result] = await pool.query<ResultSetHeader>(
            `INSERT INTO review (user_id, store_id, star, content, picture)
             VALUES (?, ?, ?, ?, ?);`,
            [data.userId, storeId, data.star, data.content, data.picture || null]
          );
        
          return result.insertId;
        };
        
        //3) 리뷰 반환
        export const getReview = async (reviewId : number) : Promise<any|null> => {
            const [reviews] = await pool.query<RowDataPacket[]>(
                `SELECT * FROM review WHERE review_id = ?;`,
                [reviewId]
            );
        
            if(reviews.length===0)
                return null;
        
            return reviews[0];
        }  
        ```
        
        1. 로직 작성
        
        1) storeId로 가게 조회
        
        2) 없으면 에러
        
        3) 있으면 review 추가
        
        4) 생성된 reviewId로 다시 조회
        
        ```jsx
        import { CreateReviewRequest, responseFromReview } from "../dtos/review.dto.js";
        import { addReview, getReview, getStore } from "../repositories/review.repository.js";
        
        export const createReview = async (storeId:number,data:CreateReviewRequest) => {
            
            //리뷰 가게 있는지 확인
            const store = await getStore(storeId);
            if(store===null)
                throw new Error("존재하지 않는 가게입니다.");
        
            //리뷰 추가
            const reviewId = await addReview(storeId, data);
            
            //추가한 거 조회
            const review = await getReview(reviewId);
            if(review===null)
                throw new Error("리뷰 생성 후 조회 실패했습니다.");
        
            return responseFromReview(review);
        }
        ```
        
        1. 컨트롤러 1-1랑 유사
        
        ```jsx
        import { Request, Response, NextFunction } from "express";
        import { StatusCodes } from "http-status-codes";
        import { createReview } from "../services/review.service.js";
        import { bodyToReview, CreateReviewRequest } from "../dtos/review.dto.js";
        
        // 핸들러 만들기
        // 1) storeId 꺼내기
        // 2) body를 DTO로 정리해서 service에 넘기기
        // 3) 성공 응답 보내기
        
        export const handleCreateReview= async (req: Request, res: Response, next: NextFunction ) => {
            try {
            // 1) storeId 꺼내기
            const storeId = Number(req.params.storeId);
        
            // 2) body를 DTO로 정리해서 service에 넘기기
            const review = await createReview(
              storeId,
              bodyToReview(req.body as CreateReviewRequest)
            );
        
            // 3) 성공 응답 보내기
            res.status(StatusCodes.OK).json({
              result: review,
            });
        
          } catch (err) {
            next(err);
          }
        }
        
        ```
        
        테스트
        
        ![image.png](attachment:e06ab5b4-ddf1-4fcb-80f1-166d29b4bbfa:image.png)
        
        ![image.png](attachment:70029dad-7862-4a44-8f86-7ac3111ed63b:image.png)
        
        1-1에서 만들었던 store에 리뷰를 달았다.
        
    - **1-4. 가게의 미션을 도전 중인 미션에 추가(미션 도전하기) API**
        
        ![image.png](attachment:2b62b905-da26-47db-8a69-1fa4a1d0c73c:image.png)
        
        ![image.png](attachment:1e8104e4-43ac-4f2e-9edf-a0fbde2432af:image.png)
        
        똑같이..
        POST /api/v1/missions/:missionId/challenge
        
        ```jsx
        - Headers
        Content-Type: application/json
        (우선 인증 없는 구조라 Authorization: Bearer accessToken 제외 시킴)
        
        - Path Variable
        missionId: number
        
        - Request Body
        {
          "userId": 6
        }
        
        - Response
        {
          "result": {
            "userMissionId": 1,
            "userId": 6,
            "missionId": 1,
            "status": "READY",
            "createdAt": "2026-05-12T...",
            "updatedAt": "2026-05-12T..."
          }
        }
        ```
        
        생각..
        
        ```jsx
        1. missionId에 해당하는 미션이 존재하는지 확인
        2. userId에 해당하는 유저가 존재하는지 확인
        3. 이미 user_mission에 같은 user_id + mission_id가 있는지 확인
        4. 없으면 user_mission에 INSERT
        5. 생성된 user_mission 다시 조회
        6. 응답 반환
        ```
        
        1. dtos
        
        위 erd랑 request, response 맞게 구현
        
        ```jsx
        export interface ChallengeMissionRequest {
          userId: number;
        }
        
        export interface UserMissionRow {
          user_mission_id: number;
          user_id: number;
          mission_id: number;
          status: "READY" | "DONE";
          created_at: string;
          updated_at: string;
        }
        
        export const bodyToChallengeMission = (body: ChallengeMissionRequest) => {
          return {
            userId: body.userId,
          };
        };
        
        export const responseFromUserMission = (userMission: UserMissionRow) => {
          return {
            userMissionId: userMission.user_mission_id,
            userId: userMission.user_id,
            missionId: userMission.mission_id,
            status: userMission.status,
            createdAt: userMission.created_at,
            updatedAt: userMission.updated_at,
          };
        };
        ```
        
        1. db 구현
        
        1) 미션 존재 확인 2) 도전 중인지 3) 미션 추가 4) 추가한 거 조회
        
        ```jsx
        import { ResultSetHeader, RowDataPacket } from "mysql2";
        import { pool } from "../../../db.config.js";
        import { ChallengeMissionRequest } from "../dtos/mission.dto.js";
        
        // 1. 미션 존재
        export const getMission = async (missionId:number):Promise<any|null>=>{
            const [missions]=await pool.query<RowDataPacket[]>(
                `SELECT * FROM mission WHERE missionId = ?;`,
                [missionId]
            );
        
            if(missions.length===0)
                return null;
        
            return missions[0];
        }
        
        // 2. 도전 중인가?
        export const getUserMission = async (userId:number, missionId : number) : Promise<any | null> => {
            const [userMissions] = await pool.query<RowDataPacket[]>(
                `SELECT * FROM user_mission WHERE user_id = ? AND mission_id = ?;`,
                [userId,missionId]
            );
            if(userMissions.length===0)
                return null;
            return userMissions[0];
        }
        
        // 3. 추가
        export const addUserMission = async (
          missionId: number,
          data: ChallengeMissionRequest
        ): Promise<number> => {
          const [result] = await pool.query<ResultSetHeader>(
            `INSERT INTO user_mission (user_id, mission_id, status)
             VALUES (?, ?, ?);`,
            [data.userId, missionId, "READY"]
          );
        
          return result.insertId;
        };
        
        // 4. 추가된 도전 미션 조회
        export const getUserMissionById = async (
          userMissionId: number
        ): Promise<any | null> => {
          const [userMissions] = await pool.query<RowDataPacket[]>(
            `SELECT * FROM user_mission 
             WHERE user_mission_id = ?;`,
            [userMissionId]
          );
        
          if (userMissions.length === 0) {
            return null;
          }
        
          return userMissions[0];
        };
        ```
        
        1. 서비스 로직
        
        ```jsx
        import { ChallengeMissionRequest, responseFromUserMission } from "../dtos/mission.dto.js";
        import { addUserMission, getMission, getUserMission, getUserMissionById } from "../repositories/mission.repository.js";
        
        export const challengeMission = async ( missionId: number, data: ChallengeMissionRequest) => {
          // 1. 미션 존재 여부 확인
          const mission = await getMission(missionId);
        
          if (mission === null) {
            throw new Error("존재하지 않는 미션입니다.");
          }
        
          // 2. 이미 도전 중인지 확인
          const userMission = await getUserMission(data.userId,missionId);
        
          if (userMission !== null) {
            throw new Error("이미 도전 중인 미션입니다.");
          }
        
          // 3. user_mission 추가
          const userMissionId = await addUserMission(missionId,data);
        
          // 4. 추가된 user_mission 조회
          const createdUserMission = await getUserMissionById(
            userMissionId
          );
        
          if (createdUserMission === null) {
            throw new Error("도전 미션 조회에 실패했습니다.");
          }
        
          // 5. 응답 형태 변환
          return responseFromUserMission(createdUserMission);
        };
        ```
        
        1. 컨트롤러
        
        ```jsx
        export const handleCreateStore= async (req: Request, res: Response, next: NextFunction ) => {
            try {
            // 1) missionId 꺼내기
            const missionId = Number(req.params.missionId);
        
            // 2) body를 DTO로 정리해서 service에 넘기기
            const userMission = await challengeMission(
              missionId,
              bodyToChallengeMission(req.body as ChallengeMissionRequest)
            );
        
            // 3) 성공 응답 보내기
            res.status(StatusCodes.OK).json({
              result: userMission,
            });
        
          } catch (err) {
            next(err);
          }
        }
        
        ```
        
        테스트
        
        ![image.png](attachment:20c15480-4a52-4036-95c4-990713ff4aad:image.png)
        
        ![image.png](attachment:7cedd354-dd0b-49a4-a1b3-0469a8ede0fb:image.png)

https://www.notion.so/makeus-challenge/Chapter-5-API-350b57f4596b81899651f0214ce61fe5
ㄴ 노션 링크