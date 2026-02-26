---
name: draw-swimlane
description: |
  業務フローをswimlane（水泳レーン）形式でビジュアライズし、draw.ioファイルとして出力するスキル。
  ユーザーが業務フロー、ワークフロー、プロセスフロー、業務手順、業務設計などを図にしたい場合に必ず使用すること。
  「フローを図にしたい」「ワークフローを可視化したい」「業務の流れを整理したい」「swimlaneで描いて」「draw.ioで出力して」などの要求でトリガーすること。
---

# Workflow Swimlane ビジュアライザー

業務フローの情報を受け取り、ヒアリングを経てdraw.io形式のswimlaneダイアグラムを生成する。

---

## フェーズ0：インプット受け取り

最初に、ユーザーがどの形式で情報を提供するかを確認する。

### インプットの2パターン

**パターンA：対話形式**
ユーザーがテキストで業務フローを説明してくる場合。そのままフェーズ1に進む。

**パターンB：ファイル指定**
ユーザーが「このファイルを読んで」「〇〇.txt を元に作って」のようにファイルを指定してくる場合。
- 指定されたファイルをすべて読み込む
- 対応形式：テキスト（.txt / .md）、Excel（.xlsx / .csv）、Word（.docx）、PDF（.pdf）など
- 複数ファイル指定も可（例：設計書＋業務定義書の組み合わせ）
- 読み込んだ内容を元にフェーズ1の分析を行う

**インプット置き場（任意）：**
ユーザーがファイルパスを明示しない場合、以下のディレクトリを確認し、ファイルがあれば自動で読み込む：
```
.claude/skills/draw-swimlane/inputs/
```

ユーザーから明示的な指定がなければ「対話形式で進めますか？それともファイルを指定しますか？」と聞く。

---

## フェーズ1：ワークフロー分析 & ヒアリング

インプット（会話またはファイル内容）から以下を抽出・分析する：

- **登場人物/役割**：誰が（部署・役職・システム）関わっているか
- **プロセスの流れ**：タスクの順序と依存関係
- **判断分岐**：条件分岐（Yes/Noの判断）がどこにあるか
- **システム連携**：外部システムとのやり取り

分析後、**不足している情報だけ**を質問する。すでに読み取れた情報は再確認しない。

```
【基本情報】
- このフローのタイトル・目的は？
- 対象とするスコープ（開始〜終了のイベント）は？

【プロセス情報】
- 主要なタスク・ステップをすべて教えてください
- タスクの順序・依存関係は？
- 並行して行われる作業はありますか？

【スイムレーン設定】
- 関係する役割・部署・システムは？（各スイムレーンになる）
- 各タスクの担当者・システムは？

【特殊要件】
- 判断分岐（承認/却下、条件分岐）はどこにありますか？
- タイムアウトや例外処理はありますか？

【付加情報】
- 重要なマイルストーンや特記事項は？
```

不明点をリスト形式で提示し、ユーザーの回答を待つ。

---

## フェーズ2：Draw.io XML 生成

ヒアリング完了後、以下の設計原則に従ってdraw.io XMLを生成する。

### 設計原則

- **流れの方向**：左から右（水平方向）
- **グリッド準拠**：10px単位でスナップ
- **スイムレーン**：役割・部署・システムごとにレーンを分割
- **色分け**：役割ごとに異なる背景色を使用
- **矢印スタイル**：`orthogonalEdgeStyle`（縦横線のみ）
- **フォント**：タイトル20pt以上、本文12pt以上

### 図形の種類

| 要素 | 図形 | XML スタイル |
|------|------|-------------|
| 開始/終了 | 楕円 | `ellipse;whiteSpace=wrap;` |
| タスク/作業 | 角丸矩形 | `rounded=1;whiteSpace=wrap;` |
| 判断分岐 | ひし形 | `rhombus;whiteSpace=wrap;` |
| 外部システム | 矩形（点線枠） | `dashed=1;whiteSpace=wrap;` |

### 色パレット（スイムレーンごと）

| レーン番号 | 背景色 | ストローク色 |
|-----------|--------|------------|
| 1 | `#dae8fc` | `#6c8ebf` |
| 2 | `#d5e8d4` | `#82b366` |
| 3 | `#fff2cc` | `#d6b656` |
| 4 | `#f8cecc` | `#b85450` |
| 5 | `#e1d5e7` | `#9673a6` |

### swimlane の向きについて（重要）

- `horizontal=0` を必ず指定する → 役割名が**左側の列**として表示される
- `startSize=120` → 左側ラベル列の幅（px）
- ノードの x 座標は `startSize`（120）以上から開始 → ラベル列と重ならない
- `horizontal=0` を省略または `horizontal=1` にすると役割名が上部ヘッダーになるので注意

### Draw.io XML テンプレート

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1654" pageHeight="1169" math="0" shadow="0">
  <root>
    <mxCell id="0" />
    <mxCell id="1" parent="0" />

    <!-- スイムレーン1: horizontal=0 でラベルを左側に配置 -->
    <mxCell id="s1" value="[役割1]" style="swimlane;startSize=120;horizontal=0;fillColor=#dae8fc;strokeColor=#6c8ebf;fontStyle=1;fontSize=14;" vertex="1" parent="1">
      <mxGeometry x="30" y="30" width="1560" height="160" as="geometry" />
    </mxCell>
    <!-- ノード: x は startSize(120) 以上から開始 -->
    <mxCell id="n1" value="[開始]" style="ellipse;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=12;" vertex="1" parent="s1">
      <mxGeometry x="130" y="50" width="80" height="60" as="geometry" />
    </mxCell>
    <mxCell id="n2" value="[タスク名]" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=12;" vertex="1" parent="s1">
      <mxGeometry x="270" y="50" width="120" height="60" as="geometry" />
    </mxCell>

    <!-- スイムレーン2 -->
    <mxCell id="s2" value="[役割2]" style="swimlane;startSize=120;horizontal=0;fillColor=#d5e8d4;strokeColor=#82b366;fontStyle=1;fontSize=14;" vertex="1" parent="1">
      <mxGeometry x="30" y="190" width="1560" height="160" as="geometry" />
    </mxCell>
    <mxCell id="n3" value="[判断]" style="rhombus;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;fontSize=11;" vertex="1" parent="s2">
      <mxGeometry x="270" y="40" width="120" height="80" as="geometry" />
    </mxCell>

    <!-- 同一レーン内エッジ: parent = そのレーンのID -->
    <mxCell id="e1" style="edgeStyle=orthogonalEdgeStyle;rounded=0;" edge="1" source="n1" target="n2" parent="s1">
      <mxGeometry relative="1" as="geometry" />
    </mxCell>
    <!-- クロスレーンエッジ: parent="1"（ルート）にする -->
    <mxCell id="e2" style="edgeStyle=orthogonalEdgeStyle;rounded=0;" edge="1" source="n2" target="n3" parent="1">
      <mxGeometry relative="1" as="geometry" />
    </mxCell>
  </root>
</mxGraphModel>
```

---

### ファイル出力

生成した XML を `.drawio` ファイルとして保存する。

**出力先ディレクトリ（固定）：**
```
.claude/skills/draw-swimlane/outputs/[フロー名].drawio
```

ファイル名はフロー名をスネークケースにする（例：受注処理フロー → `order_processing_flow.drawio`）。

ファイル保存後、以下を伝える：
- 保存したファイルの絶対パス
- Draw.io で開く方法（draw.io デスクトップアプリ、またはブラウザで [app.diagrams.net](https://app.diagrams.net) を開いてインポート）
- 「修正したい箇所があればお知らせください」

---

## よくある詰みポイントと対処法

| 問題 | 対処 |
|------|------|
| レーン数が多すぎる | サブプロセスを別図に分割することを提案 |
| 分岐が複雑 | ヒアリング時に分岐条件を細かく確認 |
| XMLエラー | IDの重複がないか確認、parent の参照が正しいか確認 |
| ノードが重なる | x 座標を 160px 間隔、y 座標を中央揃えで配置 |
| ファイルが読み込めない | ファイルパスを確認、Read ツールで直接読み込む。または `inputs/` に置いてもらう |
