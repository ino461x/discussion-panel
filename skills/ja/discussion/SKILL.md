---
name: discussion
description: >
  独立したサブエージェントを起動し、前提を疑い、盲点を発見し、代替案を浮上させる多角的分析パネル。
  ユーザーが /discussion、/panel、/challenge と発言した時に発動する。また、セカンドオピニオン、
  ストレステスト、「本当にこのアプローチで正しい？」、設計判断前のトレードオフ議論、行き詰まりの
  打開、長時間の無批判な合意の打破を求められた場合にも発動する。
---

# Discussion — 多角的分析パネル

新鮮なサブエージェントで会話の惰性と確証バイアスを断ち切る。真に独立した専門家ではなく、
同一基盤モデルによる**構造化されたセルフレビュー** — 新鮮なコンテキストと役割固定が有効に働く。

## 呼び出し方

```
/discussion [トピック]                 標準 (パネリスト2名)
/discussion full [トピック]            フルパネル (パネリスト4名)
/discussion max [トピック]             MAXパネル (パネリスト5名)
/discussion [トピック] --independent   従来の独立探索モード (パネリスト各自に Read/Grep を渡す)
```

デフォルトはオーケストレーターが1回だけ探索して **Shared Context Pack** を作り全パネリストに
配布する方式。`--independent` は旧方式で、通常は不要。詳細は `references/context-pack.md`。

## モード選択

モード指定なしで呼び出された場合、トピックの重要度を評価する:

- **軽量** (アーキ非影響、巻き戻し容易、単一ファイル): Standard + Sonnet をサイレント実行
- **中程度 / 重量級**: AskUserQuestion で規模とモデルをまとめて提示 (推奨: Full + Balanced / Max + Balanced)

**重要**: Max のデフォルトは **Balanced** (All Opus ではない)。All Opus は明示的 opt-in。
Realist と Outsider は Sonnet で十分品質が出る。

```json
{
  "questions": [
    {
      "question": "パネルの規模を選んでください。",
      "header": "Scale",
      "multiSelect": false,
      "options": [
        {"label": "Standard", "description": "2視点 (Critic, Realist)"},
        {"label": "Full", "description": "4視点 (+ Architect, Outsider)"},
        {"label": "Max", "description": "5視点 (+ Contrarian)"}
      ]
    },
    {
      "question": "パネリストのモデルを選んでください。",
      "header": "Model",
      "multiSelect": false,
      "options": [
        {"label": "All Sonnet", "description": "全員 Sonnet。高速・低コスト"},
        {"label": "Balanced (Recommended)", "description": "Critic/Architect/Contrarian=Opus、他=Sonnet"},
        {"label": "All Opus", "description": "全員 Opus。最高精度だがコスト大"}
      ]
    }
  ]
}
```

ユーザーが明示指定、「とりあえずやって」、急いでいる場合は質問をスキップしデフォルト (Balanced) を使用。

### Balanced モデル割り当て

| パネリスト | モデル | 理由 |
|----------|-------|-----|
| Critic | Opus | 深い前提挑戦にはより強い推論が必要 |
| Architect | Opus | 体系的分析に深い洞察が有利 |
| Contrarian | Opus | 整合性ある反論構築が最も難度が高い |
| Realist | Sonnet | 実践的トレードオフ評価に Sonnet で十分 |
| Outsider | Sonnet | 初心者目線に Opus は不要 |

## パネリスト一覧

**Starting Artifact** は思考の足場として**内部で実行する** (出力はしない)。これが同一モデルからの
多様性を生む。構成: Standard = Critic + Realist / Full は Architect + Outsider 追加 / Max は Contrarian 追加。

| 役割 | フォーカス | 認知フレームワーク | Starting Artifact (内部実行) |
|------|-----------|-----------------|----------------------------|
| **Critic** | 欠陥のある前提、見落としたリスク、何がうまくいかないか | プレモーテム + 5 Whys | 3つの失敗シナリオを思考 |
| **Realist** | 実装コスト、保守負担、よりシンプルな代替案 | 具体的な見積もり | フェーズ別の人日見積もり |
| **Architect** | 根本原因、体系的影響、長期的帰結 | 第一原理分解 | 依存関係のチェーン |
| **Outsider** | 不要な複雑さ、分かりにくい命名、初心者目線 | 初心者の目 + 異分野アナロジー | ソフト以外の分野での類似事例 |
| **Contrarian** | 現在アプローチとは正反対の最強の主張を構築 | スチールマン・インバージョン | 現在のアプローチが提案されなかった世界 |

## 実行フロー

### ステップ 1: ブリーフ抽出

トピックを構造化ブリーフに要約する。カテゴリとフォーマットは `references/brief-format.md`。
トピックが曖昧な場合は、続行前に1つだけ明確化質問を行う。

### ステップ 2: Shared Context Pack 作成 (技術トピックのみ)

オーケストレーターが Read/Grep/Glob で1回だけコード素材を収集する (3-5k tokens 目安)。
手順とスキップ条件は `references/context-pack.md`。Pack は Outsider を除く全パネリストに配布する
(Outsider は白紙状態を保つため渡さない)。

### ステップ 3: 情報配分と Artifact パラメータ化

各パネリストはブリーフの異なるビューを受け取る。配分ルールと動的 Artifact 注入の詳細は
`references/information-distribution.md`。

### ステップ 4: パネリスト並列起動

全パネリストを Agent ツールで並列起動する (単一メッセージ内で複数 Agent 呼び出し)。
各 Agent に `model` パラメータを設定。**パネリストにはツールを渡さない** — Pack とブリーフだけで
分析させる (`--independent` フラグ時のみ Read/Grep/Glob を渡す)。

プロンプトテンプレート全文は `references/panelist-prompt.md`。出力は **Findings 1-2件のみ**、
各項目に重要度ラベル (CRITICAL/HIGH/MEDIUM/LOW) と 2-3文の根拠。Reasoning chain は廃止。

### ステップ 5: Collision Analysis (Full / Max のみ)

Standard では省略し、2人の Findings をインラインで比較する。Full/Max の場合: 専用サブエージェント
(会話履歴なし) に Findings と各パネリストの Artifact 要約 (2-3行) を渡し、矛盾検出と合意リスク
チェックを依頼する。プロンプト全文は `references/collision-analyst.md`。

### ステップ 6: Findings Validation (Full / Max のみ)

これはサブエージェント起動ではなく、オーケストレーター自身が各 Findings を検証する。確認事項:
(1) 実際の設計/コードを正しく参照しているか、(2) 既に対処済みではないか、(3) パネル自身の出力と
矛盾していないか。評価ラベル: ◎ (正確) / ○ (妥当だが限定的) / △ (部分的に正しい) / ✕ (事実誤認)。

### ステップ 7: 結果提示

**サマリーを最初に** — 大半のユーザーはここしか読まない。出力フォーマットは
`references/output-format.md`。**発見**は真に新しい洞察のみ記載し、既知懸念の補強にすぎない場合は
「なし」と正直に書く (水増しより誠実なフレーミング優先)。

### ステップ 8: ファシリテーション

提案をすぐに採用/却下しない。どの点が響いたか尋ねる。掘り下げたい点があれば議論するか、
焦点を絞ったフォローアップを起動する。パネルは情報を提供する。決めるのはユーザーである。

## 使用すべきでない場合

- 明確な答えがある単純な事実の質問
- ユーザーが議論ではなく実行を求めている場合
- スピード優先の高速イテレーションループ
- 判断を誤ってもコストが無視できる些細な決定

## コスト意識

基準: 単一 Sonnet 呼び出し相当を 1.0 とした概算値。

| モード | パネリスト | デフォルトモデル | トークン倍率 (概算) |
|--------|-----------|---------------|-----------|
| Standard | 2 | Sonnet | 約 2-3× |
| Full | 4 | Balanced | 約 3-4× |
| Max | 5 | Balanced | 約 4-5× |

All Opus は Balanced の約 1.5× のコスト。誤った選択のコストが分析コストを大幅に上回る重大な
意思決定でのみ使用する。Haiku は非推奨 (探索効率が悪く同等品質で Sonnet の4倍トークンを消費)。
