<h1 align="center">RPG-Kit</h1>

<p align="center">
  <a href="README.md">English</a> |
  <a href="README.zh-CN.md">简体中文</a> |
  <a href="README.ja-JP.md">日本語</a> |
  <a href="README.ko-KR.md">한국어</a> |
  <a href="README.hi-IN.md">हिन्दी</a>
</p>

## AI coding agents को पूरे repository को समझने दें

AI coding agents शक्तिशाली होते हैं, लेकिन वे अक्सर file-by-file काम करते हैं। जैसे-जैसे project बढ़ता है, वे requirements, architecture, dependencies, और पिछले design decisions का track खो सकते हैं।

RPG-Kit इस समस्या को **Repository Planning Graph (RPG)** maintain करके हल करने में मदद करता है: एक structured map जो requirements, features, files, components, और dependencies को जोड़ता है।

जब आप चाहते हैं कि AI agents isolated prompts के बजाय repository-level context के साथ काम करें, तब RPG-Kit का उपयोग करें।

### RPG-Kit क्यों?

| AI coding agents की common problem | RPG-Kit कैसे मदद करता है |
|---|---|
| Agent कुछ prompts के बाद requirements भूल जाता है | Requirements RPG में encode की जाती हैं |
| Agent related files को समझे बिना एक file edit करता है | Files, components, और dependencies graph में connected होते हैं |
| Generated code original plan से drift हो जाता है | Planning artifacts और code aligned रखे जाते हैं |
| Existing repositories को agents के लिए समझना कठिन होता है | Codebase को RPG में encode किया जा सकता है |
| Targeted edits hidden dependencies तोड़ सकते हैं | Edits graph-aware context के साथ किए जाते हैं |

### अपना workflow चुनें

| Goal | Workflow | Start here |
|---|---|---|
| Requirements से नया project create करें | Forward workflow | [`Quick Start: नया Repository`](#quick-start-new-repository) |
| Existing codebase को समझें या update करें | Reverse workflow | [`Quick Start: मौजूदा Repository`](#quick-start-existing-repository) |
| Precise repository-aware edit करें | Surgical edit workflow | [`Quick Start: मौजूदा Repository`](#quick-start-existing-repository) |

नीचे इस repository के लिए generated graph visualization का एक हिस्सा है। `/rpgkit.encode` चलाएँ और full interactive graph explore करने के लिए `rpg.html` खोलें।

![RPG-Kit repository graph visualization](../docs/rpgkit_visualized_graph.png)

## Installation

### पूर्वापेक्षाएँ

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- Git
- installed और authenticated AI coding agent CLI: [GitHub Copilot](https://docs.github.com/en/copilot) या [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup)

### RPG-Kit install करें

```bash
# Persistent installation (Recommended)
uv tool install rpgkit-cli --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit"
rpgkit check

# One-time usage
uvx --from "git+https://github.com/microsoft/RPG-ZeroRepo.git#subdirectory=RPG-Kit" rpgkit init <project-name>
```

<a id="quick-start-new-repository"></a>

## Quick Start: नया Repository

जब आप चाहते हैं कि RPG-Kit requirements को एक नए codebase में बदले, तो इस path का उपयोग करें।

> [!WARNING]
> जिन projects में generated code की मात्रा बड़ी हो, उनमें `/rpgkit.design_interfaces` और `/rpgkit.code_gen` का runtime लंबा हो सकता है। एक typical example: feature count 100 होने पर runtime लगभग 30 minutes होता है।

1. नया project initialize करें:

   ```bash
   rpgkit init my-project
   cd my-project
   ```

   सामान्य variants:

   ```bash
   rpgkit init my-project --ai claude --script sh
   rpgkit init my-project --ai copilot
   rpgkit init my-project --github-token $GITHUB_TOKEN
   ```

2. **[Optional]** अपनी requirement documents को `my-project/docs/` में रखें।

3. project directory में अपना AI coding agent launch करें।

4. forward pipeline run करें:

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

RPG-Kit क्रमिक रूप से `.rpgkit/data/rpg.json` बनाता है और इसका उपयोग requirements, planning artifacts, generated code, और dependency information को aligned रखने के लिए करता है।

<a id="quick-start-existing-repository"></a>

## Quick Start: मौजूदा Repository

जब आपके पास पहले से repository हो और आप चाहते हों कि AI agent उसे RPG context के साथ समझे या edit करे, तो इस path का उपयोग करें।

> [!WARNING]
> बड़े projects के लिए, `rpgkit init . --encode` और `/rpgkit.encode` का runtime लंबा हो सकता है। एक typical example: source code files 200 होने पर runtime 100 minutes होता है।

1. repository root में RPG-Kit initialize करें और initial graph build करें:

   ```bash
   mkdir my-project
   cp -r existing-repo/ my-project/
   cd my-project
   rpgkit init . --encode
   ```

   अगर आप non-empty directory के confirmation prompt को skip करना चाहते हैं:

   ```bash
   rpgkit init . --force --encode
   ```

2. repository में अपना AI coding agent launch करें।

3. generated RPG को MCP tools और slash commands के माध्यम से उपयोग करें:

   ```text
   /rpgkit.encode                                  # जरूरत पड़ने पर full RPG rebuild करें
   /rpgkit.update_rpg                              # manual incremental update fallback
   /rpgkit.rpg_edit <edit instructions>            # graph-aware code edit
   ```

4. commits के बाद, RPG-Kit hooks `.rpgkit/data/rpg.json`, `.rpgkit/data/dep_graph.json`, और `.rpgkit/data/rpg.html` को code changes के साथ aligned रखते हैं। अगर hook fail या skip हो जाए, तो `/rpgkit.update_rpg` run करें।

## क्या जोड़ा जाता है

`rpgkit init` run करने के बाद भी workspace root आपके project repository का root रहता है। RPG-Kit command definitions, runtime scripts, MCP configuration, और generated graph data को आपके code के साथ जोड़ता है।

```text
my-project/
├── docs/                 # /rpgkit.feature_spec के लिए optional requirement docs
├── .github/ or .claude/  # AI assistant command definitions और settings
├── .vscode/              # applicable होने पर Copilot/VS Code MCP configuration
└── .rpgkit/              # RPG-Kit runtime
    ├── scripts/          # Pipeline scripts और support packages
    ├── data/             # Generated artifacts, जिनमें rpg.json और dep_graph.json शामिल हैं
    ├── logs/             # Per-stage execution logs
    └── reports/          # Generated review और diagnostic reports
```

Full layout और data file reference के लिए [docs/project-structure.md](docs/project-structure.md) देखें।

## Supported Platforms

| प्लेटफ़ॉर्म             | Claude Code | GitHub Copilot | Codex |
| ----------------------- | ----------- | -------------- | ----- |
| CLI उपयोग               | ✅          | ✅(MCP नहीं)   | ⌛    |
| VS Code extension उपयोग | ✅          | ✅             | ⌛    |

| Script | Linux | Windows | Mac |
| ------ | ----- | ------- | --- |
| sh     | ✅    | ⌛      | ⌛  |
| ps     | N/A   | ⌛      | ⌛  |

## Documentation

- [Slash command reference](docs/commands.md) — हर `/rpgkit.*` command, inputs, outputs, और examples।
- [CLI reference](docs/cli-reference.md) — `rpgkit init`, `rpgkit update`, `rpgkit check`, `rpgkit version`, और सभी options।
- [Configuration](docs/configuration.md) — AI assistant setup, MCP registration, hooks, auto-approval, और troubleshooting।
- [Project structure](docs/project-structure.md) — RPG-Kit द्वारा बनाए गए files और directories।

## आगामी फीचर्स

- **सरल decoder commands:** मौजूदा decoder flow को कम commands में merge करना, जिसमें end-to-end repository generation के लिए `/rpgkit.generate_repo`, और feature generation तथा RPG planning के लिए `/rpgkit.generate_feature` plus `/rpgkit.plan` शामिल हैं।
- **Multi-language support:** Go, C++, Rust, JavaScript/TypeScript, और अन्य के लिए support जोड़ना।
- **अधिक platform integrations:** अलग-अलग systems पर अलग-अलग AI coding agents के लिए CLI और VS Code extension workflows में RPG-Kit support करना।

## Troubleshooting

**AI assistant CLI नहीं मिला:** `rpgkit check` run करें, selected assistant CLI install और authenticate करें, फिर `rpgkit init` या `rpgkit update` दोबारा run करें।

**MCP tools `rpg_unavailable` report करते हैं:** `.rpgkit/data/rpg.json` create करने के लिए `/rpgkit.encode` run करें।

**Incremental update failed:** `.rpgkit/logs/update_rpg.log` inspect करें, फिर `/rpgkit.update_rpg` run करें।

**Rate limits या private repo access के कारण template download fail होता है:** `--github-token $GITHUB_TOKEN` pass करें या `GH_TOKEN` / `GITHUB_TOKEN` set करें।

## License

MIT License - विवरण के लिए [LICENSE](LICENSE) देखें।

## Acknowledgements

[GitHub Spec-Kit](https://github.com/github/spec-kit) पर आधारित।
