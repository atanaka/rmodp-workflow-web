# インストール手順 / Installation Guide

[English](#english) | [日本語](#japanese)

---

<a name="japanese"></a>
## 日本語

### 前提条件

- Claude.ai アカウント（Pro / Team / Enterprise プラン）
- Claude.ai のスキル機能が有効になっていること

### インストール手順

#### 1. リポジトリをクローン（またはダウンロード）

```bash
git clone https://github.com/atanaka/rmodp-workflow-web.git
cd rmodp-workflow-web
```

#### 2. スキルのアップロード

Claude.ai のスキル設定画面を開き、以下の 7 つのスキルをそれぞれアップロードします。
各スキルは `skills/` 配下のディレクトリ単位でアップロードしてください。

| # | スキル名 | ディレクトリ | 役割 |
|---|---|---|---|
| 1 | `rmodp-workflow-web` | `skills/rmodp-workflow-web/` | オーケストレーター（必須） |
| 2 | `rmodp-enterprise-view-web` | `skills/rmodp-enterprise-view-web/` | Enterprise Viewpoint |
| 3 | `rmodp-information-view-web` | `skills/rmodp-information-view-web/` | Information Viewpoint |
| 4 | `rmodp-computational-view-web` | `skills/rmodp-computational-view-web/` | Computational Viewpoint |
| 5 | `rmodp-engineering-view-web` | `skills/rmodp-engineering-view-web/` | Engineering Viewpoint |
| 6 | `rmodp-technology-view-web` | `skills/rmodp-technology-view-web/` | Technology Viewpoint |
| 7 | `rmodp-correspondence-web` | `skills/rmodp-correspondence-web/` | Correspondence |

> **重要**: 7つすべてのスキルをアップロードしてください。
> オーケストレーター（`rmodp-workflow-web`）は実行時に他のスキルの SKILL.md を
> `view` ツールで動的に読み込むため、すべてが必要です。

#### 3. 動作確認

スキルアップロード後、Claude.ai の新しい会話で以下を入力します：

```
rmodp-workflow-web テスト
```

初期化フェーズが起動し、業務概要の入力を求めるプロンプトが表示されれば成功です。

### アップロード時の注意事項

- スキル名はディレクトリ名（`rmodp-workflow-web` など）と一致させてください
- `rmodp-workflow-web` の `assets/` フォルダ（進捗テンプレート）も必ず含めてください
- アップロードサイズ制限がある場合は、各スキルを個別にアップロードしてください

---

<a name="english"></a>
## English

### Prerequisites

- Claude.ai account (Pro, Team, or Enterprise plan)
- Claude.ai Skills feature enabled on your account

### Installation Steps

#### 1. Clone (or download) the repository

```bash
git clone https://github.com/atanaka/rmodp-workflow-web.git
cd rmodp-workflow-web
```

#### 2. Upload the skills

Open Claude.ai's Skills settings and upload each of the following 7 skills.
Upload each skill as a directory unit from the `skills/` folder.

| # | Skill Name | Directory | Role |
|---|---|---|---|
| 1 | `rmodp-workflow-web` | `skills/rmodp-workflow-web/` | Orchestrator (required) |
| 2 | `rmodp-enterprise-view-web` | `skills/rmodp-enterprise-view-web/` | Enterprise Viewpoint |
| 3 | `rmodp-information-view-web` | `skills/rmodp-information-view-web/` | Information Viewpoint |
| 4 | `rmodp-computational-view-web` | `skills/rmodp-computational-view-web/` | Computational Viewpoint |
| 5 | `rmodp-engineering-view-web` | `skills/rmodp-engineering-view-web/` | Engineering Viewpoint |
| 6 | `rmodp-technology-view-web` | `skills/rmodp-technology-view-web/` | Technology Viewpoint |
| 7 | `rmodp-correspondence-web` | `skills/rmodp-correspondence-web/` | Correspondence |

> **Important**: All 7 skills must be uploaded.
> The orchestrator (`rmodp-workflow-web`) dynamically loads other skills' SKILL.md files
> at runtime using the `view` tool, so all skills are required.

#### 3. Verify installation

After uploading, start a new conversation in Claude.ai and type:

```
rmodp-workflow-web test
```

If the initialization phase starts and prompts for a business overview, the installation is successful.

### Upload Notes

- Match the skill name exactly to the directory name (e.g., `rmodp-workflow-web`)
- Always include the `assets/` folder of `rmodp-workflow-web` (progress templates)
- If there are upload size limits, upload each skill individually
