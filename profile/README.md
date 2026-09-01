# <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/swarm_icon.png" width="40"/> Swarm

**연령별 인지제약 기반 AI 사용자 시뮬레이션 엔진**

기존 AI 기반 UX 테스트 도구는 웹 페이지의 DOM 정보를 폭넓게 인식하고 이를 기반으로 행동한다.  
하지만 실제 사용자는 페이지의 모든 정보를 동일하게 인식하지 않는다.

Swarm은 인간의 인지적 한계를 프롬프트가 아닌 코드 레벨에서 적용하여,  
연령과 디지털 리터러시에 따른 인지적 제약을 반영하는 AI 사용자 시뮬레이션 엔진이다.

---

## 핵심 차별점

**기존 Prompt 기반 방식**

```text
"70대 사용자처럼 행동하세요."
        ↓
AI는 여전히 전체 DOM 정보를 기반으로 판단
```

**Swarm 방식**

```text
DOM → 인지제약 필터링 → Persona가 인식 가능한 정보 → AI
```

Swarm은 페르소나의 인지적 특성에 따라 **AI에게 전달되는 정보 자체를 코드 레벨에서 제한**한다.

예를 들어 고령 페르소나에게는 작은 글씨나 낮은 대비 등 설정된 인지 기준을 충족하지 못하는 요소가 전달되지 않는다.

따라서 특정 연령대의 행동을 프롬프트로 모방하도록 지시하는 것이 아니라,  
**각 페르소나가 인식할 수 있는 정보 범위 안에서 행동을 결정하도록 설계했다.**

---

## 시스템 구조

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/system_architecture.png" width="700"/>
</p>

Swarm은 **Web Application과 AI Simulation Engine을 분리한 구조**로 설계했다.

1. 사용자가 React 대시보드에서 시뮬레이션을 생성한다.
2. Spring Boot 서버가 요청을 처리하고 작업을 Redis + Celery Queue로 전달한다.
3. FastAPI 기반 Simulation Worker가 페르소나 단위로 작업을 병렬 수행한다.
4. 시뮬레이션 메타데이터와 결과 요약은 PostgreSQL(RDS)에 저장한다.
5. 로그와 스크린샷 등의 원본 데이터는 AWS S3에 저장한다.
6. Auditor AI가 저장된 로그를 기반으로 반복적인 UX 실패 패턴을 분석한다.

---

## AI Framework 구조

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/ai_architecture.png" width="700"/>
</p>

웹 DOM을 표준화한 뒤 보편적 인지제약과 연령별 인지제약을 순차적으로 적용하고,  
필터링된 정보만 Navigator AI에 전달하여 다음 행동을 결정한다.

---

## 핵심 기술

### 인지제약 2-Layer 시스템

- **Layer 1 (보편적 제약)**  
  WCAG 2.1 색상 대비, Miller's Law 작업 기억 한계, Pre-attentive Processing 기반 시각 필터링

- **Layer 2 (연령별 개인차)**  
  20대 / 50대 / 70대 앵커 값 기반 선형 보간 및 디지털 리터러시 수준 반영

### 섹션 기반 절차적 탐색 알고리즘

DOM을 헤더 / 네비게이션 / 메인 / 사이드 / 푸터 등의 의미 단위로 분할하고,  
각 섹션 내부 요소를 시각적 현저성 기준으로 3단계 Layer로 재분류한다.

AI가 한 번에 전체 DOM을 탐색하는 대신 단계별로 제한된 요소를 탐색하도록 하여  
인간의 순차적인 정보 탐색 방식을 구조적으로 반영한다.

### 대규모 시뮬레이션 최적화

- MutationObserver 기반 증분 파싱으로 변경된 노드만 처리
- pHash 캐싱을 통한 Vision AI 중복 호출 제거
- Guide AI 선행 실행을 통한 ParsingCache 사전 워밍업
- Redis + Celery 기반 페르소나 단위 병렬 실행
- Map-Reduce 체인 + 클러스터링 기반 대규모 로그 분석

---

## 서비스 화면

### 시뮬레이션 생성

대상 사이트 URL, 성공 조건, 연령별 페르소나 비율, 디지털 리터러시 수준,  
시뮬레이션 횟수 등의 실험 조건을 설정한다.

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/create.png" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/create_option.png" width="700"/>
</p>

### 시뮬레이션 진행

설정된 조건을 기반으로 페르소나별 AI 사용자가 대상 서비스를 탐색한다.

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/polling.png" width="700"/>
</p>

### 결과 대시보드

시뮬레이션 결과를 5개의 분석 화면으로 제공한다.

#### 개요

전체 성공률, 연령대별 성공/실패율, 평균 완료 시간 비교

<table>
  <tr>
    <td width="50%">
      <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/overview1.png" width="100%"/>
    </td>
    <td width="50%">
      <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/overview2.png" width="100%"/>
    </td>
  </tr>
</table>

#### 주요 이슈

Auditor AI가 시뮬레이션 로그를 분석하여 도출한 UI/UX 문제와 우선순위

<table>
  <tr>
    <td width="50%">
      <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/issues1.png" width="100%"/>
    </td>
    <td width="50%">
      <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/issues2.png" width="100%"/>
    </td>
  </tr>
</table>

#### 히트맵

실제 사이트 스크린샷 위에 페르소나별 인식 및 실패 지점을 시각화

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/heatmap.png" width="700"/>
</p>

#### WCAG

axe-core 기반 웹 접근성 자동 검사 결과

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/wcag.png" width="700"/>
</p>

#### AI 수정

Code Fixer AI가 분석된 이슈를 기반으로 실제 적용 가능한 코드 수정안을 생성

<p align="center">
  <img src="https://raw.githubusercontent.com/S-warm/.github/main/images/ai_fix.png" width="700"/>
</p>

---

## 시연 영상

추후 업로드 예정

---

## 기술 스택

| 영역 | 기술 |
|---|---|
| Frontend | React, TypeScript |
| Backend | Java, Spring Boot, PostgreSQL |
| AI Framework | Python, FastAPI, Playwright |
| AI | GPT-4o-mini, GPT-4o, Claude Vision, Claude Haiku |
| Message Queue | Redis, Celery |
| Infrastructure | AWS EC2, RDS, S3, Step Functions, Lambda, Docker |

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
| 황필호 (팀장) | 기획, AI Framework 전체 설계 및 개발, 캐싱 시스템, 파이프라인 최적화 |
| 최수종 | 테스트 페이지 개발, Spring Boot 백엔드 서버 개발, PostgreSQL DB 설계, Docker 인프라 구축 |
| 공지현 | 프론트엔드 API 연동, Spring Boot API 명세 정의, DB 연동 환경 구축 |
| 김필중 | 프론트엔드 대시보드 UI 개발, 로그데이터 피드백, 연령별 페르소나 자료조사 |

---

## 프로젝트 활동

### 2026 캡스톤 디자인

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/77e16298-0b7c-4ced-872a-dd329c5ab43a"
    width="500"
    alt="Team Swarm"
  />
</p>

<p align="center">
  <b>Team Swarm</b>
</p>

<table align="center">
  <tr>
    <td width="50%" align="center">
      <img
        src="https://github.com/user-attachments/assets/9e9356a4-48ab-4db3-8576-aa7d9db9959b"
        width="300"
        alt="프로젝트 현장 시연"
      />
      <br/>
      <b>프로젝트 현장 시연</b>
    </td>
    <td width="50%" align="center">
      <img
        src="https://github.com/user-attachments/assets/cd899a49-a8a8-4d90-918b-dc08caa414c6"
        width="300"
        alt="Swarm 시뮬레이션 구동 환경"
      />
      <br/>
      <b>Swarm 시뮬레이션 구동 환경</b>
    </td>
  </tr>
</table>

<p align="center">
  Swarm을 실제 캡스톤 디자인 현장에서 시연하며,<br/>
  연령별 AI 사용자 시뮬레이션 과정과 UX 분석 결과를 소개했습니다.
</p>

---

> 2026학년도 1학기 캡스톤 디자인 | 한성대학교
