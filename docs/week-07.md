# Week 07 팀 활동지 / Team Activity

| 팀명 / Team | 작성일 / Date | 참여자 / Members |
| --- | --- | --- |
| Team형인 | 2026/10/14 | 김형인,강준우,박태현,최지범 |

## 1. 아키텍처 다이어그램 / Architecture Diagram

손그림 사진을 올리고 링크하십시오 / Upload a photo and link it.

<img width="1075" height="592" alt="image" src="https://github.com/user-attachments/assets/215bf910-cb19-414f-a20f-53503010853a" />


| 계층 / Layer | 모듈·파일 / Modules | 의존하지 않는 것 / Does not depend on |
| --- | --- | --- |
| 표현 / Presentation | core.js의 패널·오버레이 (highlight, bubble, modal, el, CSS) | 팁 문구, 모드 로직 (표시할 내용은 인자로 받음) |
| 로직 / Logic | modes/tutorial.js, modes/practice.js, modes/tip.js, core.js의 클릭 차단(guard)·모드 등록 | 화면의 CSS 선택자 (SG.rows·SG.findButton 등을 거쳐 접근), chrome.* API, 외부 서버 |
| 데이터 / Data | tips.js (팁 데이터), core.js의 화면 읽기 (screen, rows, cell, 선택자 기준은 docs/selectors.md) | 모드, 오버레이, 네트워크 (페이지 DOM에서만 읽음) |

## 2. 이슈 / Issues

> 이슈 = **앞으로 할 일** / An issue is work to be done.
개발할 때 커밋에 이슈 번호를 붙이고(`#3 알레르기 필터 추가`), 완료 조건을 만족하면 이슈를 닫습니다.
Add the issue number to commits, and close the issue when its criteria are met.
> 

| # | 제목 / Title | 라벨 / Label | 담당 / Assignee | 완료 조건 / Acceptance Criteria |
| --- | --- | --- | --- | --- |
| D0 | 확장 공통 기반 (완료, 이슈 없음) | feat | 강준우 | 목업에서 세 모드를 켜고 끌 수 있고, 켜진 동안 실제 신청 버튼을 눌러도 목업의 신청 내역이 바뀌지 않는다 |
| D1 | 실전 재현 엔진 (기능 2) | feat | 박태현 | 판정 스크립트 통과: 같은 입력이면 같은 결과가 나오고, 희망/제한 비율이 높은 과목이 먼저 마감된다 |
| D2 | 자유 팁 모드(기능 3) | feat | 최지범 | 요소 25개 이상이고, 신입생 질문 사례에 나온 주제(사용법·이수제한·인원·학점·일정)에 해당하는 요소가 모두 들어 있으며, 요소마다 출처가 있다 |
| D3 | 신입생 튜토리얼 (기능 1) | feat | 최지범 | 어느 화면에서 시작해도 수강편람으로 이동한 뒤 마지막 단계까지 진행되고, 단계 문구가 팁 데이터와 글자까지 같다 |
| D4 | 가짜 신청 버튼·결과 모달 (기능 2) | feat | 박태현 | 마감 전에 누르면 성공, 마감 후에 누르면 정원 마감 모달이 뜨고, 요약에 모든 과목의 결과가 나온다 |
| D5 | 대기열 팝업 재현 | feat | 박태현 | 팝업이 뜬 상태에서 신청 버튼을 다시 누르면 대기 순번이 늘어나고 안내 문구가 뜬다 |
| D6 | 추가 실패 모달 | feat | 박태현 | 각 상황을 만들었을 때 해당 모달이 뜬다 |

## 3. 마일스톤 / Milestones

| 마일스톤 / Milestone | 마감 / Due | 포함 이슈 / Issues |
| --- | --- | --- |
| MVP 완료 | 11주차 | D0(완료),1,2,3,4 |
| 실행 패키지 | 13주차 | D 5,6 |

> AI를 사용했다면 `PROMPTS.md`에 기록 / Log any AI use in `PROMPTS.md`
>
