---
name: discussion
description: >
  独立したサブエージェントを起動し、前提を疑い、盲点を発見し、代替案を浮上させる多角的分析パネル。
  ユーザーが /discussion、/panel、/challenge と発言した際にこのスキルを使用する。
  また、「セカンドオピニオンが欲しい」「このアイデアをストレステストしたい」
  「本当にこのアプローチで正しい？」と尋ねた場合、設計判断前にトレードオフを議論したい場合、
  方向性に行き詰まっている場合、または会話が長時間にわたり前提を疑わず合意し続けている場合にも
  発動する。対象領域: アーキテクチャ判断、リファクタリング方針、優先度決定、ツール/ライブラリ選定、
  UX方針、および技術・非技術を問わず「XとYのどちらにすべきか？」というジレンマ全般。
---

# Discussion — 多角的分析パネル

## 存在理由

AIと1対1で作業していると以下が起きる:
1. **確証バイアスの漂流** — AIが会話のフレーミングに適応し反論しなくなる
2. **トンネルビジョン** — 双方とも既に議論された内容しか見えなくなる
3. **場当たり的な修正** — 根本原因が未解決のままパッチが蓄積
4. **疑われない前提** — 誰も再検証しない仮定が生き残る

このスキルは、現在の会話の結論に利害関係を持たない新鮮なサブエージェントを起動してこれを打破する。
**真に独立した専門家ではなく、同一基盤モデルによる構造化されたセルフレビュー** — 新鮮なコンテキストが
会話の惰性を取り除き、構造化された役割が見落とされた角度の検証を強制する。

## 呼び出し方

```
/discussion [トピック]                 標準 (パネリスト2名)
/discussion full [トピック]            フルパネル (パネリスト4名)
/discussion max [トピック]             MAXパネル (パネリスト5名)
/discussion [トピック] --independent   パネリストに Read/Grep を渡す (旧 --ctx、通常非推奨)
```

**重要な設計変更**: 本スキルは従来の「パネリストが各自 Read/Grep で探索する」方式を廃止し、
オーケストレーターが1回だけ探索して **Shared Context Pack** を作り、全パネリストに同じ素材を
渡す方式をデフォルトとする。これにより:
- 5人分の重複探索が1回に圧縮される (トークン 60-80% 削減)
- 全パネリストが同じコードを見ているため、Collision Analysis が「視点の衝突」として明確に出る
- 事実誤認が減り、Findings Validation が軽量化される

`--independent` で従来の独立探索に戻せるが、通常は Shared Context で十分。

## モード選択

モード指定なしで呼び出された場合、トピックの重要度を評価する:

- **軽量トピック** (アーキテクチャ非影響、巻き戻し容易、単一ファイル): Standard + Sonnet をサイレント実行
- **中程度 / 重量級**: AskUserQuestion で規模とモデルをまとめて提示

中程度は Full + Balanced、重量級は Max + Balanced を推奨とする。

**重要**: Max モードのデフォルトは **Balanced** (All Opus ではない)。All Opus は明示的 opt-in。
Realist と Outsider は Sonnet で十分品質が出ることが実証されており、Opus は過剰設定。

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

ユーザーがモード/モデルを明示指定した場合、「とりあえずやって」と言った場合、または急いでいる場合は
質問をスキップしてデフォルト (Balanced) を使用する。

### Balanced モデル割り当て

| パネリスト | モデル | 理由 |
|----------|-------|-----|
| Critic | Opus | 深い前提挑戦にはより強い推論が必要 |
| Architect | Opus | 体系的分析に深い洞察が有利 |
| Contrarian | Opus | 整合性ある反論構築は最も難度が高い |
| Realist | Sonnet | 実践的トレードオフ評価に Sonnet で十分 |
| Outsider | Sonnet | 初心者目線に Opus は不要 |

## パネリスト一覧

各パネリストには認知フレームワークと Starting Artifact がある。Starting Artifact は思考の足場として
**内部で実行する** (出力はしない)。これにより同一モデルからでも多様性が生まれる。

| 役割 | フォーカス | 認知フレームワーク | Starting Artifact (内部実行) |
|------|-----------|-----------------|----------------------------|
| **Critic** | 欠陥のある前提、見落としたリスク、何がうまくいかないか | プレモーテム + 5 Whys | 3つの失敗シナリオを思考 |
| **Realist** | 実装コスト、保守負担、よりシンプルな代替案 | 具体的な見積もり | フェーズ別の人日見積もり |
| **Architect** | 根本原因、体系的影響、長期的帰結 | 第一原理分解 | 依存関係のチェーン |
| **Outsider** | 不要な複雑さ、分かりにくい命名、初心者目線 | 初心者の目 + 異分野アナロジー | ソフト以外の分野での類似事例 |
| **Contrarian** | 現在アプローチとは正反対の最強の主張を構築 | スチールマン・インバージョン | 現在のアプローチが提案されなかった世界 |

Standard = Critic + Realist / Full = + Architect, Outsider / Max = + Contrarian

## 実行フロー

### ステップ 1: ブリーフ抽出

トピックを構造化ブリーフに要約する。詳細カテゴリとフォーマットは
`references/brief-format.md` を参照。要点:
- トピック(1-2文) + ステークス
- トピックに関連するカテゴリだけ抽出 (技術トピックでは技術的制約に集中、
  ビジネス系カテゴリは必要な時のみ)
- ユーザーの主張と過去の試行を明示的に含める (Critic/Contrarian の攻撃対象)

トピックが曖昧な場合は、続行前に1つの明確化質問を行う。

### ステップ 2: Shared Context Pack 作成 (技術トピックのみ)

オーケストレーターが自ら Read/Grep/Glob を使い、トピックに関連するコード素材を1回だけ収集する。
詳細は `references/context-pack.md` を参照。要点:
- 関連ファイルパスと行番号付きの raw snippets を集める
- 解釈/評価は加えない (純粋な素材のみ)
- Pack 全体で 3-5k tokens を目安
- 非技術トピックではスキップ

Pack は全パネリスト (Outsider を除く) に共通で渡す。**Outsider だけは白紙状態を保つため Pack を渡さない。**

### ステップ 3: 情報配分と Artifact パラメータ化

各パネリストはブリーフの異なるビューを受け取る。配分ルールと動的 Artifact パラメータ化の詳細は
`references/information-distribution.md` を参照。要点:
- Critic: 全ブリーフ + 暗黙の前提を強調
- Realist: 全ブリーフ + 技術/ビジネス制約を強調
- Architect: 全ブリーフ + 依存関係情報
- Outsider: トピックとステークスのみ (白紙状態、Pack も渡さない)
- Contrarian: 全ブリーフ + ユーザーの主張を強調

### ステップ 4: パネリスト並列起動

全パネリストを Agent ツールで並列起動する (単一メッセージ内で複数 Agent 呼び出し)。
各 Agent 呼び出しに `model` パラメータを設定する。

**パネリストにはツールを渡さない** — Read/Grep/Glob は使わせない。Pack とブリーフだけで分析させる。
`--independent` フラグ時のみ、従来通り Read/Grep/Glob を渡す。

パネリストに渡すプロンプトテンプレートは `references/panelist-prompt.md` を参照。要点:
- Starting Artifact は **内部で実行** (思考の足場として使うが出力しない)
- 出力は Findings 1-2件のみ、各項目に重要度ラベルと 1-2文の根拠
- Reasoning chain は廃止 (Findings の根拠文に統合)
- CRITICAL/HIGH/MEDIUM/LOW の重要度ラベル必須

### ステップ 5: Collision Analysis (Full / Max のみ)

Standard モードでは省略 (2人の Findings をインラインで比較するだけ)。

Full/Max の場合: 専用サブエージェント (会話履歴なし) を起動する。
各パネリストの Findings と、オーケストレーターが抽出した「各パネリストが前提とした核心」を
2-3行の要約として渡す。プロンプト全文は `references/collision-analyst.md` を参照。

Collision Analyst のタスク:
1. パネリスト間の矛盾を特定 → 第三の結論を提案
2. 全員合意の盲点チェック → 反対論者の主張を仮構築

### ステップ 6: Findings Validation (Full / Max のみ)

オーケストレーター (サブエージェントではない) が各 Findings を検証する。
Shared Context Pack を使った場合、事実誤認は大幅に減っているはずだが、以下は確認する:
1. 所見は実際の設計/コードを正しく参照しているか
2. 既に対処済みか (パネリストが知らなかった設計要素)
3. パネル自身の出力と矛盾していないか

評価ラベル: ◎ (正確) / ○ (妥当だが限定的) / △ (部分的に正しい) / ✕ (事実誤認)

### ステップ 7: 結果提示

**サマリーを最初に** — 大半のユーザーはここしか読まない。
出力フォーマットは `references/output-format.md` を参照。要点:
- サマリー: 合意点 / 緊張点 / 発見
- Findings テーブル (重要度順)
- Collision Analysis (Full/Max のみ)
- Findings Validation (Full/Max のみ)

**発見**について: 真に新しい洞察のみ記載する。パネリストが既知の懸念を繰り返しただけの場合は
「なし — パネリストは既存懸念を補強した」と正直に書く。水増しより誠実なフレーミング優先。

### ステップ 8: ファシリテーション

- 提案をすぐに採用しない / すぐに却下もしない
- どの点が響いたか尋ねる
- 掘り下げたい点があれば議論するか、焦点を絞ったフォローアップを起動する

パネルは情報を提供する。指示はしない。決めるのはユーザーである。

## 使用すべきでない場合

- 明確な答えがある単純な事実の質問
- ユーザーが議論ではなく実行を求めている場合
- スピード優先の高速イテレーションループ
- 判断を誤ってもコストが無視できる些細な決定

## コスト意識

| モード | パネリスト | デフォルトモデル | おおよそのトークン倍率 |
|--------|-----------|---------------|-----------|
| Standard | 2 | Sonnet | 約2-3倍 |
| Full | 4 | Balanced | 約3-4倍 |
| Max | 5 | Balanced | 約4-5倍 |

旧方式 (パネリスト各自探索) は約1.5-2倍の追加コストがかかっていた。Shared Context Pack により
この重複が解消されている。`--independent` を明示した場合のみ旧方式に戻る。

All Opus は Balanced の約1.5倍のコスト。誤った選択のコストが分析コストを大幅に上回る重大な
意思決定でのみ使用する。

**モデル選択の根拠**:
- **Sonnet**: コスト対品質比が最良。Standard および Full の Realist/Outsider のデフォルト。
- **Opus**: 深い前提挑戦・体系分析・整合性ある反論構築が必要な Critic/Architect/Contrarian。
- **Haiku**: 非推奨。探索効率が悪く同等品質で Sonnet の4倍トークンを消費する。

## 参照ファイル

詳細な指示は以下のファイルを必要時に読む:
- `references/brief-format.md` — ブリーフ抽出のカテゴリとフォーマット
- `references/context-pack.md` — Shared Context Pack の作成手順
- `references/information-distribution.md` — パネリストごとの情報配分と Artifact パラメータ化
- `references/panelist-prompt.md` — パネリストプロンプトテンプレート全文
- `references/collision-analyst.md` — Collision Analyst プロンプト全文
- `references/output-format.md` — 結果提示フォーマット
