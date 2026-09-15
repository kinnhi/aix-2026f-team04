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

## [이슈 #__] 제목 / Title

**목표(스펙) / Spec**
- 입력 Input:
- 처리 Processing:
- 출력 Output:
- 실패 조건 Failure:

**요청한 프롬프트 요지 / Prompt (summary)**

**결과에 대한 판단 / Decisions**
- 채택한 부분과 이유 / Accepted, because:
- 수정한 부분과 이유 / Changed, because:
- 폐기한 부분과 이유 / Rejected, because:

**검증 방법 / How it was verified**

---
(이슈 단위로 반복 / repeat per issue)
