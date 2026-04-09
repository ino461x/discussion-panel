# Shared Context Pack — 作成手順

**なぜ共有するのか**: 従来スキルはパネリスト各自が Read/Grep でコード探索していたが、これは
5人分の重複探索を生み、トークンが scale out する。さらに各自が違うファイルを読むため、Collision
Analysis が「視点の衝突」ではなく「情報の不一致」になっていた。

オーケストレーターが1回だけ探索して Pack を作り、全パネリスト (Outsider を除く) に同じ素材を
配布することで、トークン 60-80% 削減しつつ Collision の質を上げる。

## 作成手順

### 1. トピックから対象領域を特定

ブリーフのトピック文から、探索すべきキーワードとファイル類型を抽出する。例:
- 「AI chat 関連のリファクタリング」 → `services/ai_chat_service.py`, `routes_ai.py`, tests
- 「API パフォーマンス問題」 → 該当 endpoint, repo, migrations
- 「認証フローの再設計」 → auth middleware, session, user routes

### 2. Glob で関連ファイル特定

```
Glob: services/*chat*.py
Glob: routes/*ai*.py
Glob: tests/*ai_chat*
```

3-5 個の主要ファイルに絞る。広く取りすぎないこと。

### 3. Grep で核心部分を特定

特定のシンボルや関数名が既知の場合は Grep で行番号を取る:
```
Grep: "def execute_chat" services/ai_chat_service.py
Grep: "_build_bookmark_context" --output content -n
```

### 4. Read で raw snippets を抽出

各主要ファイルから、トピックに関連するブロックを行番号付きで抜き出す。
**要約や解釈は加えない**。生の素材だけを集める。

### 5. Pack としてまとめる

```markdown
# Shared Context Pack

## 関連ファイル構造
- services/ai_chat_service.py (2100行): execute_chat の3パス統合、ExclusionCtx
- routes_ai.py (450行): LINE webhook, search API adapters
- gemini/chat.py (320行): stable_context/dynamic_hint 分離

## Snippet 1: services/ai_chat_service.py:450-490 (execute_chat entrypoint)
```python
def execute_chat(user_id, message, history, ...):
    # ... raw code ...
```

## Snippet 2: services/ai_chat_service.py:780-810 (ExclusionCtx.filter_history)
```python
...
```

## 関連する最近のコミット (対象あれば)
- 70d6977 perf(stats): /api/stats を RPC 1回で集計
- c60b9c6 refactor(ai-chat): 主経路から legacy context 生成を除去
```

## サイズの目安

- Pack 全体で **3-5k tokens** を目標
- 大きくなりすぎる場合はファイル数を削る (snippet の行範囲を短くするより、対象ファイルを絞る方が良い)
- 「なぜこの snippet を入れたか」は書かなくてよい。パネリストが役割に応じて解釈する

## スキップすべき時

以下の場合は Pack を作らない:
- 非技術トピック (プロダクト戦略、優先順位付け、UX 判断)
- コードベースアクセス不要な判断 (名前の付け方、ドキュメント構成)
- ユーザーが `--independent` を明示した場合 (従来の独立探索を希望)

## Outsider への配布ルール

Outsider は**意図的に白紙状態**を保つ。Pack は渡さない。ステークス記述からも実装固有の用語
(関数名、ファイルパス、ライブラリ名) を除去する。Outsider にはトピックとステークスだけを渡し、
「ソフトウェア以外の分野で同じ種類の問題を解決した事例」を起点に考えさせる。

## `--independent` フラグ時

ユーザーが `/discussion [topic] --independent` と明示した場合のみ、従来の方式 (パネリスト各自が
Read/Grep/Glob ツールを持って探索) に戻る。Pack は作らない。

この場合のみ、パネリスト起動時に Read/Grep/Glob ツールを渡し、`references/panelist-prompt.md` の
「independent mode addendum」セクションを追加する。
