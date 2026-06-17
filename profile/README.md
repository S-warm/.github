# <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/swarm_icon.png" width="40"/> Swarm

**연령별 인지제약 기반 AI 사용자 시뮬레이션 엔진**

기존 UX 검증 도구는 AI가 웹 페이지의 모든 요소를 완벽하게 인식한다. 실제 사용자는 그렇지 않다.  
Swarm은 인간의 인지적 한계를 프롬프트가 아닌 코드 레벨에서 구현하여, AI 에이전트가 연령대별로 실제 사람처럼 실수하고 포기하도록 만드는 시뮬레이션 엔진이다.

---

## 핵심 차별점

**기존 방식**
```
"당신은 70대처럼 행동하세요" → AI가 척만 함
```

**Swarm 방식**
```
70대 기준 미달 요소를 DOM에서 사전 제거 → AI가 정말로 못 봄
```

AI에게 전달되는 정보 자체를 페르소나 기준으로 필터링하여, 70대 페르소나는 물리적으로 70대가 인식할 수 있는 정보만으로 판단한다.

---

## 서비스 화면

로그인 / 회원가입

<table>
  <tr>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/login.png" width="100%"/></td>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/signup.png" width="100%"/></td>
  </tr>
</table>

시뮬레이션 생성

<img src="https://raw.githubusercontent.com/S-warm/.github/main/images/create.png" width="700"/>
<img src="https://raw.githubusercontent.com/S-warm/.github/main/images/create_option.png" width="700"/>

시뮬레이션 진행


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/polling.png" width="700"/>


결과 대시보드 (5개 탭)

개요 - 전체 성공률, 연령대별 성공/실패율, 평균 완료 시간 비교

<table>
  <tr>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/overview1.png" width="100%"/></td>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/overview2.png" width="100%"/></td>
  </tr>
</table>

주요이슈 - Auditor AI가 도출한 UI/UX 문제 목록 및 우선순위

<table>
  <tr>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/issues1.png" width="100%"/></td>
    <td width="50%"><img src="https://raw.githubusercontent.com/S-warm/.github/main/images/issues2.png" width="100%"/></td>
  </tr>
</table>

히트맵 - 실제 사이트 스크린샷 위에 연령대별 실패 지점 오버레이


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/heatmap.png" width="700"/>


WCAG - axe-core 기반 접근성 자동 검사 결과


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/wcag.png" width="700"/>


AI 수정 - Code Fixer AI가 생성한 실제 적용 가능한 코드 수정안


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/ai_fix.png" width="700"/>


---

## 시연 영상

추후 업로드 예정

---

## 시스템 구조


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/system_architecture.png" width="700"/>


사용자가 React 대시보드에서 시뮬레이션을 설정하면 Spring Boot가 요청을 받아 Redis + Celery 큐에 작업을 분배한다. FastAPI 시뮬레이션 엔진의 다수 워커가 페르소나 단위로 병렬 실행되며, 로그와 스크린샷은 S3에, 메타데이터와 결과 요약은 PostgreSQL(RDS)에 저장된다. Auditor AI(GPT-4o)는 S3의 로그 데이터에 RAG 기반으로 접근하여 패턴 분석을 수행한다.

---

## AI Framework 구조


  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/ai_architecture.png" width="700"/>


---

## 주요 기능

**인지제약 2-Layer 시스템**

- Layer 1 (보편적 제약): WCAG 2.1 색상 대비, Miller's Law 작업 기억 한계, Pre-attentive Processing 기반 시각 필터링
- Layer 2 (연령별 개인차): 20대/50대/70대 앵커 값 기반 선형 보간, 디지털 리터러시 수준 반영

**섹션 기반 절차적 탐색 알고리즘**

DOM을 헤더/네비게이션/메인/푸터 섹션으로 분할하고, 각 섹션 내부 요소를 시각적 현저성 기준으로 3단계 레이어로 재분류한다. AI는 구조적으로 인간이 페이지를 읽는 순서를 따를 수밖에 없다.

**대규모 시뮬레이션 최적화**

- MutationObserver 기반 증분 파싱으로 변경 노드만 처리
- pHash 캐싱으로 Vision API 중복 호출 제거
- Guide AI 선행 실행으로 ParsingCache 사전 워밍업, 캐시 히트율 100% 달성
- Map-Reduce 체인 + 클러스터링 기반 로그 분석

---

## 기술 스택

| 영역 | 기술 |
|---|---|
| AI Framework | Python, FastAPI, Playwright, GPT-4o-mini, GPT-4o, Claude Vision, Claude Haiku |
| Backend | Spring Boot, PostgreSQL (RDS) |
| Frontend | React, TypeScript |
| Infra | AWS EC2, S3, Redis, Celery |

---

## 레포지토리 구성

| 레포 | 설명 |
|---|---|
| [BE](https://github.com/S-warm/BE) | Spring Boot 서버, PostgreSQL DB, Docker 인프라 |
| [FE](https://github.com/S-warm/FE) | React + TypeScript 대시보드 |
| [AI](https://github.com/S-warm/BE_AI_Framework) | AI Framework, 시뮬레이션 엔진, 인지제약 레이어 |

---

## 팀 구성

| 이름 | 역할 |
|---|---|
| 황필호 (팀장) | AI Framework 전체 설계 및 개발, 캐싱 시스템, 파이프라인 최적화 |
| 최수종 | 백엔드 테스트 페이지 개발, Spring Boot 서버, PostgreSQL DB 설계, Docker 인프라 |
| 공지현 | 프론트엔드 API 연동, Spring Boot API 명세 정의, DB 연동 환경 구축 |
| 김필중 | 프론트엔드 대시보드 UI 개발, 로그데이터 피드백, 연령별 페르소나 자료조사 |

---

> 2026학년도 1학기 캡스톤 디자인 | 한성대학교
