# AI 협업 기록 / AI Collaboration Log

작성 원칙: 프롬프트 나열이 아니라 **판단 근거**를 남긴다.
Principle: record your **reasoning**, not just prompts.
---

## [이슈 #1] 메모 검색 기능 (2주차 활동)

**목표(스펙) / Spec**
- 입력 Input: GET /memos/search?q={키워드} — 쿼리 파라미터 q (문자열)
- 처리 Processing: req.user.id 기준으로 본인 소유 메모 중 title 또는 body에 q가 포함된 메모를 조회. LIKE 검색 시 특수문자(%, _)는 escapeLikePattern()으로 이스케이프 처리
- 출력 Output: { ok: true, data: [메모 목록] } — 목록 항목은 기존 조회 API와 동일하게 id, title, created_at 포함
- 실패 조건 Failure: q가 없거나 빈 문자열이면 400과 함께 { ok: false, error: 'QUERY_REQUIRED' } 반환 (에러 코드는 CONVENTIONS.md 명명 규칙에 맞춰 대문자+밑줄)

**요청한 프롬프트 요지 / Prompt (summary)**

A조 프롬프트
``` markdown
메모 검색 기능 만들어줘. 제목이랑 본문에서 키워드로 찾을 수 있게.
```
B조 프롬프트
```` markdown
[지시]
메모 검색 기능을 추가해줘. 제목과 본문에서 키워드로 검색된다.

[규약]
(CONVENTIONS.md 내용 전체를 붙여넣기)
# 프로젝트 규약

이 문서는 코드를 작성할 때 지켜야 할 규칙입니다.

## 계층 분리

- `routes.js`는 HTTP 요청과 응답만 다룹니다. SQL을 직접 쓰지 않습니다.
- 데이터베이스 접근은 `service.js`에만 둡니다.

## 응답 형식

모든 응답은 다음 두 형태 중 하나입니다.

```json
{ "ok": true,  "data": ... }
{ "ok": false, "error": "ERROR_CODE" }
```

에러 코드는 대문자와 밑줄로 씁니다. (예: `MEMO_NOT_FOUND`)

## 명명 규칙

- 함수명은 동사로 시작합니다. `list`, `get`, `create`, `update`, `remove`
- 데이터베이스 컬럼은 스네이크 케이스를 씁니다. `user_id`, `created_at`
- 자바스크립트 변수는 카멜 케이스를 씁니다. `userId`, `createdAt`

## 입력 검증

- 사용자 입력은 반드시 검증합니다.
- 검증에 실패하면 400과 함께 `{ ok: false, error }` 를 반환합니다.

## 권한

- 모든 조회와 수정은 **본인 소유 데이터로 한정**합니다.
- 모든 쿼리에 `user_id` 조건을 포함합니다.


[근거]
(schema.sql, service.js, routes.js 내용 전체를 붙여넣기)
//schema.sql
# 프로젝트 규약

이 문서는 코드를 작성할 때 지켜야 할 규칙입니다.

## 계층 분리

- `routes.js`는 HTTP 요청과 응답만 다룹니다. SQL을 직접 쓰지 않습니다.
- 데이터베이스 접근은 `service.js`에만 둡니다.

## 응답 형식

모든 응답은 다음 두 형태 중 하나입니다.

```json
{ "ok": true,  "data": ... }
{ "ok": false, "error": "ERROR_CODE" }
```

에러 코드는 대문자와 밑줄로 씁니다. (예: `MEMO_NOT_FOUND`)

## 명명 규칙

- 함수명은 동사로 시작합니다. `list`, `get`, `create`, `update`, `remove`
- 데이터베이스 컬럼은 스네이크 케이스를 씁니다. `user_id`, `created_at`
- 자바스크립트 변수는 카멜 케이스를 씁니다. `userId`, `createdAt`

## 입력 검증

- 사용자 입력은 반드시 검증합니다.
- 검증에 실패하면 400과 함께 `{ ok: false, error }` 를 반환합니다.

## 권한

- 모든 조회와 수정은 **본인 소유 데이터로 한정**합니다.
- 모든 쿼리에 `user_id` 조건을 포함합니다.
``` javascript
//service.js
const db = require('./db');

/**
 * 사용자의 메모 목록을 최신순으로 조회한다.
 */
function listMemos(userId) {
  return db.all(
    `SELECT id, title, created_at
       FROM memos
      WHERE user_id = ?
      ORDER BY created_at DESC`,
    [userId]
  );
}

/**
 * 메모 한 건을 조회한다. 본인 메모가 아니면 null을 반환한다.
 */
function getMemo(userId, memoId) {
  return db.get(
    `SELECT id, title, body, created_at
       FROM memos
      WHERE id = ? AND user_id = ?`,
    [memoId, userId]
  );
}

/**
 * 메모를 생성한다.
 */
function createMemo(userId, title, body) {
  return db.run(
    `INSERT INTO memos (user_id, title, body, created_at)
     VALUES (?, ?, ?, datetime('now'))`,
    [userId, title, body]
  );
}

module.exports = { listMemos, getMemo, createMemo };
//routes.js
const express = require('express');
const service = require('./service');

const router = express.Router();

// 메모 목록 조회
router.get('/memos', async (req, res) => {
  const memos = await service.listMemos(req.user.id);
  res.json({ ok: true, data: memos });
});

// 메모 단건 조회
router.get('/memos/:id', async (req, res) => {
  const memo = await service.getMemo(req.user.id, req.params.id);

  if (!memo) {
    return res.status(404).json({ ok: false, error: 'MEMO_NOT_FOUND' });
  }

  res.json({ ok: true, data: memo });
});

// 메모 생성
router.post('/memos', async (req, res) => {
  const { title, body } = req.body;

  if (!title || !body) {
    return res.status(400).json({ ok: false, error: 'TITLE_AND_BODY_REQUIRED' });
  }

  const result = await service.createMemo(req.user.id, title, body);
  res.status(201).json({ ok: true, data: { id: result.lastID } });
});

module.exports = router;
```
[종료조건]
- GET /memos/search?q=키워드 로 호출된다
- 제목 또는 본문에 키워드가 포함된 메모만 반환한다
- 본인 메모만 반환한다
- q가 비어 있으면 400과 { ok: false, error } 를 반환한다
````
**결과에 대한 판단 / Decisions**
- 채택한 부분과 이유 / Accepted, because: 새로운 함수 escapeLikePattern(), searchMemos()를 포함해 전부 채택
- 수정한 부분과 이유 / Changed, because: 없음
- 폐기한 부분과 이유 / Rejected, because: 없음

**검증 방법 / How it was verified**
- 제목에만 키워드가 있는 메모, 본문에만 있는 메모, 둘 다 없는 메모로 각각 검색해 결과 포함 여부 확인
- 다른 사용자의 메모가 검색 결과에 노출되지 않는지 확인 (권한 스코프 검증)
- q 파라미터 누락/빈 문자열로 요청 시 400 + 지정된 에러 코드 반환 확인
- %, _ 등 LIKE 특수문자가 포함된 키워드로 검색해 escapeLikePattern()이 의도대로 이스케이프하는지 확인
- 응답 바디가 CONVENTIONS.md의 { ok, data } / { ok, error } 형식을 따르는지 확인
---

## [이슈 # 2] 3주차 활동

- 무엇을 하려고 썼는가

  아이디어 기획안 구체화하기 위해 사용
  
- 넣은 프롬프트 **원문 그대로** — 요약하지 않습니다

  신입생 튜토리얼 기획안을 작성하자. 신입생, 유학생 등 학교 시스템이 익숙하지 않은 사용자를 위해서 수강신청, 교수님께 이메일쓰는법, 강의실 찾아가기, 술자리에서 등 팁을 배울 수 있는 시뮬레이션을 만들면 좋을거 같아.

사용자 Who
상황 When/Where
페인포인트 What — 무엇이 얼마나
성공 기준 Done

이거 간단하게 작성해줘

- 나온 것 중 **쓴 것 / 버린 것**

  **who** 
  
  학교 시스템이 낯선 국내 신입생과 외국인 유학생
  
  **when/where** 
  
  입학 전 2월부터 개강 후 3주까지, 주로 모바일로 틈틈이 사용
  
  **what**
  
  • 수강신청 규칙을 몰라 원하는 과목을 놓침
  
  • 교수님께 보낼 이메일 형식을 몰라 한 통에 30분 이상 걸림
  
  • 건물 약칭과 호실 코드를 몰라 첫 주에 지각함
  
  • 술자리에서 거절하는 법을 몰라 원치 않는 음주를 함
  
  **done**
  
  • 시뮬레이션 완주율 70% 이상
  
  • 수강신청 관련 학사팀 문의 30% 감소
  
  • 개강 첫 주 "강의실 못 찾음" 응답이 미참여자의 절반 수준

- **버렸다면 왜 버렸는가** — 이 한 줄을 봅니다

  버리지않았음

- 무엇을 하려고 썼는가

  우리가 세운 가정의 사실 여부를 판단하는 조건을 세우기 위해 사용

- 넣은 프롬프트 **원문 그대로** — 요약하지 않습니다

  3	C	신입생은 학점 제한과 장바구니 규칙을 몰라서 원하는 과목을 놓친다	기존 데이터	학교 커뮤니티의 수강신청 실패 게시글 30개를 원인별로 분류	강준우	4주차
  4	C	교수님께 보내는 이메일 한 통에 30분 이상 걸린다	관찰	신입생 3명에게 결석 사유 메일을 실제로 쓰게 하고 소요 시간 측정	강준우	4주차
  5	C	건물 약칭과 호실 코드만 보고는 강의실을 찾기 어렵다	직접 해보기	가본 적 없는 건물의 강의실을 시간표 코드만 보고 찾아가며 시간과 헤맨 지점 기록	강준우	4주차
  
  내가 적은 가정 확인 계획인데, 아래 부분을 채워야해. 해당하는 내용이 있다면 작성해줘  
  관찰·데이터·직접 해보기를 한다면 / If you observe instead
  * 무엇을 셀 것인가 (숫자가 나와야 합니다):
  * 언제 어디서:

-  나온 것 중 **쓴 것 / 버린 것**

  **3번 (수강신청 실패 게시글)**
  
  - 가정이 맞다고 볼 조건: 30개 중 '학점 제한·장바구니 규칙 오해'가 원인인 글이 10개 이상(약 33% 이상)이거나, 원인별 순위에서 1~2위 안에 든다.
  - 가정이 틀렸다고 볼 조건: 5개 이하(약 17% 이하)이고, 서버 지연이나 정원 마감 같은 다른 원인이 훨씬 많다.
  - 기준 근거: 원인이 서너 가지로 나뉜다고 보면, 3분의 1 이상은 "주요 원인"이라고 말할 수 있는 수준이에요.
  
  **4번 (결석 사유 메일 작성)**
  
  - 가정이 맞다고 볼 조건: 3명 중 2명 이상이 30분을 넘긴다.
  - 가정이 틀렸다고 볼 조건: 3명 모두 20분 이내에 끝낸다.
  - 기준 근거: 가정 자체가 "30분 이상"이라서 그 숫자를 그대로 기준으로 썼어요. 3명이라 표본이 작으니, 과반(2명)이 넘으면 맞는 쪽으로 봅니다.
  
  **5번 (강의실 찾아가기)**
  
  - 가정이 맞다고 볼 조건: 찾아간 강의실 3곳 중 2곳 이상에서 입구부터 도착까지 10분을 넘기거나, 잘못 간 횟수가 2회 이상 나온다.
  - 가정이 틀렸다고 볼 조건: 모든 강의실에 5분 이내로 도착하고, 잘못 간 적이 없다.
  - 기준 근거: 쉬는 시간이 보통 10~15분이라, 건물 안에서만 10분이 걸리면 실제로 지각할 위험이 있다고 판단했어요.

- **버렸다면 왜 버렸는가** — 이 한 줄을 봅니다
  버리지않았음


---
(이슈 단위로 반복 / repeat per issue)
