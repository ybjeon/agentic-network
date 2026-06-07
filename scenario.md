# Scenario

## Entity
- User
- Resource server
- Trusted domain
  - Authentication server
  - Gateway (iPaaS)
  - Agent (Authenticated)
  - MCP Server (Authenticated)
  - Audit Log
- Untrusted Domain
  - Agent A (Unauthenticated)
  - Agent B (Unauthenticated)
  - MCP Server (Unauthenticated)

## Backgrounds
- Agent의 자율성 때문에 의도하지 않은 action들이 많이 일어난다.
- 사내에 여러 Agent 들이 존재하여 거버넌스가 필요하다.
- 모든 Agent와 MCP Server의 트래픽은 Proxy를 통해서 통신하고 있다.
- 신규 Gateway 도입
    - Threat 들로부터 Agent를 지키기 위해 Gateway를 설치하고, 인증받은 MCP Server와 Agent들의 권한도 관리하고자 한다.
    - MCP Server와 Agent는 iPaaS 등록 시 정적 검증을 받는다.
    - Authenticated Agent와 MCP 서버는 iPaaS에 등록해야 한다.
    - Authenticated Agent<->Agent, MCP<-> Agent는 Gateway를 통해서만 통신한다.
    - User는 Authenticated Agent에 요청을 보내고, Authenticated Agent가 Authentication Server에 인증을 요청한다.
    - Authenticated Agent는 인증 결과로 받은 Access Token을 이용하여 Gateway를 통해 통신한다.
    - MCP server도 마찬가지로 인증 과정을 거쳐서 Access Token을 발급받고, Gateway를 통해 통신한다.
    - 최종적으로 Gateway를 거친 모든 Agent와 MCP Server traffic은 auditing 된다.
    - Gateway는 Resource server와 직접 연결하지 않고, MCP Server를 거쳐서 통신한다.
- Unauthenticated Agent끼리도 Gateway를 거치지 않고 통신을 할 수 있다.
- 고려해야 될 문제
  - Unauthenticated Agent끼리도 Gateway를 거치지 않고 통신을 할 수 있어서 문제가 될 수 있다.
  - Agent마다 사용자가 여러명 존재할 수 있다. 사용자마다 Resource server 접근 권한이 달라 권한 관리가 필요하다. 사용자마다 속해있는 조직이 존재하고, 조직마다 Resource server 접근 권한이 달라서 권한 관리가 필요하다. 조직끼리는 chinese wall이 존재한다.
  - Agent가 Resource 서버에서 데이터를 가져오기 위해서는 MCP 서버를 통해서 제공하도록 가이드를 하지만, unauthenticated Agent가 PAT token을 Agent에 내장하여 resource server에 직접 통신하거나 자체 MCP 서버를 거쳐서 리소스 서버에 접근한다.


## Gateway 도입 이유
- 인증되지 않은 Agent와 MCP Server가 리소스 서버에 직접 접근하는 것을 방지하기 위해
- 인증된 Agent와 MCP Server의 권한을 관리하기 위해
- 모든 Agent와 MCP Server의 트래픽을 감시하고 감사하기 위해