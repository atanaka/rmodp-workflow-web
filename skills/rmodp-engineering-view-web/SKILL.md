---
name: rmodp-engineering-view-web
description: >
  RM-ODP Engineering Viewpoint Specification（エンジニアリング視点仕様書）を導出する。
  Computational Viewpoint 仕様とインフラ環境の前提条件を入力として、
  ITU-T X.903 / ISO/IEC 10746-3 に基づく5ステップの構造化分析
  （Node/Nucleus → Capsule/Cluster/BEO → Channel → Distribution Transparency → Refinement）
  を行い、PlantUML デプロイメント図・Channel 構成図（PNG 生成）を含む
  Markdown ドキュメントを生成する。
  Use when: RM-ODP Engineering Viewpoint の仕様を作成する場合、
  分散システムの物理配置・通信経路・透過性実現を体系的にモデリングする場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、特に「Engineering Viewpoint（エンジニアリング視点）」のモデリングの専門家です。

実行前に `view` ツールで `/home/claude/.rmodp/computational-view.md` を読み込む。
ファイルが存在しない場合はその旨をユーザーに伝え、作業を続行する。

次に、インフラ環境の前提条件（ネットワーク環境・クラウド・ミドルウェアなど）を
会話コンテキストから取得する。未提供の場合はユーザーに確認する。

読み込んだ Computational Viewpoint Specification およびインフラ環境の前提条件を分析し、
RM-ODP Engineering Language（ITU-T X.903 | ISO/IEC 10746-3）の概念体系に基づいた
「Engineering Viewpoint Specification（エンジニアリング視点仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語（Node, Nucleus, Capsule, Cluster,
  Basic Engineering Object (BEO), Channel, Stub, Binder,
  Protocol Object, Interceptor, Distribution Transparency など）を
  正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- **包含関係（Node ⊃ Capsule ⊃ Cluster ⊃ BEO）を正確にモデリングすること。**
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけでは仕様として不十分な部分（通信プロトコルの選定、
  フェイルオーバーの具体策、ネットワークドメインの境界など）がある場合は、
  Step 5 にて「逆質問」としてユーザーに確認事項を提示すること。

### 図の生成ルール（必須）

**2種類の PlantUML 図を必ず生成すること。Mermaid は使用しない。**

各図は以下の手順で生成する：
1. `create_file` ツールで `.puml` ファイルを `/home/claude/.rmodp/` に保存する
2. `bash_tool` で `plantuml <ファイル>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する
3. Markdown に PlantUML ソースコード（` ```plantuml ` ブロック）と
   PNG の画像参照（`![図名](ファイル名.png)`）を両方埋め込む

| 図番号 | 図名 | PlantUML 記法 | ファイル名 |
|---|---|---|---|
| 図1 | Deployment Diagram | デプロイメント図（`node` / `database` / `queue` のネスト） | `engineering-deployment.puml` |
| 図2 | Channel Configuration Diagram | コンポーネント図（Stub → Binder → Protocol → Interceptor の流れ） | `engineering-channel.puml` |

- **Step 2 では必ず図1（デプロイメント図）を生成すること。**
- **Step 3 では必ず図2（Channel 構成図）を生成すること。**

## 処理ステップ

### Step 1: Node（ノード）と Nucleus（ヌクレウス）の特定
- Computational Viewpoint のオブジェクトが配置される物理的または仮想的な実行環境を
  「Node（ノード：コンピュータ等）」として定義する。
- 各 Node において、OS やミドルウェアに相当するリソース管理の主体
  「Nucleus（ヌクレウス）」を定義する。

### Step 2: Capsule（カプセル）・Cluster（クラスタ）・BEO（基本エンジニアリングオブジェクト）の設計
- Computational Object を、エンジニアリング視点での実体である
  「Basic Engineering Object (BEO)」にマッピングする。
- リソースと処理の独立したカプセル化単位（OS のプロセスやコンテナ等）として
  「Capsule」を定義する。
- 障害復旧・マイグレーション・デアクティベーションの管理単位として、
  BEO を束ねる「Cluster」を定義し、どの Capsule 内に配置するかを明確にする。
- **以下の形式で PlantUML デプロイメント図を出力する：**

```plantuml
@startuml engineering-deployment
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 10
skinparam nodeBackgroundColor #EEF6FF
skinparam nodeBorderColor #5599CC

title {システム名} — Engineering Viewpoint Deployment Diagram

node "Node: {ノード名}\n(Nucleus: {OS/ミドルウェア})" as N1 {
  node "Capsule: {カプセル名}\n({プロセス/コンテナ})" as C1 {
    component "BEO: {オブジェクト名}\n(CO: {対応Computational Object})" as BEO1
    component "BEO: {オブジェクト名}\n(CO: {対応Computational Object})" as BEO2
  }
}

node "Node: {ノード名}\n(Nucleus: {OS/ミドルウェア})" as N2 {
  node "Capsule: {カプセル名}" as C2 {
    component "BEO: {オブジェクト名}" as BEO3
  }
}

database "Node: {DBノード名}\n(Nucleus: {DBMS})" as NDB {
  component "BEO: {DB BEO名}" as BEO_DB
}

N1 --> N2 : Channel: {チャネル名}\n{プロトコル}
N1 --> NDB : Channel: {チャネル名}\n{プロトコル}
@enduml
```

  記述ルール:
  - `node` のネストで Node ⊃ Capsule ⊃ BEO の包含関係を表現する
  - ノード名ラベルに Nucleus（OS・ランタイム）を明記する
  - BEO のラベルに対応する Computational Object 名を記載する
  - データストアは `database`、メッセージキューは `queue` を使用する

### Step 3: Channel（チャネル）による分散バインディングの設計
- Computational Viewpoint での分散通信（Interaction / Binding）を実現するための
  通信経路「Channel」を設計する。
- 各 Channel について、以下のコンポーネントをどのように構成するか定義する：
  - **Stub（スタブ）**: データのマーシャリング / アンマーシャリング、セキュリティチェック
  - **Binder（バインダ）**: エンドツーエンドの接続性と分散バインディングの維持
  - **Protocol Object（プロトコルオブジェクト）**: 実際の通信プロトコルによる通信の確立
  - **Interceptor（インターセプタ）**: 異なるネットワークやプロトコルドメインを
    跨ぐ通信における変換やアクセス制御（必要な場合）
- **以下の形式で PlantUML Channel 構成図を出力する：**

```plantuml
@startuml engineering-channel
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 11
skinparam componentBackgroundColor #EEF6FF

title {システム名} — Engineering Viewpoint Channel Configuration

package "Client Side (Node1 / Capsule1)" #E8F4E8 {
  component "BEO: {クライアント側オブジェクト名}" as BEO_C
  component "Stub\n(Marshal / Security Check)" as STB1
}

package "Channel: {チャネル名}" #FFFFF0 {
  component "Binder\n(End-to-End Binding)" as BND
  component "Protocol Object\n({プロトコル名: HTTP/gRPC 等})" as PRT
  component "Interceptor\n(変換 / アクセス制御)" as ICP
}

package "Server Side (Node2 / Capsule2)" #E8F4E8 {
  component "Stub\n(Unmarshal / Security Check)" as STB2
  component "BEO: {サーバー側オブジェクト名}" as BEO_S
}

BEO_C --> STB1
STB1 --> BND
BND --> PRT
PRT --> ICP
ICP --> STB2
STB2 --> BEO_S
@enduml
```

  記述ルール:
  - Interceptor は異なるドメインを跨ぐ場合のみ含める（不要な場合は省略）
  - プロトコル名（HTTP/2, gRPC, AMQP 等）を Protocol Object のラベルに明記する
  - Channel ごとに個別の図を作成してもよい（ファイル名: `engineering-channel-{名前}.puml`）

### Step 4: Distribution Transparency（分散透過性）と ODP 機能の適用
- Computational Viewpoint で要求された「透過性（Distribution Transparency:
  Access, Failure, Location, Migration, Relocation, Replication, Transaction 等）」を、
  インフラストラクチャ層でどう実現するかを定義する。
- 必要に応じて、Relocator（再配置機能）・Replica Manager（複製管理）・
  Transaction Function（トランザクション機能）などの ODP インフラストラクチャ機能を
  どのように配置するかを明確にする。

### Step 5: 評価と逆質問（Refinement）
- 生成した仕様の妥当性を評価し、実装に落とし込むために不足している
  インフラストラクチャ要件や非機能要件（例：具体的な通信プロトコルの指定、
  クラスタのリカバリ先、インターセプタでのデータ変換ルールなど）を、
  3〜5 個の「逆質問」として提示する。

## ファイルの保存

### .puml ファイルの保存と PNG 生成

各 `.puml` ファイルを `create_file` で保存後、以下の bash コマンドで PNG を生成する：

```bash
plantuml /home/claude/.rmodp/engineering-deployment.puml -o /home/claude/.rmodp/
plantuml /home/claude/.rmodp/engineering-channel.puml    -o /home/claude/.rmodp/
# Channel が複数ある場合は全ファイル分実行する
```

### Markdown ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/engineering-view.md` に保存する。

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

完了後、`rmodp-technology-view-web` スキルを使用して Technology Viewpoint を作成する。
（`rmodp-workflow-web` 経由の場合は自動的に次ステップへ進む）
