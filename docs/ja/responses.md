<!-- 翻訳時の注意: 見出し（## Note, ### Login and switch result など）は英語のまま
     にしてください。commands.md と output-and-errors.md から `responses.md#...` で
     リンクしており、日本語の見出しにするとアンカーが壊れます（VitePress は見出し
     ID を NFKD 正規化するため、濁点を含む日本語アンカーはリンク先と一致しません）。 -->

# レスポンスリファレンス

[English](../en/responses.md) | [日本語](../ja/responses.md)

各 `rimo` コマンドが返す値をフィールド単位でまとめたリファレンスです。コマンド
自体（構文・フラグ・例）については [コマンド](commands.md) を、出力の取り決めと
エラー形式については [出力とエラー](output-and-errors.md) を参照してください。

## このページの読み方

- 他ページからリンクされる見出しは英語のままにしています（アンカーを GitHub と
  ドキュメントサイトの両方で安定させるためです）。翻訳しないでください。
- **有無** は **常時**（すべてのレスポンスに含まれる）か **任意** のどちらかです。
  任意のフィールドは値がないとき**キーごと省略**されます。`null` や `""` として
  返るわけではないので、空の値と比較するのではなくキーの有無を確認してください。
  唯一の例外が [`Participant.user_id`](#participant) で、常に含まれますが Rimo
  アカウントを持たない話者では `""` になります。
- タイムスタンプは呼び出し側のローカルタイムゾーンの RFC 3339 文字列です。例:
  `"2026-07-05T15:45:20.47178+09:00"`。
- キーの順序は取り決めの一部ではありません。順序に依存しないでください。

## 各コマンドが返すもの

| コマンド | 形式 | 形状 |
|---------|--------|-------|
| [`rimo auth login`](commands.md#rimo-auth-login) | JSON | [ログインと切り替えの結果](#login-and-switch-result) |
| [`rimo auth logout`](commands.md#rimo-auth-logout) | JSON | [ログアウト結果](#logout-result) |
| [`rimo auth status`](commands.md#rimo-auth-status) | JSON | [認証ステータス](#auth-status) |
| [`rimo auth switch`](commands.md#rimo-auth-switch) | JSON | [ログインと切り替えの結果](#login-and-switch-result) |
| [`rimo note list`](commands.md#rimo-note-list) | JSON | `{notes: `[Note](#note)`[], next_page_token}` |
| [`rimo note get`](commands.md#rimo-note-get) | JSON | `{note: `[Note](#note)`}` |
| `rimo note get --list-documents` | JSON | `{documents: `[Document](#document)`[]}` |
| `rimo note get --transcript` / `--document` / `--full` / `--meeting-chat` / `--document-id` | プレーンテキスト | [Note content](#note-content) |
| [`rimo note search`](commands.md#rimo-note-search) | JSON | `{notes: `[Search result](#search-result)`[], total_count}` |
| [`rimo note ask`](commands.md#rimo-note-ask) | プレーンテキスト | [Ask answer](#ask-answer) |
| [`rimo note create`](commands.md#rimo-note-create) | JSON | `{note: `[Note](#note)`, document: `[Document](#document)`}` |
| [`rimo note append`](commands.md#rimo-note-append) | JSON | `{document: `[Document](#document)`}` |
| [`rimo team list`](commands.md#rimo-team-list) | JSON | `{teams: `[Team](#team)`[], next_page_token}` |
| [`rimo version`](commands.md#rimo-version) | プレーンテキスト | バージョン 1 行 |
| [`rimo upgrade`](commands.md#rimo-upgrade) | プレーンテキスト | ステータス 1 行 |
| [`rimo mcp`](commands.md#rimo-mcp) | — | stdio 上の MCP サーバーであり、データを返すコマンドではありません。[MCP](mcp.md) を参照。 |

`rimo login` と `rimo logout` は `rimo auth login` / `rimo auth logout` の
エイリアスで、同じ形状を返します。

どのコマンドも失敗する可能性があり、その場合は
[エラーオブジェクト](output-and-errors.md#エラー形式) を出力して終了コード 1 で終わります。

## Envelopes

すべての JSON レスポンスは、ペイロード（本体）をコンテナキーで包み、スカラーの
メタデータをその隣に並べます。この包みをエンベロープと呼びます。

| エンベロープ | 返すコマンド |
|----------|-------------|
| `{notes: [...], next_page_token: "..."}` | `note list` |
| `{notes: [...], total_count: <int>}` | `note search` |
| `{teams: [...], next_page_token: "..."}` | `team list` |
| `{note: {...}}` | `note get` |
| `{documents: [...]}` | `note get --list-documents` |
| `{note: {...}, document: {...}}` | `note create` |
| `{document: {...}}` | `note append` |
| `{accounts: [...], active_account: "..."}` | `auth status` |

`--fields` と `--excludes` はエンベロープの**内側のレコード**に適用され、スカラーの
メタデータ（`next_page_token`、`total_count`、`active_account`）は常に保持されます。
つまり `rimo note list --fields id,title` は各ノートを絞り込みつつ
`next_page_token` は残します。詳しくは
[出力とエラー](output-and-errors.md) の「フィールドのフィルタリング」を参照して
ください。

`next_page_token` はカーソルです。`--page-token` に渡すと次のページを取得でき、
返らなくなったら終わりです。

## Note

`note get` と `note create` は `{note: ...}` の中で、`note list` は
`{notes: [...]}` の中で返します。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `id` | string | 常時 | ノートの ID。`note get`、`note append` などのノートコマンドに渡します。 |
| `title` | string | 常時 | ノートのタイトル。 |
| `state` | string | 常時 | 録画・文字起こしパイプライン上の状態。[ノートの state](#note-states) を参照。 |
| `duration` | integer | 常時 | メディアの長さ（**ミリ秒**）。録音のないノートは `0`。 |
| `created_at` | string | 常時 | ノートの作成日時。 |
| `updated_at` | string | 常時 | ノートの最終更新日時。`note list --updated-since` で絞り込めます。 |
| `user_id` | string | 任意 | ノートの所有者のユーザー ID。 |
| `organization_id` | string | 任意 | ノートが属する組織の ID。個人ノートでは省略されます。 |
| `team_id` | string | 任意 | ノートが属するチームの ID。個人ノートでは省略されます。`note list --team` に渡せます。 |
| `locale` | string | 任意 | 文字起こしの言語設定。例: `ja-JP`。 |
| `media_type` | string | 任意 | 添付メディアの種別。例: `audio`、`none`。 |
| `source` | string | 任意 | ノートの作成経路。例: `none`、`zoom_import`。連携の追加に伴い値は増えます。 |
| `held_at` | string | 任意 | 会議の開催日時。`note list --since` / `--until` で絞り込めます。 |
| `memo` | string | 任意 | ノートに追加された自由記述のメモ。 |
| `share_mode` | string | 任意 | URL 共有の設定: `nothing`（共有しない）、`view`、`edit`。 |
| `custom_template_id` | string | 任意 | 議事録に使ったカスタムテンプレートの ID。 |
| `webhook_url` | string | 任意 | 処理完了時に呼ばれる Webhook。 |
| `document_markdown` | string | 任意 | メインのドキュメントの Markdown。下の注意を参照。 |
| `tags` | string[] | 任意 | ノートに付与されたタグ名。下の注意を参照。 |
| `participants` | [Participant](#participant)[] | 任意 | 会議の参加者。下の注意を参照。 |

**`note list` と `note get` はメタデータのみを返します。** `document_markdown`、
`tags`、`participants` は返しません。ノートのコンテンツの読み込みが重い処理であり、
多くの呼び出し側はメタデータだけを必要とするためです。コンテンツを読むには専用の
フラグ（`note get --document`、`--transcript`、`--full`、`--list-documents`）を
使ってください。これらは [プレーンテキスト](#note-content) または
[Document](#document) を返します。

### Note states

`state` は録画・音声変換・音声認識を通したノートの状態を表します。値は増えうる
オープンな集合として扱い、必要な値だけを判定し、未知の値も安全に扱えるように
してください。

| state | 意味 |
|-------|---------|
| `NOTE_CREATED` | 音源を登録せず、ノートを作成しただけの状態（`note create` など）。 |
| `NOT_SCHEDULED` | Bot による自動録画が予約されていない状態。 |
| `RECORDING_SCHEDULED` | Bot による自動録画が予約済みの状態。 |
| `RECORDING_WAITING` | Bot が会議に参加し、録画開始（ホストの承認など）を待っている状態。 |
| `RECORDING_STARTED` | Bot による自動録画が進行中の状態。 |
| `RECORDING_PAUSED` | Bot による自動録画が一時停止中の状態。 |
| `RECORDING_DONE` | Bot による自動録画が終了した状態。 |
| `RECORDING_FAILED` | Bot による自動録画が失敗し、基本的にリトライできない状態。 |
| `MIC_RECORDING_REQUESTED` / `MIC_RECORDING_STARTED` / `MIC_RECORDING_DONE` | マイク録音が未開始／進行中／終了の状態。 |
| `HARDWARE_RECORDING_REQUESTED` / `HARDWARE_RECORDING_STARTED` | ハードウェア録音が未開始／進行中の状態。 |
| `TRANSCRIPTION_DONE` | リアルタイム文字起こしは完了したが、メディアがアップロードされていない状態。 |
| `MEDIA_PREPARING` | 動画・音声のアップロード中の状態。 |
| `VC_WAITING` | 音声がアップロードされ、解析許可を待っている状態。 |
| `VC_REQUESTED` | 音声変換の処理中。 |
| `VC_ERROR` | 音声変換処理が失敗した状態。 |
| `ASR_PROCESSING` | 音声認識の処理中。 |
| `ASR_ERROR` | 音声認識処理が失敗した状態。 |
| `ASR_DONE` | 音声認識処理が完了し、文字起こしと議事録が利用できる状態。 |

## Participant

[Note](#note) の `participants` 配列の中で返る会議の参加者です。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `user_id` | string | 常時 | 参加者の Rimo ユーザー ID。Rimo アカウントを持たない話者では空になります。`note search --participant` に渡せます。 |
| `id` | string | 任意 | 内部の参加者 ID。 |
| `name` | string | 任意 | 表示名。 |
| `email` | string | 任意 | メールアドレス。 |
| `calendar_name` | string | 任意 | カレンダー予定から作成されたノートでの、予定上の参加者名。 |
| `speaker_id` | string | 任意 | 話者分離の識別子。文字起こしの各行をこの参加者に対応づけるのに使います。 |

## Document

ノートに紐づくドキュメント（議事録）です。1 つのノートに複数（翻訳版、別テンプレート
など）持つことができ、`primary` がメインのドキュメントを示します。

`note get --list-documents` は `{documents: [...]}` の中で、`note create` と
`note append` は `{document: ...}` の中で返します。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `id` | string | 常時 | ドキュメントの ID。`note append` や `note get --document-id` に渡します。 |
| `note_id` | string | 常時 | このドキュメントが属するノートの ID。 |
| `title` | string | 常時 | ドキュメントのタイトル。 |
| `primary` | boolean | 常時 | ノートのメインの議事録なら `true`。 |
| `created_at` | string | 常時 | ドキュメントの作成日時。 |
| `updated_at` | string | 常時 | ドキュメントの最終更新日時。 |
| `export_markdown` | string | 任意 | ドキュメント本文（Markdown）。 |
| `locale` | string | 任意 | ドキュメントの言語。例: `ja-JP`。 |
| `category` | string | 任意 | ドキュメントの種別。例: `agenda`、`summary`、`translation`。 |
| `template_mode` | string | 任意 | 生成に使われたテンプレート。例: `minutes`。 |
| `custom_template_id` | string | 任意 | 使われたカスタムテンプレートの ID。 |

**`--list-documents` はすべてのドキュメントの `export_markdown` を丸ごと含みます。**
そのためレスポンスはノートのコンテンツ量に比例して大きくなり、翻訳版がいくつかある
ノートでは数十 KB になることもあります。ID とタイトルだけが必要なときは本文を落として
ください:

```bash
rimo note get <note_id> --list-documents --excludes export_markdown
```

## Team

`team list` が `{teams: [...]}` の中で返します。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `id` | string | 常時 | チームの ID。`note list --team`、`note search --team`、`note create --team` に渡せます。 |
| `name` | string | 常時 | チーム名。Rimo アプリ上で見えるフォルダ名です。 |
| `category` | string | 任意 | チームフォルダなら `"team"`。`"organization"` は組織自身のフォルダで、`--include-organization` を指定したときだけ返ります。`id` は組織の ID です。 |
| `is_private_channel` | boolean | 常時 | チームのメンバーだけがノートを閲覧できる場合に `true`。 |
| `member_ids` | string[] | 常時 | チームに所属するユーザーの ID。 |
| `created_at` | string | 常時 | チームの作成日時。 |
| `updated_at` | string | 常時 | チームの最終更新日時。 |
| `parent_id` | string | 任意 | 親チームの ID。 |
| `description` | string | 任意 | チームの説明。 |

## Search result

`note search` が `{notes: [...], total_count}` の中で返す検索ヒットです。完全な
[Note](#note) ではなく、検索インデックスが返した内容だけを持ちます。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `id` | string | 常時 | ノートの ID。`note get` に渡せます。 |
| `title` | string | 任意 | ノートのタイトル。 |
| `held_at` | string | 任意 | 会議の開催日時。 |
| `created_at` | string | 任意 | ノートの作成日時。 |
| `owner_name` | string | 任意 | ノート所有者の表示名。`--mode=filter` のみ。 |
| `snippet` | object | 任意 | ヒット箇所の抜粋。`--mode=filter` のみ。下記参照。 |

`--mode=filter` では、`total_count` は権限フィルター前の生のマッチ数です。現在の
ページではなく結果全体を数えており、これが `--page` / `--per` によるページ送りを
成り立たせています。`--mode=semantic` では返却された一意なノート数になります。

**`snippet`** はヒット箇所の抜粋を持ち、マッチした語は強調表示のため HTML タグで
囲まれます。各キーは文字列の配列で、4 つのキーはすべて存在します（その箇所が
マッチしなかった場合は空配列）。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `transcripts` | string[] | 常時 | 文字起こしからの抜粋。 |
| `headings` | string[] | 常時 | 見出しからの抜粋。 |
| `annotations` | string[] | 常時 | アノテーションからの抜粋。 |
| `document_markdowns` | string[] | 常時 | ドキュメント本文からの抜粋。 |

## 認証系のレスポンス

### Login and switch result

`auth login`（および `rimo login`）と `auth switch` が返します。すべての
フィールドが常に含まれます。

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `status` | string | `auth login` では `logged_in`、`auth switch` では `switched`。 |
| `alias` | string | アカウントのエイリアス。`--account` で使います。 |
| `email` | string | サインインしたメールアドレス。 |
| `name` | string | 表示名。未設定ならメールアドレス、次にユーザー ID にフォールバック。 |
| `org` | string | 組織名。未設定なら組織 ID、次に `Personal` にフォールバック。 |

### Logout result

`auth logout`（および `rimo logout`）が返します。すべてのフィールドが常に
含まれます。

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `status` | string | 常に `logged_out`。 |
| `alias` | string | ログアウトしたエイリアス。 |
| `active_account` | string | ログアウト後のアクティブなアカウント。自動昇格はしないため、アクティブだったアカウントをログアウトした場合は空になります。 |

### Auth status

`auth status` が返します。

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `active_account` | string | 常時 | アクティブなアカウントのエイリアス。 |
| `accounts` | object[] | 常時 | 保存されているすべてのアカウント。下記参照。 |
| `active_credential` | string | 任意 | 環境変数が設定済みアカウントを上書きしている場合のみ含まれます: `env:RIMO_API_KEY` または `env:RIMO_TOKEN`。含まれるときは、`active_account` ではなくその認証情報がリクエストを認証しています。 |
| `api_key_hint` | string | 任意 | キーをマスクした表示。`env:RIMO_API_KEY` のときのみ含まれます。 |

`accounts` の各要素:

| フィールド | 型 | 有無 | 説明 |
|-------|------|----------|-------------|
| `alias` | string | 常時 | アカウントのエイリアス。`--account` で使います。 |
| `name` | string | 常時 | 表示名。 |
| `org` | string | 常時 | 組織名。 |
| `active` | boolean | 常時 | アクティブなアカウントなら `true`。 |
| `token_status` | string | 常時 | `valid`、`expiring_soon`、`expired`、`unknown`（有効期限を判定できなかった場合）。 |
| `email` | string | 任意 | アカウントのメールアドレス。 |

`auth status` は報告の前に期限切れ・期限間近のトークンを更新するため、
`token_status` は認証情報ストアに最後に書かれた値ではなく現在の状態を反映します。

## プレーンテキストの出力

以下のコマンドは、JSON で包むとかえって扱いにくくなるため、成功時にプレーンテキスト
を出力します。エラーは JSON のままです —
[出力とエラー](output-and-errors.md#エラー形式) を参照してください。

### Note content

`note get` にコンテンツ系フラグを付けると、ノートの中身をそのまま出力します。

| フラグ | 出力 |
|------|--------|
| `--transcript` | 文字起こしのセグメントごとに 1 行、`Speaker: content` 形式。話者を特定できなかったセグメントは本文のみを出力します。空のセグメントはスキップされます。 |
| `--document` | メインのドキュメントの Markdown。先頭に `# <title>` が付きます。 |
| `--full` | 文字起こし、空行、メインのドキュメントの順。 |
| `--document-id <id>` | 指定したドキュメントの Markdown。`--document` と同じ形式です。 |
| `--meeting-chat` | ウェブ会議チャットのメッセージごとに 1 行、`[HH:MM] sender: text` 形式。`[HH:MM]` と `sender:` は取得できない場合に省略され、空のメッセージはスキップされます。 |
| `--timestamps` | `--transcript` / `--full` と併用すると、文字起こしの各行に `[HH:MM:SS]` を付与。**録音開始からの経過時間**であり、実時刻ではありません。開始時刻のないセグメントには付きません。 |

文字起こしやチャットのないノートでは何も出力せず、終了コード 0 で終わります。

### Ask answer

`note ask` は回答を生成されるそばからストリーミングし、続けて `Sources:` ブロック
（回答本文にインラインの引用は含まれないため、ここが正式な引用元の表示です）、
`Fetch a note:` ブロックを出力します。例外的にこの 3 つはすべて **stdout** に
出力されるため、stdout が機械可読でない唯一のコマンドです。実際の例は
[`rimo note ask`](commands.md#rimo-note-ask) を参照してください。

### Version and upgrade

どちらも stdout に 1 行だけ出力します。正確な文字列は
[`rimo version`](commands.md#rimo-version) と
[`rimo upgrade`](commands.md#rimo-upgrade) を参照してください。

## Dry-run output

書き込み系コマンド（`note create`、`note append`）に `--dry-run` を付けると、
リクエストは送信されません。レスポンス形状の**代表的な例**に `"dry_run": true` を
加えたものが返ります:

```json
{
  "document": { "id": "doc_abc123", "primary": true, "...": "..." },
  "dry_run": true
}
```

値は API スキーマ由来のプレースホルダーであり、入力内容のプレビューではありません。
レスポンスの形状の確認とコマンドが正しく解釈されるかの確認に使い、作成されるノートの
中身を見る用途には使えません。

読み取り系コマンドで `--dry-run` を指定すると
`dry-run not supported for read operations` で拒否されます。
