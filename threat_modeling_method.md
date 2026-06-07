# Threat Modeling Agent 구축 절차

## 1단계: 방법론 선택

먼저 어떤 위협 모델링 프레임워크를 사용할지 결정합니다.

| 프레임워크 | 특징 | 적합한 대상 |
|---|---|---|
| **STRIDE** | 6가지 위협 범주 분류 | 일반 시스템/API |
| **PASTA** | 공격자 관점 중심, 7단계 프로세스 | 비즈니스 리스크 중심 |
| **LINDDUN** | 프라이버시 위협 전문 | 개인정보 처리 시스템 |
| **MITRE ATT&CK** | 실제 공격 기술 기반 | 엔터프라이즈/ICS |

---

## 2단계: Agent 입력 정의

Agent가 분석하기 위해 받아야 할 입력을 명확히 합니다.

```
필수 입력:
- 시스템 아키텍처 다이어그램 또는 설명
- 데이터 흐름도 (DFD)
- 신뢰 경계 (Trust Boundary) 정의
- 기술 스택 (언어, 프레임워크, 인프라)

선택 입력:
- 기존 보안 통제 목록
- 비즈니스 자산 우선순위
- 규정 준수 요건 (PCI-DSS, GDPR 등)
```

---

## 3단계: Agent 파이프라인 설계

```
[입력 수집] → [자산 식별] → [위협 열거] → [취약점 분석] → [리스크 평가] → [완화 방안] → [보고서 생성]
```

**Sub-agent 구성 (역할 분리):**

```
ThreatModelingAgent (Orchestrator)
├── AssetIdentificationAgent     # 자산 및 신뢰 경계 파악
├── ThreatEnumerationAgent       # STRIDE/ATT&CK 기반 위협 열거
├── VulnerabilityAnalysisAgent   # 실제 취약점 매핑
├── RiskScoringAgent             # CVSS/DREAD 점수 산정
└── MitigationAgent              # 완화 방안 제안
```

---

## 4단계: 각 Sub-agent 프롬프트 설계

**AssetIdentificationAgent 예시:**
```
주어진 시스템 설명에서 다음을 식별하라:
1. 데이터 자산 (민감도 분류 포함)
2. 서비스/컴포넌트 목록
3. 외부 인터페이스 및 신뢰 경계
4. 데이터 흐름 경로

출력 형식: JSON 구조체
```

**ThreatEnumerationAgent (STRIDE) 예시:**
```
각 컴포넌트/데이터흐름에 대해 STRIDE 위협을 열거하라:
- Spoofing: 인증 우회 가능 여부
- Tampering: 데이터 무결성 침해 가능 여부
- Repudiation: 부인 방지 미비 여부
- Information Disclosure: 정보 노출 가능 여부
- Denial of Service: 가용성 침해 가능 여부
- Elevation of Privilege: 권한 상승 가능 여부
```

---

## 5단계: 도구(Tool) 정의

Agent가 사용할 도구를 구현합니다.

```python
tools = [
    # 아키텍처 파싱
    parse_architecture_diagram(),   # 이미지/텍스트에서 컴포넌트 추출

    # 위협 데이터베이스 조회
    query_mitre_attack(),           # ATT&CK 기술/전술 검색
    query_cve_database(),           # 기술 스택별 CVE 조회
    query_owasp_top10(),            # OWASP 매핑

    # 리스크 계산
    calculate_cvss_score(),         # CVSS v3.1 점수 계산

    # 보고서 생성
    generate_threat_report(),       # Markdown/HTML 보고서 출력
    create_dfd_diagram(),           # 데이터 흐름도 생성
]
```

---

## 6단계: 리스크 평가 기준 정의

```
DREAD 모델 활용:
- Damage Potential (1-10)
- Reproducibility (1-10)
- Exploitability (1-10)
- Affected Users (1-10)
- Discoverability (1-10)

우선순위: Critical (40+) > High (30-39) > Medium (20-29) > Low (<20)
```

---

## 7단계: 출력 형식 정의

```markdown
# Threat Model Report

## 1. 시스템 개요
## 2. 자산 목록 및 신뢰 경계
## 3. 위협 목록 (우선순위 정렬)
   | ID | 위협 | 컴포넌트 | STRIDE | 리스크 점수 | 상태 |
## 4. 완화 방안
   - 단기 (즉시 조치)
   - 중기 (1-3개월)
   - 장기 (아키텍처 개선)
## 5. 잔여 리스크
```

---

## 8단계: 구현 스택 선택

```
Claude API (claude-opus-4-8 권장)
├── Agent SDK: anthropic python SDK
├── 멀티턴 대화: 시스템 설명을 단계별로 수집
├── Tool Use: 외부 DB 조회 연동
└── Structured Output: JSON 모드로 위협 목록 출력
```

---

## 9단계: 검증 및 반복

```
1. 알려진 취약 시스템으로 테스트 (DVWA, Juice Shop 등)
2. 전문가 리뷰와 Agent 출력 비교
3. False negative / False positive 측정
4. 프롬프트 및 도구 반복 개선
```
