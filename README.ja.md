[English](README.md) | **日本語**

<p align="center">
  <img src="assets/banner.png" alt="Discussion Panel">
</p>

<h1 align="center">Discussion Panel</h1>

<p align="center">
  <strong>Claude Code 向け多視点分析スキル</strong><br>
  確証バイアスから脱却し、あらゆる意思決定に新鮮な視点を。
</p>

<p align="center">
  <a href="#インストール">インストール</a> &bull;
  <a href="#使い方">使い方</a> &bull;
  <a href="#モード">モード</a> &bull;
  <a href="#出力フォーマット">出力</a> &bull;
  <a href="#仕組み">仕組み</a> &bull;
  <a href="#詳細解説">詳細解説</a> &bull;
  <a href="#使用例">使用例</a>
</p>

---

## v4.0.0 の新機能

- **Shared Context Pack** — オーケストレーターがコードベースを**1 回だけ**探索し、raw snippets を
  全パネリストに配布します。従来の「5 人がそれぞれ Read/Grep で独立探索」方式は廃止されました。
  トークン消費が約 50-60% 減少し、Collision Analysis が「情報の不一致」ではなく本物の「視点の衝突」
  を検出できるようになります。
- **Findings のみ出力** — パネリストは 150-200 語の Reasoning chain を出力しなくなりました。
  推論は内部思考で実行し、出力は 1-2 件の Findings（各 2-3 文の根拠付き）のみです。
  パネリスト 1 人あたりの出力トークンが約 60-70% 削減されます。
- **Balanced がデフォルト** — Max モードのデフォルトが Balanced になりました（All Opus ではない）。
  All Opus は明示的 opt-in です。Realist と Outsider は Sonnet で十分な品質が出ることが実証済みです。
- **スケルトン分割** — SKILL.md は短いスケルトンになり、詳細な手順は `references/` 以下に分割されて
  必要時のみ読み込まれます（Anthropic の Progressive Disclosure パターン）。
- **`--independent` フラグ** — 従来の「パネリスト各自探索」モードに戻します。通常は不要で、
  Shared Context Pack がほとんどのトピックに十分です。

## 問題

AI と長時間 1 対 1 で作業していると、危険なパターンが生まれます：

- **確証ドリフト** — AI があなたの思考の枠組みに適応し、反論しなくなる
- **視野狭窄** — すでに議論されたことしか見えなくなる
- **その場しのぎの修正** — 根本原因ではなく表面的なパッチだけ当てる
- **前提の放置** — 誤った前提が誰にも再検討されないまま生き残る

## 解決策

`/discussion` は**新鮮なコンテキストを持つ独立したサブエージェント**を召喚し、異なる視点からトピックを分析します。彼らは現在の会話の結論に縛られることがありません。

各パネリストは異なる**認知フレームワーク**（プレモーテム、第一原理、スティールマン反転など）を使用し、**事実の異なる切り口**を受け取ります。同じモデルでありながら、思考が本当に分岐するのはこのためです。

## 要件

- **Claude Code**（CLI、デスクトップアプリ、または IDE 拡張機能）
- **Claude Pro / Max / Team / Enterprise プラン**（サブエージェントには Agent ツールが必要）
- Standard モードはエージェント 2 体、Full は 4 体、Max は 5 体を召喚

## インストール

```bash
git clone https://github.com/ino461x/discussion-panel.git
```

スキルディレクトリをプロジェクトの `.claude/skills/` フォルダにコピーします。**`references/`
ディレクトリも必ず含めてください** — v4.0.0 は参照ファイルに依存しています：

```bash
# プロジェクトルートから実行
mkdir -p .claude/skills
cp -r discussion-panel/skills/ja/discussion .claude/skills/
cp -r discussion-panel/skills/ja/panel .claude/skills/
```

`/discussion` と `/panel` はどちらも同じスキルを呼び出します。`panel` は利便性のための短縮エイリアスです。

### アップデート

```bash
cd discussion-panel && git pull
cp -r skills/ja/discussion /path/to/your/project/.claude/skills/
cp -r skills/ja/panel /path/to/your/project/.claude/skills/
```

## 使い方

```
/discussion REST から GraphQL に移行すべきか？
/panel このデータベーススキーマは適切か？
```

以上です。スキルがトピックの重要度を評価し、パネルの設定を確認します：

**Q1: スケール** — パネリストは何人？

| Standard | Full | Max |
|----------|------|-----|
| 2 人 | 4 人 | 5 人 |
| Critic, Realist | + Architect, Outsider | + Contrarian |
| 簡易確認 | 設計判断 | 重大な意思決定 |

**どのモードをいつ使うか：**

| 状況 | 推奨 |
|------|------|
| 簡単なサニティチェック、単一ファイルの変更、容易に元に戻せる変更 | Standard |
| 設計判断、複数ファイルのリファクタリング、アプローチの選択 | Full |
| 重大なアーキテクチャ判断、元に戻しにくい移行、チーム全体への影響 | Max |

**Q2: モデル** — 品質レベルは？

| All Sonnet | Balanced (推奨) | All Opus |
|------------|-----------------|----------|
| 高速・低コスト | コスト/品質バランス最良 — **Max モードのデフォルト** | 最大限の深さ、明示的 opt-in |
| 全パネリストが Sonnet | Critic/Architect/Contrarian が Opus、他は Sonnet | 全パネリストが Opus |

軽いトピックの場合は、確認なしで直接実行します。

## モード

各パネリストは**認知フレームワーク**と**Starting Artifact**を持ちます。Starting Artifact は
Findings を出す前に**内部で実行**する思考演習です（v4.0.0 変更点：出力には含まれません）。
これにより、同じ基盤モデルからでも本物の思考の分岐が生まれます。

### Standard（2 人）

| ロール | 焦点 | フレームワーク | Starting Artifact |
|--------|------|--------------|-------------------|
| **Critic** | 前提・リスク・代替案 | プレモーテム + 5 Whys | 失敗シナリオを 3 つ思考 |
| **Realist** | コスト・保守性・より簡単な道 | コストベネフィット推定 | フェーズ別の人日を思考 |

### Full（4 人）

追加：

| ロール | 焦点 | フレームワーク | Starting Artifact |
|--------|------|--------------|-------------------|
| **Architect** | 根本原因・システム的影響・見落とした相乗効果 | 第一原理 | 依存関係のチェーンをマッピング |
| **Outsider** | 不必要な複雑さ・「なぜそうしないの？」 | 初心者の視点 | この問題を解決した非ソフトウェア領域を挙げる |

### Max（5 人）

追加：

| ロール | 焦点 | フレームワーク | Starting Artifact |
|--------|------|--------------|-------------------|
| **Contrarian** | **逆のアプローチ**に対する最も強力な論拠 | スティールマン反転 | このアプローチが提案されなかった世界で何が構築されたかを描写 |

## 出力フォーマット

各パネリストが生成するもの：
1. 深刻度評価付きの **1〜2 件の Findings**
2. 2-3 文の根拠（Starting Artifact が何を導いたかを明示）

結果は以下にまとめられます：

| セクション | 内容 |
|-----------|------|
| **サマリー** | 合意点・緊張点・発見 |
| **Findings Table** | 深刻度順（CRITICAL > HIGH > MEDIUM > LOW）に並べた全 Findings |
| **Collision Analysis** | （Full/Max のみ）パネリストが矛盾した箇所と、そこから導かれる第三の結論 |
| **Findings Validation** | （Full/Max のみ）オーケストレーターによる設計/コード照合 |

深刻度は各パネリストが分析に基づいて割り当て、最終統合でソートされます。

深刻度の定義：

| 深刻度 | 意味 |
|--------|------|
| **CRITICAL** | 無視すると進行が止まるか、失敗を引き起こす |
| **HIGH** | 重大なリスクまたは見逃した機会 |
| **MEDIUM** | 考慮する価値はあるが緊急ではない |
| **LOW** | 軽微な改善点または細かい指摘 |

## 仕組み

1. **ブリーフ抽出** — 事実をカテゴリ別に整理（技術・ビジネス・ユーザー行動・暗黙の前提）。ユーザーの主張を攻撃可能なターゲットとして保持。
2. **Shared Context Pack 作成** — オーケストレーターがコードベースを**1 回だけ**探索し、raw snippets を集める（v4.0.0 変更点）。Pack は Outsider を除く全パネリストに配布。
3. **差別化された入力** — 各パネリストは異なる切り口を受け取る。Outsider はトピックとステークスのみ、意図的な白紙状態。
4. **Starting Artifact（内部実行）** — Findings の前に必須の思考演習（失敗シナリオ、コスト見積もり、依存関係マップ）。内部で実行し、出力には含めない。
5. **パネリスト並列起動** — Agent ツールで並列起動。`--independent` 以外ではツールを渡さない。
6. **Collision Analysis**（Full/Max） — 新たなサブエージェントが Findings + Artifact 要約を検討し矛盾を探す：「両方が正しいとしたら、どんな第三の結論が導かれるか？」
7. **Findings Validation**（Full/Max） — オーケストレーターが各 Finding を実際の設計/コードに照合。
8. **ファシリテーション** — 何を実行に移すかはあなたが決める。

## 詳細解説

### 差別化された入力

各パネリストは同じ事実の**異なる切り口**を受け取ります：

| パネリスト | 情報の切り口 |
|-----------|------------|
| Critic | 全事実 + 暗黙の前提を強調表示；さらにブリーフが**触れていない** 3 つの問いをリストアップするよう求められる；Shared Context Pack あり |
| Realist | 全事実 + 技術・ビジネス上の制約を強調表示；Shared Context Pack あり |
| Architect | 全事実 + 依存関係・構造情報を強調表示；Shared Context Pack あり |
| Outsider | **トピックとステークスのみ**（意図的な白紙状態；非技術トピックにはトピックのみ）；**Pack は渡さない** |
| Contrarian | 全事実 + ユーザーの主張を前面に配置；Shared Context Pack あり |

### ファイル構成 (v4.0.0)

```
skills/ja/discussion/
├── SKILL.md                    # スケルトン — 呼び出し方、モード、フロー
└── references/
    ├── brief-format.md         # ブリーフ抽出ルール
    ├── context-pack.md         # Shared Context Pack 作成手順
    ├── information-distribution.md
    ├── panelist-prompt.md      # パネリストプロンプトテンプレート全文
    ├── collision-analyst.md    # Collision Analyst プロンプト全文
    └── output-format.md        # 結果提示フォーマット
```

オーケストレーターは `references/*.md` を必要時のみ読み込みます（Progressive Disclosure）。
常駐するのは SKILL.md だけです。

## 使用例

### アーキテクチャレビュー

```
/discussion full routes レイヤーが太りすぎていないか？
```

> **Findings**
>
> | # | 深刻度 | Finding | パネリスト |
> |---|--------|---------|-----------|
> | 1 | CRITICAL | `handle_payment` にトランザクション境界がない — 失敗時に部分書き込みが発生 | Architect |
> | 2 | HIGH | routes が DB を直接インポートし、サービスレイヤーをバイパスしている | Critic |
> | 3 | HIGH | コードをサービスに移しても fat が移動するだけ — ドメインで分割が必要 | Realist |
> | 4 | MEDIUM | 既存の auth モジュールのようにリソース別（users, orders, payments）に routes を分割する | Outsider |

> **Collision Analysis**：
> Critic は「routes がサービスレイヤーをバイパスしている」と指摘し、Realist は「サービスに移しても fat が移動するだけ」と述べた。
> 両方が正しいなら：修正はサービスへの抽出ではなく、まずドメインで分割し、それから抽出することだ。

## フラグ

| フラグ | 効果 |
|-------|------|
| `--independent` | 従来の「パネリスト各自探索」モードに戻す（パネリストに Read/Grep/Glob を渡す）。通常は不要 — Shared Context Pack がデフォルトで、ほとんどのケースで十分。 |

**v4.0.0 変更点**：旧来の `--ctx` / `--no-ctx` フラグは廃止されました。技術トピックでは常に
Shared Context Pack を作成し（オーケストレーターが 1 回だけ探索）、非技術トピックでは Pack を
作成しません。`--independent` はパネリストに個別探索させたい場合のみ使用してください。

## 正直な前置き

これらのパネリストは同じ基盤モデルからロールプレイされた視点であり、真に独立した思考者ではありません。それでも価値があるのは：

- **認知フレームワーク**が異なるラベルだけでなく、異なる思考プロセスを強制するから
- **差別化された入力**によって各パネリストが文字通り異なる情報を見るから
- **Starting Artifacts**が意見を述べる前に考えさせるから
- **Collision Analysis**が矛盾から新たな洞察を引き出すから

これらは実際のピアレビュー・ドメイン専門知識・ユーザーテストの**代替にはなりません**。

## ライセンス

MIT
