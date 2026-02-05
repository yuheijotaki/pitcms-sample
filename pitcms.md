# PitCMS 設定ファイル仕様書

PitCMSはGit-basedのヘッドレスCMSです。pitcms.jsonc（またはpitcms.json）ファイルでコンテンツ構造を定義します。

## 基本構造

```json
{
  "schemaVersion": 1,
  "media": {
    "storagePath": "public/images",
    "publicPath": "/images"
  },
  "collections": [...]
}
```

- schemaVersion: 必須。現在は1のみサポート
- media: 画像アップロード先の設定（任意、両方指定時のみ有効）
  - storagePath: リポジトリ内の保存先パス
  - publicPath: 公開URL用のパス
- collections: コレクション定義の配列

## コレクション定義

```json
{
  "name": "posts",           // 必須: 一意の識別子（英数字、ハイフン）
  "label": "ブログ記事",      // 任意: UI表示名
  "description": "説明文",    // 任意: 説明
  "path": "content/posts",   // 必須: ファイル保存ディレクトリ
  "extension": "md",         // 必須: ファイル拡張子（md または json）
  "singleton": false,        // 任意: true=単一ファイル、false=複数ファイル（デフォルト）
  "generateFilename": false, // 任意: true=ファイル名を自動生成（デフォルト: false）
  "fields": [...],           // 必須: フィールド定義配列
  "layout": [...],           // 任意: UIレイアウト定義
  "pit": {...}               // 任意: CMS UI設定
}
```

### singleton について

- false（デフォルト）: 複数のファイルを持つコレクション（ブログ記事など）
- true: 単一ファイルのみ（サイト設定など）。pathはファイル名（拡張子なし）を指定

### generateFilename について

- false（デフォルト）: ユーザーがファイル名（slug）を入力
- true: ファイル名を自動生成（entry-{ランダム文字列}）。ユーザーがslugを意識しなくてよい場合に便利

## フィールド共通プロパティ

すべてのフィールドで使用可能：

```json
{
  "name": "fieldName", // 必須: フィールド識別子
  "label": "表示名", // 任意: UI表示名
  "note": "入力のヒント", // 任意: フィールド下に表示される補足テキスト
  "required": true // 任意: 必須フィールドか
}
```

## フィールドタイプ

### text - 1行テキスト

```json
{
  "type": "text",
  "name": "title",
  "label": "タイトル",
  "required": true,
  "default": "",
  "minLength": 1,
  "maxLength": 200,
  "pattern": "^[a-z0-9-]+$"
}
```

### textarea - 複数行テキスト

```json
{
  "type": "textarea",
  "name": "description",
  "label": "説明",
  "minLength": 0,
  "maxLength": 1000
}
```

### richtext - リッチテキストエディタ

```json
{
  "type": "richtext",
  "name": "content",
  "label": "本文",
  "disabledFeatures": ["taskList", "codeBlock"]
}
```

disabledFeaturesで無効化できる機能:

- 見出し: heading（全て）, heading1, heading2, heading3, heading4, heading5
- ブロック: bulletList, orderedList, taskList, blockquote, codeBlock, horizontalRule, image
- インライン: bold, italic, underline, strike, code, highlight, link

### number - 数値

```json
{
  "type": "number",
  "name": "order",
  "label": "並び順",
  "default": 0,
  "min": 0,
  "max": 100
}
```

### boolean - チェックボックス

```json
{
  "type": "boolean",
  "name": "draft",
  "label": "下書き",
  "default": true
}
```

### date - 日付/日時

```json
{
  "type": "date",
  "name": "publishedAt",
  "label": "公開日",
  "includeTime": false,
  "default": "now"
}
```

- includeTime: true=日時、false=日付のみ
- default: "now"で新規作成時に現在日時、または固定値（"2024-01-01"）を指定可能

### image - 画像

```json
{
  "type": "image",
  "name": "thumbnail",
  "label": "サムネイル",
  "options": {
    "maxSizeBytes": 10485760
  }
}
```

- options.maxSizeBytes: 最大ファイルサイズ（バイト、上限10MB）

ファイルサイズ制限:

- GitHub（リポジトリ）: 1MB（固定、maxSizeBytesは無視）
- Cloudflare R2: デフォルト5MB、maxSizeBytesで最大10MBまで拡張可能
- Cloudinary: デフォルト5MB、maxSizeBytesで最大10MBまで拡張可能

許可されているファイル形式:

- image/jpeg, image/png, image/gif, image/webp, image/svg+xml

注意: 画像フィールドを使用するにはmedia設定、または外部画像プロバイダ（R2/Cloudinary）の設定が必要です。

### select - 単一選択

```json
{
  "type": "select",
  "name": "category",
  "label": "カテゴリ",
  "options": [
    { "value": "news", "label": "ニュース" },
    { "value": "blog", "label": "ブログ" }
  ],
  "default": "news",
  "multiple": false
}
```

- multiple: true=複数選択可

### checkbox - 複数選択

```json
{
  "type": "checkbox",
  "name": "tags",
  "label": "タグ",
  "options": [
    { "value": "featured", "label": "おすすめ" },
    { "value": "new", "label": "新着" }
  ],
  "default": []
}
```

### relation - 他コレクションへの参照

```json
{
  "type": "relation",
  "name": "author",
  "label": "著者",
  "collection": "authors",
  "multiple": false,
  "displayField": "name"
}
```

- collection: 参照先コレクション名
- multiple: true=複数参照可
- displayField: 表示に使うフィールド

### loop - 繰り返しフィールド

同じ構造のデータを複数入力できるフィールド（著者リスト、FAQ、ギャラリーなど）:

```json
{
  "type": "loop",
  "name": "authors",
  "label": "著者",
  "minItems": 1,
  "maxItems": 5,
  "itemLabel": "name",
  "layout": [["name", "role"], ["avatar"]],
  "fields": [
    { "type": "text", "name": "name", "label": "名前", "required": true },
    { "type": "text", "name": "role", "label": "役割" },
    { "type": "image", "name": "avatar", "label": "アバター" }
  ]
}
```

- fields: 必須。各アイテムのフィールド定義配列
- minItems: 最小アイテム数
- maxItems: 最大アイテム数
- itemLabel: アイテム表示ラベルに使うフィールド名（省略時は番号表示）
- layout: アイテム内のUIレイアウト（省略時は縦積み）

制限事項:

- loopフィールドのネストは不可（loop内にloopは配置できない）
- pit.title, pit.bodyなどには指定不可

保存形式:

- md拡張子: 正式なYAML配列形式
  ```yaml
  authors:
    - name: "田中太郎"
      role: "執筆"
    - name: "鈴木花子"
      role: "監修"
  ```
- json拡張子: JSON配列形式
  ```json
  "authors": [
    { "name": "田中太郎", "role": "執筆" },
    { "name": "鈴木花子", "role": "監修" }
  ]
  ```

## layout - UIレイアウト

フィールドの配置を2次元配列で定義（1行最大3カラム）:

```json
"layout": [
  ["title", "slug"],
  ["category", "tags"],
  ["publishedAt"],
  ["content"]
]
```

- 指定しない場合は全フィールドが縦積み
- pit設定のフィールド（createdAt, updatedAt, draft, order）は自動的に除外される

## pit - CMS UI設定

```json
{
  "pit": {
    "title": "title",
    "body": "content",
    "createdAt": "createdAt",
    "updatedAt": "updatedAt",
    "draft": "draft",
    "order": "order"
  }
}
```

- title: ファイル一覧での表示名に使うフィールド
- body: 本文エリアに表示するフィールド（md拡張子のみ有効）
- createdAt: 作成日時フィールド（新規作成時に自動設定）
- updatedAt: 更新日時フィールド（保存時に自動設定）
- draft: 下書きステータスフィールド
- order: 並び順フィールド（number型必須、ドラッグ&ドロップで並び替え可能）

## ファイル形式

### Markdown (extension: "md")

YAMLフロントマター + 本文:

```markdown
---
title: "記事タイトル"
date: "2024-01-20"
draft: false
---

本文のMarkdownコンテンツ
```

pit.bodyで指定されたフィールドが本文として扱われる。

### JSON (extension: "json")

全フィールドがJSON:

```json
{
  "title": "記事タイトル",
  "date": "2024-01-20",
  "draft": false,
  "content": "本文"
}
```

## 設計ガイドライン

1. フィールド名は英数字とアンダースコアのみ使用
2. 日本語ラベルはlabelプロパティで指定
3. 必須フィールドは明示的にrequired: trueを設定
4. 日付はISO 8601形式（YYYY-MM-DD または YYYY-MM-DDTHH:mm:ssZ）
5. singletonはサイト設定など単一のコンテンツに使用
6. relationは循環参照を避ける
7. 画像フィールドを使用する場合は必ずmedia設定を追加
8. ユーザーがslugを意識する必要がない場合はgenerateFilename: trueを検討

## 完全な例

```json
{
  "schemaVersion": 1,
  "media": {
    "storagePath": "public/images",
    "publicPath": "/images"
  },
  "collections": [
    {
      "name": "posts",
      "label": "ブログ記事",
      "path": "content/posts",
      "extension": "md",
      "fields": [
        {
          "type": "text",
          "name": "title",
          "label": "タイトル",
          "required": true
        },
        {
          "type": "text",
          "name": "slug",
          "label": "スラッグ",
          "required": true,
          "pattern": "^[a-z0-9-]+$",
          "note": "URLに使用される識別子（英小文字、数字、ハイフンのみ）"
        },
        { "type": "textarea", "name": "excerpt", "label": "概要" },
        {
          "type": "richtext",
          "name": "content",
          "label": "本文",
          "required": true
        },
        { "type": "image", "name": "thumbnail", "label": "サムネイル" },
        {
          "type": "select",
          "name": "category",
          "label": "カテゴリ",
          "options": [
            { "value": "tech", "label": "テクノロジー" },
            { "value": "life", "label": "ライフスタイル" }
          ]
        },
        {
          "type": "checkbox",
          "name": "tags",
          "label": "タグ",
          "options": [
            { "value": "featured", "label": "おすすめ" },
            { "value": "popular", "label": "人気" }
          ]
        },
        {
          "type": "relation",
          "name": "author",
          "label": "著者",
          "collection": "authors"
        },
        {
          "type": "loop",
          "name": "faq",
          "label": "よくある質問",
          "itemLabel": "question",
          "fields": [
            {
              "type": "text",
              "name": "question",
              "label": "質問",
              "required": true
            },
            {
              "type": "textarea",
              "name": "answer",
              "label": "回答",
              "required": true
            }
          ]
        },
        { "type": "date", "name": "publishedAt", "label": "公開日" },
        {
          "type": "date",
          "name": "createdAt",
          "label": "作成日時",
          "includeTime": true
        },
        {
          "type": "date",
          "name": "updatedAt",
          "label": "更新日時",
          "includeTime": true
        },
        {
          "type": "boolean",
          "name": "draft",
          "label": "下書き",
          "default": true
        },
        { "type": "number", "name": "order", "label": "並び順", "default": 0 }
      ],
      "layout": [
        ["title", "slug"],
        ["category", "author"],
        ["publishedAt"],
        ["thumbnail"],
        ["excerpt"],
        ["content"]
      ],
      "pit": {
        "title": "title",
        "body": "content",
        "createdAt": "createdAt",
        "updatedAt": "updatedAt",
        "draft": "draft",
        "order": "order"
      }
    },
    {
      "name": "authors",
      "label": "著者",
      "path": "content/authors",
      "extension": "json",
      "fields": [
        { "type": "text", "name": "name", "label": "名前", "required": true },
        { "type": "text", "name": "email", "label": "メールアドレス" },
        { "type": "image", "name": "avatar", "label": "アバター" },
        { "type": "textarea", "name": "bio", "label": "自己紹介" }
      ],
      "pit": { "title": "name" }
    },
    {
      "name": "notes",
      "label": "メモ",
      "description": "シンプルなメモ。ファイル名は自動生成。",
      "path": "content/notes",
      "extension": "md",
      "generateFilename": true,
      "fields": [
        {
          "type": "text",
          "name": "title",
          "label": "タイトル",
          "required": true
        },
        { "type": "richtext", "name": "content", "label": "内容" },
        {
          "type": "date",
          "name": "createdAt",
          "label": "作成日時",
          "includeTime": true
        },
        {
          "type": "date",
          "name": "updatedAt",
          "label": "更新日時",
          "includeTime": true
        }
      ],
      "pit": {
        "title": "title",
        "body": "content",
        "createdAt": "createdAt",
        "updatedAt": "updatedAt"
      }
    },
    {
      "name": "settings",
      "label": "サイト設定",
      "path": "data/settings",
      "extension": "json",
      "singleton": true,
      "fields": [
        {
          "type": "text",
          "name": "siteName",
          "label": "サイト名",
          "required": true
        },
        { "type": "textarea", "name": "description", "label": "サイト説明" },
        { "type": "image", "name": "logo", "label": "ロゴ" },
        { "type": "text", "name": "copyright", "label": "コピーライト" }
      ]
    }
  ]
}
```
