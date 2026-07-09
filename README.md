# ALM — 방법론 실행 플랫폼 (Methodology Execution Platform)

단순 문서 저장소가 아니라, 개발 방법론을 **실제 개발 프로세스에 강제 적용**하고
산출물의 **생성·통합관리·Gate 검증까지 자동화**하는 실행 플랫폼의 설계 저장소입니다.

> 지향: "방법론을 문서로 제공" → **"방법론을 실행·강제하고 산출물 생성·검증을 자동화"**
> GxP 등 규정 준수·감사 추적이 중요한 환경을 1차 타깃으로 한다.

## 설계 관점: 오케스트레이션 우선(Orchestration-first)

전부 자체 구현하지 않는다. 검증된 툴을 엮고, 우리 플랫폼은 그 위에서
**방법론 정의 · 프로젝트 자동생성 · 추적성 · Gate 상태 관리**를 담당하는 *척추(spine)* 역할을 한다.

| 영역 | 역할 | 대표 구현 | 본 설계에서의 위치 |
|---|---|---|---|
| 방법론 관리 | 방법론·프로세스·역할·산출물 정의 | IBM RMC, EPF, Confluence | **Core**(Methodology/Stage/Template) |
| 가이드 제공 | 단계별 절차·체크리스트 | Confluence, Backstage TechDocs | Portal(위임) |
| 산출물 관리 | 템플릿·작성예시·버전관리 | Git, Confluence, Nextcloud, SharePoint | Git/Confluence(위임) + Core 메타 |
| 실행 자동화 | 프로젝트 생성·표준 구조 생성 | Backstage Software Templates, Cookiecutter | **Scaffolder**(오케스트레이션) |
| Gate 검증 | 필수 산출물·품질 기준 자동확인 | Jenkins, GitHub Actions, GitLab CI | **Gate Engine** + CI 실행위임 |
| 추적성 | 요구사항↔설계↔개발↔테스트↔릴리즈 | Jira, ALM, RTM | **Core**(WorkItem/TraceLink) |

## 현실적인 툴 조합 (권장 스택)

| 툴 | 담당 |
|---|---|
| **Confluence** | 방법론·가이드·템플릿 문서 |
| **Backstage** | Golden Path, 프로젝트 생성 자동화(Software Templates) |
| **Jira** | 작업·요구사항 추적 |
| **GitLab / GitHub** | 소스·문서 버전관리 |
| **Jenkins** (또는 Actions/GitLab CI) | Gate 검증 및 CI/CD |
| **Nextcloud** | 대용량 산출물·파일 관리(필요 시) |

우리 플랫폼(ALM Core)은 이들을 어댑터로 연결하고, 방법론 상태·Gate·추적성을 관장한다.

## 흐름 한눈에

```
방법론 Portal ──▶ 프로젝트 생성(Scaffold) ──▶ 개발 진행 ──▶ Gate 검증·승인
  가이드          표준 디렉터리                산출물 작성      필수 산출물/품질
  체크리스트       문서 자동생성                추적성 자동관리   자동확인 + 전자서명
  템플릿          Jira/Git/CI 생성                             통과 못하면 다음 단계 차단
```

## 문서 구성

| 문서 | 내용 |
|---|---|
| [00-overview](docs/architecture/00-overview.md) | MEP 포지셔닝, 설계 원칙, 오케스트레이션 아키텍처 |
| [01-data-model](docs/architecture/01-data-model.md) | WorkItem 중심 통합 데이터 모델(척추) |
| [02-artifact-automation](docs/architecture/02-artifact-automation.md) | Git/이슈/CI 이벤트 → 산출물 자동생성 |
| [03-gate-workflow](docs/architecture/03-gate-workflow.md) | Gate 상태전이 + 전자서명 |
| [04-screen-flow](docs/architecture/04-screen-flow.md) | 화면 흐름·정보구조 |
| [05-integration-api](docs/architecture/05-integration-api.md) | 웹훅/REST API, 툴 어댑터 |
| [06-baseline-audit](docs/architecture/06-baseline-audit.md) | 베이스라인/스냅샷, 감사 추적성 |
| **[07-platform-reference-architecture](docs/architecture/07-platform-reference-architecture.md)** | **툴 매핑 레퍼런스 아키텍처(오케스트레이션)** |
| **[08-project-scaffolding](docs/architecture/08-project-scaffolding.md)** | **프로젝트 자동생성·산출물 자동생성(SRS/SDS/RTM…)** |
| **[09-gate-validation](docs/architecture/09-gate-validation.md)** | **단계별 CI 기반 Gate 자동검증·차단 규칙** |
| **[10-tailoring](docs/architecture/10-tailoring.md)** | **방법론 등록·선택·테일러링(병합/생략)·단계별 진행** |
| **[11-authoring](docs/architecture/11-authoring.md)** | **방법론 저작 — 3레벨 WBS 사용자 등록·편집·발행** |

원 벤치마크 조사: [`docs/benchmark.md`](docs/benchmark.md)

## 방법론 단계 (기준 예시)

```
요구사항 → 설계 → 구현 → 테스트 → 릴리즈
   G1      G2     G3     G4       G5
```
