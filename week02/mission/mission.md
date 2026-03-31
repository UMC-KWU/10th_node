1. 미션(홈)
  
    #### 1) 진행중 미션 쿼리
  
    SELECT
        member_mission.id,
        member_mission.status,
        member_mission.started_at,
        mission.id,
        mission.title,
        mission.description,
        mission.reward_point,
        mission.end_at,
        DATEDIFF(mission.end_at, NOW()) AS d_day,
        store.id,
        store.name
    FROM member_mission
    JOIN mission
        ON member_mission.mission_id = mission.id
    JOIN store
        ON mission.store_id = store.id
    WHERE member_mission.member_id = 1  #일단 id=1
      AND member_mission.status = 'CHALLENGING'
    ORDER BY member_mission.started_at DESC
    LIMIT 10;
    
    
    진행중인 미션은 리뷰 작성이 불가능하다 → 따라서 진행 중과 진행 완료를 분리하여 작성하였다
    미션 정보와 가게 정보를 함께 표시하므로 mission, store테이블을 join하였고, 정렬은 최신순이라는 가정으로 completed_at을 기준으로 내림차순 정렬하였다.
    
    #### 2) 완료 미션 쿼리
  
    SELECT
        member_mission.id,
        member_mission.status,
        member_mission.completed_at,
        mission.id,
        mission.title,
        mission.description,
        mission.reward_point,
        store.id,
        store.name,
        review.id
    FROM member_mission
    JOIN mission
        ON member_mission.mission_id = mission.id
    JOIN store
        ON mission.store_id = store.id
    LEFT JOIN review
        ON review.member_mission_id = member_mission.id
    WHERE member_mission.member_id = 1
      AND member_mission.status = 'COMPLETED'
    ORDER BY member_mission.completed_at DESC
    LIMIT 10;
    
    완료된 미션에 대해서는 리뷰 작성이 가능하다. 나머지는 진행중인 미션과 동일하고, 리뷰 작성 여부를 확인하기 위해서 review테이블을 추가로 left join하였다.
    review.id가 null일때만(아직 리뷰를 남기지 않았음을 의미) 리뷰 남기기 버튼을 활성화 시켰다.
    
2. 점포관리
    
    리뷰 작성하는 쿼리,
    * 사진의 경우는 일단 배제
 
    SELECT
        review.id,
        member.nickname,
        review.rating,
        review.content,
        review.created_at
    FROM review
    JOIN member ON review.member_id = member.id
    JOIN member_mission ON review.member_mission_id = member_mission.id
    JOIN mission ON member_mission.mission_id = mission.id
    JOIN store ON mission.store_id = store.id
    WHERE store.id = 1
    ORDER BY review.created_at DESC;

    점포 관리 리뷰 화면은 특정 가게에 작성된 리뷰 목록을 조회하는 화면이다.
    지난주 erd에서 review테이블은 store을 직접 참조하지 않기 때문에 review→ member_review→ mission→ store경로를 통해
    조인하여 해당 리뷰가 어떤 가게에 대한 건지 알 수 있도록 하였다.
    
    리뷰 작성자의 정보를 표시해야 하므로 member테이블을 조인하였고, 그 중에 닉네임을 표시하도록 하였다. 
    
    리뷰 정렬 순서 또한 최신 리뷰순이라는 가정하에 review.created_at을 기준으로 내림차순 정렬하였다.
    
3. 홈
    
    홈 화면 쿼리
    (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
    
    #### 1) 홈 화면 상단
    
    SELECT
        region.id,
        region.name,
        member.point,
        COUNT(DISTINCT mission.id) as total_mission_count,#전체 미션 카운트
        COUNT(DISTINCT CASE
            WHEN member_mission.status = 'COMPLETED' THEN member_mission.mission_id
        END) as completed_mission_count #미션 상태가 완료인 미션 카운트
    FROM member
    JOIN store ON store.region_id = region.id
    JOIN mission ON mission.store_id = store.id
    LEFT JOIN member_mission ON member_mission.mission_id = mission.id AND member_mission.member_id = member.id
    WHERE member.id = 1 AND mission.start_at <= NOW() AND mission.end_at >= NOW() AND region.id = 1
    GROUP BY region.id, region.name, member.point;
    
    상단 요약 영역과 하단 미션 목록 영역으로 구성되어 있으며, 두 영역은 조회 목적이 다르기 때문에 각각 별도의 쿼리로 구성하였다.
    전체미션 수, 완료한 미션 수, 현재 보유 포인트를 보여주기 위해 region, store, mission테이블을 조인하여
    해당 지역에 속한 전체 미션을 조회하고 member_mission테이블을 통해 해당 회원이 완료한 미션 수를 집계하였다.
    CASE문: IF문같은 집계함수, DISTINCT: 중복제거
    mission테이블의 start_at, end_at컬럼을 이용해서 현재 진행중인 미션만 집계할 수 있게 조건을 설정하였다.
    
    #### 2) 하단 My mission
 
    SELECT
        mission.id,
        store.name,
        store.category,
        mission.title,
        mission.description,
        mission.reward_point,
        DATEDIFF(mission.end_at, NOW()) AS d_day, #남은 날짜 계산
        member_mission.status
    FROM mission
    JOIN store ON mission.store_id = store.id
    JOIN region ON store.region_id = region.id
    LEFT JOIN member_mission ON member_mission.mission_id = mission.id AND member_mission.member_id = 1
    WHERE region.id = 1  AND mission.start_at <= NOW() AND mission.end_at >= NOW()
    ORDER BY mission.created_at DESC
    LIMIT 10;
  
    홈 화면의 미션 리스트는 특정 지역의 미션 정보를 조회하기 위해 mission, store, region 테이블을 조인하여 구성하였다.
    또한 사용자의 참여 여부를 확인하기 위해 member_mission을 LEFT JOIN 하였으며(참여 안한 미션도 보여줘야하기 때문에),
    현재 진행 중인 미션만 조회하기 위해 start_at과 end_at 조건을 사용하였다.
    최신순 정렬과 페이징 처리를 위해 ORDER BY와 LIMIT 10을 적용하였다.
    
 4.  마이페이지
    
    마이 페이지 화면 쿼리
  
    SELECT
        member.id,
        member.nickname,
        member.email,
        member.phone,
        member.phone_verified,
        member.point
    FROM member
    WHERE member.id = 1;
    
    
    member 테이블에서 회원의 아이디, 닉네임, 이메일, 전화번호, 전화번호 인증 여부, 포인트 정보를 조회하여 마이페이지에 표시한다.

    노션 링크:https://makeus-challenge.notion.site/Chapter-2-SQL-Query-31fb57f4596b814e8147d42b9e32d1b5?source=copy_link
