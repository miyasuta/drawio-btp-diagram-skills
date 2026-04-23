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
- `diagrams/diagram.md` （Mermaid）
- `diagrams/diagram.drawio` （draw.io XML）
```

CLAUDE.md に出力先を指定していない場合、スキルはこのデフォルトを提案する。

複数の図を残したい場合は、`diagrams/<yyyy-mm-dd>-<nn>/` のような日付別サブディレクトリに分けるルールを CLAUDE.md に書く（連番は当日既存の max + 1、既存図の修正は元ディレクトリを編集）。

## 使い方

### サンプルプロンプト（推奨: 曖昧な状態から始める）

プロジェクトに実装がある場合、細かい構造は Claude がコードを読めば把握できるので、**高レベルの関係だけ** 伝えれば十分:

```
このプロジェクト（CAP + UI5）のアーキテクチャ図を描いてください。
- UI は Work Zone からアクセスさせる
- CAP は S/4HANA (Private Cloud) につながっている
```

Claude は `package.json` / `mta.yaml` / `xs-app.json` / `package.json > cds` 等を読んでプロジェクト構造を把握し、足りない情報を **番号付きで聞き返す**:

```
以下を確認させてください:
1. End User の接続元は Application Clients（ブラウザ）経由でよいですか？
2. 認証は Cloud Identity Services 経由でしょうか？ 3rd-party IdP 連携はありますか？
3. S/4HANA への接続は Destination Service + Connectivity Service + Cloud Connector の標準パターンでよいですか？
4. Work Zone の中に置くアプリは UI5 アプリ 1 つだけですか？ My Inbox 等は含めますか？
5. Task Center は使いますか？
```

番号で簡潔に返答すれば Mermaid → 棚卸し → draw.io XML の生成に進む:

```
1. はい（ブラウザのみ）/ 2. CIS 経由、3rd-party なし / 3. はい / 4. UI5 1 つだけ / 5. 不要
```

### ランドスケープが確定している場合

既に構成が決まっているなら、構造化リストで一気に渡すと往復が減る:

````markdown
以下のランドスケープをダイアグラム化してください。

**BTP 内**:
- Subaccount (BTP 内)
  - SAP Build Work Zone
  - SAP Task Center
- SAP Cloud Identity Services (BTP 内、Subaccount 外)

**BTP 外**:
- End User (非SAP)
- Application Clients: Mobile or Desktop (非SAP)
- 3rd-party Identity Provider (非SAP)
- SAP On-Premise Solutions (SAP)
- SAP Cloud Solutions (SAP)

**関係**:
- End User → Application Clients → Build Work Zone → Task Center → On-Prem / Cloud Solutions
- Subaccount ↔ Cloud Identity Services (trust)
- Cloud Identity Services ↔ 3rd-party IdP (delegates auth)
````

**含めると精度が上がる情報**:

- 構成要素のリスト（サービス名・役割）
- 階層・所属関係（何が何の中に入るか）
- 関係性（データフロー、trust、認証委譲など）
- SAP / 非SAP の区別（色分けに影響）

### ワークフローの流れ

プロンプトを送ると、スキルは以下の順で進む（詳細は `skills/drawio-sap-diagrams/SKILL.md`）。

1. **Mermaid 生成** → `diagram.md` に論理構造を出力。VSCode の Mermaid プレビューで確認し、違えば言葉で修正指示
2. **要素棚卸し**（座標なしのセル一覧をテキストで提示）→ 構造ミスがあればここで修正
3. **draw.io XML 生成** → `diagram.drawio`
4. **構造レビュー**（`diagram-review` スキルが自動実行）
5. 人間が draw.io に貼り付けてスクリーンショットを会話に添付
6. **ビジュアルレビュー**（`diagram-review` スキルが自動実行）

### よく使う追加指示

- 「アイコンの差し替えはこちらで行うので、それ以外の修正のみ適用してください」— Generic Icon を触らせたくないとき
- 「コンテナ埋め込みの状態で、文字のスタイルのみ変更してください」— ラベルが別セル化されるのを防ぐ
- 「この新しいルールを `references/layout.md` に追加してください」— 会話中の合意を永続化する

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
