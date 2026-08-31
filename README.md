# dev-team — Claude Code AI 에이전트 팀 플러그인

기획부터 배포까지 개발 전 과정을 커버하는 전문 AI 에이전트 팀입니다. 오케스트레이터 스킬 1개 + 전문 서브에이전트 9개 + 슬래시 커맨드 3개로 구성되어 있으며, 다음 오픈소스 프로젝트들의 검증된 패턴을 차용했습니다.

- **[Graft](https://github.com/NanoNets/Graft) / [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)** → 세션이 끝나도 사라지지 않는 파일 기반 프로젝트 메모리 (`.claude/memory/`)
- **[agency-agents](https://github.com/msitarzewski/agency-agents)** → 정체성·임무·규칙·산출물·성공지표를 명시한 서브에이전트 정의 템플릿
- **[OpenMontage](https://github.com/calesthio/OpenMontage)** → 단계별 파이프라인 + 승인 게이트 + 재개 가능한 체크포인트 + 셀프 리뷰
- **[Orca](https://github.com/stablyai/orca)** → git worktree 격리 병렬 실행 → 결과 비교 후 승자 병합

## 설치

### 방법 1: 플러그인 마켓플레이스 (권장)

Claude Code에서:

```
/plugin marketplace add nsh-cmd/Skill
/plugin install dev-team@nsh-skill
```

### 방법 2: 수동 복사 (전역 설치)

```bash
git clone https://github.com/nsh-cmd/Skill.git
cp -r Skill/agents/*.md ~/.claude/agents/
cp -r Skill/skills/dev-team ~/.claude/skills/
cp -r Skill/commands ~/.claude/commands/   # 선택
```

설치 후 Claude Code를 재시작하고 `/agents`에서 9개 에이전트가 보이는지 확인하세요.

### 기존 vibecoding 스킬 사용자라면

dev-team은 vibecoding의 후속 격입니다. 같은 프롬프트에 두 스킬이 동시에 반응할 수 있으니, 기존 전역 vibecoding 스킬은 비활성화하거나 제거하는 것을 권장합니다.

## 에이전트 로스터

| 에이전트 | 역할 | 읽기 전용 |
|---------|------|:---:|
| `planner` | 모호한 요청 → 수용 기준이 명확한 작업 계획 | ✅ |
| `architect` | 코드 작성 전 구조·인터페이스 계약·트레이드오프 설계 | ✅ |
| `implementer` | 승인된 설계대로 코드베이스 컨벤션에 맞춰 구현, 빌드/린트/테스트 검증 후 반환 | |
| `test-engineer` | 수용 기준을 증명하는 테스트 작성·**실제 실행**, 커버리지 갭 보고 | |
| `code-reviewer` | diff를 severity 랭킹(BLOCKER~NIT)으로 리뷰, file:line 명시 | ✅ |
| `security-auditor` | 인젝션·인가·시크릿·의존성 등 취약점 감사, 공격 시나리오 포함 | ✅ |
| `debugger` | 재현 → 가설 → 검증 순서의 근본원인 분석, 최소 수정 | |
| `devops-engineer` | CI/CD·빌드·버저닝·릴리스 준비 (배포는 반드시 사용자 승인) | |
| `docs-writer` | 코드 실제 동작과 일치하는 문서 — 모든 예제 검증 후 작성 | ✅* |

*docs-writer는 문서 파일만 수정 가능

## 핵심 개념

### 파이프라인과 승인 게이트

비자명한 작업은 6단계 파이프라인으로 진행됩니다:

```
plan → design → implement → test → review → ship
```

- **plan / design / ship 단계는 항상 사용자 승인이 필요**합니다. implement/test/review는 fast 모드 선택 시 결과가 깨끗하면 자동 진행됩니다.
- 각 단계 산출물은 `.claude/pipeline/<작업-슬러그>/`에 저장되고 `state.json`으로 상태를 추적하므로, **세션이 끊겨도 이어서 재개**할 수 있습니다.
- 거부된 산출물은 같은 에이전트가 피드백을 반영해 재작업하며, 3회 초과 시 사용자에게 에스컬레이션됩니다.

### 프로젝트 메모리

`.claude/memory/`에 시스템별 노트(`system-*.md`), 개념 노트(`concept-*.md`), 결정 기록(`decision-*.md`)을 축적합니다. 모든 에이전트는 탐색 전에 `INDEX.md`를 먼저 읽어 **매 세션 반복되는 코드베이스 재탐색 비용을 제거**합니다. 작업이 끝나면 발견한 사실이 메모리에 다시 기록됩니다.

### 병렬 팬아웃

접근 방식이 애매하거나 리스크가 큰 구현은 git worktree로 격리된 N개(권장 2~3)의 병렬 시도로 실행하고, 테스트 통과 → diff 크기 → 컨벤션 적합도 → 복잡도 순으로 비교해 승자를 병합합니다. 토큰 비용이 약 N배이므로 필요할 때만 사용합니다.

## 사용 예시

**전체 파이프라인:**

```
/dev-team:pipeline 사용자 프로필에 아바타 업로드 기능 추가
```

plan부터 ship까지 단계별로 진행되며, 핵심 단계마다 승인을 요청합니다.

**단일 에이전트 위임 (자연어로도 트리거됩니다):**

```
이 diff 코드 리뷰해줘          → code-reviewer
이 테스트 왜 실패하는지 원인 찾아줘  → debugger
보안 점검해줘                → security-auditor
```

**병렬 팬아웃:**

```
/dev-team:fanout 2 검색 자동완성을 디바운스 방식과 캐시 방식으로 각각 구현해서 비교
```

**프로젝트 메모리:**

```
/dev-team:memory init    # 프로젝트 스캔 후 초기 노트 생성
/dev-team:memory update  # 오래된 노트를 코드와 대조해 갱신
/dev-team:memory show    # 인덱스와 노트 신선도 확인
```

## 트러블슈팅

- **에이전트가 안 보여요** — `/agents` 목록 확인. 플러그인 설치라면 `/plugin` 에서 dev-team이 enabled인지, 수동 설치라면 `~/.claude/agents/`에 파일이 있는지 확인 후 Claude Code 재시작.
- **커맨드가 안 보여요** — `/help`에서 `dev-team` 네임스페이스 확인. 플러그인 커맨드는 `/dev-team:pipeline` 형식입니다.
- **한국어 프롬프트에 반응이 약해요** — 각 에이전트 description에 한국어 트리거가 포함되어 있지만, 확실하게 하려면 에이전트 이름을 직접 언급하세요 ("planner로 계획 세워줘").
- **파이프라인이 중간에 끊겼어요** — 그냥 이어서 요청하세요. `.claude/pipeline/*/state.json`을 감지해 재개를 제안합니다.

## 라이선스

MIT
