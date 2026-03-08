---
name: rmodp-workflow-web
description: >
  RM-ODP 全6スキル（5 Viewpoint + Correspondence）をオーケストレーションする統合ワークフロー。
  業務シナリオから Enterprise → Information → Computational → Engineering → Technology → Correspondence
  の順に仕様を導出し、視点間の整合性を検証する。フル実行・ステップ指定・再開・検証のみの4モードに対応。
  Use when: RM-ODP の複数 Viewpoint を一貫して開発する場合、
  「rmodp-workflow-web」「RM-ODP全体」「フルパイプライン」等の指示を受けた場合。
argument-hint: "[システム名 または 業務概要]"
environment: claude.ai (Web)
---

# RM-ODP Integrated Workflow — Web 版

RM-ODP 全 Viewpoint を統合的にオーケストレーションする。
6つのサブスキルを依存関係に従い順次実行し、`/home/claude/.rmodp/` に仕様群を生成する。

> **Web 環境での動作**: 各サブスキルは `view` ツールで SKILL.md を読み込みインラインで実行する。
> ダイアグラムは各サブスキルが PlantUML 形式で `.puml` ファイルを生成し、`bash_tool` で PNG に変換後、
> PlantUML ソースと PNG 画像参照の両方を仕様書（Markdown）に埋め込む。
> ファイル書き込みは `create_file` ツールを使用する。

## パイプライン全体図

```
業務シナリオ + インフラ前提 + 技術スタック
    │
    ▼
┌─ Phase A: 要件モデリング ──────────────────────────────────┐
│  Step 1: Enterprise VP  (6 steps) → enterprise-view.md    │
│  Step 2: Information VP (5 steps) → information-view.md   │
└────────────────────────────────────────────────────────────┘
    │ Enterprise + Information 仕様を入力
    ▼
┌─ Phase B: システム設計 ────────────────────────────────────┐
│  Step 3: Computational VP (5 steps) → computational-view.md│
│  Step 4: Engineering VP   (5 steps) → engineering-view.md  │
│  Step 5: Technology VP    (4 steps) → technology-view.md   │
└────────────────────────────────────────────────────────────┘
    │ 全5 Viewpoint を入力
    ▼
┌─ Phase C: 整合性検証 ──────────────────────────────────────┐
│  Step 6: Correspondence  (4 steps) → correspondence.md    │
└────────────────────────────────────────────────────────────┘
```

## 出力言語の決定（Output Language Detection）

1. 入力テキスト（業務概要 + 業務シナリオ）の主要言語を判定する
2. 入力が英語 → 英語で出力（進捗テンプレート: `assets/progress-template-en.md`）
3. 入力が日本語 → 日本語で出力（進捗テンプレート: `assets/progress-template.md`）
4. 混在している場合 → 量が多い方の言語で出力する
5. 判定不能の場合 → 日本語で出力する（後方互換性）
6. `.rmodp/` に既存ファイルがある場合 → 既存ファイルの言語に合わせる（一貫性維持）

**RM-ODP 用語の扱い:**
- RM-ODP 公式用語は常に英語で記述する
- 日本語出力の場合のみ括弧書きで日本語訳を添える（例: `Community（コミュニティ）`）

> ⚠️ **言語強制ルール（Language Enforcement Rule）**
> 本 SKILL.md は日本語で記述されているが、**英語出力モードで実行中はユーザーへの一切の出力を英語にすること。**
> 具体的には以下の日本語ラベルを英語に置き換えて出力すること：
>
> | SKILL.md 内部ラベル（出力禁止） | 英語モードでの出力 |
> |---|---|
> | 完了 / 完了チェック | Complete / Completion Check |
> | 入力 | Input |
> | 完了後 | After completion |
> | 実行手順 | Execution Steps |
> | 新規プロジェクト | New project |
> | 要件モデリング | Requirements Modeling |
> | システム設計 | System Design |
> | 整合性検証 | Consistency Verification |
>
> **各ステップの開始アナウンス形式（英語モード）:**
> 英語モードで各 Step を開始する際は、以下の形式で必ずアナウンスすること。自発的に日本語で要約してはならない。
> ```
> Now proceeding to Phase X — Step N: <Viewpoint Name>.
> <One-line English description of what will be done in this step.>
> ```
>
> **この SKILL.md に書かれた日本語の見出しや内部ラベルは、実行指示であってユーザーへの出力テキストではない。英語モード実行時にそのまま出力してはならない。**

## 実行モード

| モード | 説明 | 実行ステップ |
|---|---|---|
| **フルワークフロー** | 全6ステップを順次実行 | Step 1 → 6 (Phase A → B → C) |
| **ステップ指定実行** | 指定したステップのみ実行 | ユーザー指定 |
| **途中再開** | 指定ステップから再開 | Step N → 6 |
| **整合性検証のみ** | Correspondence のみ実行 | Step 6 のみ |

追加オプション：
- **Phase 分割**: Phase A 完了後に一旦停止し、新しい会話で Phase B から再開（コンテキスト節約）
- **インタラクティブ**: 各ステップ完了後にユーザー確認を挟む

---

## 実行手順

### PHASE 0: 初期化

#### 0.1 プロジェクト情報の取得

$ARGUMENTS またはユーザーとの対話から以下を取得：

- **システム名**: kebab-case に変換（例：「受注管理システム」→ `order-management`）
- **業務概要**: システムの全体像
- **業務シナリオ**: 具体的な業務フローとルール

#### 0.2 出力ディレクトリの準備

```bash
mkdir -p /home/claude/.rmodp
```

#### 0.3 既存成果物の確認

```bash
ls -la /home/claude/.rmodp/ 2>/dev/null || echo "New project (新規プロジェクト)"
```

既存ファイルがある場合、どのステップまで完了しているか判定してユーザーに提示：

| ファイル | 対応ステップ | 状態 |
|---|---|---|
| `enterprise-view.md` | Step 1 | ✅ 存在 / ❌ 未作成 |
| `information-view.md` | Step 2 | ✅ 存在 / ❌ 未作成 |
| `computational-view.md` | Step 3 | ✅ 存在 / ❌ 未作成 |
| `engineering-view.md` | Step 4 | ✅ 存在 / ❌ 未作成 |
| `technology-view.md` | Step 5 | ✅ 存在 / ❌ 未作成 |
| `correspondence.md` | Step 6 | ✅ 存在 / ❌ 未作成 |

#### 0.4 実行モードの確認

実行モードをユーザーに確認する（未指定の場合のみ）。
既存成果物がある場合は「途中再開」を推奨として提示。

#### 0.5 追加入力の一括収集（モードに応じて）

- Step 4 (Engineering) を含む場合 → **インフラ環境の前提条件** を取得
- Step 5 (Technology) を含む場合 → **技術スタック・非機能要件** を取得

取得した追加入力は会話コンテキストに保持する。
**該当ステップの実行時に再度ユーザーへ質問してはならない。**

#### 0.6 workflow-status.json の初期化

`create_file` ツールで `/home/claude/.rmodp/workflow-status.json` に保存：

```json
{
  "project": "{システム名}",
  "started_at": "{ISO 8601 datetime}",
  "current_step": 0,
  "steps": {
    "1_enterprise":     { "status": "pending", "file": null },
    "2_information":    { "status": "pending", "file": null },
    "3_computational":  { "status": "pending", "file": null },
    "4_engineering":    { "status": "pending", "file": null },
    "5_technology":     { "status": "pending", "file": null },
    "6_correspondence": { "status": "pending", "file": null }
  },
  "inputs": {
    "business_overview": "...",
    "business_scenario": "...",
    "infra_prerequisites": "...",
    "tech_stack": "..."
  }
}
```

---

### PHASE A: Requirements Modeling (Step 1-2) / 要件モデリング

#### Step 1: Enterprise Viewpoint

`view` ツールで `/mnt/skills/user/rmodp-enterprise-view-web/SKILL.md` を読み込み、
そのスキルの指示に従って Enterprise Viewpoint を導出する。

**Input:**
- 業務概要・業務シナリオ（PHASE 0 で取得済み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/enterprise-view.md` が生成されている
- [ ] Community, Objective, Role, Process, Deontic token, Policy, Accountability が定義されている
- [ ] 逆質問（3〜5個）がユーザーに提示され、回答が反映されている

workflow-status.json を更新：`"1_enterprise": { "status": "completed", "file": "enterprise-view.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 1 — Enterprise Viewpoint complete.
   Defined Community, Roles, Processes, Deontic tokens, and Policies → enterprise-view.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 1 — Enterprise Viewpoint 完了
   Community, Role, Process, Deontic token, Policy を定義 → enterprise-view.md
```

---

#### Step 2: Information Viewpoint

`view` ツールで `/mnt/skills/user/rmodp-information-view-web/SKILL.md` を読み込み、
そのスキルの指示に従って Information Viewpoint を導出する。

**Input:**
- Enterprise VP 仕様（Step 1 の出力を `view` ツールで読み込み参照）
- 業務概要・業務シナリオ（PHASE 0 で取得済み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/information-view.md` が生成されている
- [ ] Information Object（属性定義付き）が特定されている
- [ ] Invariant Schema（常に真の制約）が定義されている
- [ ] Invariant Schema Diagram（クラス図＋制約 note）が PlantUML + PNG で出力されている
- [ ] Static Schema Diagram（`<<object>>` オブジェクト図）が PlantUML + PNG で出力されている
- [ ] Dynamic Schema Diagram（全 Information Object 複合状態遷移図）が PlantUML + PNG で出力されている
- [ ] 逆質問（3〜5個）がユーザーに提示され、回答が反映されている

workflow-status.json を更新：`"2_information": { "status": "completed", "file": "information-view.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 2 — Information Viewpoint complete.
   Defined Information Objects, Invariant/Static/Dynamic Schemas, generated state diagrams → information-view.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 2 — Information Viewpoint 完了
   Information Object, Invariant/Static/Dynamic Schema を定義、状態遷移図を生成 → information-view.md
```

---

### ⚠️ AUTO-SPLIT CHECKPOINT — Phase A 完了後

フルワークフローモードかつ Phase 分割が有効な場合：

1. Phase A の成果物を `present_files` ツールで提示
2. 以下のメッセージを表示して **停止する（Step 3 に進まない）**：

**日本語出力の場合:**
```
========================================
✅ PHASE A COMPLETE（要件モデリング）
========================================
Project: {システム名}
Steps completed: 1-2

Generated Artifacts:
  📄 Enterprise Viewpoint  → .rmodp/enterprise-view.md
  📄 Information Viewpoint → .rmodp/information-view.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📢 NEXT: 新しい会話で以下を指示してください：

  「{システム名} の rmodp-workflow-web を Step 3 から再開」

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**英語出力の場合:**
```
========================================
✅ PHASE A COMPLETE (Requirements Modeling)
========================================
Project: {system-name}
Steps completed: 1-2

Generated Artifacts:
  📄 Enterprise Viewpoint  → .rmodp/enterprise-view.md
  📄 Information Viewpoint → .rmodp/information-view.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📢 NEXT: In a new conversation, say:

  "Resume rmodp-workflow-web for {system-name} from Step 3"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Phase 分割なし、またはユーザーが「このまま続行」と指示した場合は Phase B に進む。

---

### PHASE B: System Design (Step 3-5) / システム設計

#### Step 3: Computational Viewpoint

`view` ツールで `/mnt/skills/user/rmodp-computational-view-web/SKILL.md` を読み込み、
そのスキルの指示に従って Computational Viewpoint を導出する。

**Input:**
- Enterprise VP 仕様（`view` ツールで読み込み）
- Information VP 仕様（`view` ツールで読み込み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/computational-view.md` が生成されている
- [ ] Computational Object とその役割・責務が定義されている
- [ ] Interface（Operation / Signal / Stream）が定義されている
- [ ] Interaction 種別（Interrogation / Announcement / Flow）と Binding が定義されている
- [ ] PlantUML コンポーネント図（PNG）が出力されている
- [ ] 主要ユースケースの PlantUML シーケンス図（PNG）が出力されている
- [ ] Environment Contract と Distribution Transparency が定義されている
- [ ] 逆質問（3〜5個）がユーザーに提示され、回答が反映されている

workflow-status.json を更新：`"3_computational": { "status": "completed", "file": "computational-view.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 3 — Computational Viewpoint complete.
   Defined Computational Objects, Interfaces, Interactions, generated component and sequence diagrams → computational-view.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 3 — Computational Viewpoint 完了
   Computational Object, Interface, Interaction を定義、コンポーネント図・シーケンス図を生成 → computational-view.md
```

---

#### Step 4: Engineering Viewpoint

`view` ツールで `/mnt/skills/user/rmodp-engineering-view-web/SKILL.md` を読み込み、
そのスキルの指示に従って Engineering Viewpoint を導出する。

**Input:**
- Computational VP 仕様（`view` ツールで読み込み）
- インフラ環境の前提条件（PHASE 0 で取得済み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/engineering-view.md` が生成されている
- [ ] Node と Nucleus が定義されている
- [ ] 包含関係（Node ⊃ Capsule ⊃ Cluster ⊃ BEO）が正しくモデリングされている
- [ ] PlantUML デプロイメント図（PNG）が出力されている
- [ ] Channel の構成（Stub / Binder / Protocol Object / Interceptor）が定義されている
- [ ] PlantUML Channel 構成図（PNG）が出力されている
- [ ] Distribution Transparency と ODP 機能の適用が定義されている
- [ ] 逆質問（3〜5個）がユーザーに提示され、回答が反映されている

workflow-status.json を更新：`"4_engineering": { "status": "completed", "file": "engineering-view.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 4 — Engineering Viewpoint complete.
   Defined Nodes, Capsules, BEOs, Channels, generated deployment and channel diagrams → engineering-view.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 4 — Engineering Viewpoint 完了
   Node, Capsule, BEO, Channel を定義、デプロイメント図・Channel 構成図を生成 → engineering-view.md
```

---

#### Step 5: Technology Viewpoint

`view` ツールで `/mnt/skills/user/rmodp-technology-view-web/SKILL.md` を読み込み、
そのスキルの指示に従って Technology Viewpoint を導出する。

**Input:**
- Engineering VP 仕様（`view` ツールで読み込み）
- 技術スタック・非機能要件（PHASE 0 で取得済み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/technology-view.md` が生成されている
- [ ] Engineering Object → Technology Object の対応表が出力されている
- [ ] PlantUML 技術スタック図（製品名・バージョン付き、PNG）が出力されている
- [ ] Implementable Standard が各 Technology Object に割り当てられている
- [ ] IXIT（テスト用実装付加情報）のプロフォーマが定義されている
- [ ] 逆質問（3〜5個）がユーザーに提示され、回答が反映されている

workflow-status.json を更新：`"5_technology": { "status": "completed", "file": "technology-view.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 5 — Technology Viewpoint complete.
   Mapped Engineering Objects to Technology Objects, defined Implementable Standards and IXIT → technology-view.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 5 — Technology Viewpoint 完了
   Engineering Object → Technology Object マッピング、Implementable Standard・IXIT を定義 → technology-view.md
```

---

### PHASE C: Consistency Verification (Step 6) / 整合性検証

#### Step 6: Correspondence Specifications

`view` ツールで `/mnt/skills/user/rmodp-correspondence-web/SKILL.md` を読み込み、
そのスキルの指示に従って Correspondence を導出する。

**Input:**
- 全5 Viewpoint 仕様（各ファイルを `view` ツールで順次読み込み）

**Completion Check / 完了チェック:**
- [ ] `/home/claude/.rmodp/correspondence.md` が生成されている
- [ ] Enterprise を起点とした4つの対応関係（→Information / →Computational / →Engineering / →Technology）が定義されている
- [ ] 基本3視点間の3つの対応関係（Computational↔Information / Engineering↔Computational / Technology↔Engineering）が定義されている
- [ ] PlantUML トレーサビリティマップ（PNG）が出力されている
- [ ] Markdown テーブルによるトレーサビリティマトリクスが出力されている
- [ ] 孤立要素・機能欠落・矛盾の検証結果が記述されている
- [ ] 逆質問（3〜5個）がユーザーに提示されている

workflow-status.json を更新：`"6_correspondence": { "status": "completed", "file": "correspondence.md" }`

**英語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 6 — Correspondence Specifications complete.
   Verified cross-viewpoint traceability, identified orphan elements and gaps → correspondence.md
```

**日本語出力モードでの完了メッセージ（必ずこの形式で出力すること）:**
```
✅ STEP 6 — Correspondence Specifications 完了
   視点間トレーサビリティを検証、孤立要素・ギャップを特定 → correspondence.md
```

---

### PHASE FINAL: Complete / 完了

全ステップ完了後、以下を実行する：

#### 1. 成果物の一括提供

```bash
# outputs にコピー
cp -r /home/claude/.rmodp /mnt/user-data/outputs/{システム名}-rmodp

# ZIP 作成
cd /mnt/user-data/outputs
zip -r {システム名}-rmodp.zip {システム名}-rmodp/
```

`present_files` ツールで以下を提示（優先順）：
1. `{システム名}-rmodp.zip`（一括ダウンロード）
2. `enterprise-view.md`
3. `information-view.md`
4. `computational-view.md`
5. `engineering-view.md`
6. `technology-view.md`
7. `correspondence.md`

#### 2. 最終サマリの表示

**日本語出力の場合:**
```
========================================
✅ RM-ODP WORKFLOW COMPLETE
========================================
Project: {システム名}

Generated Specifications:
  📋 Enterprise Viewpoint    → .rmodp/enterprise-view.md
  📊 Information Viewpoint   → .rmodp/information-view.md
  ⚙️  Computational Viewpoint → .rmodp/computational-view.md
  🏗️  Engineering Viewpoint   → .rmodp/engineering-view.md
  🔧 Technology Viewpoint    → .rmodp/technology-view.md
  🔗 Correspondence Specs    → .rmodp/correspondence.md

Verification:
  Orphan Elements:    {x} 件
  Gaps:               {x} 件
  Contradictions:     {x} 件
  Overall Status:     ✅ OK / ⚠️ 要対応

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📢 NEXT ACTIONS:
  - 不整合がある場合 → 該当 Viewpoint を修正後、rmodp-correspondence-web を再実行
  - 個別 Viewpoint の修正 → 対応するサブスキルで再実行
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**英語出力の場合:**
```
========================================
✅ RM-ODP WORKFLOW COMPLETE
========================================
Project: {system-name}

Generated Specifications:
  📋 Enterprise Viewpoint    → .rmodp/enterprise-view.md
  📊 Information Viewpoint   → .rmodp/information-view.md
  ⚙️  Computational Viewpoint → .rmodp/computational-view.md
  🏗️  Engineering Viewpoint   → .rmodp/engineering-view.md
  🔧 Technology Viewpoint    → .rmodp/technology-view.md
  🔗 Correspondence Specs    → .rmodp/correspondence.md

Verification:
  Orphan Elements:    {x} detected
  Gaps:               {x} detected
  Contradictions:     {x} detected
  Overall Status:     ✅ OK / ⚠️ Action Required

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📢 NEXT ACTIONS:
  - If inconsistencies found → Fix the relevant Viewpoint, then re-run rmodp-correspondence-web
  - Individual Viewpoint updates → Re-run the corresponding sub-skill
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## エラーハンドリング

| エラー | 対処 |
|---|---|
| 前提ファイルが存在しない | 依存ステップの実行を促す |
| サブスキル SKILL.md が見つからない | スキルの内容をインラインで実行する |
| 逆質問への回答が不足 | 仮定値で進め、Step 6 で再検証 |
| コンテキストウィンドウ不足 | Phase 分割を提案 |

## 進捗管理

ステップ完了ごとに `create_file` ツールで `/home/claude/.rmodp/workflow-status.json` を上書き保存する。
再開時は `view` ツールで読み込んで状態を復元する。

進捗テンプレートは `assets/progress-template.md`（日本語）または
`assets/progress-template-en.md`（英語）を参照。

## Claude Code 版との差異

| 項目 | Claude Code 版 | Web 版（本スキル） |
|---|---|---|
| サブスキル実行 | `/rmodp-enterprise-view` スラッシュコマンド | `view` で SKILL.md を読み込みインライン実行 |
| ダイアグラム | PlantUML（`.puml` → `.png` 生成） | PlantUML（`.puml` → `.png` 生成、PNG + ソースを仕様書に埋め込み） |
| ファイル書き込み | bash で直接書き込み | `create_file` ツール |
| 成果物提供 | ファイルシステム上に残る | `present_files` ツールでダウンロード提供 |
| 進捗再開 | bash で `workflow-status.json` を読み書き | `view` / `create_file` ツールで読み書き |
