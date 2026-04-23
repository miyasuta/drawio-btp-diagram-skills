# SAP 公式 BTP Solution Diagram ガイドライン（要約）

このドキュメントは SAP 公式ガイドライン [SAP/btp-solution-diagrams](https://github.com/SAP/btp-solution-diagrams) の主要ルールを日本語で要約し、オフラインで参照できるようにしたもの。

## このドキュメントの位置付け

- **SAP 公式ガイドライン（このファイル）= source of truth**
- **[`styles/`](styles/) = 公式ガイドラインを draw.io XML として具体化した実装層**
- `styles/` と公式が食い違う場合、原則として `styles/` 側を公式に合わせて修正する（変更の前にユーザーに確認する）

公式原文は英語で、数値仕様の多くは画像（PNG）や draw.io ライブラリに含まれている。本ファイルはテキストで読み取れるルールのみを拾っており、詳細（Text の4階層サイズ、Numbers の具体スタイル、Icon の厳密サイズ等）は原文画像と draw.io ライブラリを参照する。

原文リポジトリ: https://github.com/SAP/btp-solution-diagrams

---

## Foundation（基礎要素）

原文: [`guideline/docs/btp_guideline/foundation.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/foundation.md)

### 色（Colors）

Horizon は SAP プロダクトの既定ビジュアルスタイル。"Its color balance helps to draw the user's attention to the essential information and functions."（配色バランスがユーザーの注意を本質的な情報と機能に向ける）

#### Primary Colors

| 用途 | Border | Fill |
|---|---|---|
| **SAP / BTP Area** | `#0070F2` | `#EBF8FF` |
| **Non-SAP Area** | `#475E75` | `#F5F6F7` |

| 用途 | 色 |
|---|---|
| **Text — Title** | `#1D2D3E` |
| **Text — Body** | `#556B82` |

#### Semantic Colors

| 意味 | Border | Fill |
|---|---|---|
| **Positive** | `#188918` | `#F5FAE5` |
| **Critical** | `#C35500` | `#FFF8D6` |
| **Negative** | `#D20A0A` | `#FFEAF4` |

#### Accent / Emphasized Colors

| 名称 | Border | Fill |
|---|---|---|
| **Teal** | `#07838F` | `#DAFDF5` |
| **Indigo** | `#5D36FF` | `#F1ECFF` |
| **Pink** | `#CC00DC` | `#FFF0FA` |

### 線種（Line Styles）

原文: *"a solid arrow means direct communication, while a dashed arrow implies indirect communication."*

推奨される BTP Solution Diagram 用の線種：

| 線種 | 用途 |
|---|---|
| **Solid（実線）** | direct, synchronous request-response data flows（直接・同期のリクエスト-レスポンス） |
| **Dashed（破線）** | indirect, asynchronous data flows（間接・非同期のデータフロー） |
| **Dotted（点線）** | optional data flows（オプションのフロー） |
| **Thick（太線）** | firewalls only（ファイアウォール専用） |

### スペーシング（Spacing）

原文: *"Spacing around objects should be even and roughly the height of the SAP Logo."*

オブジェクト間の余白は均等に、SAP ロゴの高さ程度を目安にする。具体数値は画像（spacing.png）参照。

---

## Diagram Components

原文ディレクトリ: [`guideline/docs/btp_guideline/diagr_comp/`](https://github.com/SAP/btp-solution-diagrams/tree/main/guideline/docs/btp_guideline/diagr_comp)

### Areas（矩形・エリア）

原文: [`diagr_comp/areas.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/areas.md)

- **角丸半径**: *"A fixed corner radius of 16 pixels is recommended"* — **16 px 固定**
- **色**: Foundation の Primary Colors に準拠（SAP=青、非SAP=グレー）
- **Accent 色の使用**: 図全体を圧倒しないよう使いすぎない
- **ネスティング**: *"alternate between using a fill and not using a fill"* — エリアを入れ子にする際、塗りの有無を交互にしてコントラストを確保する
- **カーディナリティ**: 同種要素のグルーピングはスタック表現で、スタイルの一貫性を保つ
- **階層**: 親レイヤは BTP（青）スタイル、子要素は塗りパターンを交互に

### Icons（アイコン）

原文: [`diagr_comp/icons.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/icons.md)

2 種類のアイコンセットがある。

#### SAP BTP Service Icons

- SAP Discovery Center に掲載されている全サービス用のアイコン
- **必須ルール**: *"For diagram visualization it is mandatory to use the version with grey background circle."* — 図解表現では**グレー背景円付きバージョンの使用が必須**
- draw.io ライブラリが GitHub リポジトリで提供されている

#### Generic Icons

- デバイス・データベースなどの汎用要素、または専用視覚表現がないコンテンツ用
- *"The primary color used is neutral grey."* — 主要色はニュートラルグレー
- ソフトグラデーション付きアイコンセット
- draw.io ライブラリが GitHub リポジトリで提供されている

### Lines / Connectors（線・コネクタ）

原文: [`diagr_comp/lines_connectors.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/lines_connectors.md)

定義: *"Connectors are non-, one- or bidirectional arrows."*

#### 方向（Data Flow）

| 種類 | 意味 |
|---|---|
| **One-directional（単方向）** | *"typically represent a request-response from client to server"* — クライアント→サーバーのリクエスト-レスポンス |
| **Bidirectional solid arrows（双方向実線矢印）** | *"may represent mutual trust"* — 相互信頼（mutual trust）を表す |

#### セマンティック色の対応

再発するフロー種別には標準化されたセマンティック色を使う：

| フロー種別 | 色 |
|---|---|
| **Trust flows（信頼）** | Pink（`#CC00DC`） |
| **Authentication flows（認証）** | Green（`#188918`） |
| **Authorization flows（認可）** | Indigo（`#5D36FF`） |

非セマンティックなパスやフローの強調には Accent 色を使う。

#### ファイアウォール / ネットワーク境界

太いグレー線（thick grey lines）で表現する。

#### インターフェース

専用の視覚表現が draw.io ライブラリで提供されている（詳細は原文の interfaces 画像参照）。

### Numbers（番号付け）

原文: [`diagr_comp/numbers.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/numbers.md)

- *"Numbers can be used to describe specific paths in diagrams."* — 図内の特定のパスを説明するために番号を使える
- 具体的なスタイル（サイズ・色・フォント）は原文の画像（numbers.png）および draw.io ライブラリ参照
- 原文テキストには数値仕様の記載なし

### Product Names（製品名表記）

原文: [`diagr_comp/product_names.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/product_names.md)

- *"SAP product names must be paired with the SAP logo"* — **SAP 製品名は SAP ロゴとペアで表示する**
- *"It is not recommended to use too many SAP logos in the same diagram"* — 同一図内で多数の SAP ロゴを使うのは非推奨
- ロゴの過剰使用を避けるため、**テキストのみの要素**を代替として使う

### Text（テキスト）

原文: [`diagr_comp/text.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/text.md)

- *"To create hierarchies four text styles were derived from Fiori Horizon."* — 階層構造を作るため、Fiori Horizon から派生した **4 つのテキストスタイル**がある
- *"Texts defined in this document may need to be scaled up to fit the specifications of the target medium."* — テキストは対象媒体に合わせてスケールする必要がある
- 4 階層の具体的なフォントサイズ・色・字体は画像（text_styles.png, text_element_hierachy.png）および PowerPoint / draw.io ライブラリ参照
- 原文テキストには数値仕様の記載なし

---

## 公式原文に当たるときの Tips

- 具体数値が必要なとき（番号スタイル、Text 階層、アイコン px 等）は、まず原文ページの画像を参照、次に GitHub リポジトリの [`assets/shape-libraries-and-editable-presets/draw.io`](https://github.com/SAP/btp-solution-diagrams/tree/main/assets/shape-libraries-and-editable-presets/draw.io) 配下の draw.io ライブラリ XML を参照する
- 本ファイルは原文テキストから拾えたルールのみを収録している。画像のみに記載されているルールは別途原文で確認すること
- 公式のルールと `styles/` の実装に食い違いを発見したら、勝手に修正せずユーザーに報告してから対応する
