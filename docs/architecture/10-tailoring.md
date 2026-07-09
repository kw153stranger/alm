# 10. 방법론 등록 · 선택 · 테일러링 · 단계 진행

방법론 실행의 4대 골격.

1. **등록** — 조직이 방법론(마스터)을 등록·버전관리한다.
2. **선택** — 프로젝트마다 방법론(버전)을 선택한다.
3. **테일러링** — 프로젝트 특성에 맞게 단계를 **병합·생략**(및 산출물 조정)한다.
4. **진행 관리** — 프로젝트별로 (테일러링된) 단계 진행을 관리한다.

```
Methodology(마스터, 버전)  ──선택──▶  Project
                                        │
                                   TailoringProfile (병합/생략 + 근거·승인)
                                        │  재계산(materialize)
                                        ▼
                                  EffectiveStage[]  ← 이 프로젝트가 실제로 밟는 파이프라인
                                        │
                                   단계별 진행 상태(state) 관리
```

## 1. 등록 — Methodology (마스터)

조직 표준 방법론을 정의·발행한다. [01](01-data-model.md)의 `METHODOLOGY` / `STAGE` /
`ARTIFACT_TEMPLATE` / `GATE` 가 마스터 정의다.

- **버전관리**: 방법론 개정 시 새 버전 발행. 프로젝트는 특정 버전에 고정(pin)되어, 마스터가
  바뀌어도 진행 중 프로젝트가 흔들리지 않는다.
- **상태**: `draft → published → deprecated`. `published` 만 신규 프로젝트가 선택 가능.
- **여러 방법론 공존**: IEC 62304, ISO 26262, 자체 애자일 등 복수 등록.

## 2. 선택 — Project ↔ Methodology

프로젝트 생성 시 `methodology_id` + `methodology_version` 을 선택해 고정한다([08](08-project-scaffolding.md)).
선택 즉시 마스터 단계를 복제한 **초기 EffectiveStage** 집합이 생성된다(테일러링 전 = 마스터 그대로).

## 3. 테일러링 — 프로젝트 맞춤 재구성

마스터를 프로젝트 특성(규모·안전등급·기간)에 맞게 조정한다. **핵심 연산은 병합·생략.**

### 테일러링 연산 (TailoringOp)

| 연산 | 의미 | 예시 |
|---|---|---|
| **merge (병합)** | 2개 이상 단계를 하나로 합침 | `요구사항 + 설계 → 분석·설계`, Gate `G1+G2 → G12` |
| **omit (생략)** | 단계를 제외 | Class A 저위험 → 별도 `릴리즈` 단계 생략(테스트에 흡수) |
| **modify_artifact** | 산출물을 필수↔선택↔면제로 조정 | `SDS` 를 optional, `ValidationReport` 는 유지 |
| **add (추가, 선택)** | 프로젝트 고유 단계 추가 | `PoC` 단계 삽입 |

> 생략·병합은 **산출물과 Gate 를 함께 재구성**한다.
> - 병합 시: 두 단계의 필수 산출물 합집합이 새 단계의 산출물이 되고, Gate 통과조건도 합쳐진다.
> - 생략 시: 그 단계의 필수 산출물이 사라지므로, 다른 단계로 이관(reassign)할지 면제(waive)할지 명시해야 한다.

### 거버넌스 — 근거·승인 (규제 대응 필수)

테일러링은 "방법론 준수"의 예외이므로 **정당화와 승인**이 강제된다.

- 각 `TailoringOp` 는 `rationale`(근거) 필수.
- `TailoringProfile` 은 `draft → in_review → approved → active` 상태를 가지며,
  `approved` 되어야 프로젝트에 적용된다. 승인자·승인시각·서명은 감사로그에 기록([06](06-baseline-audit.md)).
- 테일러링 변경 이력 자체가 버전관리된다(무엇을 왜 생략/병합했는가의 추적성).
- **강제 산출물 보호**: 방법론이 `mandatory: true` 로 잠근 산출물(예: 21 CFR 대응 ValidationReport)은
  테일러링으로 생략 불가(정책으로 차단).

### 결과 — EffectiveStage (재계산본)

테일러링 승인 시 마스터 + Ops 를 합성해 **EffectiveStage[]** 를 materialize 한다.
이것이 프로젝트의 파이프라인·Gate·산출물의 실제 기준이 된다.

```
마스터:   요구사항 · 설계 · 구현 · 테스트 · 릴리즈   (G1 G2 G3 G4 G5)
Ops:      merge(요구사항+설계) , omit(릴리즈→테스트 흡수)
─────────────────────────────────────────────
Effective: 분석·설계 · 구현 · 테스트            (G12 G3 G4*)
           derived_from                        G4*=G4+G5 통합
           [요구사항,설계]
```

## 4. 진행 관리 — 단계별 상태 (프로젝트별)

각 `EffectiveStage` 는 프로젝트별 진행 상태를 가진다([03](03-gate-workflow.md) Stage 상태전이).

`locked → active → in_gate_review → passed`

- 파이프라인 뷰는 마스터가 아니라 **EffectiveStage** 를 렌더한다([04](04-screen-flow.md)).
- Gate 검증·자동생성·추적성 커버리지 모두 EffectiveStage 기준으로 동작([09](09-gate-validation.md)).
- 대시보드는 프로젝트별 "몇 번째 단계 / 몇 개 Gate 통과 / 다음 Gate 조건" 을 집계.

## 5. 데이터 모델 (요약)

상세는 [01-data-model §테일러링](01-data-model.md#7-테일러링-계층-tailoring).

```
METHODOLOGY 1─* STAGE            (마스터 정의, 버전)
PROJECT *─1 METHODOLOGY(version) (선택·고정)
PROJECT 1─1 TAILORING_PROFILE 1─* TAILORING_OP   (병합/생략/조정 + 근거·승인)
PROJECT 1─* EFFECTIVE_STAGE      (재계산본, state 로 진행관리)
  EFFECTIVE_STAGE.derived_from → STAGE[]          (추적: 어느 마스터 단계에서 왔나)
```

## 6. 화면 (목업 대응)
- **방법론 등록**: 등록된 방법론 목록(버전·단계수·컴플라이언스·상태) + 마스터 단계 상세.
- **테일러링**: 마스터 파이프라인 → 연산(병합/생략) → Effective 파이프라인 비교, 근거 입력·승인 상태.
- **파이프라인**: EffectiveStage 기준 진행 현황(기존 [04](04-screen-flow.md) 화면이 이를 렌더).
