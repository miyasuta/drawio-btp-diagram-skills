# drawio-btp-diagram-skills

Claude Code / Claude Desktop 用の Skill バンドル。SAP BTP アーキテクチャ図を draw.io で描く **5 ステップの協働ワークフロー** と、生成された XML / スクリーンショットの自動レビューを提供する。

## 含まれるスキル

| スキル | 役割 | トリガー |
|---|---|---|
| `drawio-sap-diagrams` | 言葉 → Mermaid → draw.io XML の 5 ステップ協働フロー | SAP BTP アーキテクチャ図の作成依頼 |
| `diagram-review` | 生成された draw.io XML / スクリーンショットのレビュー | Step 4 で XML 生成完了 / 人間がスクリーンショットを共有 |

2 スキルは独立しており、`diagram-review` は他のワークフローからも単独で呼び出せる。

## インストール（symlink 推奨）

```bash
git clone https://github.com/<user>/drawio-btp-diagram-skills.git ~/projects/claude-drawio-skill/drawio-btp-diagram-skills
ln -s ~/projects/claude-drawio-skill/drawio-btp-diagram-skills/skills/drawio-sap-diagrams ~/.claude/skills/drawio-sap-diagrams
ln -s ~/projects/claude-drawio-skill/drawio-btp-diagram-skills/skills/diagram-review     ~/.claude/skills/diagram-review
```

symlink なら `git pull` が即反映される。`cp -r` は上書き忘れのリスクがあるので非推奨。

### 確認

```bash
ls -la ~/.claude/skills/drawio-sap-diagrams/       # symlink 解決
ls ~/.claude/skills/drawio-sap-diagrams/references/  # layout / elements / guidelines の 3 ファイル
```

新規 Claude Code セッション開始時のスキル一覧に `drawio-sap-diagrams` と `diagram-review` が並ぶはず。

## 利用プロジェクトの CLAUDE.md テンプレート

プロジェクト固有の出力先ルールは **各プロジェクトの `CLAUDE.md`** に書く。スキル側にはプロジェクト固有パスを持たせない。

```markdown
## ダイアグラム作成

SAP BTP アーキテクチャ図は `drawio-sap-diagrams` スキルで作成する。

出力先:
- `diagrams/<yyyy-mm-dd>-<nn>/diagram.md` （Mermaid）
- `diagrams/<yyyy-mm-dd>-<nn>/diagram.drawio` （draw.io XML）

連番は当日既存の max + 1。既存図の修正は元ディレクトリを編集する。
```

CLAUDE.md に出力先を指定していない場合、スキルはデフォルト（上記と同じパターン）を提案する。

## リポジトリ構成

```
drawio-btp-diagram-skills/
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    ├── drawio-sap-diagrams/
    │   ├── SKILL.md          # 5-step workflow + 9 conventions (inline)
    │   └── references/
    │       ├── layout.md     # draw.io 実装詳細（px サイズ・座標・エッジ接続点）
    │       ├── elements.md   # draw.io 要素スタイル定義（BTP, ロゴ, アイコン, 線）
    │       └── guidelines.md # SAP 公式ガイドライン日本語要約（source of truth）
    └── diagram-review/
        └── SKILL.md          # 構造レビュー + ビジュアルレビューの 2 フェーズ
```

## （任意）draw.io MCP サーバーの併用

Chrome に [Draw.io MCP Extension](https://chromewebstore.google.com/detail/drawio-mcp-extension/okdbbjbbccdhhfaefmcmekalmmdjjide) をインストールして MCP サーバーを立ち上げれば、Claude から draw.io を直接編集できる。ただし本スキルは **XML を手貼りする** 前提で設計されている（ツール呼び出し回数が少なく速いため）。MCP が活きるのは細かい修正フェーズ（`edit-cell` / `delete-cell-by-id`）であり、初稿生成では不要。

導入は利用プロジェクト側の裁量。スキル本体には MCP 依存はない。

## ライセンス

[MIT](LICENSE)
