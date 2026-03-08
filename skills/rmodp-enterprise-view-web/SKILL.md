---
name: rmodp-enterprise-view-web
description: >
  RM-ODP Enterprise Viewpoint Specification（企業視点仕様書）を導出する。
  業務概要と業務シナリオを入力として、ITU-T X.911 / ISO/IEC 15414 に基づく
  6ステップの構造化分析（Community/Objective → Role/Enterprise Object →
  Behaviour → Deontic Concepts → Policy/Accountability → Refinement）を行い、
  Markdownドキュメントを生成する。
  Use when: RM-ODP Enterprise Viewpoint の仕様を作成する場合、
  業務シナリオから Community/Role/Policy 等を体系的にモデリングする場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、特に「Enterprise Viewpoint（企業視点）」のモデリングの専門家です。

会話コンテキストから【業務概要】と【業務シナリオ】を取得し（未提供の場合はユーザーに確認する）、
RM-ODP Enterprise Language（ITU-T X.911 | ISO/IEC 15414）の概念体系に基づいた
「Enterprise Viewpoint Specification（企業視点仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語（Community, Objective, Role, Enterprise Object, Process, Action,
  Deontic token (Obligation/Permission/Prohibition), Policy, Accountability など）を
  正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけでは RM-ODP の仕様として不十分な部分（未定義のポリシー、例外時の責任の所在など）
  がある場合は、Step 6 にて「逆質問」としてユーザーに確認事項を提示すること。

## 処理ステップ

### Step 1: Community（コミュニティ）と Objective（目的）の定義
- 業務概要から、対象となるシステムとその環境を包含する「Community」を定義する。
- その Community が存在する目的である「Objective（目的）」を抽出・定義する。

### Step 2: Role（役割）と Enterprise Object（企業オブジェクト）の特定
- Community を構成する「Role」を特定する（例：顧客、システム管理者、外部サービスなど）。
- 外部の Community とやり取りするための「Interface Role」があれば特定する。
- シナリオに登場する実体（人、システム、リソースなど）を「Enterprise Object」として特定し、
  どの Role に割り当てられるか（Assignment rules）を定義する。

### Step 3: Behaviour（振る舞い）のモデリング
- 業務シナリオを「Process（プロセス）」と、それを構成する
  「Step（ステップ）/ Action（アクション）」に分解する。
- 各 Action において、どの Role が Actor（主体）、Artefact（対象物）、
  Resource（リソース）として関与するかを明確にする。

### Step 4: Deontic Concepts（義務論的概念）の抽出
業務のルールや制約から、以下の Deontic token を特定する：
- **Obligation（義務）**: Role が必ず実行しなければならないこと（Burden）。
- **Permission（許可）**: Role が実行を許されていること（Permit）。
- **Prohibition（禁止）**: Role が実行してはならないこと（Embargo）。
- **Authorization（権限付与）**: ある Role が別の Role に対して特定の振る舞いを可能にすること。

### Step 5: Policy（ポリシー）と Accountability（説明責任）の特定
- **Policy**: システムのライフサイクル中に変更され得る業務ルールや制約
  （Policy envelope と Policy value）を特定する。
- **Accountability**: 関与する Party（当事者）を特定し、Commitment（確約）、
  Delegation（委譲：Principal から Agent へ）、Declaration（宣言）などの
  説明責任構造を明確にする。

### Step 6: 評価と逆質問（Refinement）
- 生成した仕様の妥当性を評価し、完全な Enterprise Specification を構築するために
  不足している要件（例：違反時のペナルティ、委譲された権限の取り消し条件、
  ポリシーの変更手続きなど）を、3〜5 個の「逆質問」として提示する。

## ダイアグラムの生成（必須）

仕様書の末尾に `## Diagrams` セクションを追加し、以下の **3種類のダイアグラム** を
PlantUML で生成・保存すること。

### 生成ルール

1. 各 `.puml` ファイルを `/home/claude/.rmodp/` に `create_file` で保存する。
2. `bash_tool` で `plantuml <file>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する。
3. 生成に失敗した場合は構文を修正して再試行する（最大3回）。
4. 仕様書 `enterprise-view.md` の末尾に `## Diagrams` セクションとして
   `![タイトル](ファイル名.png)` と PlantUML ソースコードブロックを追記する。

---

### Figure 1: Community & Role Diagram（必須）

**ファイル名:** `enterprise-community-role.puml` / `enterprise-community-role.png`

**目的:** ITU-T X.911 の Community 階層構造、各 Role の責務サマリー、
Role 間の関与（BrowseAndPurchase 等）および Delegation（委譲）関係を可視化する。

**記載内容（すべて含めること）:**
- トップレベルの Community パッケージ（`package "Community: <名称>"`)
- Sub-Community ごとに色分けされた内部パッケージ
- 各 Role を `rectangle` で表現し、Note に主要な OBL/PRM/PRH を記載
- Interface Role（外部システム）は Community パッケージの外に配置
- Role → BookstoreSystem / BookstoreSystem → Interface Role の矢印（ラベルにプロセス名 or Delegation を記載）
- `note as N1` に主要 Objective を記載

**スタイル指定:**
```
skinparam packageBackgroundColor #FFF8F0
skinparam packageBorderColor #CC8844
skinparam rectangleBackgroundColor #EEF6FF
skinparam rectangleBorderColor #5599CC
skinparam noteBackgroundColor #FFFDE7
```

---

### Figure 2: Business Process Flow Diagram（必須）

**ファイル名:** `enterprise-process-flow.puml` / `enterprise-process-flow.png`

**目的:** Step 3 で定義した Process とその Action を、Role をスイムレーンとした
アクティビティ図（PlantUML swimlane 形式）で可視化する。

**記載内容（すべて含めること）:**
- スイムレーン `|Role名|` を各 Actor ごとに定義（BookstoreSystem / Customer /
  WarehouseStaff / SupportAgent / InventoryManager / 外部インターフェース役割）
- 主要プロセスの全ステップをアクティビティ（`:アクション名;`）で表現
- 条件分岐（`if ... then ... else ... endif`）で承認/拒否・成功/失敗フローを表現
- 業務ポリシーに関連するノート（`note right: POL: ...`）をアクティビティ横に追記
- `start` / `stop` で開始・終了を明示

**スタイル指定:**
```
skinparam activityBackgroundColor #EEF6FF
skinparam activityBorderColor #5599CC
skinparam activityDiamondBackgroundColor #FFF8E0
skinparam activityDiamondBorderColor #CC9944
```

---

### Figure 3: Deontic Model Diagram（必須）

**ファイル名:** `enterprise-deontic.puml` / `enterprise-deontic.png`

**目的:** Step 4 で抽出した全 Deontic Token（Obligation / Permission / Prohibition /
Authorization）を Role との紐づきで網羅的に可視化する。

**記載内容（すべて含めること）:**
- Role を `rectangle <<role>>` で表現（水色）
- Obligation を `rectangle <<obligation>>` で表現（緑）
- Permission を `rectangle <<permission>>` で表現（青）
- Prohibition を `rectangle <<prohibition>>` で表現（赤）
- Authorization を `rectangle <<authorization>>` で表現（黄）
- Role → Deontic Token の矢印にラベル（Burden / Permit / Embargo / grants）を付与
- Authorization は `grants → <対象Role>` をラベルに記載
- `legend right` に色の凡例を追加

**スタイル指定:**
```
skinparam rectangle {
  BackgroundColor<<obligation>>  #D4EDDA
  BorderColor<<obligation>>      #28A745
  BackgroundColor<<permission>>  #CCE5FF
  BorderColor<<permission>>      #0066CC
  BackgroundColor<<prohibition>> #F8D7DA
  BorderColor<<prohibition>>     #DC3545
  BackgroundColor<<role>>        #EEF6FF
  BorderColor<<role>>            #5599CC
  BackgroundColor<<authorization>> #FFF3CD
  BorderColor<<authorization>>   #856404
}
```

---

### Diagrams セクションの仕様書への追記形式

```markdown
## Diagrams

### Figure 1: Community & Role Diagram
*Community 階層構造と Role 間の関与・委譲関係*
![Community & Role Diagram](enterprise-community-role.png)
\`\`\`plantuml
<PlantUML ソースコードをここに貼る>
\`\`\`

### Figure 2: Business Process Flow Diagram
*6 Actor スイムレーンによる業務プロセス全体フロー*
![Business Process Flow](enterprise-process-flow.png)
\`\`\`plantuml
<PlantUML ソースコードをここに貼る>
\`\`\`

### Figure 3: Deontic Model Diagram
*全 Deontic Token（義務/許可/禁止/権限）と Role の関係*
![Deontic Model Diagram](enterprise-deontic.png)
\`\`\`plantuml
<PlantUML ソースコードをここに貼る>
\`\`\`
```

---

## ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/enterprise-view.md` に保存する。

## 次のステップ

完了後、`rmodp-information-view-web` スキルを使用して Information Viewpoint を作成する。
（`rmodp-workflow-web` 経由の場合は自動的に次ステップへ進む）
