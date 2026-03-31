# TRPG エージェント共通プロトコル

この文書は詳細仕様の退避先である。通常ターンでは読まなくてよい。`context_*.md` で参照を求められた場合、または挙動が曖昧な場合のみ読む。

## 共通原則

- 毎ターンの主入力は `session/msg/context_<agent>_<turn>.md` とする
- `session/state/` を次ターン実行用の最小状態の正本とする
- `session/log.md` はリプレイ用の完全記録として保持する
- 通常ターンでは `session/log.md` を再読しない
- 補助ファイルは `context_*.md` の `read_if_needed` に挙がったものだけ読む

## セッション構成

```text
session/
  log.md
  public_scene_<N>.md
  system_feedback.md
  state/
    session.json
    gm.md
    pc1.md
    pc2.md
    pc3.md
  msg/
    context_<agent>_<turn>.md
    gm_<turn>.md
    pc1_<turn>.md
    ...
```

## context ファイル

推奨セクション：

- `role`
- `scene`
- `objective`
- `current_state`
- `recent_events`
- `public_delta_start_line`
- `public_delta_end_line`
- `public_delta`
- `open_threads`
- `read_if_needed`
- `output`

`read_if_needed` は必要時のみ読む補助ファイル一覧である。
`public_delta` は、その担当がまだ読んでいない公開発言・公開描写だけを載せる。

`public_delta_start_line`, `public_delta_end_line` は、`session/public_scene_<N>.md` のどの行範囲を差分として切り出したかを示す。差分が空なら両方とも直前の `last_line` と同値でよい。

`public_delta` の切り出し規則：

- 主キーは `session/public_scene_<N>.md` の行番号とする
- フォールバックとして各公開イベントに `event_id` を付ける
- 通常は `last_line` 以降の差分を渡す
- `context` には `public_delta_start_line` と `public_delta_end_line` を明記する
- 行編集や再整形で行番号が信用できない場合は、`last_event_id` の次から復元する

## state 更新方針

- state は全文要約ではなく最小継続情報を保存する
- 直近詳細は最大 2 件までに抑える
- 古い内容は 1 行に圧縮する
- `session/state/session.json` は共有の機械可読状態を持つ
- `session/state/session.json` には各担当の `public_cursor` を持たせる

`session/state/session.json` の最小項目例：

```json
{
  "scene": 1,
  "turn": 4,
  "location": "旧礼拝堂",
  "open_threads": ["祭壇の血痕", "地下室の鍵"],
  "public_flags": { "altar_opened": false },
  "next_expected": "GM",
  "public_cursor": {
    "gm": { "last_line": 18, "last_event_id": "E018" },
    "pc1": { "last_line": 18, "last_event_id": "E018" },
    "pc2": { "last_line": 15, "last_event_id": "E015" },
    "pc3": { "last_line": 18, "last_event_id": "E018" }
  }
}
```

`public_cursor` の運用規則：

- 通常更新: その担当へ渡した `public_delta` の末尾行を `last_line` に保存する
- 同時に、末尾イベントの `event_id` を `last_event_id` に保存する
- `session/public_scene_<N>.md` を追記以外で触った場合は、次回差分生成で `last_event_id` を優先して再同期する
- 行番号とイベント ID が両方失効したときだけ、当該シーンの公開要約から再構築する

公開共有の運用：

- GM の公開描写、PC の公開発言、公開裁定は `session/public_scene_<N>.md` に追記する
- 各担当には全文ではなく未読差分だけを `context` の `public_delta` に載せる
- 長くなったらシーン終了時に state へ短く圧縮する

公開イベントの推奨表記：

```markdown
[E019] GM: 礼拝堂の扉が開き、冷気が流れ込む。
[E020] PC1: 「誰だ、お前は」
[E021] PC2: 祭壇の血痕を調べる。
```

## ログ追記方針

- `session/log.md` への追記は毎ターン 1 回の `cat` リダイレクトで行う
- 既存ログは編集しない
- 1 ターンの追記ブロックに、そのターンの自身の活動を漏れなく含める
- リプレイとして読める流れを保つ
- `system`, `system_feedback`, 進行メモなどのメタ情報は、本文と分離してブロック引用形式で記録する

公開共有の再現は `session/log.md` ではなく `session/public_scene_<N>.md` と `public_delta` で行う。`session/log.md` はリプレイ記録、`session/public_scene_<N>.md` は次ターン入力用の公開正本として役割を分ける。

`system_feedback` がある場合のみ `session/system_feedback.md` にも 1 回の `cat` リダイレクトで追記する。

## next

- `GM`
- `PC名`
- `scene_end`
- `retrospective`
- `session_end`

許可される値は役割と局面に従う。
