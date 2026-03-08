---
name: rmodp-information-view-web
description: >
  RM-ODP Information Viewpoint Specification（情報視点仕様書）を導出する。
  業務概要と業務シナリオを入力として、ITU-T X.903 / ISO/IEC 10746-3 に基づく
  5ステップの構造化分析（Information Object → Invariant Schema → Static Schema →
  Dynamic Schema → Refinement）を行い、PlantUML クラス図・制約図・状態遷移図
  （PNG 生成）を含む Markdown ドキュメントを生成する。
  Use when: RM-ODP Information Viewpoint の仕様を作成する場合、
  業務データの構造・制約・状態遷移を体系的にモデリングする場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、特に「Information Viewpoint（情報視点）」のモデリングの専門家です。

実行前に `view` ツールで `/home/claude/.rmodp/enterprise-view.md` を読み込み、
Enterprise Viewpoint の内容（Community・Role・Process・Deontic token）を参照する。
ファイルが存在しない場合はその旨をユーザーに伝え、作業を続行する。

会話コンテキストから【業務概要】と【業務シナリオ】を取得し（未提供の場合はユーザーに確認する）、
RM-ODP Information Language（ITU-T X.903 | ISO/IEC 10746-3）の概念体系に基づいた
「Information Viewpoint Specification（情報視点仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語（Information Object, Invariant schema, Static schema,
  Dynamic schema など）を正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけでは情報モデルの仕様として不十分な部分（未定義のデータ状態、
  例外時の状態遷移、データのライフサイクルなど）がある場合は、
  Step 5 にて「逆質問」としてユーザーに確認事項を提示すること。

### 図の生成ルール（必須）

**3種類の PlantUML 図を必ず生成すること。Mermaid は使用しない。**

各図は以下の手順で生成する：
1. `create_file` ツールで `.puml` ファイルを `/home/claude/.rmodp/` に保存する
2. `bash_tool` で `plantuml <ファイル>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する
3. Markdown に PlantUML ソースコード（` ```plantuml ` ブロック）と
   PNG の画像参照（`![図名](ファイル名.png)`）を両方埋め込む

| 図番号 | 図名 | PlantUML 記法 | ファイル名 | 生成ステップ |
|---|---|---|---|---|
| 図1 | Invariant Schema Diagram | クラス図（`class` + `note` による制約注記） | `information-invariant.puml` | Step 2 |
| 図2 | Static Schema Diagram | クラス図（`<<object>>` ステレオタイプによるオブジェクト図） | `information-static.puml` | Step 3 |
| 図3 | Dynamic Schema Diagram | 状態図（`state` / `-->` による状態遷移、エンティティごとに個別ファイル） | `information-dynamic-{エンティティ名}.puml` | Step 4 |

- **Step 2 では必ず図1（Invariant Schema Diagram）を生成すること。**
- **Step 3 では必ず図2（Static Schema Diagram）を生成すること。**
- **Step 4 では必ず図3（Dynamic Schema Diagram）をエンティティごとに生成すること。**

## 処理ステップ

### Step 1: Information Object（情報オブジェクト）の特定
- 業務シナリオから、システムが意味を管理し、状態を追跡する必要がある対象
  （エンティティ、概念、記録など）を「Information Object」として抽出・特定する。
- 各 Information Object がどのような情報を持つか（属性の概要）を定義する。

### Step 2: Invariant Schema（不変スキーマ）の定義
- システムのライフサイクルを通じて「常に真（True）」でなければならない
  ビジネスルールや制約を特定する。
- データの整合性を保つための前提条件や、状態変化に関わらず維持される関係性を記述する
  （例：貸出冊数は上限を超えない、貸出記録は必ず実在する会員と蔵書に紐づく、など）。
- **以下の形式で PlantUML Invariant Schema Diagram を出力する：**
  ITU-T X.903 に基づき、Information Object をクラスとして表現し、
  常に成立すべき制約（Invariant）を `note` ブロックで各クラスに付記する。
  クラス間の必須参照関係（参照整合性制約）は関連矢印で表現する。

```plantuml
@startuml information-invariant
skinparam backgroundColor #FAFAFA
skinparam classBackgroundColor #EEF6FF
skinparam classBorderColor #5599CC
skinparam defaultFontSize 11
skinparam noteBackgroundColor #FFFDE7
skinparam noteBorderColor #BBAA44

title {システム名} — Information Viewpoint Invariant Schema

class "{InformationObject1名}" as IO1 {
  +{属性名}: {型}
  +{属性名}: {型}
}
note right of IO1
  INV-E1: [存在制約の説明]
  INV-V1: {属性名} の値制約
end note

class "{InformationObject2名}" as IO2 {
  +{属性名}: {型}
}
note bottom of IO2
  INV-A1: [集約制約の説明]
end note

IO1 "1" --> "0..*" IO2 : {関連名}\n{INV-R1: 参照整合性制約}

note as N_global
  Invariant（全体）:
  INV-R2: IO2 は必ず有効な IO1 に紐づく
end note
@enduml
```

  記述ルール:
  - クラス名は Information Object 名（RM-ODP での論理名）を使用する
  - 属性は `+属性名: 型` の形式で記述する（具体的な値ではなく型定義）
  - **各クラスの `note` に、そのオブジェクトに課される Invariant（INV-xx）を列挙する**
  - 参照整合性制約は関連矢印のラベルにも記載し、双方から確認できるようにする
  - 全体に関わる制約は `note as N_global` でフローティングノートとして記述する
  - 多重度は `"1"`, `"0..*"`, `"1..*"` 等で両端に必ず明示する

### Step 3: Static Schema（静的スキーマ）の定義
- 特定の時点（スナップショット）における Information Object の状態（State）や、
  オブジェクト間の関連性（Relationship）を定義する。
- 業務プロセスの中で意味を持つ主要な状態
  （例：初期状態、貸出中状態、延滞状態など）を定義し、
  その状態にあるときのオブジェクトの条件を明確にする。
- **以下の形式で PlantUML Static Schema Diagram（オブジェクト図）を出力する：**
  ある時点のシステム状態（インスタンスとその関係）を表現する。
  `<<object>>` ステレオタイプを付与したクラス記法でインスタンスを表現し、
  業務プロセスの主要な状態ごとにオブジェクト図を作成する。

```plantuml
@startuml information-static
skinparam backgroundColor #FAFAFA
skinparam classBackgroundColor #F0FFF0
skinparam classBorderColor #44AA66
skinparam defaultFontSize 11
skinparam stereotypeCBackgroundColor #DDFFDD

title {システム名} — Information Viewpoint Static Schema\n（スナップショット: {状態名} 時点）

class "{インスタンス名}_{型名}" as obj1 {
  <<object>>
  {属性名} = "{具体的な値}"
  {属性名} = {具体的な値}
}

class "{インスタンス名}_{型名}" as obj2 {
  <<object>>
  {属性名} = "{具体的な値}"
}

obj1 "{多重度}" --> "{多重度}" obj2 : {関連名}
@enduml
```

  記述ルール:
  - クラス名は `インスタンス名_型名` 形式（例: `loan001_LoanRecord`）、ハイフン不可
  - `<<object>>` ステレオタイプを必ず付与してクラス図（型定義）と区別する
  - 属性は `属性名 = 値` の形式で**具体的な値**を記述する（型宣言ではなく）
  - 多重度は `"1"`, `"0..*"`, `"1..*"` 等で両端に明示する
  - Invariant Schema で定義した制約を満たしている状態を具体的な値で示す
  - 業務プロセスの主要な状態（初期状態・処理中・完了等）ごとにオブジェクト図を作成する
  - **複数の状態スナップショットがある場合は、同一ファイル内に `newpage` で区切って出力する**

```plantuml
' 複数スナップショットの例
@startuml information-static
' ... （1枚目: 貸出前の状態）
newpage {状態名2}
' ... （2枚目: 貸出中の状態）
@enduml
```

### Step 4: Dynamic Schema（動的スキーマ）の定義
- 業務シナリオ（基本フロー、代替フロー）で発生するアクションによって、
  Information Object の状態がどのように変化するのか（状態遷移）を定義する。
- Information Object の生成（Creation）・更新（Modification）・削除（Deletion）の
  タイミングと、そのトリガーとなるアクションを明確にする。
- **エンティティごとに個別の PlantUML 状態図（State Diagram）を出力する：**

```plantuml
@startuml information-dynamic-{エンティティ名}
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 11
skinparam stateBackgroundColor #EEF6FF
skinparam stateBorderColor #5599CC
skinparam ArrowColor #444444

title {システム名} — Dynamic Schema: {エンティティ名}

[*] --> {初期状態名} : {Creation trigger}

{初期状態名} : {成立条件や説明（任意）}
{状態名2} : {成立条件や説明（任意）}

{初期状態名} --> {状態名2} : {Modification action}\n[{guard condition}]
{状態名2} --> {状態名3} : {Modification action}
{状態名3} --> [*] : {Deletion trigger}
@enduml
```

  記述ルール:
  - ファイル名は `information-dynamic-{エンティティ名小文字}.puml`（例: `information-dynamic-loanrecord.puml`）
  - 生成は `[*] --> 初期状態`、削除は `最終状態 --> [*]` で表す
  - 遷移ラベルは `アクション名 / [ガード条件]` の形式（ガード条件がある場合のみ）
  - 各状態の説明（成立条件・属性値の変化）を状態ノード内にコメントとして記述する
  - **エンティティごとに個別の `.puml` ファイルを生成する**

### Step 5: 評価と逆質問（Refinement）
- 生成した仕様の妥当性を評価し、完全な Information Specification を構築するために
  不足しているデータ要件（例：データの保持期間、特定の状態からの復帰条件、
  排他制御の必要性など）を、3〜5 個の「逆質問」として提示する。

## ファイルの保存

### .puml ファイルの保存と PNG 生成

各 `.puml` ファイルを `create_file` で保存後、以下の bash コマンドで PNG を生成する：

```bash
plantuml /home/claude/.rmodp/information-invariant.puml -o /home/claude/.rmodp/
plantuml /home/claude/.rmodp/information-static.puml    -o /home/claude/.rmodp/
plantuml /home/claude/.rmodp/information-dynamic-{エンティティ名}.puml -o /home/claude/.rmodp/
# Dynamic Schema が複数エンティティある場合は全ファイル分実行する
```

### Markdown ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/information-view.md` に保存する。

Markdown には各図について以下の形式で埋め込むこと：

```markdown
### {図名}
*（図の説明）*

![{図名}]({ファイル名}.png)

```plantuml
{PlantUML ソースコード}
```
```

## 次のステップ

完了後、`rmodp-computational-view-web` スキルを使用して Computational Viewpoint を作成する。
（`rmodp-workflow-web` 経由の場合は自動的に次ステップへ進む）
