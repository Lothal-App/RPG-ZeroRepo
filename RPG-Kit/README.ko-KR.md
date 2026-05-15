<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI 기반 양방향 Repository-RPG 툴킷

RPG-Kit은 LLM 기반 에이전트를 활용하여 **Repository Planning Graphs (RPG)** — 기능, 파일, 컴포넌트, 의존성을 연결하는 통합 그래프 — 를 양방향으로 다룹니다:

- **정방향(요구사항 → RPG → 코드):** AI 코딩 에이전트가 구동하는 다단계 파이프라인을 통해 고수준 요구사항을 테스트된 구조화 리포지토리로 변환합니다
- **역방향(코드 → RPG):** 기존 코드베이스를 RPG 그래프로 인코딩하여 AI 지원 검색, 탐색, 증분 업데이트에 사용합니다
- **외과적 편집(지시 → RPG + 코드):** 코드, RPG, 의존성 그래프의 동기화를 유지하면서 대상이 명확한 변경을 적용합니다

### 예정 기능

- **더 단순한 디코더 명령:** 현재 디코더 흐름을 더 적은 명령으로 병합합니다. 여기에는 엔드투엔드 리포지토리 생성을 위한 `/rpgkit.generate_repo`, 기능 생성과 RPG 계획을 위한 `/rpgkit.generate_feature` 및 `/rpgkit.plan`이 포함됩니다.
- **다중 언어 지원:** Go, C++, Rust, JavaScript/TypeScript 등에 대한 지원을 추가합니다.
- **더 많은 플랫폼 통합:** 다양한 시스템에서 여러 AI 코딩 에이전트를 위한 CLI 및 VS Code 확장 워크플로 전반에 RPG-Kit을 지원합니다.

| 플랫폼                 | Claude Code | GitHub Copilot | Codex |
| ---------------------- | ----------- | -------------- | ----- |
| CLI 사용               | ✅          | ✅(MCP 없음)   | ⌛    |
| VS Code 확장 사용      | ✅          | ✅             | ⌛    |

| 스크립트 | Linux | Windows | Mac |
| -------- | ----- | ------- | --- |
| sh       | ✅    | ⌛      | ⌛  |
| ps       | N/A   | ⌛      | ⌛  |

## 필수 조건

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- 설치 및 인증이 완료된 AI 코딩 에이전트 CLI: [GitHub Copilot](https://docs.github.com/en/copilot) 또는 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

설치 후 `rpgkit check`를 사용하여 로컬 도구 사용 가능 여부를 확인하세요.

## 설치

### 옵션 1: 영구 설치(권장)

```bash
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
```

### 옵션 2: 일회성 사용

```bash
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

## 빠른 시작

### 1. 프로젝트 초기화

```bash
# 새 프로젝트 생성
rpgkit init my-project

# 또는 현재 디렉터리에서 초기화
rpgkit init .

# 프라이빗 리포지토리에는 token 필요
rpgkit init my-project --github-token $GITHUB_TOKEN

# 기존 코드베이스는 초기 RPG를 즉시 구축 가능
rpgkit init . --encode

# 프로젝트 디렉터리로 이동
cd my-project
```

### 2. `/rpgkit` 명령 사용

AI 코딩 에이전트를 실행하고 `/rpgkit.*` 명령을 순서대로 실행하세요:

```text
# 1단계: 기능 명세
/rpgkit.feature_spec <feature description>
/rpgkit.feature_build
/rpgkit.feature_refactor
/rpgkit.feature_edit <edit instructions>       # 선택 사항, 스켈레톤 계획 전

# 2단계: RPG 구성 및 계획
/rpgkit.build_skeleton
/rpgkit.build_data_flow
/rpgkit.design_base_classes
/rpgkit.design_interfaces
/rpgkit.plan_tasks

# 3단계: 코드 생성
/rpgkit.code_gen
/rpgkit.rpg_edit <edit instructions>           # 선택 사항, RPG/코드가 존재한 후

# 역방향: 기존 리포지토리 → RPG 인코딩
/rpgkit.encode                                  # 전체 인코딩
/rpgkit.update_rpg                              # 수동 증분 업데이트 폴백
```

## `/rpgkit` 명령

RPG-Kit은 13개의 slash command를 제공합니다. 정방향 파이프라인은 요구사항에서 코드를 생성합니다. 인코더는 기존 코드에서 RPG를 구축합니다. `rpg_edit`는 대상이 명확한 변경을 적용하면서 RPG와 코드의 동기화를 유지합니다.

> 각 명령의 자세한 사용법은 [docs/commands.md](docs/commands.md)를 참조하세요.

### 1단계: 기능 명세

| 명령 | 설명 |
| ---- | ---- |
| `/rpgkit.feature_spec <desc>` | 사용자 입력 또는 `docs/` 파일에서 구조화된 기능 명세를 생성합니다 |
| `/rpgkit.feature_build` | 명세에서 기능 트리를 생성하고 확장합니다 |
| `/rpgkit.feature_refactor` | 기능 트리를 모듈식 컴포넌트 아키텍처로 리팩터링합니다 |
| `/rpgkit.feature_edit <instr>` | 스켈레톤 계획 전에 기능 트리 노드를 편집합니다 — 선택 사항 |

### 2단계: RPG 구성 및 계획

| 명령 | 설명 |
| ---- | ---- |
| `/rpgkit.build_skeleton` | 컴포넌트 아키텍처에서 리포지토리 파일 스켈레톤을 구축합니다; `.rpgkit/data/rpg.json`을 생성합니다 |
| `/rpgkit.build_data_flow` | 컴포넌트 간 데이터 흐름 DAG를 구축하고 RPG를 업데이트합니다 |
| `/rpgkit.design_base_classes` | 공유 베이스 클래스와 데이터 구조를 설계합니다 |
| `/rpgkit.design_interfaces` | 타입 힌트와 docstring이 포함된 함수/클래스 인터페이스를 설계합니다 |
| `/rpgkit.plan_tasks` | 의존성 순서에 따른 구현 작업 배치를 계획합니다 |

### 3단계: 코드 생성 및 외과적 편집

| 명령 | 설명 |
| ---- | ---- |
| `/rpgkit.code_gen` | 반복적인 test-code-fix 사이클을 포함한 TDD 기반 구현 |
| `/rpgkit.rpg_edit <instr>` | 자연어 지시에서 RPG 그래프, 코드, 의존성 그래프를 외과적으로 편집합니다 — 선택 사항 |

### RPG 인코더(역방향: 코드 → RPG)

| 명령 | 설명 |
| ---- | ---- |
| `/rpgkit.encode` | 기존 리포지토리를 `.rpgkit/data/rpg.json`으로 인코딩합니다 |
| `/rpgkit.update_rpg` | 자동 hook이 건너뛰어지거나 실패했을 때 증분 RPG 업데이트를 수동으로 실행합니다 |

두 방향 모두 `.rpgkit/data/rpg.json`에 동일한 RPG 구조를 생성하여 AI 에이전트가 **MCP server**(`search_rpg`, `explore_rpg`, `get_node_detail`, `list_rpg_tree`)를 통해 그래프를 질의할 수 있게 합니다. 아래의 [MCP 통합](#mcp-통합)을 참조하세요.

## CLI 참조

### `rpgkit init`

최신 템플릿에서 새 프로젝트를 초기화합니다.

```bash
rpgkit init <project-name> [options]
rpgkit init --here [options]
rpgkit init . [options]
```

| 옵션 | 설명 |
| ---- | ---- |
| `--ai <agent>` | AI assistant: `copilot` 또는 `claude` |
| `--script <type>` | 스크립트 유형: `sh`(POSIX) 또는 `ps`(PowerShell) |
| `--here` | 현재 디렉터리에서 초기화합니다 |
| `--force` | 비어 있지 않은 현재 디렉터리에 대한 확인을 건너뜁니다 |
| `--no-git` | git 초기화를 건너뜁니다 |
| `--no-mcp` | MCP server 구성을 건너뜁니다 |
| `--ignore-agent-tools` | AI 에이전트 CLI 도구 검사를 건너뜁니다 |
| `--github-token <token>` | 프라이빗 리포지토리 또는 더 높은 rate limit을 위한 GitHub token |
| `--pre` | 최신 프리릴리스 템플릿을 다운로드합니다 |
| `--skip-tls` | SSL/TLS 검증을 건너뜁니다 |
| `--encode/--no-encode` | init 마지막에 초기 RPG 인코딩을 실행하거나 건너뜁니다 |
| `--debug` | 자세한 진단 출력을 표시합니다 |

**지원되는 AI Assistants:**

| 에이전트 | 폴더 | 설명 | 상태 |
| -------- | ---- | ---- | ---- |
| `copilot` | `.github/`, `.vscode/` | GitHub Copilot | 검증됨 |
| `claude` | `.claude/` | Claude Code | 검증됨 |

> RPG-Kit은 현재 CLI에서 **GitHub Copilot**과 **Claude Code**만 지원합니다. 추가 에이전트는 향후 릴리스에서 적용될 수 있습니다.

**예시:**

```bash
rpgkit init my-project
rpgkit init my-project --ai claude --script sh
rpgkit init . --force
rpgkit init --here --ai copilot
rpgkit init --here --github-token $GITHUB_TOKEN
rpgkit init --here --encode
```

### `rpgkit update`

기존 프로젝트의 RPG-Kit 템플릿 파일, 스크립트, 명령 정의, MCP 구성, gitignore 규칙 및 hooks를 업데이트합니다. AI assistant는 가능한 경우 기존 프로젝트 구성에서 자동 감지됩니다.

```bash
rpgkit update
rpgkit update --ai claude
rpgkit update --pre
rpgkit update --no-mcp
rpgkit update --github-token $GITHUB_TOKEN
```

| 옵션 | 설명 |
| ---- | ---- |
| `--ai <agent>` | AI assistant, 지정하지 않으면 자동 감지됩니다 |
| `--script <type>` | 스크립트 유형: `sh`(POSIX) 또는 `ps`(PowerShell) |
| `--github-token <token>` | 프라이빗 리포지토리 또는 더 높은 rate limit을 위한 GitHub token |
| `--pre` | 최신 프리릴리스 템플릿을 다운로드합니다 |
| `--no-mcp` | MCP server 구성을 건너뜁니다 |
| `--skip-tls` | SSL/TLS 검증을 건너뜁니다 |
| `--debug` | 자세한 진단 출력을 표시합니다 |

### `rpgkit check`

필요한 도구가 설치되어 있는지 확인합니다.

```bash
rpgkit check
```

### `rpgkit version`

버전 및 시스템 정보를 표시합니다.

```bash
rpgkit version
```

## 워크플로

```text
정방향: 요구사항 → RPG → 코드

 1단계: 기능 명세                 2단계: RPG 구성 및 계획                                      3단계
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ feature  │ │ feature  │ │ feature  │ │  build   │ │  build   │ │ design   │ │ design   │ │  plan    │ │          │
│  _spec   ├─▶  _build  ├─▶_refactor ├─▶ skeleton ├─▶  data    ├─▶  base    ├─▶interfaces├─▶  tasks  ├─▶ code_gen │
│          │ │          │ │          │ │          │ │  flow    │ │ classes  │ │          │ │          │ │   (TDD)  │
└──────────┘ └──────────┘ └────┬─────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────┬─────┘
 feature_     feature_        │        skeleton     data_flow    base_        interfaces   tasks        source
 spec/        build           │        .json        .json        classes      .json        .json        code
 feature_     .json           │        skeleton_    data_flow    .json
 spec.json                    │        summary.txt  _viz.html
                              │
                       ┌──────▼──────┐
                       │ feature_edit│ feature_tree.json에 대한 선택적 사전 계획 편집
                       └─────────────┘
                                        ╰───── rpg.json(생성 → 점진적으로 보강) ─────╯
                                                                            │
                                                                            ▼
                                                                     ┌──────────┐
                                                                     │ rpg_edit │ 선택적 동기화 RPG + 코드 + dep_graph 편집
                                                                     └──────────┘

역방향: 코드 → RPG

┌──────────────────┐         ┌──────────┐       ┌──────────┐
│ Existing Codebase│────────▶│  encode  │──────▶│update_rpg│
│                  │         │  (full)  │       │ (manual  │
└──────────────────┘         └──────────┘       │ fallback)│
                              rpg.json          └──────────┘
                              dep_graph.json     rpg.json / dep_graph.json
                                                  ▲
                                                  │ post-commit hook은 일반적으로 증분 업데이트를 실행합니다

MCP Server: search_rpg / explore_rpg / get_node_detail / list_rpg_tree
```

## 프로젝트 구조

`rpgkit init`을 실행하면 workspace root가 프로젝트 리포지토리 루트가 됩니다. RPG-Kit은 에이전트 명령 정의, 런타임 스크립트, MCP 구성, 생성된 데이터를 코드와 함께 추가합니다.

```text
my-project/
├── docs/                 # /rpgkit.feature_spec용 선택적 요구사항 문서
├── .github/ or .claude/  # AI assistant 명령 정의 및 설정
├── .vscode/              # 해당되는 경우 Copilot/VS Code MCP 구성
└── .rpgkit/              # RPG-Kit 런타임
    ├── scripts/          # 파이프라인 스크립트 및 지원 패키지
    ├── data/             # rpg.json 및 dep_graph.json을 포함한 생성 아티팩트
    ├── logs/             # 단계별 실행 로그
    └── reports/          # 생성된 리뷰 및 진단 보고서
```

전체 디렉터리 레이아웃 및 데이터 파일 참조는 [docs/project-structure.md](docs/project-structure.md)를 참조하세요.

## MCP 통합

`rpgkit init`은 `--no-mcp`가 전달되지 않는 한 RPG-Kit의 MCP server(`rpg-tools`)를 AI assistant에 자동으로 등록합니다. 이 server는 `.rpgkit/data/rpg.json`을 읽고 네 개의 읽기 전용 도구를 노출합니다:

| 도구 | 목적 |
| ---- | ---- |
| `search_rpg` | 키워드, 이름, 경로, 함수, 클래스 또는 기능으로 노드를 검색합니다 |
| `explore_rpg` | 시작 노드에서 의존성 및 호출 체인 엣지를 순회합니다 |
| `get_node_detail` | 노드의 전체 레코드와 선택적으로 소스 코드를 가져옵니다 |
| `list_rpg_tree` | 리포지토리의 기능 아키텍처를 트리로 렌더링합니다 |

MCP 구성은 `rpgkit init`을 실행한 프로젝트로 제한됩니다. 사용자 수준 assistant 설정은 수정되지 않습니다. 그래프가 아직 생성되지 않은 경우 MCP 도구는 에이전트에게 `/rpgkit.encode`를 실행하라고 알려주는 `rpg_unavailable` 힌트를 반환합니다.

MCP 등록, 자동 승인, hooks 및 초기화 옵션은 [docs/configuration.md](docs/configuration.md)를 참조하세요.

## 문제 해결

**AI assistant CLI를 찾을 수 없음:**

```bash
rpgkit check
```

선택한 assistant CLI를 설치하고 인증한 다음 `rpgkit init` 또는 `rpgkit update`를 다시 실행하세요.

**MCP 도구가 `rpg_unavailable`를 보고함:**

```text
/rpgkit.encode
```

MCP server는 구성되어 있지만 `.rpgkit/data/rpg.json`이 아직 생성되지 않았습니다.

**증분 업데이트 실패:**

```bash
tail -n 200 .rpgkit/logs/update_rpg.log
```

그런 다음 실행하세요:

```text
/rpgkit.update_rpg
```

**rate limit 또는 프라이빗 리포지토리 접근으로 인해 템플릿 다운로드 실패:**

```bash
rpgkit init my-project --github-token $GITHUB_TOKEN
# 또는 환경 변수 설정:
export GH_TOKEN=your_token
```

## 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.

## 감사의 말

[GitHub Spec-Kit](https://github.com/github/spec-kit)을 기반으로 합니다.
