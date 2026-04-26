# drawio-btp-diagram-skills

Claude Code / Claude Desktop 用の Skill バンドル。**言葉で伝えた設計を draw.io で描けるアーキテクチャ図に変換する** 協働ワークフローを提供します。

---

## 概要 / 何ができるか

SAP BTP を題材にアーキテクチャ図を描くとき、draw.io で直接書くのは時間がかかり、公式ガイドライン（色・サイズ・配置）を正しく当てはめるのも面倒です。このスキルは:

1. 言葉で設計を伝える
2. Claude が **Mermaid で論理構造** に起こす（`diagram.md`）
3. 合意が取れたら Claude が **draw.io XML** を生成する（`diagram.drawio`）
4. **構造レビュー**（XML のチェック）+ **ビジュアルレビュー**（スクショのチェック）が自動で走る

という 3 段論法で、レビューしやすい順に詰めていきます。draw.io XML を直接読むのはレビューがつらいので、Mermaid で論理を固めてから XML に落とすのがポイントです。

### 含まれる 2 つのスキル

| スキル | 役割 |
|---|---|
| `drawio-sap-diagrams` | 5 ステップ協働ワークフロー（言葉 → Mermaid → XML） |
| `sap-diagram-review` | 生成された XML / スクリーンショットの自動レビュー |

`sap-diagram-review` は独立トリガー（XML 生成完了 / スクショ共有）を持つため、他のワークフローからも単独で呼び出せます。

### 対象ユーザー

- SAP BTP でアプリやソリューションを組み立てているアーキテクト・開発者
- 公式 SAP ビジュアルガイドラインに沿った図を書きたい人

---

## Getting Started

### 前提

- Claude Code がインストール済み
- `git` / `ln` が使える（macOS / Linux）

### 1. clone + symlink

```bash
git clone https://github.com/<user>/drawio-btp-diagram-skills.git ~/projects/claude-drawio-skill/drawio-btp-diagram-skills
ln -s ~/projects/claude-drawio-skill/drawio-btp-diagram-skills/skills/drawio-sap-diagrams ~/.claude/skills/drawio-sap-diagrams
ln -s ~/projects/claude-drawio-skill/drawio-btp-diagram-skills/skills/sap-diagram-review     ~/.claude/skills/sap-diagram-review
```

symlink にしておくと `git pull` の結果が即反映されます（`cp -r` は更新漏れの原因になるため非推奨）。

確認:

```bash
ls -la ~/.claude/skills/drawio-sap-diagrams/        # symlink 解決
ls ~/.claude/skills/drawio-sap-diagrams/references/ # layout / elements / guidelines
```

### 2. Claude Code を再起動

セッション開始時のスキル一覧に `drawio-sap-diagrams` と `sap-diagram-review` が並んでいれば準備完了。

### 3. 最初の 1 図を描く

適当なプロジェクトで試してみてください:

```
このプロジェクト（CAP + UI5）のアーキテクチャ図を描いてください。
- UI は Work Zone からアクセスさせる
- CAP は S/4HANA (Private Cloud) につながっている
```

Claude が不足情報を番号付きで聞き返してくるので、番号で返答すると Mermaid → draw.io XML の順に生成されます。XML を draw.io にペーストしてスクリーンショットを会話に貼ると、ビジュアルレビューも自動で走ります。

詳しい流れや他のプロンプト例は次の [使い方](#使い方) を参照。

---

## 使い方

### サンプルプロンプト（推奨: 曖昧な状態から始める）

プロジェクトに実装がある場合、細かい構造は Claude がコードを読めば把握できるので、**高レベルの関係だけ** 伝えれば十分です:

```
このプロジェクト（CAP + UI5）のアーキテクチャ図を描いてください。
- UI は Work Zone からアクセスさせる
- CAP は S/4HANA (Private Cloud) につながっている
```

情報が足りないと Claude は **番号付きで聞き返してきます**。番号で簡潔に返答すれば、合意がとれたところから Mermaid → 棚卸し → draw.io XML の生成に進みます。

```
1. はい / 2. CIS 経由、3rd-party なし / 3. はい / 4. 不要
```

何を聞き返してくるかは依頼内容と Claude のプロジェクト把握度次第です（上記はあくまで返答スタイルの例）。

### ランドスケープが確定している場合

既に構成が決まっているなら、構造化リストで一気に渡すと往復が減ります:

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

含めると精度が上がる情報:

- 構成要素のリスト（サービス名・役割）
- 階層・所属関係（何が何の中に入るか）
- 関係性（データフロー、trust、認証委譲など）
- SAP / 非SAP の区別（色分けに影響）

### ワークフローの流れ

プロンプトを送ると、スキルは以下の順に進みます:

1. **Mermaid 生成** → `diagram.md` に論理構造を出力。Mermaid プレビューで確認し、違えば言葉で修正指示
2. **要素棚卸し**（座標なしのセル一覧をテキストで提示）→ 構造ミスがあればここで修正
3. **draw.io XML 生成** → `diagram.drawio`
4. **構造レビュー**（`sap-diagram-review` が自動実行）
5. draw.io に貼り付けて、スクリーンショットを会話に添付
6. **ビジュアルレビュー**（`sap-diagram-review` が自動実行）

### よく使う追加指示

- 「アイコンの差し替えはこちらで行うので、それ以外の修正のみ適用してください」— Generic Icon を触らせたくないとき
- 「コンテナ埋め込みの状態で、文字のスタイルのみ変更してください」— ラベルが別セル化されるのを防ぐ
- 「この新しいルールを `references/layout.md` に追加してください」— 会話中の合意を永続化する（次節 [スキルの育て方](#スキルの育て方) を参照）

---

## スキルの育て方

このスキルは **使いながら育つ** よう設計されています。出荷時の `references/` に載っていない要素や、自分の好み・運用ルールに気付いたとき、それを会話で伝えるだけで Claude が適切なファイルに永続化してくれます。

### 育て方の例

| きっかけ | 永続化される場所 |
|---|---|
| 未登録の SAP サービスアイコン名を確認した | `references/elements.md` の SAPIcon 表 |
| 新しいスタイル・色・サイズ（公式に載っているがスキル側に未定義） | `references/elements.md` または `references/layout.md` |
| 「このパターンは原則こう配置して欲しい」というレイアウト方針 | `SKILL.md` の Conventions 節 |
| 「このサービス群はいつもセットで描く」という設計ルール | `SKILL.md` の Conventions 節 |
| 「上向き矢印は避けたい」など、自分の図の好み | `SKILL.md` の Conventions 節 |

Claude 自身が「これは再利用できそう」と判断したときは追記を提案してくるので、承認すれば書き込まれます。明示的に指示するのもありです:

```
このレイアウト方針は今後ずっと使いたいので、SKILL.md の Conventions に追記してください。
```

### symlink 経由なので即反映

symlink でインストールしているため、Claude が `references/` や `SKILL.md` を編集すると **リポジトリ本体のファイルが直接書き換わります**。次のセッションから新しいルールが効きます。

### 自動コミット（強く推奨）

skill リポが symlink されていると、**利用プロジェクト側のセッション** で更新が走っても、そのセッションで `git commit` はされません。放置すると未コミット変更が溜まります。

`~/.claude/settings.json` に Stop フックを仕込むと、**セッション終了時に** skill リポの未コミット変更を自動でコミットしてくれます:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "SKILL_REPO=$HOME/projects/claude-drawio-skill/drawio-btp-diagram-skills; if [ -d \"$SKILL_REPO/.git\" ] && [ -n \"$(git -C \"$SKILL_REPO\" status --porcelain 2>/dev/null)\" ]; then git -C \"$SKILL_REPO\" add -A && FILES=$(git -C \"$SKILL_REPO\" diff --cached --name-only | paste -sd , - | sed 's/,/, /g') && STAT=$(git -C \"$SKILL_REPO\" diff --cached --shortstat | sed 's/^ *//') && git -C \"$SKILL_REPO\" commit -m \"Auto-update $FILES ($STAT)\"; fi || true"
          }
        ]
      }
    ]
  }
}
```

挙動:

- skill リポに未コミット変更がないセッション → 何もしない
- 変更があるセッション → `Auto-update skills/drawio-sap-diagrams/references/elements.md (1 file changed, 1 insertion(+))` のような差分情報入りメッセージでコミット
- 利用プロジェクト側のリポジトリには触れない（`git -C "$SKILL_REPO"` で scope 固定）

設定を書いたら Claude Code を再起動するか、`/hooks` を一度開くと反映されます。

### チームで共有する

1 人で育てたルールをチームに展開するには、`drawio-btp-diagram-skills` リポジトリを GitHub に push するだけです。他のメンバーは同じ clone + symlink 手順で入れ、`git pull` で最新を取り込めます。

---

## プロジェクトごとのカスタマイズ

プロジェクト固有の出力先ルールは **各プロジェクトの `CLAUDE.md`** に書きます。スキル側にはプロジェクト固有パスを持たせません。

### デフォルト

CLAUDE.md に出力先を指定していない場合、スキルは以下をデフォルトとして提案します:

```
diagrams/diagram.md       # Mermaid（論理構造）
diagrams/diagram.drawio   # draw.io XML（貼り付け用）
```

### 日付別に複数図を残したいとき

CLAUDE.md にこう書きます:

```markdown
## ダイアグラム作成

SAP BTP アーキテクチャ図は `drawio-sap-diagrams` スキルで作成する。

出力先:
- `diagrams/<yyyy-mm-dd>-<nn>/diagram.md` （Mermaid）
- `diagrams/<yyyy-mm-dd>-<nn>/diagram.drawio` （draw.io XML）

連番は当日既存の max + 1。既存図の修正は元ディレクトリを編集する。
```

---

## リポジトリ構成

```
drawio-btp-diagram-skills/
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    ├── drawio-sap-diagrams/
    │   ├── SKILL.md          # 5-step workflow + Conventions (inline)
    │   └── references/
    │       ├── layout.md     # draw.io 実装詳細（px サイズ・座標・エッジ接続点）
    │       ├── elements.md   # draw.io 要素スタイル定義（BTP, ロゴ, アイコン, 線）
    │       └── guidelines.md # SAP 公式ガイドライン日本語要約（source of truth）
    └── sap-diagram-review/
        └── SKILL.md          # 構造レビュー + ビジュアルレビューの 2 フェーズ
```

---

## ライセンス

[MIT](LICENSE)
