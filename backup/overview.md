# 시스템 개요 (Overview)

> 대상 시스템: Enterprise AI Agent Gateway 아키텍처  
> 분석 방법론: STRIDE  
> 주언어: 한글 / 보조언어: 영어

---

## Assumption

1. 모든 Authenticated Agent와 MCP Server는 Gateway(iPaaS)에 등록 및 정적 검증을 완료한 상태이다.
2. Authentication Server는 OAuth 2.0 기반의 Access Token을 발급하며, 토큰에는 Agent/MCP Server의 권한 범위(Scope)와 소속 조직 정보가 포함된다.
3. 사용자는 반드시 특정 조직(Organization)에 소속되어 있으며, 조직 간에는 Chinese Wall 정책이 적용된다.
4. Gateway는 Authenticated Agent와 MCP Server 간의 유일한 공식 통신 경로이다. Agent-to-Agent 및 Agent-to-MCP 통신은 반드시 Gateway를 경유한다.
5. Unauthenticated Agent와 MCP Server는 현재 Gateway를 우회하여 Resource Server에 직접 접근 가능한 상태이다 (위협 시나리오).
6. 모든 Authenticated 트래픽은 TLS로 암호화된다.
7. Audit Log는 Trusted Domain 내에서만 접근 가능하며, Gateway만이 쓰기 권한을 가진다.
8. Resource Server는 MCP Server를 통해서만 접근하도록 설계 가이드가 존재하나, 이를 강제하는 네트워크 통제가 현재 불완전하다.
9. Unauthenticated Agent 간의 직접 통신은 현재 기술적으로 차단되어 있지 않다.
10. iPaaS 등록 시 정적 검증(Static Validation)을 수행하지만, 런타임 동작에 대한 동적 검증은 제한적이다.

---

## Scope

### In Scope

- Gateway(iPaaS)를 통한 Agent 및 MCP Server 인증/인가 체계
- Trusted Domain과 Untrusted Domain 간의 신뢰 경계 보안
- 사용자 및 조직 기반 Resource Server 접근 권한 관리 (Chinese Wall 포함)
- Unauthenticated Agent/MCP Server의 Resource Server 직접 접근 위협
- Gateway를 통한 트래픽 감사(Audit) 체계
- Agent 자율성(Autonomy)으로 인한 의도치 않은 행위 위협 (Prompt Injection 포함)
- Authenticated Agent 및 MCP Server의 권한 범위 초과 위협

### Out of Scope

- Resource Server 내부 데이터 처리 로직 및 내부 보안
- 개별 Agent의 AI 모델 자체 취약점 (Hallucination, Model Poisoning 등)
- 인프라(네트워크, OS, 하드웨어) 수준의 보안
- 사용자 단말기 및 클라이언트 측 보안
- Gateway 내부 구현 세부 사항 (코드 수준 취약점)

---

## Assets

| 자산 (Asset) | 유형 | 민감도 | 설명 |
|---|---|---|---|
| Access Token | 데이터 | 높음 | 인증된 Agent/MCP Server의 API 호출 권한 증명 |
| PAT Token | 데이터 | 높음 | Unauthenticated Agent가 내장할 수 있는 개인 접근 토큰 |
| 토큰 서명 키 (Private Key) | 데이터 | 매우 높음 | Authentication Server의 토큰 서명에 사용되는 비밀 키 |
| Resource Server 데이터 | 데이터 | 매우 높음 | 사용자/조직의 비즈니스 데이터 (조직 간 격리 필요) |
| 사용자 자격증명 | 데이터 | 높음 | 사용자 인증 정보 |
| 감사 로그 | 데이터 | 높음 | Gateway를 통과한 모든 트래픽의 행위 기록 |
| 조직/사용자 권한 정책 | 데이터 | 높음 | Chinese Wall 정책 및 RBAC 규칙 |
| Authentication Server | 서비스 | 매우 높음 | 토큰 발급 및 검증의 핵심 인프라 (SPOF 위험) |
| Gateway (iPaaS) | 서비스 | 매우 높음 | 모든 인증 트래픽의 단일 진입점 (SPOF 위험) |
| Authenticated Agent | 서비스 | 높음 | 사용자 요청을 처리하는 인증된 자율 에이전트 |
| Authenticated MCP Server | 서비스 | 높음 | Resource Server 접근을 제공하는 인증된 MCP 서비스 |

---

## Use Cases

- **UC1**: 사용자가 Authenticated Agent에 업무 작업을 요청한다.
- **UC2**: Authenticated Agent가 Authentication Server에 Access Token을 요청하고 발급받는다.
- **UC3**: Authenticated MCP Server가 Authentication Server에 Access Token을 요청하고 발급받는다.
- **UC4**: Authenticated Agent가 Access Token을 포함하여 Gateway를 통해 Authenticated MCP Server에 API를 호출한다.
- **UC5**: Authenticated Agent가 Access Token을 포함하여 Gateway를 통해 다른 Authenticated Agent에 요청을 전달한다.
- **UC6**: Gateway가 Access Token 유효성을 Authentication Server에 검증한 후 요청을 인가된 대상으로 포워딩한다.
- **UC7**: Authenticated MCP Server가 Gateway로부터 받은 요청을 처리하여 Resource Server에서 데이터를 조회한다.
- **UC8**: Gateway가 모든 통과 트래픽을 Audit Log에 기록한다.
- **UC9**: iPaaS 등록 시 MCP Server/Agent에 대한 정적 검증(서명 검사, 메타데이터 유효성 등)을 수행한다.
- **UC10**: Unauthenticated Agent가 PAT Token을 내장하여 Gateway를 우회하고 Resource Server에 직접 접근한다. *(위협 시나리오)*
- **UC11**: Unauthenticated Agent가 자체 Unauthenticated MCP Server를 경유하여 Resource Server에 직접 접근한다. *(위협 시나리오)*
- **UC12**: Unauthenticated Agent 간에 Gateway를 거치지 않고 직접 통신한다. *(위협 시나리오)*

---

## Context Diagram

> Source: [context.mmd](./output/context.mmd)

---

## DFD

> 전체 DFD: [dfd.mmd](./output/dfd.mmd)

| 서브 DFD | 대상 Use Cases | 설명 |
|---|---|---|
| [dfd_1_auth.mmd](./output/dfd_1_auth.mmd) | UC2, UC3, UC6, UC9 | 인증 및 등록 플로우 |
| [dfd_2_request.mmd](./output/dfd_2_request.mmd) | UC1, UC4, UC5, UC7, UC8 | 정상 요청 처리 플로우 |
| [dfd_3_threat.mmd](./output/dfd_3_threat.mmd) | UC10, UC11, UC12 | Unauthenticated 우회 위협 시나리오 |

---

## Threat Identification

### 평가 기준

| 점수 | Impact / Likelihood |
|---|---|
| 5 | 매우 높음 |
| 4 | 높음 |
| 3 | 보통 |
| 2 | 낮음 |
| 1 | 매우 낮음 |

**Risk Score = Impact × Likelihood**

| 우선순위 | Risk Score |
|---|---|
| Critical | 20 ~ 25 |
| High | 12 ~ 19 |
| Medium | 6 ~ 11 |
| Low | 1 ~ 5 |

> 하나의 Threat이 여러 Category에 속할 수 있음

---

### Category 1: Authentication

인증 인프라의 가용성, 토큰 발급/검증, 신원 확인과 관련된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-01 | Authentication Server 사칭으로 가짜 Access Token 발급 | Authentication Server | Spoofing | 5 | 2 | 10 | Medium |
| T-02 | 토큰 서명 키(Private Key) 탈취로 임의 토큰 위조 | Authentication Server | Info Disclosure | 5 | 3 | 15 | High |
| T-03 | Authentication Server DDoS로 전체 인증 서비스 불가 | Authentication Server | DoS | 5 | 3 | 15 | High |
| T-05 | Gateway DDoS: 단일 진입점 공격으로 전체 인증 트래픽 마비 | Gateway (iPaaS) | DoS | 5 | 4 | 20 | **Critical** |
| T-06 | 위조 또는 만료된 토큰으로 Gateway 인가 우회 시도 | Gateway (iPaaS) | Spoofing | 4 | 3 | 12 | High |
| T-12 | Unauthenticated Agent가 Authenticated Agent ID 사칭 | Authenticated Agent | Spoofing | 4 | 3 | 12 | High |
| T-17 | Unauthenticated Agent 내 PAT Token 탈취로 정상 사용자 권한 도용 | Unauthenticated Agent | Info Disclosure | 5 | 5 | 25 | **Critical** |

#### Mitigation — Authentication

**[M-AUTH-1] Token Infrastructure Hardening** (T-02, T-06, T-17 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | PAT Token 신규 발급 중단 및 기존 PAT Token 만료 일정 수립 |
| 단기 | SAST/Secret Scanning으로 코드베이스 내 Token 내장 탐지 |
| 단기 | Access Token 만료 시간 단축 및 Refresh Token Rotation 적용 |
| 중기 | 토큰 서명 키를 HSM(Hardware Security Module)에서 관리 |
| 중기 | PAT Token → OAuth 2.0 Client Credentials 방식으로 전환 |
| 중기 | Token 즉시 폐기(Revoke) 메커니즘 자동화 |
| 장기 | Zero Trust 기반으로 모든 인증을 Gateway 경유 OAuth 토큰으로 통일 |

**[M-AUTH-2] Authentication Infrastructure Resilience** (T-03, T-05 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | Rate Limiting 및 IP-based 차단 규칙 적용 |
| 단기 | WAF/DDoS 방어 솔루션 Gateway 앞단 배치 |
| 중기 | Authentication Server 및 Gateway HA 구성 (Active-Active) |
| 중기 | Circuit Breaker 패턴 적용으로 연쇄 장애 방지 |
| 장기 | Gateway 수평 확장(Horizontal Scaling) 및 Multi-Region 배포 |

**[M-AUTH-3] Mutual Identity Verification** (T-01, T-12 공통 적용)

| 구분 | Mitigation |
|---|---|
| 중기 | mTLS(Mutual TLS) 적용으로 Agent/MCP Server 상호 인증 강화 |
| 중기 | Agent ID 서명 및 검증 체계 수립 (등록된 Agent만 인증 요청 허용) |

---

### Category 2: Agent 특화

AI Agent의 자율성(Autonomy), MCP 프로토콜, Prompt Injection에 특화된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-09 | Prompt Injection을 통한 Authenticated Agent 행위 조작 | Authenticated Agent | Tampering | 4 | 4 | 16 | High |
| T-10 | Agent 자율성(Autonomy)으로 인한 의도치 않은 권한 범위 초과 행위 | Authenticated Agent | EoP | 4 | 4 | 16 | High |
| T-11 | Agent가 사용자 요청에서 민감 정보를 과도하게 수집/처리 | Authenticated Agent | Info Disclosure | 4 | 3 | 12 | High |
| T-14 | MCP Tool 정의 또는 응답 변조를 통한 Indirect Prompt Injection | Authenticated MCP Server | Tampering | 4 | 3 | 12 | High |
| T-18 | Unauthenticated Agent 간 직접 통신을 통한 악의적 페이로드 전파 | Unauthenticated Agent | Tampering | 4 | 4 | 16 | High |
| T-20 | 악의적 Unauthenticated MCP Server가 조작된 데이터를 Agent에 반환 | Unauthenticated MCP Server | Tampering | 4 | 3 | 12 | High |

> T-10은 Authorization 카테고리와 중복

#### Mitigation — Agent 특화

**[M-AGENT-1] Input/Output Sanitization** (T-09, T-14, T-18, T-20 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | Unauthenticated Agent 출력을 Authenticated Agent 입력으로 허용 금지 정책 수립 |
| 중기 | Gateway 수준에서 요청/응답 내 Prompt Injection 패턴 탐지 및 차단 |
| 중기 | MCP Tool 정의 및 응답에 대한 콘텐츠 검증(Schema Validation) 및 Sanitization 적용 |
| 장기 | LLM Output Guardrail 솔루션 도입으로 Agent 출력 실시간 모니터링 |

**[M-AGENT-2] Agent Behavior Guardrails** (T-10, T-11 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | Agent가 수행 가능한 액션의 사전 정의된 Allowlist 운영 |
| 중기 | 민감 데이터 처리 및 외부 API 호출 전 사용자 확인(Human-in-the-Loop) 절차 적용 |
| 중기 | Agent 실행 결과에 대한 실시간 이상 탐지(Anomaly Detection) 및 알림 |
| 장기 | 최소 권한 원칙 기반 Agent Tool 접근 권한 설계 |

---

### Category 3: Authorization

접근 제어, 권한 분리, 감사 체계와 관련된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-04 | 낮은 권한 Agent가 높은 권한 Scope 토큰 발급 유도 (Scope Escalation) | Authentication Server | EoP | 5 | 3 | 15 | High |
| T-07 | Gateway 설정 오류로 토큰 검증 없이 요청 처리 | Gateway (iPaaS) | EoP | 5 | 2 | 10 | Medium |
| T-08 | Gateway 장애 또는 오류 시 감사 로그 미기록 | Gateway (iPaaS) | Repudiation | 3 | 2 | 6 | Medium |
| T-10 | Agent 자율성으로 인한 의도치 않은 권한 범위 초과 | Authenticated Agent | EoP | 4 | 4 | 16 | High |
| T-13 | 악의적 MCP Server가 정상 MCP Server로 위장하여 iPaaS 등록 | Authenticated MCP Server | Spoofing | 4 | 2 | 8 | Medium |
| T-15 | MCP Server 권한 오설정으로 타 조직 데이터 접근 (Chinese Wall 위반) | Authenticated MCP Server | EoP | 5 | 3 | 15 | High |
| T-16 | Unauthenticated Agent가 PAT Token 내장으로 Resource Server 직접 무단 접근 | Unauthenticated Agent | EoP | 5 | 5 | 25 | **Critical** |
| T-19 | 미검증 Unauthenticated MCP Server를 통한 Resource Server 직접 무단 접근 | Unauthenticated MCP Server | EoP | 5 | 4 | 20 | **Critical** |
| T-21 | 감사 로그 변조 또는 삭제로 포렌식 증거 훼손 | Audit Log | Tampering | 4 | 3 | 12 | High |
| T-22 | 감사 로그 저장소 과부하로 감사 기능 마비 및 로그 유실 | Audit Log | DoS | 4 | 3 | 12 | High |
| T-23 | 사용자 간 권한 혼용으로 타 사용자 데이터 노출 | Cross-Boundary | Info Disclosure | 5 | 3 | 15 | High |
| T-24 | Chinese Wall 위반: 타 조직 데이터 무단 접근 | Cross-Boundary | Info Disclosure | 5 | 3 | 15 | High |

> T-04는 Authentication 카테고리와 중복  
> T-10은 Agent 특화 카테고리와 중복

#### Mitigation — Authorization

**[M-AUTHZ-1] Gateway Bypass Prevention** (T-16, T-19, T-07 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | Resource Server 네트워크 레벨에서 Gateway IP만 허용 (IP Allowlist) |
| 단기 | Resource Server와 MCP Server 간 mTLS 적용, Gateway 인증서만 허용 |
| 중기 | PAT Token 폐지 및 OAuth 2.0 Client Credentials 방식으로 Gateway 경유 강제화 |
| 중기 | Resource Server 요청 헤더 내 Gateway 서명 검증 미들웨어 적용 |
| 중기 | MCP Server 등록 의무화 및 미등록 MCP Server의 Resource Server 접근 네트워크 차단 |
| 장기 | Service Mesh 도입으로 모든 서비스 간 통신에 인증/인가 강제화 (Zero Trust) |

**[M-AUTHZ-2] Fine-grained Access Control** (T-04, T-10, T-15, T-23, T-24 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | 최소 권한 원칙(Least Privilege) 기반으로 Agent/MCP Scope 재설계 |
| 중기 | 조직별 Chinese Wall을 Access Token Claim 및 MCP Server 권한 정책에 반영 |
| 중기 | 사용자별 Resource Server 접근 권한을 Gateway 정책으로 강제화 (RBAC) |
| 중기 | Scope Escalation 방지: Authentication Server에서 요청 Scope와 등록 Scope 일치 검증 |
| 장기 | 동적 접근 제어 정책(ABAC) 도입으로 조직/사용자/컨텍스트 기반 세밀한 권한 관리 |

**[M-AUTHZ-3] Audit Integrity** (T-08, T-21, T-22 공통 적용)

| 구분 | Mitigation |
|---|---|
| 단기 | 감사 로그를 별도 Write-Once 스토리지(예: WORM)에 저장하여 변조 방지 |
| 중기 | 로그 저장소 HA 구성 및 용량 자동 확장 설정 |
| 중기 | 로그 누락 감지 알림 체계 구축 (Gateway 장애 시 fallback 로깅) |
| 장기 | 로그 무결성 검증을 위한 Hash Chain 또는 블록체인 기반 감사 로그 관리 |

---

### 우선순위별 Threat 요약

| 우선순위 | Threat IDs | 건수 |
|---|---|---|
| Critical | T-05, T-16, T-17, T-19 | 4건 |
| High | T-02, T-03, T-04, T-06, T-09, T-10, T-11, T-12, T-14, T-15, T-18, T-20, T-21, T-22, T-23, T-24 | 16건 |
| Medium | T-01, T-07, T-08, T-13 | 4건 |
| Low | - | 0건 |

### Category별 Threat 매핑

| Threat ID | Authentication | Agent 특화 | Authorization |
|---|---|---|---|
| T-01 | ✓ | | |
| T-02 | ✓ | | |
| T-03 | ✓ | | |
| T-04 | ✓ | | ✓ |
| T-05 | ✓ | | |
| T-06 | ✓ | | |
| T-07 | | | ✓ |
| T-08 | | | ✓ |
| T-09 | | ✓ | |
| T-10 | | ✓ | ✓ |
| T-11 | | ✓ | |
| T-12 | ✓ | | |
| T-13 | | | ✓ |
| T-14 | | ✓ | |
| T-15 | | | ✓ |
| T-16 | | | ✓ |
| T-17 | ✓ | | |
| T-18 | | ✓ | |
| T-19 | | | ✓ |
| T-20 | | ✓ | |
| T-21 | | | ✓ |
| T-22 | | | ✓ |
| T-23 | | | ✓ |
| T-24 | | | ✓ |
