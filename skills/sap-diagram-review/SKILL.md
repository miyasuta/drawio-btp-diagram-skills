---
name: sap-diagram-review
description: Use after generating or editing a draw.io diagram of SAP BTP architecture, or when the user shares a diagram screenshot — checks XML structure (Phase 1) and rendered visuals (Phase 2) against SAP visual conventions and the drawio-sap-diagrams skill's references.
---

# SAP Diagram Review

## Overview

draw.io XML のレビューを **2 フェーズ** で行う。

1. **構造レビュー（Step 4.5）**: XML 生成直後に自動実行。ドキュメントに照らして構造・スタイルをチェック
2. **ビジュアルレビュー（Step 5.5）**: 人間が draw.io に貼り付けた後、スクリーンショットで見た目を確認

## When to Use

- **Step 4.5**: XML 生成完了直後に自動実行する
- **Step 5.5**: 人間がスクリーンショットを提供したときに自動実行する
- 人間が明示的に「レビューして」と言った場合にも実行可能

### 再実行が必要なケース（MUST）

既存の `diagram.drawio` に対して以下の変更を加えた後は、**構造レビュー（Phase 1）を必ず再実行する**:

- 要素の**追加・削除**（アイコン、ボックス、エッジ等）
- 要素の**移動**（コンテナを跨ぐ、または位置関係が大きく変わる — 例: 上 → 下）
- コンテナの**サイズ変更**（幅・高さ）
- 主フローの**中心軸**（Y または X）の変更

**理由**: 要素の位置やサイズが変わると、コンテナの余白・同レベルボックスのサイズ統一・座標整合などの検算結果が連動して変わる。差分が小さく見えても、関連する余白計算が全体で崩れる可能性があるため、全カテゴリを通した再評価が必要。

**過去の失敗例**: HTML5 App Repo / XSUAA をコンテナ上段から下段へ移動した際、主フローの Y 座標はそのまま据え置いたため、Subaccount 内の上部に 100 px 超の空白が発生した。このとき再レビューを走らせていれば、`layout.md §2`「無駄な余白を作らない」「縦横両方向で、不要な余白がないか確認」のチェックで即座に検出できた。

---

## Phase 1: 構造レビュー（Step 4.5）

### Process

1. 対象の `diagram.drawio` を読む
2. 対応する `diagram.md`（Mermaid）を読む
3. 以下のドキュメントを読み、現在のルールを取得する:
   - `drawio-sap-diagrams` スキルの Conventions 節
   - `../drawio-sap-diagrams/references/layout.md`
   - `../drawio-sap-diagrams/references/elements.md`
4. カテゴリごとにチェックし、問題を収集する
5. 問題があれば一覧で報告、なければ「レビュー OK」

### Review Categories

各カテゴリはドキュメント参照で判定する。固定チェックリストではなく、**ドキュメントの現在の内容** がルールになる。

| カテゴリ | 参照先 | チェック観点 |
|---|---|---|
| 構成パターン | `drawio-sap-diagrams` Conventions | End User+AC セット配置、CIS 配置位置、オンプレ接続 3 点セット、サービスボックスからの矢印集約 |
| ラベル・言語 | `drawio-sap-diagrams` Conventions | 要素名が英語か、矢印ラベルが許可リスト内か |
| コンテナの塗り | `../drawio-sap-diagrams/references/elements.md` | L0-L3 の交互ルール（#EBF8FF / #ffffff）に従っているか |
| アイコン形式 | `../drawio-sap-diagrams/references/elements.md` + `layout.md` | SAP アイコンのサイズ・`html=1;`の有無、Generic Icon がプレースホルダ形式か（既知アスペクト比の適用含む）、ラベルが別セルか |
| ラベル改行 | `../drawio-sap-diagrams/references/elements.md` | 3 単語以上のラベルが改行されているか（SAP サービスアイコン: `&#10;` / サービスボックスのタイトル: `<br>`）。未改行だと水平間隔が無駄に広がる |
| ラベル位置 | `../drawio-sap-diagrams/references/elements.md`（ラベル位置の変種）+ `SKILL.md §10` | SAP サービスアイコンのラベルが**デフォルトの下**（`verticalLabelPosition=bottom`）で統一されているか。`top` に切り替えたアイコンがあれば、それが §10 の例外条件（下辺進入を避けられない等）を**本当に満たすか**を検証する。満たさない／代替手段（下ラベルのまま矢印をラベル下から出す・接続元を動かす等）がある場合は **🚩問題として報告**し、「意図的な逸脱（修正不要）」で素通りさせない。図内でラベル位置が上下混在していたら原則フラグ |
| エッジ進入 | `../drawio-sap-diagrams/references/layout.md` §1b | ボックスへの「L字進入」（横から来て下辺/上辺に刺す等、対向しない辺への直角進入）が無いか。あれば直線化（接続元の位置調整・同じ辺の異なる位置への複数直線）を提案する |
| Line jump | `../drawio-sap-diagrams/references/elements.md`（線/エッジ） | 全エッジのスタイルに `jumpStyle=gap;jumpSize=8;`（交差部の隙間）が入っているか。抜けているエッジがあれば追加を提案する |
| コンテナサイズ | `../drawio-sap-diagrams/references/layout.md` | ラベル付きコンテナの上部余白（テキストラベル: 50〜60px）、SAP アイコンのラベルはみ出し分を含めた下部・側面余白、子要素が 1 つの場合の中心揃え、同レベルのボックスの**サイズと整列**の統一（横並びなら高さ + 上端 Y、縦並びなら幅 + 左端 X。右端/下端だけ一致して反対側の端がずれていないか座標で確認） |
| タイトルはみ出し | `../drawio-sap-diagrams/references/elements.md` + `layout.md` | サービスボックスのタイトルテキストセル幅がタイトル文字列に対して十分か（fontSize=12 の太字で 1 文字あたり約 8px を目安に推定） |
| 座標整合 | `../drawio-sap-diagrams/references/layout.md` | 直線矢印で結ぶ要素の中心座標が一致しているか |
| エッジ接続点 | `../drawio-sap-diagrams/references/layout.md` | trust / authenticate 等の重要エッジで entryX/Y, exitX/Y が明示されているか |
| 必須要素 | `diagram.md` との突合 | Mermaid に定義された全ノード・エッジが XML に存在するか |

### Output Format

問題がある場合:

```
## Diagram Review (構造)

- [ ] **構成パターン**: Cloud Connector が S/4HANA の箱の外に配置されている（Conventions §8）
- [ ] **座標整合**: SPA と DEST の中心 Y が 2px ずれている（layout.md §4）

上記を修正しますか？
```

問題がない場合:

```
## Diagram Review (構造)

レビュー OK — draw.io に貼り付けてスクリーンショットを共有してください
```

---

## Phase 2: ビジュアルレビュー（Step 5.5）

### Trigger

人間が draw.io に XML を貼り付けた後、**スクリーンショットを会話に貼り付ける** ことでトリガーされる。Claude は画像を Read で読み込み、以下の観点でチェックする。

### チェック観点

| カテゴリ | チェック内容 |
|---|---|
| ラベルはみ出し | SAP サービスアイコンのラベルがコンテナ枠からはみ出していないか |
| タイトルはみ出し | サービスボックスのタイトル（アイコン横のテキスト）がボックス枠からはみ出していないか |
| ラベル重なり | 隣接するアイコンのラベル同士が重なっていないか。下向き矢印とその発射元ラベルがくっつきすぎていないか（ラベル文字と線に隙間があるか → `layout.md §6`） |
| ラベル位置の統一 | アイコンのラベルが下で統一されているか（一部だけ上になって上下混在していないか）。混在していれば §10 の例外妥当性を確認し、不要なら下に統一を提案 |
| エッジとコンテナ枠 | コンテナ境界を跨ぐエッジの**折れ点・ラベルがコンテナ枠線に重なって**いないか（`layout.md §1c`）。重なっていれば外部ターゲットを離す／経由点で折れ点を枠外のクリア帯へ移すことを提案 |
| エッジラベルの配置 | エッジラベルが**折れ点（コーナー）の上**に乗って線形状を隠していないか、**線から離れて浮き**所属不明になっていないか（`layout.md §1d`）。直線区間で線中央に乗せる／並走ラベルの高さ（Y）を揃えることを提案 |
| ボックスサイズ・整列の統一 | 同レベルのサービスボックスが、横並びなら高さ + 上端、縦並びなら幅 + 左端が揃っているか（サイズが合っていても端がずれて不揃いに見えないか） |
| コンテナ余白 | コンテナの上下左右に適切な余白があるか（詰まりすぎ・空きすぎ） |
| 矢印の直線性 | 直線であるべき矢印が折れ曲がっていないか。ボックスへの L字進入（横から来て下辺/上辺に刺す）が無いか |
| 要素の視認性 | テキストが読める大きさか、アイコンが潰れていないか |

### Process

1. スクリーンショット画像を確認する
2. 上記チェック観点で問題を洗い出す
3. 問題があれば具体的な指摘と修正案を報告、なければ「ビジュアルレビュー OK」

### Output Format

問題がある場合:

```
## Diagram Review (ビジュアル)

- [ ] **ラベルはみ出し**: Cloud Connector のラベルが S4 ボックス下端からはみ出している → S4 の高さを拡張
- [ ] **矢印の直線性**: trust 矢印が折れ曲がっている → Sub と CIS の中心 X がずれている

修正しますか？
```

問題がない場合:

```
## Diagram Review (ビジュアル)

ビジュアルレビュー OK
```
