# TRPG エージェント補助テンプレート

この文書は完全な例が必要なときだけ読む。

## GM 出力ファイル

```markdown
## speech

[全員への描写・裁定・NPC発言。要約ではなく送信本文そのものを書く]
[場面転換時は冒頭にシーン見出しを置いてよい]

## secret:PC名

[必要時のみ。実際の秘匿内容は別ファイルに切り出す]

## state_changes

[JSON 形式の変化]

## system

[次シーンへの引き継ぎ、進行メモ]

## system_feedback

[必要時のみ]
```

## PC 出力ファイル

```markdown
## speech

[発言・行動・必要なメタ発言。要約ではなく送信本文そのものを書く]
[話題や相手が変わるたびに段落を分ける]

## inner

[内心]

## system_feedback

[必要時のみ]
```

## session/log.md の最小要素

- `## シーン開始: [シーン名]` と `## シーン終了: [シーン名]` を GM が明記する
- ターン見出し
- そのターンの導入
- 発言と行動、または描写と応答
- 内心または秘匿情報
- 判定と裁定、または状態変化
- システム情報、進行メモ、メタ情報
- 次への接続

ログ本文はダイジェストではなく、そのターンに送信した本文を読みやすく整形したものにする。台詞、描写、判定、注釈は段落を分ける。

システム情報やメタ情報は、本文とは別にブロック引用で整形する。

```markdown
> **【System】**
> 瘴気濃度 2。次は PC2 に応答を促す。

> **【Meta】**
> プレイヤー確認: ここでは探索宣言として扱う。
```

## session/public*scene*<N>.md の役割

- 同卓参加者全員が共有する公開発言・公開描写・公開裁定だけを集約する
- 次ターンでは全文を再送せず、未読差分だけを `context` の `public_delta` に載せる
- 差分管理は行番号を主とし、イベント ID をフォールバックにする
- 秘匿情報、内心、system の私的メモは入れない

公開差分の例：

```markdown
[E019] GM: 礼拝堂の扉が開き、冷気が流れ込む。
[E020] PC1: 「誰だ、お前は」
[E021] PC2: 祭壇の血痕を調べる。
[E022] GM: 知覚判定を要求。
```

`session/state/session.json` の `public_cursor` 例：

```json
{
  "pc2": { "last_line": 21, "last_event_id": "E021" }
}
```

`context` の差分範囲例：

```markdown
## public_delta_start_line

22

## public_delta_end_line

25

## public_delta

[E022] GM: 知覚判定を要求。
[E023] PC3: 「祭壇に近づくな」
```

## GM state の推奨項目

```markdown
## current_scene

[シーン名、場所、進行段階]

## open_threads

- [未解決事項]

## npc_and_hazards

- [NPC 意図、危険、伏線]

## recent_decisions

- [直近2件までの裁定結果]

## pending_for_next_turn

- [次ターンで裁定すべきこと]
```

## PC state の推奨項目

```markdown
## identity

[動機・口調・対人姿勢]

## current_emotion

[現在の感情、迷い、短期目標]

## known_facts

- [この PC が知っている事実]

## recent_actions

- [直近2件までの自分の行動]

## pending_for_next_turn

- [次ターンで言いたいこと、確かめたいこと]
```
