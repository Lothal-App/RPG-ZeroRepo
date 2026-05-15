<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI 코딩 에이전트가 전체 리포지토리를 이해하도록 하기

AI 코딩 에이전트는 강력하지만, 대개 파일 단위로 작업합니다. 프로젝트가 커질수록 요구사항, 아키텍처, 의존성, 이전 설계 결정을 놓칠 수 있습니다.

RPG-Kit은 **Repository Planning Graph (RPG)** 를 유지하여 이 문제를 해결하도록 돕습니다. RPG는 요구사항, 기능, 파일, 컴포넌트, 의존성을 연결하는 구조화된 지도입니다.

고립된 프롬프트 대신 리포지토리 수준의 컨텍스트로 AI 에이전트가 작업하기를 원할 때 RPG-Kit을 사용하세요.

### 왜 RPG-Kit인가요?

| AI 코딩 에이전트의 일반적인 문제 | RPG-Kit의 도움 방식 |
|---|---|
| 에이전트가 몇 번의 프롬프트 후 요구사항을 잊어버림 | 요구사항이 RPG에 인코딩됩니다 |
| 관련 파일을 이해하지 못한 채 한 파일만 편집함 | 파일, 컴포넌트, 의존성이 그래프에서 연결됩니다 |
| 생성된 코드가 원래 계획에서 벗어남 | 계획 산출물과 코드가 정렬된 상태로 유지됩니다 |
| 기존 리포지토리를 에이전트가 이해하기 어려움 | 코드베이스를 RPG로 인코딩할 수 있습니다 |
| 대상이 명확한 편집이 숨겨진 의존성을 깨뜨릴 수 있음 | 그래프 인식 컨텍스트로 편집됩니다 |

### 워크플로 선택

| 목표 | 워크플로 | 시작 위치 |
|---|---|---|
| 요구사항에서 새 프로젝트 생성 | 정방향 워크플로 | [`빠른 시작: 새 리포지토리`](#quick-start-new-repository) |
| 기존 코드베이스 이해 또는 업데이트 | 역방향 워크플로 | [`빠른 시작: 기존 리포지토리`](#quick-start-existing-repository) |
| 정밀한 리포지토리 인식 편집 수행 | 외과적 편집 워크플로 | [`빠른 시작: 기존 리포지토리`](#quick-start-existing-repository) |

## 설치

### 필수 조건

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- 설치 및 인증이 완료된 AI 코딩 에이전트 CLI: [GitHub Copilot](https://docs.github.com/en/copilot) 또는 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### RPG-Kit 설치

```bash
# 영구 설치(권장)
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# 일회성 사용
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

<a id="quick-start-new-repository"></a>
## 빠른 시작: 새 리포지토리

RPG-Kit이 요구사항을 새 코드베이스로 변환하도록 하려면 이 경로를 사용하세요.

> [!WARNING]
> 생성되는 코드 양이 많은 프로젝트의 경우, `/rpgkit.design_interfaces`와 `/rpgkit.code_gen`의 실행 시간이 길어질 수 있습니다. 대표적인 예로, 기능 수가 100개인 경우 실행 시간은 약 30분입니다.

1. 새 프로젝트를 초기화합니다:

   ```bash
   rpgkit init my-project
   cd my-project
   ```

   일반적인 변형:

   ```bash
   rpgkit init my-project --ai claude --script sh
   rpgkit init my-project --ai copilot
   rpgkit init my-project --github-token $GITHUB_TOKEN
   ```

2. **[선택 사항]** 요구사항 문서를 `my-project/docs/`에 넣습니다.

3. 프로젝트 디렉터리에서 AI 코딩 에이전트를 실행합니다.

4. 정방향 파이프라인을 실행합니다:

   ```text
   /rpgkit.feature_spec <feature description>
   /rpgkit.feature_build
   /rpgkit.feature_refactor
   [Optional] /rpgkit.feature_edit <edit instructions>
   /rpgkit.build_skeleton
   /rpgkit.build_data_flow
   /rpgkit.design_base_classes
   /rpgkit.design_interfaces
   /rpgkit.plan_tasks
   /rpgkit.code_gen
   [Optional] /rpgkit.rpg_edit <edit instructions>
   ```

RPG-Kit은 `.rpgkit/data/rpg.json`을 점진적으로 생성하고, 이를 사용해 요구사항, 계획 산출물, 생성된 코드, 의존성 정보를 정렬된 상태로 유지합니다.

<a id="quick-start-existing-repository"></a>
## 빠른 시작: 기존 리포지토리

이미 리포지토리가 있고 AI 에이전트가 RPG 컨텍스트로 이를 이해하거나 편집하게 하려면 이 경로를 사용하세요.

> [!WARNING]
> 규모가 큰 프로젝트의 경우, `rpgkit init . --encode`와 `/rpgkit.encode`의 실행 시간이 길어질 수 있습니다. 대표적인 예로, 소스 코드 파일 수가 200개인 경우 실행 시간은 약 100분입니다.

1. 리포지토리 루트에서 RPG-Kit을 초기화하고 초기 그래프를 구축합니다:

   ```bash
   mkdir my-project
   cp -r existing-repo/ my-project/
   cd my-project
   rpgkit init . --encode
   ```

   비어 있지 않은 디렉터리에 대한 확인 프롬프트를 건너뛰려면:

   ```bash
   rpgkit init . --force --encode
   ```

2. 리포지토리에서 AI 코딩 에이전트를 실행합니다.

3. MCP 도구와 slash command를 통해 생성된 RPG를 사용합니다:

   ```text
   /rpgkit.encode                                  # 필요할 때 전체 RPG 재구축
   /rpgkit.update_rpg                              # 수동 증분 업데이트 폴백
   /rpgkit.rpg_edit <edit instructions>            # 그래프 인식 코드 편집
   ```

4. 커밋 후 RPG-Kit hooks는 `.rpgkit/data/rpg.json`, `.rpgkit/data/dep_graph.json`, `.rpgkit/data/rpg.html`을 코드 변경과 정렬된 상태로 유지합니다. hook이 실패하거나 건너뛰어진 경우 `/rpgkit.update_rpg`를 실행하세요.

## 추가되는 항목

`rpgkit init`을 실행한 후에도 workspace root는 프로젝트 리포지토리 루트입니다. RPG-Kit은 명령 정의, 런타임 스크립트, MCP 구성, 생성된 그래프 데이터를 코드와 함께 추가합니다.

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

전체 레이아웃 및 데이터 파일 참조는 [docs/project-structure.md](docs/project-structure.md)를 참조하세요.

## 지원 플랫폼

| 플랫폼                 | Claude Code | GitHub Copilot | Codex |
| ---------------------- | ----------- | -------------- | ----- |
| CLI 사용               | ✅          | ✅(MCP 없음)   | ⌛    |
| VS Code 확장 사용      | ✅          | ✅             | ⌛    |

| 스크립트 | Linux | Windows | Mac |
| -------- | ----- | ------- | --- |
| sh       | ✅    | ⌛      | ⌛  |
| ps       | N/A   | ⌛      | ⌛  |

## 문서

- [Slash command 참조](docs/commands.md) — 모든 `/rpgkit.*` 명령, 입력, 출력, 예시.
- [CLI 참조](docs/cli-reference.md) — `rpgkit init`, `rpgkit update`, `rpgkit check`, `rpgkit version` 및 모든 옵션.
- [구성](docs/configuration.md) — AI assistant 설정, MCP 등록, hooks, 자동 승인, 문제 해결.
- [프로젝트 구조](docs/project-structure.md) — RPG-Kit이 생성하는 파일과 디렉터리.

## 예정 기능

- **더 단순한 디코더 명령:** 현재 디코더 흐름을 더 적은 명령으로 병합합니다. 여기에는 엔드투엔드 리포지토리 생성을 위한 `/rpgkit.generate_repo`, 기능 생성과 RPG 계획을 위한 `/rpgkit.generate_feature` 및 `/rpgkit.plan`이 포함됩니다.
- **다중 언어 지원:** Go, C++, Rust, JavaScript/TypeScript 등에 대한 지원을 추가합니다.
- **더 많은 플랫폼 통합:** 다양한 시스템에서 여러 AI 코딩 에이전트를 위한 CLI 및 VS Code 확장 워크플로 전반에 RPG-Kit을 지원합니다.

## 문제 해결

**AI assistant CLI를 찾을 수 없음:** `rpgkit check`를 실행하고, 선택한 assistant CLI를 설치 및 인증한 다음 `rpgkit init` 또는 `rpgkit update`를 다시 실행하세요.

**MCP 도구가 `rpg_unavailable`를 보고함:** `/rpgkit.encode`를 실행하여 `.rpgkit/data/rpg.json`을 생성하세요.

**증분 업데이트 실패:** `.rpgkit/logs/update_rpg.log`를 확인한 다음 `/rpgkit.update_rpg`를 실행하세요.

**rate limit 또는 프라이빗 리포지토리 접근으로 인해 템플릿 다운로드 실패:** `--github-token $GITHUB_TOKEN`을 전달하거나 `GH_TOKEN` / `GITHUB_TOKEN`을 설정하세요.

## 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.

## 감사의 말

[GitHub Spec-Kit](https://github.com/github/spec-kit)을 기반으로 합니다.
