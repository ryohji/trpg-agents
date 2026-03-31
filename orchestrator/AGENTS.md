# オーケストレーター契約

_更新日：2026-03-31_

本文は解釈せず、ファイル名と `next:` で進行する。毎ターン `session/msg/context_<agent>_<turn>.md` を作り、それを主入力として渡す。公開共有情報は `session/public_scene_<N>.md` に集約し、各担当には未読差分だけ渡す。

## 実行

1. `session/`, `session/msg/`, `session/state/`, `session/log.md`, `session/state/session.json`, `session/public_scene_<N>.md` を用意する
2. GM と PC を起動し、`ready` 後に GM 用 `context` を送る
3. 各返答の `wrote:` と `next:` を読む
4. 必要なら `secret:` を対象 PC の `context` にだけ載せる
5. 公開発言・公開描写・公開裁定を `session/public_scene_<N>.md` に追記する
6. 次担当向け `context` を作り、`public_delta_start_line`, `public_delta_end_line`, `public_delta` に未読差分を載せ、補助ファイルは `read_if_needed` に列挙するだけにする
7. `scene_end`, `retrospective`, `session_end` を処理する

## context の output セクションの書き方

`context` の `output` に列挙する各ファイルには、**何を書くか**を明示する。特に `session/log.md` の追記指示は以下のように書く：

```
- session/log.md に追記する
  - ターン見出しを付け、speech の本文をそのまま段落分けして整形する
  - 要約・ダイジェストにしない。speech と同じ内容をログに書く
  - system / system_feedback はブロック引用形式で本文と区別する
```

この明示がないと、サブエージェントは log.md にダイジェストを書く傾向がある。

## 必須

- `context` には最小状態、直近イベント、未解決事項、出力先を書く
- 生ログ全文は渡さない
- 同卓で共有される公開発言は `session/public_scene_<N>.md` を正本とし、個別配布ではなく未読差分として `context` に載せる
- `context` には差分範囲として `public_delta_start_line` と `public_delta_end_line` も入れる
- `session/state/session.json` は共有の機械可読状態
- `session/state/session.json` には各担当の `public_cursor` を持たせ、主に `last_line`、フォールバックで `last_event_id` を使う
- `session/state/gm.md`, `session/state/pcN.md` は短い継続情報だけ持ち、古い詳細は圧縮する

## 終了

- `scene_end`: 全員に state 更新と `done` を要求
- `retrospective`: GM から開始し、`next:` で回す
- `session_end`: 全員に最終 state 更新と `done` を要求

曖昧な場合のみ `agent_protocol.md` と `agent_templates.md` を読む。
