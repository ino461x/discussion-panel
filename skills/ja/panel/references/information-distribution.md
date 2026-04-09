# Information Distribution — 情報配分と Artifact パラメータ化

各パネリストはブリーフの異なるビューを受け取る。現在のモードに含まれるパネリストのビューだけ
構成する (Standard = Critic + Realist のみ)。

## 配分ルール

| パネリスト | 受け取る情報 |
|----------|----------|
| **Critic** | 全カテゴリ + 暗黙の前提をトップに目立つように配置。追加: 「このブリーフが扱っていない質問を3つ挙げよ — 目立って欠落しているものは何か？」 + Shared Context Pack |
| **Realist** | 全カテゴリ + 技術的制約とビジネス的制約を強調 + Shared Context Pack |
| **Architect** | 全カテゴリ + 技術的制約 + 依存関係/構造情報を強調 + Shared Context Pack |
| **Outsider** | トピックとステークスのみ — 制約なし、ユーザーの主張なし、過去の試行なし、**Pack も渡さない** (意図的な白紙状態) |
| **Contrarian** | 全カテゴリ + ユーザーの主張をトップに目立つように配置 + Shared Context Pack |

## Outsider の情報衛生

Outsider のステークス記述から実装固有の用語 (関数名、ファイルパス、ライブラリ名) を除去する。
ステークスはビジネス/ユーザー影響の言葉のみで記述する。非技術トピックの場合はステークス自体を
省略し、Outsider にはトピック文のみを渡す。

Shared Context Pack も渡さない — これが Outsider の価値の源泉で、コードを見ない外部視点が
「なぜそんな複雑なことをしているのか」を問う余地を残す。

## 動的 Artifact パラメータ化

パネリストを起動する前に、ステップ1のトピック固有コンテキストを各役割の Starting Artifact
指示に注入する:

| 役割 | 注入する要素 |
|------|-----------|
| Critic | 攻撃すべき具体的な暗黙の前提 (最初の1つ) |
| Realist | 見積もるべき具体的なアプローチ (最もコストの高いフェーズ) |
| Architect | マッピングの起点となる主要コンポーネント (最も依存関係の多いもの) |
| Contrarian | 反転すべきユーザーの核心的主張 |
| Outsider | **注入しない** (白紙状態を維持) |

フォールバック: 複数候補がある場合は最も具体的なものを優先する。

## 例: AI Chat リファクタリングがトピックの場合

**Critic への Starting Artifact 指示:**
> 3つの失敗シナリオを思考せよ。特に「ExclusionCtx が否定済み候補を Gemini 履歴から除去する」
> という前提が間違っていたらどうなるかから始めよ。

**Realist への Starting Artifact 指示:**
> フェーズ別の人日見積もりを思考せよ。特に「execute_chat の Phase 4-7 ステージ分割」の
> 実装コストとリグレッション検証コストを優先せよ。

**Architect への Starting Artifact 指示:**
> 依存関係のチェーンを思考せよ。特に `services/ai_chat_service.py` を起点として、
> routes_ai/routes_search/line_webhook の 3 パスがどこで分岐し収束するかをマッピングせよ。

**Contrarian への Starting Artifact 指示:**
> ユーザーの核心的主張「3パス統合により複雑度が下がる」が提案されなかった世界を想像せよ。
> 代わりに何が構築され、なぜそれが明らかに正しいとされたかを記述せよ。

**Outsider への指示 (注入なし):**
> (Starting Artifact のみ。トピック固有パラメータは渡さない)
