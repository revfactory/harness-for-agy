# harness-for-agy

> Google **Antigravity CLI(`agy`)** 를 위한 하네스(Harness) 메타 스킬.
> 도메인에 특화된 다중 에이전트 팀, 그들이 사용할 커스텀 스킬, 그리고 이들을 묶는 **오케스트레이터**까지 한 번에 스캐폴딩합니다.

이 프로젝트는 [`revfactory/harness`](https://github.com/revfactory/harness)(Claude Code 용 메타 스킬)를 **Antigravity CLI(`agy`)** 환경에 맞게 재구성한 포팅입니다. Claude Code 의 `~/.claude/skills/` 규약 대신, `agy` 가 인식하는 스킬 디렉토리 규약(`~/.gemini/antigravity-cli/skills/`)을 따르고, Antigravity 런타임의 `define_subagent` / `invoke_subagent` / `send_message` API 를 적극 활용하는 방향으로 확장했습니다.

---

## 무엇을 하는 스킬인가

`/harness` 한 줄로 다음 산출물을 자동 생성·점진적으로 보완합니다.

1. **프로젝트 비전 & 협업 강령** — 루트의 `AGENTS.md` (변경 이력 테이블 누적)
2. **전문 에이전트 페르소나** — `.agents/agents/<AgentName>.md` + Antigravity flat 규격 `agent.json` 실행 프로필
3. **에이전트가 협업할 때 사용할 커스텀 스킬들** — `.agents/skills/<skill-folder>/SKILL.md`
4. **오케스트레이터 스킬** — `.agents/skills/orchestrator/SKILL.md` (메인 에이전트가 서브 에이전트를 기동·조율)
5. **중간 산출물 격리 폴더** — 모든 임시 산출물을 `_workspace/` 하위로 강제 격리

즉, "React 테스트 자동화 팀 만들어줘", "데이터 분석 파이프라인 전용 에이전트 짜줘" 같은 한 문장 요청을, **곧바로 가동 가능한 에이전트 팀 + 스킬 + 오케스트레이션 루프**로 변환합니다.

---

## 설치

### 1) `agy` CLI 설치

```bash
# Google Antigravity CLI 가 이미 설치되어 있어야 합니다.
agy --version
```

설치되어 있지 않다면 [Antigravity 공식 사이트](https://antigravity.google/cli)의 안내를 따르세요.

### 2) 스킬 배치

`agy` 가 인식하는 사용자 스킬 경로(`~/.gemini/antigravity-cli/skills/`)에 `skills/harness/` 디렉토리를 그대로 복사합니다.

```bash
git clone https://github.com/revfactory/harness-for-agy.git
mkdir -p ~/.gemini/antigravity-cli/skills
cp -R harness-for-agy/skills/harness ~/.gemini/antigravity-cli/skills/harness
```

설치 후 `agy` 세션에서 `/harness` 가 슬래시 커맨드로 노출됩니다.

---

## 사용법

```text
$ agy
> /harness React + Vitest 테스트 자동화 팀을 구성해줘
```

스킬이 활성화되면 에이전트는 아래 6단계 워크플로우를 정확히 따릅니다.

| Step | 단계 | 요약 |
| --- | --- | --- |
| 1 | **Domain Analysis** | 요청 도메인·스택·사용자 수련도(초/중/고급) 판별 |
| 2 | **Architecture Design** | 6대 오케스트레이션 패턴 중 최적 패턴 선택 및 `AGENTS.md` 구조화 |
| 3 | **Agent Configuration** | `.agents/agents/<AgentName>.md` + `agent.json` 프로필 생성·갱신 |
| 4 | **Skill Configuration** | `.agents/skills/<skill-folder>/SKILL.md` 표준 규격으로 생성·리팩토링 |
| 5 | **Orchestrator Skill** | `.agents/skills/orchestrator/SKILL.md` 로 협업 루프 구현 |
| 6 | **Review & Test** | `define_subagent` 실호출로 즉시 등록 → `invoke_subagent`/`send_message` 시뮬레이션 → 트리 출력 및 자연어 활용 예시 안내 |

> 💡 같은 디렉토리에서 `/harness` 를 다시 호출하면 기존 `AGENTS.md` / `.agents/` 구조를 **진단(Review)** 하고 격차를 분석(Gap Analysis)하여 점진적으로 보완하는 모드로 동작합니다. 이 경우 별도의 구현 계획서(implementation plan) 없이 곧바로 실행됩니다.

---

## 스킬 구성 (Skill Anatomy)

이 레포는 단 하나의 메타 스킬을 정의하며, 그 스킬이 다른 스킬과 에이전트 팀을 만들어내는 구조입니다.

### 레포 디렉토리

```
harness-for-agy/
├── README.md
├── LICENSE
└── skills/
    └── harness/
        └── SKILL.md      # /harness 메타 스킬 본체
```

### `SKILL.md` Frontmatter 규약

`agy` 의 스킬은 모두 다음 형태의 YAML Frontmatter 로 시작합니다.

```yaml
---
name: <kebab-case-name>
description: <3인칭 시점의 기능 설명 + 트리거 조건>
allowed-tools:
  - <허용할 도구 1>
  - <허용할 도구 2>
---
```

본문에는 다음 3대 헤더가 **반드시** 포함되어야 합니다.

- `## When to use this skill` — 언제 호출되어야 하는가 (트리거 조건)
- `## Instructions` — 동작 시 따라야 할 세부 지침
- `## Workflow` — 순차적으로 실행할 단계

`/harness` 가 생성하는 모든 하위 스킬도 이 규약을 그대로 따릅니다. 즉, 이 메타 스킬은 동시에 **스킬 작성 표준의 참조 구현(reference implementation)** 역할을 합니다.

### 생성되는 산출물 구조

```
<your-project>/
├── AGENTS.md                                # 프로젝트 비전 + 행동 강령 + Change History 테이블
├── _workspace/                              # 모든 에이전트의 중간 산출물 격리 폴더
└── .agents/
    ├── agents/
    │   ├── <AgentName>.md                   # name/role/description Frontmatter + 행동 가이드
    │   └── <agent_name>/
    │       └── agent.json                   # Antigravity flat 규격 런타임 프로필
    └── skills/
        ├── orchestrator/SKILL.md            # 메인 에이전트가 장착하는 협업 지휘 스킬
        ├── <skill-1>/SKILL.md
        ├── <skill-2>/SKILL.md
        └── <skill-3>/SKILL.md
```

### Antigravity `agent.json` 규격

각 에이전트는 마크다운 페르소나 외에, Antigravity CLI 의 **플랫(Flat) JSON** 런타임 프로필을 가집니다.

```json
{
  "name": "에이전트 이름",
  "description": "에이전트 역할 설명",
  "system_prompt": "시스템 지침 및 협업 SOP",
  "enable_write_tools": true,
  "enable_mcp_tools": true,
  "enable_subagent_tools": false
}
```

`Step 6` 에서 메타 스킬은 이 파일을 직접 읽어 **`define_subagent` 를 실제로 호출**해 현재 세션에 에이전트들을 즉시 등록합니다. (등록 누락 방지)

---

## 오케스트레이션 아키텍처 6대 패턴

`/harness` 는 도메인 복잡도와 사용자 수련도에 따라 아래 6가지 패턴 중 최적의 구성을 선택하여 `AGENTS.md` 에 명시합니다.

1. **Pipeline** — 직렬 작업 흐름 (스펙 → 구현 → 검증)
2. **Fan-out / Fan-in** — 병렬 독립 처리 후 취합
3. **Expert Pool** — 태스크 유형에 따라 전문가 에이전트 동적 선택
4. **Producer–Reviewer** — 생성 담당 + 검토/감사 담당의 2단 협업
5. **Supervisor** — 중앙 마스터 에이전트가 하부를 실시간 디스패치
6. **Hierarchical Delegation** — 대규모 태스크를 서브에이전트에게 재귀 위임

---

## 사용자 수련도별 차등 구성

요청 분석을 통해 사용자 수련도를 판별하고 구성 난이도를 차등화합니다.

| 수련도 | 아키텍처 | 스킬 구성 | 오케스트레이션 |
| --- | --- | --- | --- |
| **초급** | 단순 Pipeline 또는 1~2개 Producer–Reviewer | 1~2개 핵심 단위 스킬 | 단선적 호출, 병렬/비동기 생략 |
| **중급** | Expert Pool / Pipeline + Producer–Reviewer 복합형 | 모듈화된 기능별 스킬 | 메시징 프로토콜 기반 세미 오토 협업 |
| **고급** | Hierarchical Delegation / Supervisor 등 하이브리드 | 자율 실행·복구 전략 포함 고급 스킬 | Fan-out/Fan-in + 동적 `define_subagent` + JSON 통신 풀 자율 루프 |

---

## Runtime Collaboration Protocol

생성된 에이전트들은 Antigravity 런타임에서 다음 표준 프로토콜로 협업합니다.

- **라이프사이클** — 세션 시작 시 `.agents/agents/<name>/agent.json` 을 일괄 로드하여 `define_subagent` 로 선등록 → 필요 시 `invoke_subagent` 로 백그라운드 기동 → 반환 `Conversation ID` 로 추적.
- **메시지 페이로드** — `send_message` 호출 시 텍스트 바디에 다음 JSON 구조를 삽입합니다.

```json
{
  "sender": "송신 에이전트명",
  "action": "REQUEST_REVIEW | TASK_COMPLETE | REFACTOR_REQUEST | STATUS_UPDATE",
  "target_artifact": "/abs/path/to/file",
  "content": "요청 사항 상세 설명",
  "metadata": {
    "task_id": "...",
    "status": "...",
    "additional_context": "..."
  }
}
```

수신 에이전트는 JSON 을 먼저 파싱해 작업 대상과 사양을 식별한 뒤 도구 호출에 임합니다.

---

## `revfactory/harness` 와의 차이점

| 항목 | `revfactory/harness` (원본, Claude Code) | `harness-for-agy` (이 레포) |
| --- | --- | --- |
| 대상 CLI | Claude Code | Google Antigravity CLI (`agy`) |
| 설치 경로 | `~/.claude/skills/` | `~/.gemini/antigravity-cli/skills/` |
| 스킬 본문 규약 | `SKILL.md` + Claude Code Frontmatter | `SKILL.md` + `agy` Frontmatter (`name`, `description`, `allowed-tools`) |
| 산출물 위치 | `.claude/agents`, `.claude/skills` | `.agents/agents`, `.agents/skills`, `_workspace/` |
| 에이전트 런타임 프로필 | — | Antigravity flat `agent.json` 규격 자동 생성 |
| 오케스트레이션 | 가이드 수준 | **6대 패턴 + 수련도별 차등 + JSON 메시징 프로토콜** 내재화 |
| 협업 도구 연동 | 일반 도구 | `define_subagent` / `invoke_subagent` / `send_message` 실호출 |
| 슬래시 호출 | `/harness` (Claude Code) | `/harness` (`agy`) |

핵심 사상 — *"메타 스킬이 도메인별 에이전트 팀과 보조 스킬을 한꺼번에 만들어낸다"* — 은 동일하게 유지하되, Antigravity CLI 의 런타임 협업 API 를 일급(first-class) 으로 끌어올렸습니다.

---

## 기여

이슈 및 PR 환영합니다. 새로운 도메인 템플릿(예: 모바일/게임/데이터 엔지니어링)을 제안하실 때는 `/harness` 가 생성한 결과 트리와 `AGENTS.md` 의 Change History 항목을 함께 첨부해 주세요.

---

## License

Apache License 2.0. 자세한 내용은 [`LICENSE`](./LICENSE) 파일을 참고하세요.

원본 메타 스킬의 아이디어와 구조에 대한 크레딧은 [`revfactory/harness`](https://github.com/revfactory/harness) 에 있습니다.
