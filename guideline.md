# Guidelines

## Instruction for agent

### Overview
- scenario.md를 참고하여 Assumption, Scope, Assets, Use cases, Context Diagram, DFD, STRIDE 분석을 작성하여 output/overview.md 파일 생성. 이미 있으면 덮어쓰기. 주언어: 한글, 보조언어: 영어.
- backup 폴더의 내용은 참고하지 않는다.

### Step by step instruction
- Assumption
- Scope
- Assets
- Use Cases
  - UC(number): (Use case description) 양식으로 작성
- Context Diagram
  - All text is in English
  - Assumption, Scope, Use cases를 참고하여 `./output/context.mmd` 파일로 작성
  - 줄바꿈은 <br> 이용
- DFD
  - All text is in English
  - Context Diagram, Assumption, Scope, Use cases를 참고해서 `./output/dfd.mmd` 생성
  - 줄바꿈은 <br> 이용
  - 전체 dfd는 `./output/dfd.mmd` 파일에 그리기
  - 필요하면 `dfd_(number)_(category).mmd` 파일로 Use case를 몇 개 묶어서 DFD 작성
- Threat identification
  - STRIDE 기반으로 모든 컴포넌트에 Threat 들을 열거한다.
  - Threat 들에 대해서 Impact, Likelihood, Risk 점수를 매긴다.
  - Threat 들을 비슷한 Category끼리 묶어서 정리한다. Threat이 여러 Category에 속할 수 있다.
    - Category: Authentication, Agent 특화, Authorization
  - Risk 점수가 높은 Threat 들에 대해서는 Mitigation 방안을 제시한다.
    - Category 내에서 같은 Mitigation으로 해결될 수 있으면 좋다.
- Validation
  - 여기에 작성되지 않은 Threat이 또 뭐가 있을지 생각. OWASP이나 MITRE 에서 검색해서 확인하면 더 확실할 듯.
  - Validation에는 추가된 내용을 요약해서 기재하고 Threat identification 섹션을 다시 업데이트 한다.

# Template

## Assumption

## Scope

## Assets

## Use Cases

## Context Diagram
> Source: [context.mmd](./output/context.mmd)

## DFD
> Source: [dfd.mmd](./output/dfd.mmd)

## Threat identification

### 평가 기준

| 점수 | Impact / Likelihood |
|---|---|
| 3 | 높음 |
| 2 | 보통 |
| 1 | 낮음 |

**Risk Score = Impact × Likelihood**

| 우선순위 | Risk Score |
|---|---|
| High | 7 ~ 9 |
| Medium | 4 ~ 6 |
| Low | 1 ~ 3 |

> 하나의 Threat이 여러 Category에 속할 수 있음

---

### Category 1: Authentication

인증 인프라의 가용성, 토큰 발급/검증, 신원 확인과 관련된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-01 | Authentication Server 사칭으로 가짜 Access Token 발급 | Authentication Server | Spoofing | 5 | 2 | 10 | Medium |

#### Mitigation — Authentication

**[M-AUTH-1] Token Infrastructure Hardening** (T-02, T-06, T-17 공통 적용)

---

#### Mitigation — Agent 특화

**[M-AGENT-1] Input/Output Sanitization** (T-09, T-14, T-18, T-20 공통 적용)

---

### 우선순위별 Threat 요약

| 우선순위 | Threat IDs | 건수 |
|---|---|---|
| Critical | T-05, T-16, T-17, T-19 | 4건 |


### Category별 Threat 매핑

| Threat ID | Authentication | Agent 특화 | Authorization |
|---|---|---|---|
| T-01 | ✓ | | |

### Validation