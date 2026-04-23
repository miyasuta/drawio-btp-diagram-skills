---
name: drawio-sap-diagrams
description: Use when designing SAP BTP architecture diagrams with draw.io — guides a 5-step human/Claude collaboration (verbal design → Mermaid → draw.io XML) following SAP official visual guidelines
---

# drawio-sap-diagrams

Claude と協働して SAP BTP アーキテクチャ図を draw.io で作成するためのスキル。人間は言葉で設計を伝え、Claude が Mermaid → draw.io XML に落とし込む。

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
4b. Claude: 座標計算パス — 棚卸しに基づき座標を計算し diagram.drawio に XML を出力
      ↓
4.5 Claude: diagram-review（構造レビュー）— XML ベースで自動チェック
      ↓
5. 人間: XML を draw.io に手動で貼り付け
      ↓
5.5 Claude: diagram-review（ビジュアルレビュー）— スクリーンショットで見た目チェック
```

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

### 10. 矢印方向とラベル配置の対応

SAP サービスアイコン（`mxgraph.sap.icon`）のラベルが矢印と重ならないよう、**矢印の入力方向に応じてラベル位置を切り替える**:

| 矢印の入力方向 | ラベル位置 |
|---|---|
| 横から入る / 上から下へ入る | **下**（デフォルト） |
| 下から上へ入る（矢印の矢先がアイコン下辺） | **上** |

- **理由**: デフォルトの下ラベルは下から入る矢印と同じ空間に描かれ、矢印がラベル文字を貫く見た目になる
- 横向きの矢印はアイコン中心（左右端）に入るため、ラベルが下でも干渉しない
- 上下左右でラベル位置を混在させても図全体の一貫性は損なわれない（矢印方向とペアで判断する）

> draw.io での実装詳細（スタイル差分）は `references/elements.md` の「ラベル位置の変種」セクションを参照。

---

## Step 4: draw.io XML 生成（2 パス構成）

ステップ4 では `references/layout.md` + `references/elements.md`（判断に迷ったら `references/guidelines.md`）を読んでから着手する。Conventions はステップ2-3 で既に適用済みのため、ここでは draw.io 固有の実装（px サイズ・座標・エッジ接続点・スタイル値）に集中する。

**なぜ 2 パスに分けるか**: 座標とセル構造を同時に書くと、後からサイズ変更した際に連鎖的に座標計算をやり直すことになり時間がかかる。先にセルの棚卸し（ID・サイズ・親子・エッジ）だけを確定させ、構造を固めてから機械的に座標を計算する方が速い。4a の時点で構造ミスを発見できれば、XML を書き直さず済む。

### Step 4 事前準備（スタイル未定義の洗い出し）

両パスに入る前に、以下を一度だけ済ませる。

1. **必要要素の棚卸し**: その図で使うサービスアイコン名・ボックス種別・新規のレイアウトパターンを列挙する
2. **references に存在するか確認**: `references/elements.md` の SAPIcon 表、`references/layout.md` の標準サイズ表、`references/guidelines.md` を突き合わせる
3. **不明点を一括でユーザーに確認**: 不明な SAPIcon 名や未定義パターンを **全部まとめて** 聞く（遭遇するたびに聞くと会話が細切れになる）

### Step 4a: 要素棚卸しパス（座標を書かない）

XML を書き始める前に、**座標なし** で全セルをテキストで棚卸しする。サイズは `references/layout.md` の標準値を引用するだけ（この時点で計算はしない）。

棚卸し項目（箇条書きで出力）:

- **コンテナ・ボックス**: ID / 種別（BTP, Subaccount, サービスボックス等）/ サイズ（標準値）/ 親コンテナ
- **アイコン・ラベル**: ID / 種別（SAPIcon 名 / Generic Icon プレースホルダ）/ 親コンテナ / ラベル文字列
- **エッジ**: ID / source → target / スタイル種別（直線 / orthogonal / trust / authenticate 等）

この時点でユーザーに共有してもよい。構造ミス（親子関係の誤り、エッジの欠落、ボックス種別の選択ミス）はここで気付けば修正コストが小さい。

### Step 4b: 座標計算パス

4a の棚卸しを入力として、機械的に座標を決めて XML を生成する。計算順序:

1. **メインフロー軸の決定**: 横並びなら中心 Y、縦並びなら中心 X を 1 つ固定する
2. **コンテナ境界の計算**: 親コンテナのサイズから子要素の配置領域を算出
3. **分岐（trust / authenticate 等）の配置**: メインフローから外れるサービスの位置を決める
4. **エッジの接続点**: 直線で結ぶエッジは両端のセル中心座標が一致するよう揃える（`references/layout.md` §4）

### 座標の粒度

Claude が XML を書く際、座標は粗く決めて良い。**最終的な位置微調整は人間が draw.io 上で行う**。Claude が厳密に合わせるべきなのは以下だけ:

- **直線矢印で結ぶ要素の中心座標**（中心が 1 px でもずれると orthogonal に折れ曲がる、`references/layout.md` §4 参照）
- **コンテナの内外関係**（BTP の中 / Subaccount の中 / ボックスの中、の所属）

それ以外（コンテナの幅・高さ、余白、矢印のベンド点など）はおおよそで良い。ピクセル単位の計算に時間をかけるより、粗い XML を早く出してユーザーのフィードバックを受ける方が速い。

### スタイルの source of truth

- **`references/guidelines.md` = SAP 公式ガイドラインの日本語要約 = source of truth**
- **`references/layout.md` / `references/elements.md` = guidelines を draw.io XML として具体化した実装層**
- 実装層と公式が食い違う場合は、原則として実装層を公式に合わせて修正する。Claude は食い違いを見つけても勝手に直さず、**ユーザーに報告してから対応** する（過去の iteration で意図的にずらしている可能性があるため）。

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

ステップ4 で `diagram.drawio` を生成した直後、Claude は **diagram-review スキル（Phase 1: 構造レビュー）** を自動実行する。Conventions と `references/` の現在のルールに照らして XML をチェックし、問題があれば報告する。問題がなければ「レビュー OK」と表示してステップ5 に進む。

## Step 5: draw.io に貼り付け + スクリーンショット共有

生成された `diagram.drawio` の内容を draw.io にペーストし、**スクリーンショットを会話に貼り付ける**。

## Step 5.5: ビジュアルレビュー（自動実行）

人間が共有したスクリーンショットを元に、Claude は **diagram-review スキル（Phase 2: ビジュアルレビュー）** を実行する。ラベルのはみ出し・重なり、コンテナ余白、矢印の直線性などを画像から確認し、問題があれば修正案とともに報告する。問題がなければ「ビジュアルレビュー OK」と表示する。

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
