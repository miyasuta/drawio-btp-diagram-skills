# レイアウトガイド

draw.io XML を生成するときの**実装レベルの詳細**（px サイズ・座標計算・エッジ接続点・余白）を定義する。**Claude はステップ4（draw.io XML 生成）の際にこのファイルを読む**。

論理的な配置ルール（何をどこに置くか、どう組み合わせるか）は [`../rules/conventions.md`](../rules/conventions.md) を参照。個別要素のスタイル定義（BTP コンテナ、アイコンなど）は [`elements.md`](elements.md) を参照。

このファイルは SAP 公式ガイドラインを draw.io で具体化した実装層。source of truth は [`../GUIDELINES.md`](../GUIDELINES.md)。公式と食い違いを見つけたら `styles/` 側を公式に合わせる（変更前にユーザーに確認）。

---

## 1. 直線 vs orthogonal の使い分け

> 論理原則「データフローは一直線に並べる」→ [`../rules/conventions.md` §3](../rules/conventions.md#3-データフローは一直線に並べる)

一直線に並んだ要素間の接続は**直線（まっすぐな矢印）**で描く。曲がる必要がないのに orthogonal を使うと不自然なベンドが入るため避ける。一直線に並ばない要素間のみ **orthogonal（折れ曲がった直線）** を使う。

> 関連: 公式 Foundation の Spacing ルール「オブジェクト間は均等に、SAPロゴの高さ程度」 → [`../GUIDELINES.md` の Spacing セクション](../GUIDELINES.md#スペーシングspacing)

---

## 2. コンテナの無駄な余白を作らない

Subaccount や BTP コンテナのサイズは、内部要素を配置したうえで**実際に必要な大きさに詰める**。アイコン + ラベルだけのサービスは意外に小さいので、旧サイズのまま大きな箱を残すと余白が目立つ。

**ルール**:
- 子要素の配置が決まったら、親コンテナの width / height を見直す
- 縦横両方向で、不要な余白（特に下部・右側）がないか確認する
- 子要素を削除・縮小した後は、必ず親コンテナも縮小する
- **子要素が 1 つだけの場合、水平方向はコンテナの中心に揃える**（`子要素 x = (コンテナ幅 - 子要素幅) / 2`）。垂直方向はラベル領域を除いた利用可能エリアの中心に揃える
- **同レベルのサービスボックスはサイズを揃える**: 横並びの場合は高さを一致させる。縦並びの場合は幅を一致させる。内部要素の量が異なっても、視覚的な統一感を優先する

**コンテナ内の推奨余白**:
- 左右: 各 30〜55 px
- 上: 40 px（ラベルが入る場合は下記参照）
- 下: 40〜55 px
- 子要素同士の水平間隔: 100 px 前後（アイコン左端から次のアイコン左端まで 170 px が目安）

**ラベル付きコンテナの上部余白**:

ラベルの種類によって、子要素の開始 Y 位置（コンテナ基準）が変わる:

| ラベル種別 | 例 | 子要素の開始 Y |
|---|---|---|
| SAP ロゴ + タイトル（別セル） | BTP コンテナ | **90〜110 px** |
| 埋め込みテキストラベル | Subaccount | **50〜60 px** |
| サービスアイコン + タイトル（別セル） | BWZ サービスボックス | **50〜60 px** |

埋め込みテキストラベル（`spacingTop=10` + `fontSize=12`）の場合、ラベル下端は約 y=28。子要素を y=50 以降に配置することで、ラベルとの間に **20 px 以上の隙間**を確保する。

**SAP サービスアイコンのラベルはみ出しに注意**:

SAP サービスアイコン（`verticalLabelPosition=bottom`）のラベルはアイコンの geometry（50×50）の**外側**に描画される。コンテナサイズの計算時はアイコン底辺ではなく**ラベル底辺**を基準にする。

- **下方向**: 2 行ラベル（`fontSize=12`）はアイコン底辺から約 **30〜35 px** 下に伸びる。コンテナの下部余白はラベル底辺から 20 px 以上確保する
- **左右方向**: ラベルはアイコン中心に対して左右均等に配置される。ラベル幅がアイコン幅（50 px）を超えると左右にはみ出す。コンテナ端に近いアイコンは**ラベル幅の半分**以上の側面余白を確保する

**実測値の参考**（2 サービスを横並びに配置した Subaccount）:
- Subaccount: 330×210
- 内部の SAP サービスアイコン 2 個（56×56）が同じ Y で横並び
- BTP 全体（Subaccount + その下に CIS コンテナ 280×150）: 410×460

---

## 3. 標準サイズ

> 関連: 公式 Icons ルール（SAP BTP Service Icon はグレー背景円付きバージョン必須 / Generic Icon はニュートラルグレー） → [`../GUIDELINES.md` の Icons セクション](../GUIDELINES.md#iconsアイコン)。公式テキストには具体 px 仕様が無いため、下記の標準サイズはプロジェクト独自の運用値。

| 要素 | 標準サイズ | 備考 |
|---|---|---|
| SAP サービスアイコン（`mxgraph.sap.icon`、単独配置） | **50×50** | Subaccount 直下などで単独配置する場合 |
| SAP サービスアイコン（ボックス内左上の識別用） | **32×32** | サービスボックス（BWZ・CF 等）の左上にアイコン+タイトルとして置く場合。詳細は [`elements.md` の「サービスアイコン付きサービスボックス」](elements.md#サービスアイコン付きサービスボックス) |
| Generic Icon（`shape=image` + base64 SVG） | **40×40（正方形）** | 既知のアスペクト比があるアイコンは下表を使う。未知のアイコンは正方形で仮配置し、人間が後で調整する |
| SAP On-Premise / Cloud Solutions 矩形 | 120×60 | |
| 3rd-party Identity Provider 矩形 | 120×60 | |

**Generic Icon の既知アスペクト比**:

| アイコン | アスペクト比 (w:h) | プレースホルダサイズ (幅=40 基準) |
|---|---|---|
| Application Clients | 28:23 | **40×33** |

上記に該当しないアイコンは 40×40（正方形）で仮配置する。

**Generic Icon のプレースホルダ配置**:
- Claude は Generic Icon の base64 データを扱わない（WORKFLOW.md 参照）
- Claude は **40×40 の正方形の空矩形**（`value=""`、`dashed=1` の破線枠）を placeholder として仮配置する
- **ラベルはプレースホルダの子要素（`parent="<placeholder_id>"`）として配置する**。これにより、draw.io 上でプレースホルダを移動するとラベルも一緒に動く（SAP サービスアイコンと同じ操作感）。ラベルの `value` はプレースホルダではなく別セルに持たせるため、人間が SVG 画像に差し替えてもラベルは消えない
- ラベル text セルの例: `text;html=1;align=center;verticalAlign=middle;resizable=0;points=[];autosize=0;strokeColor=none;fillColor=none;fontSize=11;fontColor=default;fontFamily=Helvetica;` を `parent="<placeholder_id>"` で配置。座標はプレースホルダ基準の**相対座標**で、プレースホルダ底辺から 4〜6 px 下（例: `y="44"`）、ラベル幅がプレースホルダより広い場合は負の `x` でセンタリングする（例: ラベル幅 90、プレースホルダ幅 40 → `x="-25"`）
- **縦方向の矢印はラベルから出す**: プレースホルダとラベルが縦に並んでいる場合、縦方向に伸びる矢印の source は**ラベル text セル**にする（[§6 参照](#6-エッジの接続点を明示的に指定する)）。ラベルが子要素であっても、エッジの source/target として指定できる
- アスペクト比が既知のアイコンは上記の表に従ったサイズで仮配置する。未知のアイコンは正方形で仮配置し、人間が差し替え時にサイズ調整する
- 重要なのは**位置を先に確定させる**こと。アイコン差し替え後に人間が大きさを微調整しても、中心座標さえ合っていればレイアウトは崩れない

---

## 4. 接続要素の軸揃え（直線矢印を成立させる）

矢印で繋がる一連の要素は、**中心座標（X または Y）を完全に一致**させる。中心が 1 px でもずれると、draw.io は自動的に orthogonal に折れ曲げてしまう。

**横並びの実例**（同じ Y 中心 = 210）:
- `ac`: y=190, height=40 → 中心 Y = 210
- `bwz`: y=182, height=56 → 中心 Y = 210
- `tc`: y=182, height=56 → 中心 Y = 210

**縦並びの実例**（同じ X 中心 = 160）:
- `eu`: x=140, width=40 → 中心 X = 160
- `ac`: x=140, width=40 → 中心 X = 160

異なるサイズの要素を一直線に揃えるには、`y = center − height/2`（または X も同様）を計算して配置する。

> 関連: 公式 Lines / Connectors の方向とセマンティック色のルール → [`../GUIDELINES.md` の Lines / Connectors セクション](../GUIDELINES.md#lines--connectors線コネクタ)

---

## 5. CIS 配置の実装詳細

> 論理原則「CIS は Subaccount の下に配置する」→ [`../rules/conventions.md` §5](../rules/conventions.md#5-sap-cloud-identity-services-は-subaccount-の下に配置する)

**実装ルール**:
- CIS アイコン（50×50）を Subaccount の下辺中央付近、BTP コンテナ内の下段に置く
- Subaccount ↔ CIS の trust 線は短い垂直線になる（Subaccount 下辺中央 → CIS 上辺中央、ピンク双方向）
- BTP コンテナの高さは CIS の**ラベル底辺**まで含めた下部余白で確保する

**BTP コンテナ高さの計算式**:

```
BTP 高さ = (SA の y + SA の height) − BTP の y   ← SA 下端までの距離
         + 30                                     ← SA〜CIS 間のギャップ
         + 50                                     ← CIS アイコン高さ
         + 30                                     ← ラベル高さ（2行 × fontSize=12 の推定値）
         + 20                                     ← 下部余白
```

**実測値の参考**（2026-04-17-01 ダイアグラム）:
- BTP: y=30, SA: y=80, h=300 → SA 下端=380
- CIS: y=410（ギャップ 30px）、アイコン底辺=460、ラベル底辺 ≈ 490
- BTP 高さ = (380−30) + 30 + 50 + 30 + 20 = **480** → BTP 下端=510

---

## 6. エッジの接続点を明示的に指定する

矢印の source / target だけを指定すると、draw.io が自動でルーティングを決めるため、コンテナの位置調整で経路が崩れることがある。重要な矢印には **`entryX` / `entryY`** を明示的に指定する。

| 目的 | 指定 |
|---|---|
| 相手の上辺中央に刺す | `entryX=0.5;entryY=0;entryDx=0;entryDy=0` |
| 相手の下辺中央に刺す | `entryX=0.5;entryY=1;entryDx=0;entryDy=0` |
| 相手の左辺中央に刺す | `entryX=0;entryY=0.5;entryDx=0;entryDy=0` |
| 相手の右辺中央に刺す | `entryX=1;entryY=0.5;entryDx=0;entryDy=0` |

**使用例**:
- TC → On-Premise / Cloud Solutions: `entryX=0;entryY=0.5`（相手の左辺中央）
- Subaccount ↔ CIS (trust): `exitX=0.5;exitY=1` → `entryX=0.5;entryY=0`（SA 下辺中央 → CIS 上辺中央、垂直直線）
- Application Clients → CIS (authenticate): source は AC の**ラベル text セル**（プレースホルダではない）。`exitX=0.5;exitY=1` → `entryX=0;entryY=0.5`（ラベル下辺中央 → CIS 左辺中央、orthogonal）
- CIS → 3rd-party IdP (delegates auth): `entryX=0;entryY=0.5`

**縦方向の矢印はラベルから出す**: Generic Icon プレースホルダ（図形）とラベル（text セル）が縦に並んでいる場合、縦方向に伸びる矢印の source は**ラベル text セル**にする（図形ではない）。図形から出すとラベルを飛び越えた不自然な矢印になる。

- End User → Application Clients（縦配置）: source=`eu_label`（End User のラベル）
- Application Clients → CIS (authenticate): source=`ac_label`

source 側も同様に `exitX` / `exitY` で指定可能。複数のエッジが同じ辺から出る場合は分散させる。

---

## MCP / 実装上の制約

- `parent_id` でコンテナの子要素にはできない（drawio-mcp-server の制約）→ 絶対座標で重ねる
- `import-diagram` は動作が遅い・タイムアウトする場合あり → `add-rectangle` / `edit-cell` を使う、または XML を人間が手貼り（WORKFLOW.md 参照）
- ロゴサイズを 37×19 より大きくすると `imageAspect=0` の影響で横伸びする
