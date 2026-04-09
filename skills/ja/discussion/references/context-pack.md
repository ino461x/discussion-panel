# Shared Context Pack — 作成手順

オーケストレーターが1回だけ探索して Pack を作り、全パネリスト (Outsider を除く) に同じ素材を
配布する。5人分の重複探索を排除し、Collision Analysis を「情報の不一致」ではなく「視点の衝突」
として機能させるための仕組み。

## 作成手順

### 1. トピックから対象領域を特定

ブリーフのトピック文から、探索すべきキーワードとファイル類型を抽出する。例:
- 「チャット機能のリファクタリング」 → `services/chat_*.py`, `routes/chat.py`, tests
- 「API パフォーマンス問題」 → 該当 endpoint, repo, migrations
- 「認証フローの再設計」 → auth middleware, session, user routes

### 2. Glob で関連ファイル特定

```
Glob: services/*chat*.py
Glob: routes/*chat*.py
Glob: tests/*chat*
```

3-5 個の主要ファイルに絞る。広く取りすぎないこと。

### 3. Grep で核心部分を特定

既知のシンボル/関数名があれば行番号を取る:
```
Grep: "def handle_request" services/chat_service.py
```

### 4. Read で raw snippets を抽出

各主要ファイルから、トピックに関連するブロックを行番号付きで抜き出す。
**要約や解釈は加えない**。生の素材だけを集める。

### 5. Pack としてまとめる

````markdown
# Shared Context Pack

## 関連ファイル構造
- services/chat_service.py (約1200行): メイン処理、前処理、履歴管理
- routes/chat.py (約300行): HTTP エントリポイント、バリデーション
- tests/test_chat.py: 既存のカバレッジ範囲

## Snippet 1: services/chat_service.py:120-160 (handle_request)
```python
def handle_request(user_id, message, history, ...):
    # ... raw code ...
```

## Snippet 2: services/chat_service.py:340-370 (filter_history)
```python
...
```

## 関連する最近のコミット (対象あれば)
- abc1234 perf: reduce duplicate reads in hot path
- def5678 refactor: split request handler into stages
````

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

Outsider は意図的に白紙状態を保つため Pack を渡さない。詳細な情報衛生ルールは
`information-distribution.md` を参照。

## `--independent` フラグ時

ユーザーが `/discussion [topic] --independent` と明示した場合のみ、従来の方式 (パネリスト各自が
Read/Grep/Glob ツールを持って探索) に戻る。Pack は作らない。代わりに、パネリスト起動時に
Read/Grep/Glob ツールを渡し、`panelist-prompt.md` の「independent mode addendum」を追加する。
