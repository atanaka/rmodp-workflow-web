# rmodp-workflow-web

**RM-ODP Integrated Workflow for Claude.ai — Web 版**

[English](#english) | [日本語](#japanese)

---

<a name="english"></a>
## English

### Overview

`rmodp-workflow-web` is a Claude.ai Skills-based pipeline that systematically derives formal **RM-ODP (Reference Model of Open Distributed Processing)** specifications from business scenarios.

Following the **ITU-T X.901–X.911 / ISO/IEC 10746** standard, the workflow orchestrates 6 sub-skills across 3 phases to produce a complete, traceable distributed-system specification — from enterprise policy through to technology stack.

```
Business Scenario
    │
    ▼
┌─ Phase A: Requirements Modeling ─────────────────────────────┐
│  Step 1: Enterprise VP  (ITU-T X.911)  → enterprise-view.md  │
│  Step 2: Information VP (ITU-T X.903)  → information-view.md │
└───────────────────────────────────────────────────────────────┘
    │
    ▼
┌─ Phase B: System Design ──────────────────────────────────────┐
│  Step 3: Computational VP  → computational-view.md            │
│  Step 4: Engineering VP    → engineering-view.md              │
│  Step 5: Technology VP     → technology-view.md               │
└───────────────────────────────────────────────────────────────┘
    │
    ▼
┌─ Phase C: Consistency Verification ───────────────────────────┐
│  Step 6: Correspondence    → correspondence.md                 │
└───────────────────────────────────────────────────────────────┘
```

### Features

- **Standards-compliant** — Strictly follows ITU-T X.903 and X.911 terminology and concepts
- **Diagram-first** — Each viewpoint generates PlantUML diagrams (PNG) embedded in Markdown
- **Bilingual** — Auto-detects Japanese/English input and produces output in the same language
- **Resume support** — Workflow state is persisted in `workflow-status.json`; any step can be resumed in a new conversation
- **Phase splitting** — Phase A and Phase B/C can be run in separate Claude.ai sessions to conserve context window
- **Clarifying questions** — Each sub-skill prompts 3–5 clarifying questions to refine specifications before proceeding

### Skill Structure

| Skill | Phase | Description |
|---|---|---|
| `rmodp-workflow-web` | Orchestrator | Entry point; manages execution modes and sub-skill calls |
| `rmodp-enterprise-view-web` | Phase A / Step 1 | Enterprise VP: Community, Role, Process, Deontic tokens, Policy |
| `rmodp-information-view-web` | Phase A / Step 2 | Information VP: Information Objects, Invariant/Static/Dynamic Schemas |
| `rmodp-computational-view-web` | Phase B / Step 3 | Computational VP: Objects, Interfaces, Interactions, Bindings |
| `rmodp-engineering-view-web` | Phase B / Step 4 | Engineering VP: Node, Capsule, BEO, Channel |
| `rmodp-technology-view-web` | Phase B / Step 5 | Technology VP: Technology Objects, Implementable Standards, IXIT |
| `rmodp-correspondence-web` | Phase C / Step 6 | Correspondence: Cross-viewpoint traceability and consistency verification |

### Execution Modes

| Mode | Description |
|---|---|
| **Full workflow** | Run all 6 steps sequentially (Steps 1–6) |
| **Step-specific** | Run only the specified step |
| **Resume** | Resume from a specified step (requires prior output files) |
| **Verification only** | Run Step 6 (Correspondence) only |

### Generated Outputs

All files are generated under `/home/claude/.rmodp/` during execution and bundled as a ZIP for download.

| File | Contents |
|---|---|
| `enterprise-view.md` | Community, Roles, Processes, Deontic tokens, Policy + 3 diagrams (PNG) |
| `information-view.md` | Information Objects, Schemas + Invariant/Static/Dynamic diagrams (PNG) |
| `computational-view.md` | Computational Objects, Interfaces, Interactions + component & sequence diagrams (PNG) |
| `engineering-view.md` | Nodes, Capsules, BEOs, Channels + deployment & channel diagrams (PNG) |
| `technology-view.md` | Technology Object mapping, Implementable Standards, IXIT table + stack diagram (PNG) |
| `correspondence.md` | Cross-viewpoint correspondences, traceability matrix, orphan/gap analysis |
| `workflow-status.json` | Execution state for resume support |

### Installation

See [INSTALL.md](./INSTALL.md) for step-by-step upload instructions.

### Requirements

- Claude.ai account (Pro, Team, or Enterprise plan)
- Claude.ai Skills feature (currently in beta — available on Pro plan and above)
- `plantuml` is available in the Claude.ai execution environment (no separate installation needed)

### Example Usage

After uploading all 7 skills to Claude.ai, start a conversation with:

```
rmodp-workflow-web ライブラリ貸出管理システム
```

Or in English:

```
rmodp-workflow-web Library Lending Management System
```

The orchestrator will gather your business overview and scenario, then execute the full pipeline.

---

<a name="japanese"></a>
## 日本語

### 概要

`rmodp-workflow-web` は、業務シナリオから **RM-ODP（Open Distributed Processing の参照モデル）** の正式仕様書を体系的に導出する、Claude.ai スキルベースのパイプラインです。

**ITU-T X.901–X.911 / ISO/IEC 10746** 規格に準拠し、6つのサブスキルを3フェーズにわたってオーケストレーションし、企業ポリシーから技術スタックまで、完全でトレーサビリティのある分散システム仕様を生成します。

### 特徴

- **規格準拠** — ITU-T X.903 および X.911 の用語・概念を厳密に遵守
- **ダイアグラム生成** — 各 Viewpoint で PlantUML 図（PNG）を生成し Markdown に埋め込み
- **バイリンガル** — 入力言語（日本語/英語）を自動検出し、同じ言語で出力
- **再開対応** — `workflow-status.json` で状態を永続化。任意のステップから別会話で再開可能
- **フェーズ分割** — コンテキストウィンドウ節約のため Phase A と Phase B/C を別セッションで実行可能
- **逆質問機能** — 各サブスキルが 3〜5 個の逆質問を提示し、仕様を精緻化

### スキル構成

| スキル | フェーズ | 説明 |
|---|---|---|
| `rmodp-workflow-web` | オーケストレーター | エントリーポイント。実行モード管理とサブスキル呼び出し |
| `rmodp-enterprise-view-web` | Phase A / Step 1 | 企業視点: Community, Role, Process, Deontic token, Policy |
| `rmodp-information-view-web` | Phase A / Step 2 | 情報視点: Information Object, 不変/静的/動的スキーマ |
| `rmodp-computational-view-web` | Phase B / Step 3 | 計算視点: オブジェクト, インターフェース, インタラクション, バインディング |
| `rmodp-engineering-view-web` | Phase B / Step 4 | エンジニアリング視点: Node, Capsule, BEO, Channel |
| `rmodp-technology-view-web` | Phase B / Step 5 | 技術視点: Technology Object, 実装可能標準, IXIT |
| `rmodp-correspondence-web` | Phase C / Step 6 | 対応関係: 視点間トレーサビリティと整合性検証 |

### 実行モード

| モード | 説明 |
|---|---|
| **フルワークフロー** | Step 1〜6 を順次実行 |
| **ステップ指定** | 指定したステップのみ実行 |
| **途中再開** | 指定したステップから再開 |
| **整合性検証のみ** | Step 6（Correspondence）のみ実行 |

### インストール

[INSTALL.md](./INSTALL.md) を参照してください。

### 使用例

7つのスキルを Claude.ai にアップロード後、以下のように開始します：

```
rmodp-workflow-web ライブラリ貸出管理システム
```

---

## Repository Structure

```
rmodp-workflow-web/
├── README.md                          # This file
├── INSTALL.md                         # Installation guide (JA/EN)
├── LICENSE                            # MIT License
├── skills/
│   ├── rmodp-workflow-web/            # Orchestrator skill
│   │   ├── SKILL.md
│   │   └── assets/
│   │       ├── progress-template.md    # Progress tracker (Japanese)
│   │       └── progress-template-en.md # Progress tracker (English)
│   ├── rmodp-enterprise-view-web/     # Step 1: Enterprise VP
│   │   └── SKILL.md
│   ├── rmodp-information-view-web/    # Step 2: Information VP
│   │   └── SKILL.md
│   ├── rmodp-computational-view-web/  # Step 3: Computational VP
│   │   └── SKILL.md
│   ├── rmodp-engineering-view-web/    # Step 4: Engineering VP
│   │   └── SKILL.md
│   ├── rmodp-technology-view-web/     # Step 5: Technology VP
│   │   └── SKILL.md
│   └── rmodp-correspondence-web/      # Step 6: Correspondence
│       └── SKILL.md
└── examples/
    └── library-system/                # Example: Library lending system
        └── scenario.md                # Sample business scenario input
```

## License

MIT License — see [LICENSE](./LICENSE)

## Related Projects

- [uml-workflow-v3](https://github.com/atanaka/uml-workflow-v3) — UML-based application generation pipeline (scenario → production code)

## Author

Akira Tanaka
