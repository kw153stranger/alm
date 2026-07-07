# ALM — 방법론 기반 산출물 자동생성·관리 + Gate 승인 서비스

개발툴(IDE·Git·CI/CD·이슈트래커)과 연계하여 방법론에서 정의한 **산출물이 자동으로 생성**되고,
그 산출물이 **중앙에서 통합관리**되며, 단계별로 **Gate 승인**(전자서명/리뷰 워크플로우)이
진행되는 서비스의 설계 저장소입니다.

## 설계 배경

IBM ELM(Jazz), Siemens Polarion, PTC Codebeamer 3사의 공통 패턴을 경량화해 벤치마킹했습니다.
원 조사 문서는 [`docs/benchmark.md`](docs/benchmark.md) 를 참고하세요.

핵심 아이디어 3가지:

1. **작업항목(Work Item) 중심 통합 데이터 모델** — 요구사항·태스크·설계·산출물·테스트를 하나의
   공통 객체로 표현하고 추적성 링크로 연결 (IBM ELM / Codebeamer)
2. **Git/이슈트래커 훅 기반 자동 산출물 생성** — 커밋·PR·이슈 이벤트를 트리거로 산출물 템플릿을
   자동으로 채움 (ELM 자동 문서 생성 + Polarion Git 네이티브 연동)
3. **Gate = 상태 전이 + 전자서명** — 방법론 단계를 워크플로우 상태로 정의하고, 지정 승인자의
   전자서명이 있어야 다음 단계로 진행 (Polarion 전자서명 워크플로우)

## 문서 구성

| 문서 | 내용 |
|---|---|
| [00-overview](docs/architecture/00-overview.md) | 설계 원칙, 도메인 개념, 전체 아키텍처 |
| [01-data-model](docs/architecture/01-data-model.md) | 핵심 데이터 모델(ERD), 엔티티 정의 |
| [02-artifact-automation](docs/architecture/02-artifact-automation.md) | Git/이슈 이벤트 → 산출물 자동생성 파이프라인 |
| [03-gate-workflow](docs/architecture/03-gate-workflow.md) | Gate 승인 상태전이 다이어그램, 전자서명 |
| [04-screen-flow](docs/architecture/04-screen-flow.md) | 주요 화면 흐름과 정보구조 |
| [05-integration-api](docs/architecture/05-integration-api.md) | 웹훅/REST API, 개방형 아티팩트 스키마 |
| [06-baseline-audit](docs/architecture/06-baseline-audit.md) | 베이스라인/스냅샷, 감사 추적성 |

## 방법론 단계 (기준 예시)

```
요구사항定義 → 설계 → 구현 → 테스트 → 배포
   G1        G2     G3     G4      G5
```

각 화살표 위의 `G*` 가 Gate(단계 전환 승인 지점)입니다.
