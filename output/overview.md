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
| T-01 | Authentication Server 사칭을 통한 가짜 Access Token 발급 | Authentication Server | Spoofing | 3 | 2 | 6 | Medium |
| T-02 | 탈취된 유효 Access Token을 이용한 Replay 공격 | Gateway Token Validator | Spoofing | 3 | 2 | 6 | Medium |
| T-03 | 토큰 서명 Private Key 탈취로 인한 임의 토큰 위조 | Authentication Server | Tampering, Spoofing | 3 | 1 | 3 | Low |
| T-04 | Authentication Server를 대상으로 한 DoS/DDoS 공격 | Authentication Server | Denial of Service | 3 | 2 | 6 | Medium |
| T-05 | Gateway 토큰 검증 로직 우회 (Validation Bypass) | Gateway Token Validator | Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-06 | 최소 권한 원칙 위반 — 과도한 Scope가 포함된 토큰 발급 | Authentication Server | Elevation of Privilege | 2 | 2 | 4 | Medium |
| T-07 | 만료된 Access Token 수용 (토큰 갱신 체계 미흡) | Gateway Token Validator | Spoofing | 2 | 2 | 4 | Medium |
| T-08 | Authentication Server SPOF로 인한 전체 서비스 중단 | Authentication Server | Denial of Service | 3 | 2 | 6 | Medium |

#### Mitigation — Authentication

**[M-AUTH-1] Token Lifecycle Management** (T-01, T-02, T-07 공통 적용)
- Gateway ↔ Authentication Server 간 mTLS 적용으로 Auth Server 사칭 방지
- Access Token 수명 단기화 (15분 이하)
- Token Binding 또는 DPoP(Demonstration of Proof of Possession) 적용으로 Replay 방지
- 토큰 수락 시 `iss`, `aud`, `scope`, `exp` 엄격 검증

**[M-AUTH-2] Authentication Infrastructure Resilience** (T-04, T-08 공통 적용)
- Authentication Server HA 구성 (Active-Active, 복수 가용 영역)
- 인증 엔드포인트에 Rate Limiting 적용
- Circuit Breaker 패턴으로 Gateway의 Auth Server 의존성 완화

**[M-AUTH-3] Token Scope & Validation Hardening** (T-05, T-06 공통 적용)
- Gateway 토큰 검증 로직 정기 보안 감사 및 단위 테스트 강화
- iPaaS 등록 시 Scope 최소화 정책 수립 및 검토 프로세스 의무화
- Token Introspection 엔드포인트를 통한 실시간 유효성 재검증

---

### Category 2: Agent 특화

LLM 기반 Agent의 자율성, 동적 동작, 외부 입력 처리와 관련된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-09 | 사용자 입력을 통한 Direct Prompt Injection으로 Agent 행동 조작 | Authenticated Agent | Tampering, Elevation of Privilege | 3 | 3 | 9 | **High** |
| T-10 | 외부 데이터(Resource Server 콘텐츠 등) 경유 Indirect Prompt Injection | Authenticated Agent, Authenticated MCP Server | Tampering, Elevation of Privilege | 3 | 3 | 9 | **High** |
| T-11 | Agent 자율 행동으로 인한 의도치 않은 과도한 API 호출 | Authenticated Agent | Elevation of Privilege | 2 | 3 | 6 | Medium |
| T-12 | 런타임 Agent 비정상 행동 탐지 불가 (동적 검증 부재) | Gateway, Authenticated Agent | Repudiation | 2 | 3 | 6 | Medium |
| T-13 | Agent 응답에 민감 데이터 포함되어 외부 노출 | Authenticated Agent | Information Disclosure | 3 | 2 | 6 | Medium |
| T-14 | 악성 Agent/MCP Server 등록 (정적 검증 우회) | iPaaS 등록 프로세스 | Spoofing, Tampering | 3 | 1 | 3 | Low |
| T-15 | Unauthenticated Agent 간 직접 통신 (기술적 차단 없음) | Unauthenticated Agent A/B | Tampering, Information Disclosure | 2 | 3 | 6 | Medium |
| T-16 | Authenticated MCP Server를 경유한 SSRF (Server-Side Request Forgery) | Authenticated MCP Server | Elevation of Privilege, Information Disclosure | 2 | 2 | 4 | Medium |
| T-25 | Agent 재귀 호출/무한 루프로 인한 리소스·비용 소진 | Authenticated Agent, Gateway | Denial of Service | 2 | 2 | 4 | Medium |
| T-26 | Agent 시스템 프롬프트 노출 (System Prompt Leakage) | Authenticated Agent | Information Disclosure | 2 | 2 | 4 | Medium |
| T-27 | LLM Jailbreak를 통한 Agent 보안 제약 우회 | Authenticated Agent | Elevation of Privilege, Tampering | 3 | 2 | 6 | Medium |
| T-28 | MCP Tool Definition Poisoning (도구 정의에 악성 지시어 삽입) | Authenticated MCP Server | Tampering, Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-29 | Multi-Agent 신뢰 체인 오염 (침해된 Agent가 타 Agent에 악성 요청 전파) | Authenticated Agent (A→B) | Spoofing, Tampering, Elevation of Privilege | 3 | 2 | 6 | Medium |

#### Mitigation — Agent 특화

**[M-AGENT-1] Prompt Injection & Jailbreak Defense** (T-09, T-10, T-27, T-28 공통 적용 — High/Medium)
- 시스템 프롬프트 격리 및 사용자/외부 입력 컨텍스트와의 명확한 경계 설정
- Agent가 처리하는 외부 데이터(Resource Server 콘텐츠, MCP 응답, **MCP Tool 정의**)를 신뢰되지 않는 컨텍스트로 명시적 분리
- Tool Call 허용 목록(Whitelist) 기반 제한 — Agent가 호출 가능한 도구를 명시적으로 제한
- 입력 전처리(Sanitization) 레이어 구현: 악성 지시어 패턴 필터링
- Jailbreak 방어: LLM 안전 훈련 기반 제약 + Guardrail 레이어(별도 분류 모델로 출력 검증)
- MCP Tool 정의 메타데이터(이름·설명)에 대한 등록 시 콘텐츠 스캐닝 적용

**[M-AGENT-2] Runtime Behavior Monitoring & Loop Prevention** (T-11, T-12, T-15, T-25, T-29 공통 적용)
- Gateway에서 Agent별 API 호출 패턴 실시간 모니터링 및 이상 탐지 Rule 적용
- 고위험 작업(데이터 대량 조회, 쓰기/삭제 등)에 대한 Human-in-the-Loop 검토 게이트 설정
- Tool Call 호출 빈도 및 **최대 반복 횟수(Max Steps)** Rate Limiting 적용 — 무한 루프 차단
- Unauthenticated Agent 간 통신 네트워크 레벨 차단 (서비스 메시 정책 또는 방화벽 규칙)
- Agent-to-Agent 요청(UC5) 수신 시 요청 출처 Agent의 **현재 침해 여부 판단 컨텍스트** 검증 (신뢰 체인 오염 방지)

**[M-AGENT-3] System Prompt Protection** (T-26 적용)
- 시스템 프롬프트에 민감 정보(내부 정책, 자격증명, 타 사용자 컨텍스트) 포함 금지
- Agent 응답 후처리 필터로 시스템 프롬프트 내용 포함 여부 탐지 및 차단
- 운영 환경과 개발 환경 시스템 프롬프트 분리 관리

---

### Category 3: Authorization

리소스 접근 권한, 조직 간 격리, 감사 체계와 관련된 위협.

| ID | 위협 설명 | 대상 컴포넌트 | STRIDE | Impact | Likelihood | Risk | 우선순위 |
|---|---|---|---|---|---|---|---|
| T-17 | Unauthenticated Agent가 PAT Token 내장으로 Resource Server 직접 접근 | Resource Server, PAT Token | Elevation of Privilege, Spoofing | 3 | 3 | 9 | **High** |
| T-18 | Unauthenticated MCP Server를 경유한 Resource Server 무단 접근 | Resource Server, Unauthenticated MCP Server | Elevation of Privilege, Information Disclosure | 3 | 3 | 9 | **High** |
| T-19 | Chinese Wall 정책 위반으로 인한 조직 간 데이터 상호 접근 | Resource Server, Gateway | Information Disclosure, Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-20 | Unauthenticated 경로 접근으로 인한 Audit Trail 부재 (감사 우회) | Audit Log, Gateway Audit Logger | Repudiation | 3 | 3 | 9 | **High** |
| T-21 | Authenticated Agent/MCP Server의 할당 Scope 초과 접근 | Authenticated Agent, Authenticated MCP Server | Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-22 | MCP Server를 우회한 Resource Server 직접 접근 (네트워크 통제 불완전) | Resource Server | Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-23 | 멀티 사용자 환경에서 요청 컨텍스트 혼용에 의한 타 사용자 데이터 접근 | Authenticated Agent, Resource Server | Information Disclosure, Elevation of Privilege | 3 | 2 | 6 | Medium |
| T-24 | 감사 로그 조작/삭제로 인한 행위 사실 부인 (내부자 위협) | Audit Log | Tampering, Repudiation | 2 | 1 | 2 | Low |
| T-30 | MCP Server Confused Deputy — 저권한 사용자 요청을 MCP 고권한 토큰으로 처리 | Authenticated MCP Server, Resource Server | Elevation of Privilege | 3 | 2 | 6 | Medium |

#### Mitigation — Authorization

**[M-AUTHZ-1] Network Perimeter Enforcement** (T-17, T-18, T-22 공통 적용 — High)
- Resource Server에 대한 네트워크 레벨 접근 통제 구현: Authenticated MCP Server 이외의 모든 인바운드 연결 차단 (방화벽, 서비스 메시 정책, VPC Security Group)
- PAT Token 내장 관행 전수 감사 및 제거 — Unauthenticated Agent 코드 정책 수립
- 가이드 기반 통제에서 **강제적 기술 통제(Technical Control)**로 전환 (Design 8, Assumption 8 해소)

**[M-AUTHZ-2] Audit Trail Completeness & Integrity** (T-20 공통 적용 — High, T-24 포함)
- 네트워크 레벨 트래픽 모니터링 (IDS/IPS, VPC Flow Log)으로 Gateway 우회 접근 탐지 및 실시간 알람
- 감사 로그 암호학적 무결성 보호: Append-only 저장소, Hash Chain 또는 WORM(Write Once Read Many) 스토리지 적용
- Gateway 외 경로의 Resource Server 접근 시도를 별도 보안 이벤트로 분류 및 알람

**[M-AUTHZ-3] Fine-grained Authorization Enforcement** (T-19, T-21, T-23, T-30 공통 적용)
- 조직 소속 정보를 포함한 토큰 클레임(`org_id`, `user_id`) 기반 Chinese Wall 검증 — Gateway와 Resource Server 이중 검증
- 사용자 요청 처리 시 Agent 내 컨텍스트 격리: 요청별 독립 토큰 사용 (per-request token)
- Resource Server Row-Level Security 적용으로 조직/사용자 범위 외 데이터 자동 차단
- MCP Server가 Resource Server 접근 시 **요청 원본 사용자의 권한 클레임을 On-Behalf-Of(OBO) 방식으로 전달** — MCP 자체 토큰이 아닌 사용자 권한 기반으로 Resource Server 인가 결정 (Confused Deputy 방지)

---

### 우선순위별 Threat 요약

| 우선순위 | Threat IDs | 건수 |
|---|---|---|
| High | T-09, T-10, T-17, T-18, T-20 | 5건 |
| Medium | T-01, T-02, T-04, T-05, T-06, T-07, T-08, T-11, T-12, T-13, T-15, T-16, T-19, T-21, T-22, T-23, T-25, T-26, T-27, T-28, T-29, T-30 | 22건 |
| Low | T-03, T-14, T-24 | 3건 |

---

### Category별 Threat 매핑

| Threat ID | Authentication | Agent 특화 | Authorization |
|---|---|---|---|
| T-01 | ✓ | | |
| T-02 | ✓ | | |
| T-03 | ✓ | | |
| T-04 | ✓ | | |
| T-05 | ✓ | | ✓ |
| T-06 | ✓ | | ✓ |
| T-07 | ✓ | | |
| T-08 | ✓ | | |
| T-09 | | ✓ | ✓ |
| T-10 | | ✓ | ✓ |
| T-11 | | ✓ | ✓ |
| T-12 | | ✓ | ✓ |
| T-13 | | ✓ | ✓ |
| T-14 | ✓ | ✓ | |
| T-15 | | ✓ | ✓ |
| T-16 | | ✓ | ✓ |
| T-17 | ✓ | | ✓ |
| T-18 | | ✓ | ✓ |
| T-19 | | | ✓ |
| T-20 | | | ✓ |
| T-21 | | ✓ | ✓ |
| T-22 | | | ✓ |
| T-23 | | | ✓ |
| T-24 | | | ✓ |
| T-25 | | ✓ | |
| T-26 | ✓ | ✓ | |
| T-27 | | ✓ | |
| T-28 | | ✓ | |
| T-29 | | ✓ | ✓ |
| T-30 | ✓ | | ✓ |

---

### Validation

#### 검토 출처

| 출처 | 버전 | 적용 범위 |
|---|---|---|
| OWASP Top 10 for LLM Applications | 2025 | LLM 기반 Agent 특화 위협 |
| OWASP API Security Top 10 | 2023 | Gateway/API 구조 위협 |
| MITRE ATLAS | 2024 | AI/LLM Agent 공격 전술 |
| MITRE ATT&CK Enterprise | v15 | 인증·권한·감사 관련 공격 전술 |

#### 기존 위협 커버리지 확인

| 출처 항목 | 커버 여부 | 관련 Threat ID |
|---|---|---|
| OWASP LLM01 Prompt Injection | ✓ | T-09, T-10 |
| OWASP LLM02 Sensitive Information Disclosure | ✓ | T-13 |
| OWASP LLM03 Supply Chain Vulnerabilities | ✓ 부분 | T-14 |
| OWASP LLM04 Data and Model Poisoning | — | Out of Scope |
| OWASP LLM05 Improper Output Handling | ✓ | T-13 |
| OWASP LLM06 Excessive Agency | ✓ | T-11 |
| OWASP LLM07 System Prompt Leakage | **신규** | → T-26 |
| OWASP LLM08 Vector/Embedding Weaknesses | — | Out of Scope (RAG 없음) |
| OWASP LLM09 Misinformation | — | Out of Scope |
| OWASP LLM10 Unbounded Consumption | **신규** | → T-25 |
| OWASP API1 Broken Object Level Authorization | ✓ | T-23 |
| OWASP API2 Broken Authentication | ✓ | T-01, T-02, T-05 |
| OWASP API3 Broken Object Property Level Authorization | ✓ | T-21 |
| OWASP API4 Unrestricted Resource Consumption | ✓ 부분 | T-04 |
| OWASP API5 Broken Function Level Authorization | ✓ | T-21 |
| OWASP API6 Unrestricted Access to Sensitive Business Flows | **신규** | → T-29 |
| OWASP API7 SSRF | ✓ | T-16 |
| OWASP API8 Security Misconfiguration | ✓ 부분 | T-05, T-06 |
| OWASP API9 Improper Inventory Management | ✓ | T-14 |
| OWASP API10 Unsafe Consumption of APIs | ✓ | T-10 |
| MITRE ATLAS AML.T0051 LLM Prompt Injection | ✓ | T-09, T-10 |
| MITRE ATLAS AML.T0054 LLM Jailbreak | **신규** | → T-27 |
| MITRE ATLAS AML.T0043 Craft Adversarial Data | ✓ | T-10 |
| MITRE ATT&CK T1528 Steal Application Access Token | ✓ | T-02 |
| MITRE ATT&CK T1078 Valid Accounts | ✓ | T-17 |
| MITRE ATT&CK T1195 Supply Chain Compromise | ✓ 부분 | T-14 |
| MITRE ATT&CK T1562 Impair Defenses | ✓ | T-20 |
| MITRE ATT&CK T1550 Use Alternate Authentication Material | **신규** | → T-29 |

#### 신규 추가 위협 요약

Validation을 통해 T-25 ~ T-30 (6건)을 신규 추가하였으며, 모두 Medium 우선순위로 분류되었다. 신규 High Risk 위협은 식별되지 않았다.

| Threat ID | 위협 설명 | 출처 | 추가 Category |
|---|---|---|---|
| T-25 | Agent 재귀 호출/무한 루프로 인한 리소스·비용 소진 | OWASP LLM10:2025 | Agent 특화 |
| T-26 | Agent 시스템 프롬프트 노출 (System Prompt Leakage) | OWASP LLM07:2025 | Agent 특화 |
| T-27 | LLM Jailbreak를 통한 Agent 보안 제약 우회 | MITRE ATLAS AML.T0054 | Agent 특화 |
| T-28 | MCP Tool Definition Poisoning (도구 정의에 악성 지시어 삽입) | OWASP LLM07:2025 (Plugin Design) | Agent 특화 |
| T-29 | Multi-Agent 신뢰 체인 오염 (침해된 Agent → 타 Agent 악성 요청 전파) | OWASP API6, MITRE ATT&CK T1550 | Agent 특화, Authorization |
| T-30 | MCP Server Confused Deputy (저권한 사용자가 MCP 고권한 토큰 활용) | OWASP API1 심화 | Authorization, Authentication |

> Threat Identification 섹션을 T-25 ~ T-30 반영하여 업데이트 완료.
> 총 위협: 24건 → **30건** (High 5건 / Medium 22건 / Low 3건)

