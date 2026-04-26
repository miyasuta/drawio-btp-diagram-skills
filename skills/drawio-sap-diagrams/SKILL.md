---
name: drawio-sap-diagrams
description: Use when creating, modifying, or refining SAP BTP architecture diagrams in draw.io / drawio format. Triggers on requests like "describe this landscape", "draw a BTP architecture", "make a draw.io of <SAP services>", or any task producing diagram.drawio / .drawio files for SAP cloud solutions.
---

# drawio-sap-diagrams

Claude と協働して SAP BTP アーキテクチャ図を draw.io で作成するためのスキル。人間は言葉で設計を伝え、Claude が Mermaid → draw.io XML に落とし込む。

> **会話トーン**: この SKILL.md 内で「人間」「ユーザー」と書かれているのは役割分担の説明表記。実際に会話でユーザーに何かを依頼するときは、主語を省いた二人称の丁寧命令形を使う（例: ❌「人間が XML を draw.io に貼り付けてください」→ ✅「XML を draw.io に貼り付けてください」）。

## Overview

言葉 → Mermaid → draw.io XML の 3 段論法で進める。

- **なぜ Mermaid を間に挟むか**: draw.io XML は冗長で人間がレビューしづらい。Mermaid で論理構造（ノード・エッジ・グルーピング）だけを先に確定させることで、draw.io XML 生成時はスタイルに集中できる。
- **XML は手貼り**: draw.io への反映は人間がコピペで行う。ツール経由で直接描画するより速い。

---

## Workflow

```
1. 人間: 設計を言葉で記述
      ↓
2. Claude: Mermaid 図を作成して論理構造を可視化（diagram.md）
      ↓
3. 人間: Mermaid プレビューで論理構造を確認・修正指示
      ↓
4a. Claude: 要素棚卸しパス — セル一覧（ID/サイズ/親子/エッジ）をテキストで作成
      ↓
4b-1. Claude: 論理座標パス — 主軸変数（main_axis_Y / GAP 等）と相対式で各要素位置を定義
      ↓
4b-2. Claude: 物理座標パス — 主軸の値を 1 箇所で確定し、式を評価して XML に書き出す
      ↓
4.5 Claude: sap-diagram-review（構造レビュー）— XML ベースで自動チェック
      ↑↓ 修正発生時はここを往復
5. 人間: XML を draw.io に手動で貼り付け
      ↓
5.5 Claude: sap-diagram-review（ビジュアルレビュー）— スクリーンショットで見た目チェック
      ↑↓ 修正発生時は 4b-1 / 4b-2 まで戻り、再度 4.5 → 5 → 5.5 を回す
```

**修正ループのルール**: ステップ5 / 5.5 で問題が見つかったら、勝手に Edit せずに **必ず 4b-1（論理座標パス）まで戻る**。何が変わるか（主軸変数 / 相対式）を先に整理してから 4b-2 で値を再計算し、Step 4.5 を再実行する。これにより部分修正で全体整合が崩れる事故を防ぐ。

---

## Output Location

出力先は利用プロジェクトの `CLAUDE.md` の指示に従う。未指定の場合は以下をデフォルトとして提案する:

```
diagrams/
├── diagram.md          ← Mermaid（論理構造、ステップ2-3 の成果物）
└── diagram.drawio      ← draw.io XML（ステップ4 の成果物、手貼り用）
```

複数の図を保持したい場合は、プロジェクトの `CLAUDE.md` で日付ベースのサブディレクトリなどにカスタマイズする。

---

## Step 1: 設計を言葉で伝える

人間が Claude に以下を明示的に伝える。

- **構成要素のリスト**（サービス名・役割）
- **階層・所属関係**（何が何の中に入るか）
- **関係性**（データフロー、trust、認証委譲など）
- **SAP / 非SAP の区別**（色分けに影響）

曖昧な箇所があれば Claude が番号付きで聞き返す。

---

## Step 2-3: Mermaid で論理構造を確認

Claude は Mermaid を書く前に、後述の **Conventions** 節を必ず確認し、構成パターン（End User + Application Clients のセット配置、オンプレ接続の 3 点セット、CIS の配置場所など）・ラベル規約・言語ルールに従う。

出力先は `diagram.md`。人間は VSCode の Mermaid プレビューで確認し、間違いがあれば言葉で修正指示する。

**注意**: Mermaid ではスタイル（色・線種）まで決めない。論理構造だけに集中する。

---

## Conventions（Mermaid 段階から適用）

ダイアグラムの構成要素をどう組み合わせるか、どう配置するか、ラベルや言語をどうするかの **設計上の規約**。Mermaid（論理構造）の段階から適用する。draw.io の見た目（色・サイズ・座標の px 値）は `references/layout.md` / `references/elements.md` を参照。

### 1. 言語

特に指示がない限り、ダイアグラム上の要素名は **全て英語** とする。

### 2. 矢印ラベル

矢印にラベルは **原則つけない**。以下のように特別な意味を持つ関係のみラベルを設定する:

| ラベル | 用途 |
|---|---|
| `trust` | Subaccount ↔ Cloud Identity Services 間の信頼関係 |
| `authenticate` | 認証フロー |
| `delegates auth` | 認証委譲（CIS ↔ 3rd-party IdP） |

### 3. データフローは一直線に並べる

データフローやシーケンスで連続する要素は、**横方向でも縦方向でも一直線に揃える**。折れ線や段差を入れると視覚的な流れが断たれる。

- **横並びの例**: `Application Clients` → `SAP Build Work Zone` → `SAP Task Center` は同じ高さに揃える
- **縦並びの例**: 上から下に流れるシーケンスは同じ列に揃える

> draw.io での実装詳細（直線 vs orthogonal の使い分け、中心座標の px 揃え）は `references/layout.md` を参照。

### 4. End User → Application Clients は常にセットで配置する

End User と **Application Clients**（Generic Icon）を **縦に並べて** セットで配置し、BTP 内サービスへの入口とする。End User が上、Application Clients が下。

- End User → Application Clients → BTP 内サービス（Build Work Zone 等）
- **認証線は Application Clients から出す**（End User からではない）

### 5. SAP Cloud Identity Services は Subaccount の下に配置する

SAP Cloud Identity Services (CIS) は BTP コンテナ内・Subaccount 外の要素。配置は **Subaccount の直下（垂直下方）** に固定する。

- Subaccount ↔ CIS の trust 線は短い垂直線になる
- 左右や上に置くと他の要素と交差しやすく、trust 線の意味が読み取りにくくなる

> draw.io での実装詳細（px サイズ・余白・BTP コンテナ高さの計算）は `references/layout.md` を参照。

### 6. サービスボックスからの矢印は1本にまとめる

サービスボックス（Build Work Zone 等）の中に複数のコンポーネント（アプリ、Inbox 等）がある場合、**個々のコンポーネントから外部サービスへ矢印を引かず、外側のボックスから 1 本の矢印** として出す。

- **理由**: 複数の矢印が1つのアイコンに集中すると見た目が煩雑になる
- Mermaid でも `UI5 App → SPA` ではなく `BWZ → SPA` とする

#### 6a. サービスボックスから複数ターゲットへ出すときは「根っこ」を1点に揃える

サービスボックスが複数の異なるターゲット（例: HTML5 App Repository, XSUAA 等）へ矢印を出す場合、**ボックスの同じ 1 点（例: 上辺中央 `exitX=0.5;exitY=0`）から発射** し、そこから orthogonal ルーティングで各ターゲットへ分岐させる。

- **理由**: 矢印の出発点がボックスの複数箇所に散らばると、視覚的にどこから出ているのか読みにくくなる
- 共通の exit 点を指定すれば、draw.io の orthogonal ルータが最初の 1 セグメントを共有してくれるため、縦棒 1 本から枝分かれする見た目になる
- 直線矢印ではなく orthogonal 矢印を使う（共通の根から異なるターゲットへ向かう以上、直線にはなり得ない）

### 7. サービスボックス内のアプリケーション配置

Build Work Zone 等のサービスボックス内に **複数のアプリケーション** を配置する場合、**内側にコンテナ（箱）を追加** し、その中に Generic Icon プレースホルダ + テキストラベルを横並びで配置する。アプリが **1 つだけ** の場合は内側コンテナは不要で、サービスボックス内に直接 Generic Icon プレースホルダを配置する。

```
┌─ BWZ ─────────────────────────┐
│ 🔵 SAP Build Work Zone        │
│ ┌───────────────────────────┐ │
│ │  [icon]      [icon]       │ │
│ │  App Name    My Inbox     │ │
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

- 内側コンテナは青枠・白地・角丸（サービスボックスと同様のスタイル）
- 各アプリは Generic Icon プレースホルダ（40×40）+ 下部テキストラベルで構成
- アプリ同士は横並びに等間隔で配置

> draw.io での実装詳細（px サイズ・スタイル）は `references/elements.md` を参照。

### 8. オンプレミス／Private Cloud 接続パターン

BTP Subaccount からオンプレミスまたは Private Cloud システムへ接続する場合、以下の **3 サービスを 1 セットで Subaccount 内に配置する**:

```
Destination Service → Connectivity Service → Cloud Connector
```

- **Destination Service**: 接続先の設定管理
- **Connectivity Service**: オンプレミスへのトンネル管理
- **Cloud Connector**: 接続先システムの箱の中に配置する（BTP 外）

Cloud Connector は BTP 側のコンポーネントだが、図上は **接続先システムの箱の中** に描く（物理的にオンプレミス環境にインストールされるため）。

### 9. BTP 外の SAP 製品ボックスには SAP ロゴを追加する

BTP 外に配置する SAP 製品（SAP S/4HANA、SAP SuccessFactors 等）の矩形には、**BTP コンテナと同じパターンで SAP ロゴ + 製品名を左上に配置する**。

- 矩形の `value` は空にし、ロゴセル（`image=img/lib/sap/SAP_Logo.svg`）で製品名を表示する
- SAP 製品であることを視覚的に明示するため

> draw.io での実装詳細（フォントサイズ・配置位置）は `references/elements.md` の「SAP ロゴ」セクションを参照。

### 10. 上向き矢印は原則避ける（ラベル配置との関係）

SAP サービスアイコン（`mxgraph.sap.icon`）のラベルはデフォルトでアイコン下側に描画される。そのため **下から入る矢印（上向き矢印）はラベルを貫く見た目になる**。

**原則**: アイコンへの矢印は **横または上から入る** ように配置する（=上向き矢印を使わない）。ソース側の位置を工夫して、矢印方向を下向きまたは横向きに保つ。

- **例**: BWZ から HTML5 App Repo / XSUAA のような従属サービスへ出す場合、従属サービスを **BWZ の下** に配置する（上に配置すると上向き矢印になりラベルと衝突する）
- **理由**: 上向き矢印に合わせてラベル位置をアイコン上側に切り替えると、親コンテナ（Subaccount 等）のラベルとの距離が詰まり視認性が落ちる。また図全体でラベル位置が上下混在すると統一感が崩れる

**例外**: どうしてもレイアウトの都合で上向き矢印を避けられない場合のみ、そのアイコンのラベルを上側に切り替える（`verticalLabelPosition=top;verticalAlign=bottom`）。その場合は親コンテナのラベルとの間に **最低 20 px** の余白を確保する。

> draw.io での実装詳細（ラベル位置変種のスタイル差分）は `references/elements.md` の「ラベル位置の変種」セクションを参照。

### 11. サービス名の慣例的な短縮

ダイアグラムのラベルには、公式フルネームではなく **慣例的に使われる短縮形** を優先する。長いフルネームをそのまま入れると水平方向が無駄に広がるため、改行（`elements.md` のラベル改行ルール）の前にまず「短くできないか」を検討する。

既知の短縮形:

| 公式名 | ダイアグラム上のラベル |
|---|---|
| SAP Destination service | `Destination` |

**一般原則**:
- 識別に不要な `service` / `Services` サフィックスは省略してよい（ただし `SAP Cloud Identity Services` のようにブランドの一部となっているものは残す）
- 文脈で曖昧さが生じない場合は `SAP` プレフィックスも省略可（ただし `SAP HANA` のように他と区別する必要がある場合は残す）
- 迷ったら残す側（正式名）を選び、水平方向を圧迫するときに短縮を検討する

新しい短縮形を採用した場合は、本テーブルに追記する。

---

## Step 4: draw.io XML 生成（3 パス構成）

> **必須**: 4a に入る前に、`references/layout.md`、`references/elements.md`、`references/coordinate-passes.md` を **必ず Read する**。記憶 / 推測で書くと、過去の iteration で更新されたサイズ・座標規則・SAPIcon 名が反映されない。判断に迷ったら `references/guidelines.md` も参照。

Conventions はステップ2-3 で既に適用済み。ここでは draw.io 固有の実装（px サイズ・座標・エッジ接続点・スタイル値）に集中する。

### 3 パス概要

| パス | 役割 | 書くもの |
|---|---|---|
| **4a 棚卸し** | 構造ミスの早期発見 | 座標なしのセル一覧（ID / 種別 / 親子 / エッジ） |
| **4b-1 論理座標** | 修正に強い位置定義 | 主軸変数 + 相対式（`BWZ.x = SUB_LEFT + 35` 等） |
| **4b-2 物理座標** | px 値を確定して XML 出力 | 変数評価結果 + `<mxCell>` |

各パスの手順・主軸変数の典型例・式の書き方・修正ループのルールは [`references/coordinate-passes.md`](references/coordinate-passes.md) に集約。Step 4 を実行する前に必ず読む。

### Step 4 事前準備（スタイル未定義の洗い出し）

3 パスに入る前に、以下を一度だけ済ませる。

1. **必要要素の棚卸し**: その図で使うサービスアイコン名・ボックス種別・新規のレイアウトパターンを列挙する
2. **references に存在するか確認**: `references/elements.md` の SAPIcon 表、`references/layout.md` の標準サイズ表、`references/guidelines.md` を突き合わせる
3. **不明点を一括でユーザーに確認**: 不明な SAPIcon 名や未定義パターンを **全部まとめて** 聞く（遭遇するたびに聞くと会話が細切れになる）

---

## 不明なスタイル・アイコンに遭遇したとき

`references/` に定義されていない要素（未登録の `SAPIcon` 名、新種の矩形、初めて使う色 / 線種 など）に遭遇したら、**推測で描かずユーザーに確認する**。確認の粒度:

- **SAP サービスアイコン**: `SAPIcon=...` の正式名が不明なときは、シェイプ名を推測せずユーザーに聞く（誤った名前だと draw.io 側で空表示になる）
- **スタイル仕様**: 色・角丸・線種・フォント等で実装層に定義がない要素は、`references/guidelines.md` に該当セクションがあればそれを引用し、無ければユーザーに確認する
- **レイアウト判断**: 新しい配置パターン（スタックコンテナ、interface 記号など）は勝手に採用しない

確認の結果、**再利用可能なルールだと判断したら適切なファイルに追記** する:

- **構成パターン・配置ルール** → この SKILL.md の Conventions 節
- **draw.io スタイル・サイズ** → `references/layout.md` または `references/elements.md`

公式由来のルールなら `references/guidelines.md` の該当セクションへの相互リンクも一緒に足す。これにより references が使用と共に育っていき、次回以降は再質問が不要になる。

---

## Claude / 人間の分担

| 内容 | 担当 |
|---|---|
| SAP サービスアイコン（`mxgraph.sap.icon`）の配置 | **Claude** |
| Generic Icon（`shape=image` + base64 SVG）の配置 | **人間**（Claude は空プレースホルダを置くだけ。サイズは `references/layout.md` 参照） |
| コンテナ・線・テキスト・スタイル全般 | **Claude** |

**なぜ Generic Icon のみ人間担当か**: base64 エンコードされた SVG を 1 アイコンあたり数 KB、style 属性に埋め込む必要があり、Claude が XML 内で扱うとトークン消費・応答時間が膨れ上がる。SAP サービスアイコンは `SAPIcon=<名前>` の短い識別子で済むため問題ない。

会話の冒頭で「Generic Icon は人間が後から配置するので、SAP サービスアイコンのみ XML に記述してください」と伝えておくと誤解がない。

### 指示の Tips

- **アイコンを触らないでほしい時**: 「アイコンの差し替えはこちらで行うので、それ以外の修正のみ適用してください」
- **埋め込みラベルのスタイルだけ変えたい時**: 「コンテナ埋め込みの状態で、文字のスタイルのみ変更することはできませんか？」（別セル化を防ぐ）
- **新しいルールを永続化したい時**: 「`references/layout.md`（または `elements.md`）に追加してください」と明示的に指示する

---

## Step 4.5: 構造レビュー（自動実行）

ステップ4 で `diagram.drawio` を生成した直後、Claude は **sap-diagram-review スキル（Phase 1: 構造レビュー）** を自動実行する。Conventions と `references/` の現在のルールに照らして XML をチェックし、問題があれば報告する。問題がなければ「レビュー OK」と表示してステップ5 に進む。

## Step 5: draw.io に貼り付け + スクリーンショット共有

生成された `diagram.drawio` の内容を draw.io にペーストし、**スクリーンショットを会話に貼り付ける**。

## Step 5.5: ビジュアルレビュー（自動実行）

人間が共有したスクリーンショットを元に、Claude は **sap-diagram-review スキル（Phase 2: ビジュアルレビュー）** を実行する。ラベルのはみ出し・重なり、コンテナ余白、矢印の直線性などを画像から確認し、問題があれば修正案とともに報告する。問題がなければ「ビジュアルレビュー OK」と表示する。

問題があれば言葉で修正指示に戻る。

---

## References

| ファイル | 役割 | 読むタイミング |
|---|---|---|
| この SKILL.md（Conventions 節） | 設計規約（構成パターン・ラベル・言語） | **ステップ2-3**（Mermaid） |
| `references/layout.md` | draw.io 実装詳細（px サイズ・座標・エッジ接続点） | **ステップ4**（draw.io XML） |
| `references/elements.md` | draw.io 要素スタイル定義（BTP, ロゴ, アイコン, 線） | **ステップ4**（draw.io XML） |
| `references/guidelines.md` | SAP 公式ガイドラインの日本語要約（source of truth） | 判断に迷ったとき |

Step 4 に入る前に、`references/layout.md` と `references/elements.md` は **必ず Read する**。読まずに記憶や推測で XML を書くと、スタイル値や座標規則が古いまま反映されるリスクがある。

---

## Common Pitfalls

| 失敗 | 原因 | 対策 |
|---|---|---|
| ステップ2 で Mermaid にスタイル情報を入れてしまう | 論理構造と見た目を同時に決めようとした | Mermaid は論理構造のみ、スタイルはステップ4 で |
| ボックスサイズ変更のたびに座標計算をやり直す | 棚卸しと座標計算を同時にやった | ステップ4 を 4a（棚卸し）/ 4b（座標計算）に分ける |
| Claude が独自判断でスタイルを変えてしまう | `references/` 参照を明示しなかった | ステップ4 の冒頭で「`references/` のルールに従って」と明示する |
| ラベルが別セルで作られてしまう | Claude が「スタイル変更 = 新規セル作成」と解釈 | 「コンテナ埋め込みのまま文字スタイルだけ変えて」と指示 |
| Claude が Generic Icon の base64 を触ってしまう | 役割分担を伝えていない | ステップ4 の冒頭で「Generic Icon は人間が配置する」と明示 |
| Claude が新しく決めたルールを忘れる | 会話内だけで合意して永続化しなかった | 「`references/layout.md`（または `elements.md`）に追加してください」と指示する |
| アイコン名の確認で会話が細切れになる | XML 生成中に未登録アイコンに遭遇するたびユーザーに聞いた | ステップ4 冒頭で必要アイコンを列挙し、未登録分を **一括で** 確認する |
| Claude が座標計算に時間をかけすぎる | ピクセル単位で完璧に合わせようとした | 最終調整は人間の仕事。Claude は直線矢印の両端と所属関係だけ厳密にし、他は粗で出す |
