#### 1 + 2. API 추가 구현 및 **Controller → Service → Repository → DB로 이어지는 요청 흐름 정리**

- 1-1. 특정 지역에 가게 추가하기
    
    **1단계: API 설계 및 엔드포인트 정의**
    
    - 기능: 특정 지역(Region)에 새로운 가게(Store)를 추가
    - Method: `POST`
    - Endpoint: `/api/v1/regions/{regionId}/stores`
    - Path Variable: regionIde (지역 ID)
    - Requset Body:
    
    ```json
    {
      "name": "광운대카츠",
      "address": "서울시 노원구 광운로 12",
      "category": "일식"
    }
    ```
    
    - Response Body:
    
    ```json
    {
      "success": true,
      "code": "S200",
      "message": "가게가 성공적으로 추가되었습니다.",
      "data": { "storeId": 1 }
    }
    ```
    
    **2단계: DTO 및 유효성 검사**
    
    - 가게 추가 시 필요한 요청 데이터의 규격을 정의하는 DTO
    
    `src/modules/stores/dtos/store.dto.ts`
    
    ```tsx
    export interface CreateStoreRequestDto {
      name: string;
      address: string;
      categoryId: number; // ERD의 category_id와 매칭
    }
    ```
    
    **3단계: Controller 계층**
    
    - 클라이언트의 요청을 받아 Service 계층으로 데이터를 넘겨주는 역할
    
    `src/modules/stores/controllers/store.controller.ts`
    
    ```tsx
    import { Request, Response, NextFunction } from "express";
    import { createStore } from "../services/store.services.js"; // 확장자 .js 확인!
    import { CreateStoreRequestDto } from "../dtos/store.dtos.js";
    
    export const handleAddStore = async (req: Request, res: Response, next: NextFunction) => {
      try {
        const { regionId } = req.params;
    
        // 🚨 타입 가드: regionId가 없거나, 배열(string[])인 경우를 제외함
        if (!regionId || typeof regionId !== "string") {
          res.status(400).json({
            success: false,
            message: "유효한 지역 ID가 요청 경로에 포함되지 않았습니다."
          });
          return;
        }
    
        const result = await createStore(parseInt(regionId), req.body as CreateStoreRequestDto);
    
        res.status(200).json({
          success: true,
          code: "S200",
          message: "가게 등록 성공!",
          data: result,
        });
      } catch (error: any) {
        res.status(500).json({
          success: false,
          message: error.message
        });
      }
    };
    ```
    
    **4단계: Service 계층**
    
    - 비즈니스 로직을 처리하는 계층으로, 지역 존재 여부 등을 검증한 뒤 Repository를 호출한다.
    
    `src/modules/stores/services/store.service.ts`
    
    ```tsx
    import { addStore, checkRegionExists } from "../repositories/store.repositories";
    import { CreateStoreRequestDto } from "../dtos/store.dtos";
    
    export const createStore = async (regionId: number, storeData: CreateStoreRequestDto) => {
      const isExist = await checkRegionExists(regionId);
      if (!isExist) {
        throw new Error("존재하지 않는 지역 ID입니다.");
      }
    
      const storeId = await addStore(regionId, storeData);
      return { storeId };
    };
    ```
    
    **5단계: Repository 계층**
    
    - 데이터베이스와 직접 통신하며 쿼리문을 실행하는 계층
    
    `src/modules/stores/repositories/store.repository.ts`
    
    ```tsx
    import { pool } from "../../../db.config.js";
    import { ResultSetHeader, RowDataPacket } from "mysql2";
    import { CreateStoreRequestDto } from "../dtos/store.dtos.js";
    
    // 1. 지역 존재 여부 확인
    export const checkRegionExists = async (regionId: number): Promise<boolean> => {
      const [rows] = await pool.query<RowDataPacket[]>(
        `SELECT EXISTS (SELECT 1 FROM region WHERE id = ?) as isExist;`,
        [regionId]
      );
      return rows[0]?.isExist === 1;
    };
    
    // 2. 가게 추가 (ERD 기준: region_id, category_id 포함)
    export const addStore = async (regionId: number, data: CreateStoreRequestDto): Promise<number> => {
      const [result] = await pool.query<ResultSetHeader>(
        `INSERT INTO store (region_id, category_id, name, address) VALUES (?, ?, ?, ?);`,
        [regionId, data.categoryId, data.name, data.address]
      );
      return result.insertId;
    };
    ```
    
    **6단계:** `index.ts` 라우트 등록 및 테스트
    
    `app.post("/api/v1/regions/:regionId/stores", handleAddStore);`
    
    ![가게 추가 포스트맨](./가게%20추가%20API%20테스트.png)

    ![가게 추가 DB](./가게%20추가%20API%20테스트%20DB.png)
    
- 1-2. 가게에 리뷰 추가하기
    
    **1단계: API 설계 및 엔드포인트 정의**
    
    - 기능: 특정 가게(store)에 리뷰 등록
    - Method: `POST`
    - Endpoint: `/api/v1/stores/:storeId/reviews`
    - Request Body
    
    ```json
    {
      "memberId": 1, 
      "score": 5,
      "content": "월계동 최고의 카츠 맛집이에요!"
    }
    ```
    
    **2단계: DTO 정의**
    
    `src/modules/stores/dtos/review.dto.ts`
    
    ```tsx
    export interface CreateReviewRequestDto {
      memberId: number;
      score: number;
      content: string;
    }
    ```
    
    **3단계: Repository 계층(DB 통신)**
    
    `src/modules/stores/repositories/review.repository.ts`
    
    ```tsx
    import { pool } from "../../../db.config.js";
    import { ResultSetHeader, RowDataPacket } from "mysql2";
    import { CreateReviewRequestDto } from "../dtos/review.dto.js";
    
    // 1. 가게 존재 여부 확인
    export const checkStoreExists = async (storeId: number): Promise<boolean> => {
      const [rows] = await pool.query<RowDataPacket[]>(
        `SELECT EXISTS (SELECT 1 FROM store WHERE id = ?) as isExist;`,
        [storeId]
      );
      return rows[0]?.isExist === 1;
    };
    
    // 2. 리뷰 추가 (ERD 기준: member_id, store_id 포함)
    export const addReview = async (storeId: number, data: CreateReviewRequestDto): Promise<number> => {
      const [result] = await pool.query<ResultSetHeader>(
        `INSERT INTO review (member_id, store_id, score, content) VALUES (?, ?, ?, ?);`,
        [data.memberId, storeId, data.score, data.content]
      );
      return result.insertId;
    };
    ```
    
    **4단계: Service 계층(비즈니스 로직)**
    
    `src/modules/stores/services/review.service.ts`
    
    ```tsx
    import { addReview, checkStoreExists } from "../repositories/review.repository.js";
    import { CreateReviewRequestDto } from "../dtos/review.dto.js";
    
    export const createReview = async (storeId: number, data: CreateReviewRequestDto) => {
      // 1. 가게 검증
      const isExist = await checkStoreExists(storeId);
      if (!isExist) {
        throw new Error("존재하지 않는 가게입니다.");
      }
    
      // 2. 리뷰 등록
      const reviewId = await addReview(storeId, data);
      return { reviewId };
    };
    ```
    
    **5단계: Controller 계층**
    
    `src/modules/stores/controllers/review.controller.ts`
    
    ```tsx
    import { Request, Response, NextFunction } from "express";
    import { createReview } from "../services/review.service.js";
    import { CreateReviewRequestDto } from "../dtos/review.dto.js";
    
    export const handleAddReview = async (req: Request, res: Response, next: NextFunction) => {
      try {
        const { storeId } = req.params;
    
        if (!storeId || typeof storeId !== "string") {
          res.status(400).json({ success: false, message: "유효한 가게 ID가 필요합니다." });
          return;
        }
    
        const result = await createReview(parseInt(storeId), req.body as CreateReviewRequestDto);
    
        res.status(200).json({
          success: true,
          code: "S200",
          message: "리뷰 등록 성공!",
          data: result,
        });
      } catch (error: any) {
        res.status(500).json({ success: false, message: error.message });
      }
    };
    ```
    
    **6단계: `index.ts` 라우트 등록 및 테스트**
    `app.post("/api/v1/stores/:storeId/reviews", handleAddReview);`
    
    ![리뷰 추가 포스트맨](./리뷰%20추가%20API%20테스트.png)

    ![리뷰 추가 DB](./리뷰%20추가%20API%20테스트%20DB.png)
    
- 1-3. 가게에 미션 추가하기
    
    **1단계: API 설계 및 엔드포인트 정의**
    
    - 기능: 특정 가게(store)에 새로운 미션을 추가
    - Method: `POST`
    - Endpoint: `/api/v1/stores/:storeId/missions`
    - Request Body
    
    ```json
    {
      "reward": 500,
      "deadline": "2026-05-10 23:59:59",
      "missionSpec": "치즈카츠 인증샷 올리기"
    }
    ```
    
    **2단계: DTO 정의**
    
    `src/modules/stores/dtos/mission.dto.ts`
    
    ```tsx
    export interface CreateMissionRequestDto {
      reward: number;
      deadline: string;
      missionSpec: string;
    }
    ```
    
    **3단계: Repository 계층 (ERD 컬럼명 준수)**
    
    `src/modules/stores/repositories/mission.repository.ts`
    
    ```tsx
    import { pool } from "../../../db.config.js";
    import { ResultSetHeader } from "mysql2";
    import { CreateMissionRequestDto } from "../dtos/mission.dto.js";
    
    // 가게에 미션 추가
    export const addMission = async (storeId: number, data: CreateMissionRequestDto): Promise<number> => {
      const [result] = await pool.query<ResultSetHeader>(
        `INSERT INTO mission (store_id, reward, deadline, mission_spec) VALUES (?, ?, ?, ?);`,
        [storeId, data.reward, data.deadline, data.missionSpec]
      );
      return result.insertId;
    };
    ```
    
    **4단계: Service 계층**
    
    `src/modules/stores/services/mission.service.ts`
    
    ```tsx
    import { addMission } from "../repositories/mission.repository.js";
    import { checkStoreExists } from "../repositories/review.repository.js"; // 기존 함수 재사용
    import { CreateMissionRequestDto } from "../dtos/mission.dto.js";
    
    export const createMission = async (storeId: number, data: CreateMissionRequestDto) => {
      const isExist = await checkStoreExists(storeId);
      if (!isExist) {
        throw new Error("미션을 등록하려는 가게가 존재하지 않습니다.");
      }
    
      const missionId = await addMission(storeId, data);
      return { missionId };
    };
    ```
    
    **5단계: Controller 계층**
    
    `src/modules/stores/controllers/mission.controller.ts`
    
    ```tsx
    import { Request, Response, NextFunction } from "express";
    import { createMission } from "../services/mission.service.js";
    import { CreateMissionRequestDto } from "../dtos/mission.dto.ts";
    
    export const handleAddMission = async (req: Request, res: Response, next: NextFunction) => {
      try {
        const { storeId } = req.params;
    
        if (!storeId || typeof storeId !== "string") {
          res.status(400).json({ success: false, message: "유효한 가게 ID가 필요합니다." });
          return;
        }
    
        const result = await createMission(parseInt(storeId), req.body as CreateMissionRequestDto);
    
        res.status(200).json({
          success: true,
          code: "S200",
          message: "가게 미션이 성공적으로 등록되었습니다!",
          data: result,
        });
      } catch (error: any) {
        res.status(500).json({ success: false, message: error.message });
      }
    };
    ```
    
    **6단계: 라우트 등록 및 테스트**
    
    `app.post("/api/v1/stores/:storeId/missions", handleAddMission);`
    
    ![미션 추가 포스트맨](./미션%20추가%20API%20테스트.png)
    
    ![미션 추가 DB](./미션%20추가%20API%20테스트%20DB.png)

- 1-4. 가게의 미션을 도전 중인 미션에 추가(미션 도전하기) API
    
    **1단계: API 설계 및 엔드포인트 정의**
    
    - 기능: 특정 미션에 대해 사용자가 '도전하기'를 눌렀을 때 등록
    - Method: `POST`
    - Endpoint: `/api/v1/missions/:missionId/challenges`
    - Request Body
    
    ```json
    {
      "memberId": 1
    }
    ```
    
    **2단계: Repository 계층 (중복 검증 로직 포함)**
    
    `src/modules/stores/repositories/mission.repository.ts`
    
    (기존 파일에 추가)
    
    ```tsx
    import { pool } from "../../../db.config.js";
    import { ResultSetHeader, RowDataPacket } from "mysql2";
    
    // 1. 이미 'CHALLENGING' 상태인 미션이 있는지 확인
    export const isMissionAlreadyChallenging = async (missionId: number, memberId: number): Promise<boolean> => {
      const [rows] = await pool.query<RowDataPacket[]>(
        `SELECT EXISTS (SELECT 1 FROM member_mission WHERE mission_id = ? AND member_id = ? AND status = 'CHALLENGING') as isExist;`,
        [missionId, memberId]
      );
      return rows[0]?.isExist === 1;
    };
    
    // 2. 미션 도전하기 추가 (status는 기본값이 CHALLENGING이라 생략 가능)
    export const addMemberMission = async (missionId: number, memberId: number): Promise<number> => {
      const [result] = await pool.query<ResultSetHeader>(
        `INSERT INTO member_mission (mission_id, member_id, status) VALUES (?, ?, 'CHALLENGING');`,
        [missionId, memberId]
      );
      return result.insertId;
    };
    ```
    
    3단계: Service 계층
    
    `src/modules/stores/services/mission.service.ts` (기존 파일에 추가)
    
    ```tsx
    import { addMemberMission, isMissionAlreadyChallenging } from "../repositories/mission.repository.js";
    
    export const challengeMission = async (missionId: number, memberId: number) => {
      // 중복 도전 검증
      const alreadyChallenging = await isMissionAlreadyChallenging(missionId, memberId);
      if (alreadyChallenging) {
        throw new Error("이미 도전 중인 미션입니다.");
      }
    
      const memberMissionId = await addMemberMission(missionId, memberId);
      return { memberMissionId };
    };
    ```
    
    4단계: Controller 계층
    
    `src/modules/stores/controllers/mission.controller.ts` (기존 파일에 추가)
    
    ```tsx
    export const handleChallengeMission = async (req: Request, res: Response, next: NextFunction) => {
      try {
        const { missionId } = req.params;
        const { memberId } = req.body;
    
        if (!missionId || typeof missionId !== "string") {
          res.status(400).json({ success: false, message: "유효한 미션 ID가 필요합니다." });
          return;
        }
    
        const result = await challengeMission(parseInt(missionId), memberId);
    
        res.status(200).json({
          success: true,
          code: "S200",
          message: "미션 도전 시작!",
          data: result,
        });
      } catch (error: any) {
        res.status(500).json({
          success: false,
          message: error.message // "이미 도전 중인 미션입니다." 
        });
      }
    };
    ```
    
    5단계: 라우트 등록 및 최종 테스트
    
    `app.post("/api/v1/missions/:missionId/challenges", handleChallengeMission);`
    
    - 최초 실행 화면
    
    ![미션 현황1 포스트맨](./미션%20현황%20API%20테스트.png)
    
    - 중복 실행 화면
    
    ![미션 현황2 포스트맨](./미션%20현황%20API%20테스트2.png)
    
    ![미션 현황 DB](./미션%20현황%20API%20테스트%20DB.png)
    

#### 3. 회원가입 API에 비밀번호 해싱 과정 추가

1. member 테이블에 password 칼럼 추가

![password 컬럼 추가 DB](./password%20컬럼%20추가.png)

1. 라이브러리 설치
- 해싱 엔진: `bcrypt`

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

1. 회원가입 API 코드 구현
- DTO 정의

```tsx
export interface SignUpRequestDto {
  email: string;
  name: string;
  password: string; // 사용자로부터 받을 평문 비밀번호
  gender: 'M' | 'F';
  address: string;
  regionId: number;
  phone_number: number;
}
```

- contorller

```tsx
import { Request, Response, NextFunction } from "express";
import { memberSignUp } from "../services/member.service.js"; // 서비스 파일명 확인!
import { SignUpRequestDto } from "../dtos/member.dto.js";

export const memberSignUpController = async (req: Request, res: Response, next: NextFunction) => {
  try {
    // 1. 요청 바디를 DTO 타입으로 받기
    const signUpData: SignUpRequestDto = req.body;

    // 2. 서비스 호출 
    const result = await memberSignUp(signUpData);

    // 3. 성공 응답
    res.status(201).json({
      success: true,
      code: "M201",
      message: "회원가입이 완료되었습니다.",
        data: { memberId: result },
    });
  } catch (error: any) {
    // 4. 에러 응답 (중복 이메일 등)
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
};
```

- services

```tsx
import bcrypt from "bcrypt";
import { addMember } from "../repositories/member.repository.js";

export const memberSignUp = async (data: SignUpRequestDto) => {
  // 1. 비밀번호 해싱 (Salt Round 10 사용)
  const saltRounds = 10;
  const hashedPassword = await bcrypt.hash(data.password, saltRounds);

  // 2. 평문 비번을 해시 비번으로 교체해서 DB에 저장
  const result = await addMember({
    ...data,
    password: hashedPassword
  });

  return result;
};
```

- repositories

```tsx
import { ResultSetHeader } from "mysql2";
import { pool } from "../../../db.config.js";

export const addMember = async (data: any) => {
  const [result] = await pool.query<ResultSetHeader>(
    `INSERT INTO member (region_id, name, gender, address, email, password, phone_number, status, point) 
     VALUES (?, ?, ?, ?, ?, ?, ?, 'ACTIVE', 0);`,
    [data.regionId, data.name, data.gender, data.address, data.email, data.password, data.phone_number]
  );
  return result.insertId;
};
```

![비밀번호 해싱 포스트맨](./비밀번호%20해싱%20테스트.png)

![비밀번호 해싱 DB](./비밀번호%20해싱%20테스트%20DB.png)