# 00. 설계 개요 (Overview)

## 1. 설계 목표

본 서비스는 세 가지 사용자 요구를 만족한다.

| # | 요구 | 설계적 해석 |
|---|---|---|
| ① | 산출물이 **자연스럽게(자동) 생성** | 개발 활동(커밋·PR·이슈·빌드) 이벤트를 방법론이 정의한 산출물 템플릿에 매핑해 데이터를 자동 채움 |
| ② | 산출물이 **중앙에서 통합관리** | 모든 산출물을 단일 "작업항목" 모델로 저장하고 추적성 링크로 연결, 버전·베이스라인으로 관리 |
| ③ | 단계별 **Gate 승인** | 단계 전환을 상태전이로 모델링하고, 전이 시 지정 승인자의 전자서명을 강제 |

## 2. 핵심 설계 원칙

1. **단일 통합 객체 (Work Item)** — 요구사항·태스크·설계·산출물·테스트·결함·변경요청을 모두
   하나의 `WorkItem` 타입 계층으로 표현한다. 유형별로 필드는 다르지만, 링크·버전·상태·권한
   메커니즘은 공통이다. (IBM ELM / Codebeamer 패턴)

2. **문서형 뷰 + 구조화 DB 이중화** — 실무자에게는 Word/Markdown 유사 문서로 보여주되,
   내부는 문단/항목 단위로 구조화 저장한다. 문서는 구조화 데이터에 대한 *렌더링 뷰*이며
   원천(source of truth)이 아니다. (Polarion LiveDocs 패턴)

3. **이벤트 소싱 기반 자동생성** — 산출물은 "누가 작성"하는 것이 아니라 "개발 이벤트에서 도출"된다.
   외부 이벤트(웹훅)를 정규화하여 AutomationRule 로 산출물 필드에 반영한다.

4. **상태전이 = Gate** — 방법론의 단계는 워크플로우 상태이고, 단계 사이의 전환은 Gate이다.
   Gate 통과 조건(필수 산출물 존재, 검증 규칙, 승인 서명)을 만족해야 상태전이가 허용된다.

5. **개방형 링크 & API 우선** — OSLC 를 전면 도입하지는 않되, 모든 아티팩트는 안정적 URI 와
   REST 표현을 가지며, 추적성 링크는 표준화된 관계 타입을 사용한다. (경량 OSLC 벤치마킹)

6. **불변 스냅샷** — 감사 대응을 위해 임의 시점의 전체 산출물 상태를 베이스라인으로 동결한다.

## 3. 도메인 개념 지도

```
Methodology (방법론)
 └─ Stage[] (단계: 요구사항→설계→구현→테스트→배포)
     ├─ ArtifactTemplate[] (단계별 산출물 템플릿)
     └─ Gate (단계 종료 승인 지점)
         └─ ApprovalPolicy (필요 승인자/역할/서명 규칙)

Project (프로젝트)  — 방법론을 인스턴스화
 ├─ WorkItem[] (요구사항/태스크/설계/산출물/테스트/결함/변경요청)
 │    └─ TraceLink[] (추적성 링크: satisfies/derives/implements/verifies…)
 ├─ Artifact[] (산출물 = WorkItem 집합에 대한 문서형 뷰, 버전관리)
 ├─ GateInstance[] (각 Gate의 실제 승인 진행 레코드)
 │    └─ Approval[] → Signature (전자서명)
 └─ Baseline[] (스냅샷)

Integration
 ├─ IntegrationEvent[] (Git/CI/이슈트래커 웹훅의 정규화 이벤트)
 └─ AutomationRule[] (이벤트 → 산출물 필드 매핑 규칙)
```

## 4. 논리 아키텍처

```mermaid
flowchart LR
  subgraph Dev["개발툴"]
    Git[Git / GitHub·GitLab]
    CI[CI/CD · Jenkins·Actions]
    Issue[이슈트래커 · Jira]
  end

  subgraph Ingest["연동 계층"]
    WH[Webhook Receiver]
    NORM[Event Normalizer]
  end

  subgraph Core["ALM Core"]
    AUTO[Automation Engine\n산출물 자동생성]
    WI[(Work Item Store)]
    ART[(Artifact / LiveDoc Store)]
    GATE[Gate Engine\n상태전이 + 서명]
    TRACE[Traceability Graph]
    BASE[Baseline / Snapshot]
  end

  subgraph UX["사용자"]
    UI[Web UI\n문서형 뷰 · Gate 콘솔]
    API[REST / OSLC-lite API]
  end

  Git --> WH
  CI --> WH
  Issue --> WH
  WH --> NORM --> AUTO
  AUTO --> WI
  AUTO --> ART
  WI <--> TRACE
  ART --> GATE
  GATE --> BASE
  UI --> WI
  UI --> ART
  UI --> GATE
  API --> WI
  API --> ART
```

## 5. 3사 패턴의 반영 위치

| 벤치마크 요소 | 반영된 설계 구성요소 |
|---|---|
| ELM EWM: 커밋↔작업항목↔빌드 연결 | Integration 계층 + AutomationRule + TraceLink |
| ELM: 데이터 취합 자동 문서생성 | Artifact(LiveDoc) 렌더러 + Baseline export |
| ELM/Polarion: 전자서명 Gate | Gate Engine + Approval/Signature |
| Polarion LiveDocs | Artifact = 구조화 WorkItem에 대한 문서형 뷰 |
| Polarion Git 네이티브 | Integration(Git) + 변경↔WorkItem 추적성 |
| Codebeamer 컴플라이언스 템플릿 | Methodology / ArtifactTemplate 사전구성 |
| Codebeamer 유연 워크플로우 | Stage/Gate 를 설정(config)으로 정의 |
| OSLC 개방형 링크 | REST/OSLC-lite API + 표준 링크 타입 |
