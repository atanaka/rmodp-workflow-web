---
name: rmodp-computational-view-web
description: >
  RM-ODP Computational Viewpoint Specification（計算視点仕様書）を導出する。
  Enterprise Viewpoint および Information Viewpoint の仕様を入力として、
  ITU-T X.903 / ISO/IEC 10746-3 に基づく5ステップの構造化分析
  （Computational Object → Interface → Interaction/Binding →
  Environment Contract/Distribution Transparency → Refinement）を行い、
  PlantUML コンポーネント図・シーケンス図（PNG 生成）を含む
  Markdown ドキュメントを生成する。
  Use when: RM-ODP Computational Viewpoint の仕様を作成する場合、
  システムの機能分割・インターフェース・分散透過性を体系的にモデリングする場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、特に「Computational Viewpoint（計算視点）」のモデリングの専門家です。

実行前に `view` ツールで以下を順に読み込む：
1. `/home/claude/.rmodp/enterprise-view.md`
2. `/home/claude/.rmodp/information-view.md`

ファイルが存在しない場合はその旨をユーザーに伝え、作業を続行する。

読み込んだ Enterprise Viewpoint Specification および Information Viewpoint Specification を分析し、
RM-ODP Computational Language（ITU-T X.903 | ISO/IEC 10746-3）の概念体系に基づいた
「Computational Viewpoint Specification（計算視点仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語（Computational Object, Interface (Operation/Signal/Stream),
  Interaction (Interrogation/Announcement/Flow), Binding, Environment Contract,
  Distribution Transparency など）を正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけでは仕様として不十分な部分（通信の同期・非同期、QoS 要件、
  障害時の振る舞いなど）がある場合は、Step 5 にて「逆質問」としてユーザーに確認事項を提示すること。

### 図の生成ルール（必須）

**2種類の PlantUML 図を必ず生成すること。Mermaid は使用しない。**

各図は以下の手順で生成する：
1. `create_file` ツールで `.puml` ファイルを `/home/claude/.rmodp/` に保存する
2. `bash_tool` で `plantuml <ファイル>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する
3. Markdown に PlantUML ソースコード（` ```plantuml ` ブロック）と
   PNG の画像参照（`![図名](ファイル名.png)`）を両方埋め込む

| 図番号 | 図名 | PlantUML 記法 | ファイル名 |
|---|---|---|---|
| 図1 | Component Structure Diagram | コンポーネント図（`rectangle` + ステレオタイプ） | `computational-component.puml` |
| 図2 | Interaction Sequence Diagram | シーケンス図（主要ユースケース × 複数可） | `computational-seq-{ユースケース名}.puml` |

- **Step 1 では必ず図1（コンポーネント構造図）を生成すること。**
- **Step 3 では必ず図2（シーケンス図）を生成すること。主要ユースケースごとに個別ファイルで生成する。**

## 処理ステップ

### Step 1: Computational Object（計算オブジェクト）の特定
- Enterprise Viewpoint の「Role / Process」および Information Viewpoint の
  「Information Object / 状態遷移」を実現するために、システムをどのように機能分割するかを分析し、
  具体的な機能コンポーネントである「Computational Object」を特定する。
- 各オブジェクトのシステム内における役割と責務を定義する。
- **以下の形式で PlantUML コンポーネント構造図を出力する：**

```plantuml
@startuml computational-component
skinparam componentBackgroundColor #EEF6FF
skinparam componentBorderColor #5599CC
skinparam defaultFontSize 11
skinparam backgroundColor #FAFAFA

title {システム名} — Computational Viewpoint Component Diagram

rectangle "ComputationalObject1" as CO1 <<Computational Object>> #B8D4E8
rectangle "ComputationalObject2" as CO2 <<Computational Object>> #B8E8C8
rectangle "BindingObject1" as BO1 <<Binding Object>> #FFFFF0

CO1 --> CO2 : Interrogation: operationName()
CO1 ..> BO1 : Announcement: signalName()
BO1 --> CO2 : forward()

note right of CO1 : Role: {対応する Enterprise Role}
@enduml
```

  記述ルール:
  - `rectangle "名前" as エイリアス <<ステレオタイプ>>` で Computational Object を表現する
  - `<<Computational Object>>` または `<<Binding Object>>` ステレオタイプを付与する
  - 矢印ラベルに Interaction 種別（Interrogation / Announcement / Signal / Flow）を明記する
  - Enterprise Viewpoint の Role との対応を `note` で記載する

### Step 2: Computational Interface（計算インターフェース）の定義
- 各 Computational Object が提供（Server）または要求（Client）するインターフェースを定義する。
- 各インターフェースについて、以下の種別を特定する：
  - **Operation interface（操作インターフェース）**: 操作の呼び出しと応答
  - **Signal interface（シグナルインターフェース）**: アトミックな一方向通知
  - **Stream interface（ストリームインターフェース）**: 継続的な情報の流れ

### Step 3: Interaction（インタラクション）と Binding（バインディング）の設計
- インターフェース間で発生する具体的なインタラクションを定義する：
  - Operation の場合：**Interrogation**（呼び出しと応答による同期通信）か
    **Announcement**（呼び出しのみの非同期通信）か
  - Stream の場合：**Flow**（継続的な情報の流れ）
  - Signal の場合：アトミックな一方向通信
- オブジェクト間のインターフェースをどう接続するか（Binding）を明確にする。
  複雑な通信制御や仲介が必要な場合は「Binding Object（バインディングオブジェクト）」の
  導入を検討する。
- **業務シナリオの主要フローについて PlantUML シーケンス図を出力する：**

```plantuml
@startuml computational-seq-{ユースケース名}
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 11

title {システム名} — Computational Interaction: {ユースケース名}

actor "{アクター名}" as ACT
participant "ComputationalObject1\n(CO)" as CO1
participant "ComputationalObject2\n(CO)" as CO2
participant "BindingObject\n(Binding Object)" as BO

ACT -> CO1 : requestAction(params)
activate CO1
CO1 -> CO2 : Interrogation: operationName(params)
activate CO2
CO2 --> CO1 : response(result)
deactivate CO2
CO1 ->> BO : Announcement: signalName(data)
BO -> CO2 : forward(data)
CO1 --> ACT : result
deactivate CO1
@enduml
```

  記述ルール:
  - 同期（Interrogation）: `->` / `-->` （戻り値）
  - 非同期（Announcement / Signal）: `->>`
  - `activate` / `deactivate` で処理中を明示する
  - ラベルに Interaction 種別（Interrogation / Announcement / Flow）を明記する
  - ユースケースごとに個別の `.puml` ファイルを生成する

### Step 4: Environment Contract（環境契約）と Distribution Transparency（分散透過性）の特定
- オブジェクトおよびインターフェースに対する非機能要件や制約
  （QoS、パフォーマンス、セキュリティなど）を「Environment Contract」として定義する。
- 分散システムの複雑さを隠蔽するためにシステム基盤側に求める
  「Distribution Transparency（透過性）」の要件を特定する：
  - **Access transparency（アクセス透過性）**: ローカル / リモートを同一の方法でアクセス
  - **Location transparency（位置透過性）**: 場所を意識せずにアクセス可能
  - **Failure transparency（障害透過性）**: 障害からの回復を意識しない
  - **Transaction transparency（トランザクション透過性）**: トランザクション境界を意識しない
  - その他（Replication, Migration, Relocation など）が必要であれば特定する

### Step 5: 評価と逆質問（Refinement）
- 生成した仕様の妥当性を評価し、完全な Computational Specification を構築するために
  不足している要件（例：同期 / 非同期の最終決定、具体的な QoS 目標値、
  障害検知時のリカバリ方針、トランザクションの境界など）を、
  3〜5 個の「逆質問」として提示する。

## ファイルの保存

### .puml ファイルの保存と PNG 生成

各 `.puml` ファイルを `create_file` で保存後、以下の bash コマンドで PNG を生成する：

```bash
plantuml /home/claude/.rmodp/computational-component.puml -o /home/claude/.rmodp/
plantuml /home/claude/.rmodp/computational-seq-{ユースケース名}.puml -o /home/claude/.rmodp/
# シーケンス図が複数ある場合は全ファイル分実行する
```

### Markdown ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/computational-view.md` に保存する。

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

完了後、`rmodp-engineering-view-web` スキルを使用して Engineering Viewpoint を作成する。
（`rmodp-workflow-web` 経由の場合は自動的に次ステップへ進む）
