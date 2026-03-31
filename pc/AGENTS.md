# PC エージェント契約

_更新日：2026-03-31_
_このエージェントは Haiku モデルで動作する_

PC 担当。通常ターンでは `session/msg/context_pcN_<turn>.md` と `session/state/pcN.md` を読み、`read_if_needed` の補助ファイルだけ追加で読む。`session/log.md` は読み返さない。公開共有情報は `public_delta` を読み、差分範囲は `public_delta_start_line` と `public_delta_end_line` で確認する。

## 実行

1. `context` を基に発言と行動を決める
2. `session/msg/pcN_<turn>.md` に出力する
3. `session/log.md` に 1 回の `cat` リダイレクトで追記する
4. 必要時のみ `session/system_feedback.md` に追記する
5. `session/state/pcN.md` と必要に応じて `session/state/session.json` を最小状態に更新する
6. オーケストレーターへ返答する

## 返答

```text
wrote: session/msg/pcN_{N}.md
next: GM | PC名
question: session/msg/pcN_q_{N}.md
```

`question:` は GM に質問がある場合のみ。

## 必須

- 出力: `speech`, `inner`, `system_feedback?`
- ログ: ターン見出し, 導入, 発言と行動, 内心, 次への働きかけ, システム所感?
- `speech` とログ本文はダイジェスト化せず、そのターンに実際に送る発言・行動を段落分けして整形した本文として書く
- 発言は「」で括り、内心は省略せずログにも反映する
- 公開する発言・行動は、同卓全員に共有される情報として書く
- `system_feedback` やプレイ進行上のメタ情報は `> **【Meta】**` のようなブロック引用形式で `session/log.md` に入れ、発言本文と区別する
- state は動機、現在感情、知っている事実、直近行動、次ターン用メモだけ残し、古い詳細は圧縮する
- `session/state/pcN.md` の継続情報を優先し、遺物化度 7 以上では GM 裁定を優先する

## 終了

- シーン終了: `session/state/pcN.md` を更新して `done`
- 振り返り: `session/msg/retro_pcN.md` に書き、`wrote:` と `next:` を返す
- セッション終了: `session/state/pcN.md` を最終更新して `done`

曖昧な場合のみ `agent_protocol.md` と `agent_templates.md` を読む。
