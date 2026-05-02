https://github.com/minji-jeong516/umc_node/pull/6
readme에 캡쳐화면 첨부했습니다

+요청흐름 ex)리뷰추가
1. 사용자가 POST /api/v1/stores/:storeId/reviews 요청을 보냄
2. Controller가 storeId와 요청 body를 받음
3. DTO를 통해 요청 데이터를 변환
4. Service에서 해당 가게가 존재하는지 검증
5. Repository가 DB에서 store 조회
6. 존재하면 Repository가 review 테이블에 데이터 INSERT
7. 결과를 Controller로 반환하여 응답
