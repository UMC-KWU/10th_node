1. 내가 진행 중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

SELECT mission.content, mission.point, store.name, user_mission.status
FROM mission
JOIN store ON mission.store_id = store.store_id
JOIN user_mission ON mission.mission_id = user_mission.mission_id
WHERE user_mission.user_id = 1 # 내 user_id라 할 때
  AND user_mission.status = '진행중'
  AND CONCAT(LPAD(UNIX_TIMESTAMP(user_mission.created_at), 20, '0'), LPAD(user_mission.user_mission_id, 10, '0')) 
     < (SELECT CONCAT(LPAD(UNIX_TIMESTAMP(user_mission.created_at), 20, '0'), LPAD(user_mission.user_mission_id, 10, '0'))
       FROM user_mission
       WHERE user_mission.user_mission_id = 3)  # 커서 id 3이라 할 때
ORDER BY user_mission.created_at DESC # 미션 수행 일시 기준
LIMIT 10

- 1번 구현 과정
    1. 기본
    SELECT *
    FROM mission
    2. 가게 이름 보여주기 (store JOIN하기)
    SELECT *
    FROM mission
    JOIN store ON mission.store_id = store.store_id
    3. 성공여부 보여주기 (user_mission JOIN하기)
    SELECT *
    FROM mission
    JOIN store ON mission.store_id = store.store_id
    JOIN user_mission ON mission.mission_id = user_mission.mission_id
    4. 필요한 값만 select (가게 이름, 미션 내용, 적립포인트, 미션 상태)
    SELECT mission.content, mission.point, [store.name](http://store.name/), user_mission.status
    FROM mission
    JOIN store ON mission.store_id = store.store_id
    JOIN user_mission ON mission.mission_id = user_mission.mission_id
    5. 진행 중(진행 완료)만 필터링, 내 미션만 필터링
    SELECT mission.content, mission.point, [store.name](http://store.name/), user_mission.status
    FROM mission
    JOIN store ON mission.store_id = store.store_id
    JOIN user_mission ON mission.mission_id = user_mission.mission_id
    WHERE user_mission.user_id = 1 # 내 user_id
    AND user_mission.status = '진행중' # 클라이언트가 보낸 status 값
    6. 페이징네이션 (내릴 수록 보이게)
    SELECT mission.content, mission.point, [store.name](http://store.name/), user_mission.status
    FROM mission
    JOIN store ON mission.store_id = store.store_id
    JOIN user_mission ON mission.mission_id = user_mission.mission_id
    WHERE user_mission.user_id = 1 # 내 user_id라 할 때
    AND user_mission.status = '진행중'
    AND CONCAT(LPAD(UNIX_TIMESTAMP(user_mission.created_at), 20, '0'), LPAD(user_mission.user_mission_id, 10, '0'))
    < (SELECT CONCAT(LPAD(UNIX_TIMESTAMP(m.created_at), 20, '0'), LPAD(m.user_mission_id, 10, '0'))
    FROM user_mission m #바깥 쿼리랑 헷갈릴까봐 별칭
    WHERE m.user_mission_id = 3) # 커서 id 3이라 가정
    ORDER BY user_mission.created_at DESC # 미션 수행 일시 기준으로 구현함
    LIMIT 15

2. 리뷰 작성하는 쿼리 (사진의 경우는 일단 배제)

INSERT INTO review (store_id, star, content, user_id, created_at, updated_at)
VALUES (1, 5, '맛있음', 3, NOW(), NOW()) # 클라이언트가 보내준 값

- 2번 구현 과정
    
    INSERT INTO review (store_id, star, content, user_id, created_at, updated_at)
    VALUES (1, 5, '맛있음', 3, NOW(), NOW()) # 클라이언트가 보내준 값

- INSERT
    
    INSERT INTO 테이블명 (컬럼1, 컬럼2, 컬럼3)
    VALUES (값1, 값2, 값3)
    
    컬럼이랑 값이 순서대로 매칭됨
    컬럼1 → 값1
    컬럼2 → 값2
    컬럼3 → 값3
    
    - PK는 안 써도 됨
    - 문자열은 따옴표
    - 숫자는 따옴표 없이
    - 현재 시간은 NOW()

- 추가) 리뷰 조회 쿼리 ..
    1. 기본
    SELECT *
    FROM review
    2. 닉네임 필요 (user JOIN하기)
    SELECT *
    FROM review
    JOIN user ON review.user_id = user.user_id
    3. 필요한 거 뽑아내기 (닉네임, 별점, 리뷰 내용, 사진-배제, 생성일시)
    SELECT user.nickname, review.star, review.content, review.created_at
    FROM review
    JOIN user ON review.user_id = user.user_id
    4. 해당 가게 리뷰만 보이게
    SELECT user.nickname, review.star, review.content, review.created_at
    FROM review
    JOIN user ON review.user_id = user.user_id
    WHERE review.store_id = 1 # 해당 가게 리뷰만
    5. 포맷팅
    SELECT user.nickname, review.star, review.content, DATE_FORMAT(review.created_at, '%Y.%m.%d') AS created_at
    FROM review
    JOIN user ON review.user_id = user.user_id
    WHERE review.store_id = 1 # 해당 가게 리뷰만
    + 날짜 수정되는 건 안함!

3. 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

SELECT region.name, store.name, store.category, 
       mission.deadline, mission.point, mission.content
FROM mission
JOIN store ON store.store_id = mission.store_id
JOIN region ON region.region_id = store.region_id
LEFT JOIN user_mission ON user_mission.mission_id = mission.mission_id
	AND user_mission.user_id = 3 # 현재 로그인한 유저 id
WHERE user_mission.user_mission_id IS NULL
	AND region.name = '안암동'  # 선택한 지역
	AND mission.mission_id < 3  # 커서 id
ORDER BY mission.mission_id DESC
LIMIT 10

- 3번 구현 과정
    1. 기본
    SELECT *
    FROM mission
    2. 현재 선택된 지역과 가게 추가 (region, store JOIN)
    SELECT *
    FROM mission
    JOIN store ON store.store_id = mission.store_id
    JOIN region ON region.region_id = store.region_id
    3. 필요한 값들 선택 (가게 지역, 가게 이름, 가게 카테고리, 미션 기한, 미션 시 적립 포인트, 미션 내용)
    SELECT [region.name](http://region.name/), [store.name](http://store.name/), store.category, mission.deadline, mission.point, mission.content
    FROM mission
    JOIN store ON store.store_id = mission.store_id
    JOIN region ON region.region_id = store.region_id
    4. 내꺼만, 유저가 수행하지 않은 미션만
    SELECT [region.name](http://region.name/), [store.name](http://store.name/), store.category,
    mission.deadline, mission.point, mission.content
    FROM mission
    JOIN store ON store.store_id = mission.store_id
    JOIN region ON region.region_id = store.region_id
    LEFT JOIN user_mission ON user_mission.mission_id = mission.mission_id
    AND user_mission.user_id = 3 # 현재 로그인한 유저 id
    WHERE user_mission.user_mission_id IS NULL
    →원래는 user_mission을 JOIN으로 받아왔는데, user_mission에 기록이 있는 미션만 JOIN이 돼서 LEFT JOIN으로 수정! : mission 테이블을 기준으로 다 가져오고 user_mission에 매칭되는 row가 없으면 나머지를 NULL로 채우고 IS NULL로 진행 안한 미션인지 판단해야해서
    5. 현재 선택한 지역의 미션만
    SELECT [region.name](http://region.name/), [store.name](http://store.name/), store.category,
    mission.deadline, mission.point, mission.content
    FROM mission
    JOIN store ON store.store_id = mission.store_id
    JOIN region ON region.region_id = store.region_id
    LEFT JOIN user_mission ON user_mission.mission_id = mission.mission_id
    AND user_mission.user_id = 3 # 현재 로그인한 유저 id
    WHERE user_mission.user_mission_id IS NULL
    AND [region.name](http://region.name/) = '안암동' # 선택한 지역
    6. 페이징
    SELECT [region.name](http://region.name/), [store.name](http://store.name/), store.category,
    mission.deadline, mission.point, mission.content
    FROM mission
    JOIN store ON store.store_id = mission.store_id
    JOIN region ON region.region_id = store.region_id
    LEFT JOIN user_mission ON user_mission.mission_id = mission.mission_id
    AND user_mission.user_id = 3 # 현재 로그인한 유저 id
    WHERE user_mission.user_mission_id IS NULL
    AND [region.name](http://region.name/) = '안암동' # 선택한 지역
    AND mission.mission_id < 3 # 커서 id
    ORDER BY mission.mission_id DESC
    LIMIT 10
    →아직 도전 안한 미션이라서 mission_id 기준으로

4. 마이 페이지 화면 쿼리

SELECT user.nickname, user.email, user.phone_number, user.point
FROM user
WHERE user.user_id = 1 # 내 user_id

- 4번 구현 과정
    1. 기본
    SELECT *
    FROM user
    2. 가져올 값 선택
    SELECT user.nickname, user.email, user.phone_number, user.point
    FROM user
    3. 내 정보만
    SELECT user.nickname, user.email, user.phone_number, user.point
    FROM user
    WHERE user.user_id = 1 # 내 user_id

노션 링크
https://www.notion.so/makeus-challenge/Chapter-2-SQL-Query-31fb57f4596b8162a699c5229f1306fc