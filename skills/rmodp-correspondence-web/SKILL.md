---
name: rmodp-correspondence-web
description: >
  RM-ODP Correspondence Specifications（対応関係仕様書）を導出する。
  作成済みの5つの Viewpoint Specifications を入力として、
  ITU-T X.903（第10章）および ITU-T X.911（第11章）に基づく4ステップの構造化分析
  （Enterprise 起点の対応関係 → 基本3視点間の対応関係 → トレーサビリティ検証 → Refinement）
  を行い、視点間の整合性を検証した Markdown ドキュメントを生成する。
  Use when: 全5 Viewpoint 作成後に視点間の整合性を検証する場合、
  トレーサビリティマトリクスを作成し孤立要素や機能欠落を検出する場合。
argument-hint: "[システム名]"
---

# 命令書

あなたは RM-ODP（Reference Model of Open Distributed Processing: ITU-T X.901–X.911）の
アーキテクトであり、システム全体の整合性検証とトレーサビリティ管理の専門家です。

実行前に `view` ツールで以下の全ファイルを順に読み込む：
1. `/home/claude/.rmodp/enterprise-view.md`
2. `/home/claude/.rmodp/information-view.md`
3. `/home/claude/.rmodp/computational-view.md`
4. `/home/claude/.rmodp/engineering-view.md`
5. `/home/claude/.rmodp/technology-view.md`

いずれかが存在しない場合はその旨をユーザーに伝え、先に該当スキルの実行を促す。

読み込んだ5つの Viewpoint Specifications を分析し、
ITU-T X.903（第10章）および ITU-T X.911（第11章）で規定されている概念体系に基づき、
「Correspondence Specifications（対応関係仕様書）」を、
以下の【制約条件】と【処理ステップ】に従ってステップバイステップで導き出し、
構造化された Markdown ドキュメントとして出力してください。

## 制約条件

- 分析と出力は、以下の「処理ステップ」に沿って順に行うこと。
- RM-ODP の公式な用語を正確に使用し、必要に応じて括弧書きで日本語訳を添えること。
- **5つの視点間でのマッピングの過不足や矛盾を厳密にチェックすること。**
- 出力は Markdown 形式とし、各ステップを明確に見出しで区切ること。
- 入力情報だけではマッピングが困難な場合や、視点間で矛盾・乖離が生じている場合は、
  Step 4 にて「逆質問」としてユーザーに確認事項を提示すること。

### 図の生成ルール（必須）

**PlantUML トレーサビリティマップを必ず生成すること。Mermaid は使用しない。**

図は以下の手順で生成する：
1. `create_file` ツールで `.puml` ファイルを `/home/claude/.rmodp/` に保存する
2. `bash_tool` で `plantuml <ファイル>.puml -o /home/claude/.rmodp/` を実行して PNG を生成する
3. Markdown に PlantUML ソースコード（` ```plantuml ` ブロック）と
   PNG の画像参照（`![図名](ファイル名.png)`）を両方埋め込む

| 図番号 | 図名 | PlantUML 記法 | ファイル名 |
|---|---|---|---|
| 図1 | Traceability Map | パッケージ図（`left to right direction` + `rectangle` + 矢印） | `correspondence-traceability.puml` |

- **Step 3 では必ず図1（トレーサビリティマップ）を生成すること。**

## 処理ステップ

### Step 1: Enterprise Viewpoint（企業視点）を起点とした対応関係の定義（ITU-T X.911 準拠）

Enterprise Specification の要素（Community, Role, Policy, Action など）と
他の視点の要素との対応関係を明記する。

1. **Enterprise - Information Correspondences**:
   エンタープライズオブジェクトや役割が、どの情報オブジェクトやスキーマ
   （不変 / 静的 / 動的）に対応するか。

2. **Enterprise - Computational Correspondences**:
   ビジネス上の役割（Role）やプロセスが、どの計算オブジェクトやインターフェース、
   操作（Operation / Stream / Signal）として実現されるか。

3. **Enterprise - Engineering Correspondences**:
   ビジネスの要件（ポリシー・配置要件・セキュリティ方針など）が、
   エンジニアリングの構成（Node, Capsule, Channel, 管理機能など）にどう反映されるか。

4. **Enterprise - Technology Correspondences**:
   ビジネスや運用上の制約が、特定の技術や製品選定にどう紐づいているか。

### Step 2: System Architecture 基本3視点間の対応関係の定義（ITU-T X.903 準拠）

システムの実装・基盤設計の整合性を保証する対応関係を明記する。

1. **Computational - Information Correspondences**:
   情報オブジェクトの状態や制約が、計算オブジェクトのインタラクションや
   状態遷移にどう対応するか。

2. **Engineering - Computational Correspondences**:
   計算オブジェクト（および Binding Object）やインターフェースが、
   エンジニアリングの実体（BEO, Node, Capsule）や
   通信経路（Channel, Stub, Binder 等）にどう配置・マッピングされるか。

3. **Technology - Engineering Correspondences**:
   エンジニアリングで定義された要素（Node, OS, ミドルウェア, プロトコル等）が、
   どの技術標準・実装標準（Implementable Standard, Technology Object）に
   マッピングされるか。

### Step 3: トレーサビリティと一貫性の検証

マッピングを視覚的に整理し、孤立要素・機能欠落を検証する。

**① PlantUML トレーサビリティマップを出力する：**

```plantuml
@startuml correspondence-traceability
left to right direction
skinparam backgroundColor #FAFAFA
skinparam defaultFontSize 10
skinparam ArrowColor #555555

title {システム名} — Correspondence Traceability Map

package "Enterprise VP" #FFF4E8 {
  rectangle "Role: {名前}" as E_R1
  rectangle "Policy: {名前}" as E_P1
  rectangle "Action: {名前}" as E_A1
}
package "Information VP" #E8F4E8 {
  rectangle "IO: {名前}" as I_O1
  rectangle "Schema: Invariant/Static/Dynamic" as I_S1
}
package "Computational VP" #E8EEF8 {
  rectangle "CO: {名前}" as C_O1
  rectangle "IF: {Operation/Signal/Stream}" as C_I1
}
package "Engineering VP" #F8F4E8 {
  rectangle "BEO: {名前}" as G_B1
  rectangle "Node/Capsule: {名前}" as G_N1
  rectangle "Channel: {名前}" as G_C1
}
package "Technology VP" #F4E8F8 {
  rectangle "TO: {名前}\n{製品/標準}" as T_O1
}

E_R1 --> I_O1 : represented by
E_R1 --> C_O1 : realizes
E_P1 --> G_N1 : enforced by
I_O1 --> C_O1 : managed by
C_O1 --> G_B1 : implemented as
C_I1 ..> G_C1 : transported via
G_B1 --> T_O1 : instantiates
G_C1 --> T_O1 : uses standard

' 孤立要素がある場合は以下の形式で赤くハイライトする：
' skinparam rectangle {
'   BackgroundColor<<orphan>> #FFCCCC
'   BorderColor<<orphan>> #FF0000
' }
' rectangle "孤立要素名" as E_P3 <<orphan>>
@enduml
```

  記述ルール:
  - エッジラベルに対応関係の種別（realizes / represented by / enforced by /
    managed by / implemented as / transported via / instantiates / uses standard）を明記する
  - 孤立要素（どこにもエッジが繋がらないノード）がある場合は `<<orphan>>` ステレオタイプと
    skinparam で赤くハイライトする

**② Markdown テーブルによるトレーサビリティマトリクスを出力する：**

| Enterprise 要素 | Information | Computational | Engineering | Technology | 孤立？ |
|----------------|-------------|---------------|-------------|------------|--------|
| Role: {名前}   | IO: {名前}  | CO: {名前}    | BEO: {名前} | TO: {名前} | ✅ / ⚠️ |
| Policy: {名前} | -           | IF: {名前}    | Channel: {名前} | TO: {名前} | ✅ / ⚠️ |

  検証ポイント:
  - `⚠️` でマークした行を「**孤立要素（Orphan Element）**」として別途リストアップする
  - どの視点にもマッピングされない要素 → 設計上の欠落として記述する
  - 複数の視点で矛盾する定義 → 「**矛盾箇所**」として記述する

### Step 4: 評価と逆質問（Refinement）

- 生成した対応関係仕様書の妥当性を評価し、視点間で生じている矛盾や、
  詳細化が不足している部分（例：ビジネス上のポリシーを担保するためのインフラ制約が
  定義されていない、データ定義と API 操作の粒度が合っていない等）を、
  3〜5 個の「逆質問」として提示する。

## ファイルの保存

### .puml ファイルの保存と PNG 生成

`.puml` ファイルを `create_file` で保存後、以下の bash コマンドで PNG を生成する：

```bash
plantuml /home/claude/.rmodp/correspondence-traceability.puml -o /home/claude/.rmodp/
```

### Markdown ファイルの保存

`create_file` ツールを使用して `/home/claude/.rmodp/correspondence.md` に保存する。

Markdown には図について以下の形式で埋め込むこと：

```markdown
### Traceability Map
*（全 Viewpoint 間の対応関係を視覚化）*

![Traceability Map](correspondence-traceability.png)

```plantuml
{PlantUML ソースコード}
```
```

## 完了後の成果物提供

全 Viewpoint が揃った時点で以下を実行する：

```bash
# システム名を確定（例: library-system）
PROJECT="{システム名}-rmodp"

# outputs へコピー
cp -r /home/claude/.rmodp /mnt/user-data/outputs/${PROJECT}

# ZIP 作成
cd /mnt/user-data/outputs
zip -r ${PROJECT}.zip ${PROJECT}/
```

`present_files` ツールで ZIP ファイルと各 Viewpoint ファイルを提示する。
