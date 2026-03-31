# オーケストレーター仕様書
*更新日：2026-03-31*

## 役割：バトン渡し専任

メッセージの**内容は読まない**。ファイル名と `next:` だけを見て次の人に渡す。

---

## ディレクトリ構成

```
state/   gm.md, pc1.md, pc2.md, pc3.md, session.json
logs/    session_log.md
msg/     ターン間メッセージファイル（セッション終了後に削除）
```

---

## セッション開始

```
1. msg/ ディレクトリを作成する

2. GM をスポーン（run_in_background:true, name:"GM"）
   プロンプト：state/gm.md の内容 + 「準備ができたら "ready" と返してください」

3. PC1〜3 をスポーン（run_in_background:true, name:"PC1"/"PC2"/"PC3", model:haiku）
   プロンプト：state/pcN.md の内容 + 「準備ができたら "ready" と返してください」

4. 全員の "ready" を確認してから SendMessage → GM
   「シーンを開始してください。出力先と next を返してください」
```

---

## ターンの回し方

エージェントは応答として以下を返す：

```
wrote: msg/gm_4.md
next: PC2
secret: PC1 → msg/gm_secret_pc1_4.md   ← 秘匿がある場合のみ
```

オーケストレーターはこれを受け取ったら：

1. `next:` で指定されたエージェントを特定する
2. そのエージェントへ SendMessage する：

```
読むファイル：msg/gm_4.md [, msg/gm_secret_pc1_4.md]
出力先：msg/pc1_5.md
指示：発言・行動を書いて logs/session_log.md に追記し、next を返してください
```

3. `secret:` がある場合は、指定 PC へのメッセージにのみそのファイル名を含める

---

## シーン終了・終了の判断

`next: scene_end` が届いたら（GMが判断して返す）、全エージェントへ：

```
SendMessage：「シーン終了。state/ の担当ファイルを更新して "done" と返してください」
```

全員の `"done"` を確認したら次シーンへ。

---

## 振り返り

`next: retrospective` が届いたら GM へ SendMessage する：

```
「振り返りを書いてください。書き終えたら wrote: と next: を返してください」
```

以降はバトン渡しと同じ手順でエージェント間を回す。
全員が振り返りを書き終えて `next: session_end` が届いたら：

1. `logs/session_log.md` の末尾に各振り返りファイル名をリスト追記する（内容は読まない）
2. GM へ `「振り返り完了。セッションを終了してください」` と SendMessage する

---

## セッション終了

`next: session_end` が届いたら、全エージェントへ：

```
SendMessage：「セッション終了。最終状態を更新して "done" と返してください」
```

全員の `"done"` を確認して完了。
