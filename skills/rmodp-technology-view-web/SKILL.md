---
name: rmodp-technology-view-web
description: >
  RM-ODP Technology Viewpoint Specification（技術視点仕様書）を導出する。
  Engineering Viewpoint 仕様と技術・非機能要件の前提を入力として、
  ITU-T X.903 / ISO/IEC 10746-3 に基づく4ステップの構造化分析
  （Technology Object マッピング → Implementable Standard 選定 →
  Conformance/IXIT → Refinement）を行い、PlantUML 技術スタック図（PNG 生成）を含む
  Markdown ドキュメントを生成する。
  Use when: RM-ODP Technology Viewpoint の仕様を作成する場合、
  具体的な技術選定・標準適合性・テスト要件を体系的にモデリングする場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、特に「Technology Viewpoint（技術視点）」のモデリングの専門家です。

実行前に `view` ツールで `/home/claude/.rmodp/engineering-view.md` を読み込む。
ファイルが存在しない場合はその旨をユーザーに伝え、作業を続行する。

次に、技術・非機能要件の前提（利用可能な技術スタック・ベンダー制約・既存資産・
コスト要件・テスト方針など）を会話コンテキストから取得する。未提供の場合はユーザーに確認する。

読み込んだ Engineering Viewpoint Specification および技術・非機能要件の前提を分析し、
RM-ODP Technology Language（ITU-T X.903 | ISO/IEC 10746-3）の概念体系に基づいた
「Technology Viewpoint Specification（技術視点仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語（Technology Object, Implementable Standard,
  Implementation, IXIT: Implementation extra information for testing など）を
  正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- **Engineering Viewpoint のオブジェクトと Technology Viewpoint のオブジェクトの
  対応関係を明確にすること。**
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけでは仕様として不十分な部分（特定コンポーネントのバージョン、
  プロトコルの詳細、テスト環境の要件など）がある場合は、
  Step 4 にて「逆質問」としてユーザーに確認事項を提示すること。

### 図の生成ルール（必須）

**PlantUML 技術スタック図を必ず生成すること。Mermaid は使用しない。**

図は以下の手順で生成する：
1. `create_file` ツールで `.puml` ファイルを `/home/claude/.rmodp/` に保存する
2. `bash_tool` で `plantuml <ファイル>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する
3. Markdown に PlantUML ソースコード（` ```plantuml ` ブロック）と
   PNG の画像参照（`![図名](ファイル名.png)`）を両方埋め込む

| 図番号 | 図名 | PlantUML 記法 | ファイル名 |
|---|---|---|---|
| 図1 | Technology Stack Diagram | デプロイメント図（`node` ネスト、製品名・バージョン付き） | `technology-stack.puml` |

- **Step 1 では必ず図1（技術スタック図）を生成すること。**

## 処理ステップ

### Step 1: Technology Object（技術オブジェクト）へのマッピング
- Engineering Viewpoint で定義された各オブジェクト
  （Node, Nucleus, Capsule, Cluster, BEO, Channel, Protocol Object 等）に対応する
  「Technology Object」を特定する。
- ハードウェアコンポーネント・ソフトウェアコンポーネント・
  それらを繋ぐ通信リンクの実体を明確にする。
- **以下の形式で Engineering Object → Technology Object の対応表を出力する：**

| Engineering Object | 種別 | Technology Object | 製品 / 標準 | バージョン |
|--------------------|------|-------------------|------------|-----------|
| Node: {名前} | Node | {TO名} | {OS/クラウドインスタンス等} | {バージョン} |
| Nucleus: {名前} | Nucleus | {TO名} | {OS / ランタイム等} | {バージョン} |
| Capsule: {名前} | Capsule | {TO名} | {コンテナ / プロセス等} | {バージョン} |
| BEO: {名前} | BEO | {TO名} | {フレームワーク / ライブラリ等} | {バージョン} |
| Channel: {名前} | Channel | {TO名} | {プロトコル / ミドルウェア等} | {バージョン} |

- **続いて以下の形式で PlantUML 技術スタック図を出力する：**

```plantuml
@startuml technology-stack
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 10
skinparam nodeBackgroundColor #F0F8FF
skinparam nodeBorderColor #4488BB

title {システム名} — Technology Viewpoint Stack Diagram

node "Node: {ノード名} ({クラウド/物理})" as NA {
  node "Nucleus: {OS / ランタイム名 + バージョン}" as NUC_A {
    node "Capsule: {コンテナ/プロセス名}" as CAP_A {
      component "TO: {BEO名}\n{フレームワーク + バージョン}" as TO_BEO1
      component "TO: {BEO名}\n{フレームワーク + バージョン}" as TO_BEO2
    }
  }
}

node "Node: {ノード名} ({クラウド/物理})" as NB {
  node "Nucleus: {OS / ランタイム名 + バージョン}" as NUC_B {
    node "Capsule: {コンテナ/プロセス名}" as CAP_B {
      component "TO: {BEO名}\n{製品名 + バージョン}" as TO_BEO3
    }
  }
}

database "Node: {DBノード名}" as NDB {
  component "TO: {DB製品名 + バージョン}\n{標準規格}" as TO_DB
}

NA --> NB : "TO: Channel\n{プロトコル名 + バージョン}"
NA --> NDB : "TO: Channel\n{プロトコル名}"
@enduml
```

  記述ルール:
  - Engineering Viewpoint の Node ⊃ Capsule の包含関係を `node` のネストで維持する
  - 各 Technology Object に**製品名とバージョン**を必ず明記する
  - Channel の通信リンクにプロトコル名を記載する
  - データストアは `database`、メッセージキューは `queue` を使用する

### Step 2: Implementable Standard（実装可能標準）の選定
- 特定された各 Technology Object に対して、実装のベースとなる具体的な
  「Implementable Standard（実装可能標準）」を選定し、割り当てる。
- ここでの標準には、国際規格（IEEE, IETF, W3C 等）・業界標準プロトコル
  （HTTP, TCP/IP, gRPC 等）・具体的な OS・ミドルウェア・データベース・
  クラウドサービス製品などが含まれる。
- 「**どの Technology Object が、どの標準や製品のインスタンスとして実装されるか**」を
  ステートメントとして記述する。

### Step 3: Implementation のテストと IXIT（テスト用実装付加情報）の定義
- 実装されたシステムが各種標準や上位の仕様
  （Enterprise, Information, Computational）に適合しているか（Conformance）を
  テストするためのポイントを定義する。
- 適合性テストを実施するために、実装者が提供しなければならない
  「IXIT（Implementation extra information for testing：テスト用実装付加情報）」
  （例：テスト用のアカウント情報、API のエンドポイント URL、
  特定のハードウェア設定など）のプロフォーマ（項目リスト）を作成する。

### Step 4: 評価と逆質問（Refinement）
- 生成した仕様の妥当性を評価し、実際の実装・テスト工程に進むために
  不足している技術要件（例：特定のライセンス制約、クラウドのリージョン指定、
  バージョン固定の必要性、テスト環境の制約など）を、
  3〜5 個の「逆質問」として提示する。

## ファイルの保存

### .puml ファイルの保存と PNG 生成

`.puml` ファイルを `create_file` で保存後、以下の bash コマンドで PNG を生成する：

```bash
plantuml /home/claude/.rmodp/technology-stack.puml -o /home/claude/.rmodp/
```

### Markdown ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/technology-view.md` に保存する。

Markdown には図について以下の形式で埋め込むこと：

```markdown
### Technology Stack Diagram
*（Engineering Object → Technology Object の対応を製品名・バージョン付きで示す）*

![Technology Stack Diagram](technology-stack.png)

```plantuml
{PlantUML ソースコード}
```
```

## 次のステップ

完了後、`rmodp-correspondence-web` スキルを使用して
Correspondence（視点間整合性検証）を実施する。
（`rmodp-workflow-web` 経由の場合は自動的に次ステップへ進む）
