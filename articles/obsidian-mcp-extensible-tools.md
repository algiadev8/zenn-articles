---
title: "ObsiScripta Bridge - Obsidian用の拡張可能なMCPサーバープラグイン"
emoji: "🔌"
type: tech
topics:
  - obsidian
  - mcp
  - ai
  - plugin
published: false
---

## はじめに

Obsidianのvaultに対してAIクライアント（Claude Desktop、Cursorなど）からアクセスするためのMCPサーバープラグインを作りました。

**最大の特徴は、JavaScriptでツールを自由に拡張できること**です。

https://github.com/daichi-629/obsidian-obsiscripta-mcp

## なぜ作ったか

既存のObsidian MCP実装はいくつかありますが、提供されるツールが固定されていて「こういう操作がしたい」と思っても対応できないことがありました。

そこで**スクリプトでツールを自由に追加できる**設計にしました。vault内の`mcp-tools/`フォルダにJavaScriptファイルを置くだけで、そのツールがMCP経由で利用可能になります。

## 特徴

### 1. スクリプトでツールを自由に定義

`mcp-tools/`フォルダにJSファイルを置くだけ。ホットリロード対応なので、Obsidianを再起動せずにツールを追加・修正できます。

```javascript
// mcp-tools/my-tool.js
export default {
  name: "my_custom_tool",
  description: "自分だけのカスタムツール",
  inputSchema: {
    type: "object",
    properties: {
      query: { type: "string", description: "検索クエリ" }
    },
    required: ["query"]
  },
  handler: async (args) => {
    // Obsidian APIにフルアクセス可能
    const files = app.vault.getMarkdownFiles();
    const matched = files.filter(f => f.path.includes(args.query));

    return {
      content: [{
        type: "text",
        text: `Found ${matched.length} files`
      }]
    };
  }
};
```

### 2. Obsidian APIへのフルアクセス

スクリプト内から`app`オブジェクト経由でObsidianの全機能にアクセスできます：

- `app.vault` - ファイル操作（読み書き、作成、削除）
- `app.workspace` - ワークスペース操作
- `app.fileManager` - フロントマター編集など
- `app.metadataCache` - メタデータキャッシュ

サンドボックスなしで動作するため、Obsidianでできることは何でもスクリプトから実行可能です。

### 3. プラグインAPIとの連携

インストール済みのObsidianプラグインのAPIも利用できます：

**Dataview** - DQLクエリの実行
```javascript
// dv（DataviewのAPI）が利用可能
const result = await dv.query("LIST FROM #tag");
```

**Omnisearch** - 全文検索
```javascript
// グローバルのomnisearchが利用可能
const results = await omnisearch.search("キーワード");
```

### 4. ローカル専用・認証なし

セキュリティ的な割り切りとして：
- localhost限定（外部からアクセス不可）
- 認証なし（ローカル環境での利便性優先）
- デスクトップ版のみ（モバイル非対応）

## セットアップ

### プラグインのインストール

BRATを使う場合：
1. BRATをインストール
2. `https://github.com/daichi-629/obsidian-obsiscripta-mcp` を追加
3. ObsiScripta Bridgeを有効化

### Claude Desktopとの接続

1. [Releases](https://github.com/daichi-629/obsidian-obsiscripta-mcp/releases)からstdio bridgeバイナリをダウンロード
2. Claude Desktopの設定に追加：

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "/path/to/obsidian-mcp",
      "env": {
        "OBSIDIAN_MCP_HOST": "127.0.0.1",
        "OBSIDIAN_MCP_PORT": "3000"
      }
    }
  }
}
```



## ユースケース例

スクリプトで何でもできるので、例えば：

- **独自のノート作成ルール**を実装したツール
- **特定のフォルダ構造**に従ったファイル操作
- **カスタムメタデータ**の一括更新
- **外部サービス連携**（APIキーをスクリプト内に持てる）
- **Dataviewクエリ**をラップした便利ツール

自分のワークフローに合わせて、必要なツールを自分で作れます。

## 技術的なポイント

### ホットリロード

`mcp-tools/`フォルダを監視し、ファイルの追加・変更を検知して自動でツールを再読み込みします。開発中のイテレーションが高速です。

### TypeScriptサポート

TSファイルもそのまま配置可能です。サーバー側でトランスパイルされるため、事前のビルドは不要です。

ただし、Obsidian側で提供される型定義（`TFile`、`App`など）がインポートして使えるかは未精査です。現状はスクリプト内で独自にinterface定義するのが確実です。

## 今後の展望

### Remote MCP対応

現在はローカルホスト限定ですが、認証機構を追加してリモートからのアクセスに対応予定です。これによりモバイルやリモート環境からもvaultを操作できるようになります。

### 共通関数ライブラリ

よく使うパターン（フロントマター操作、パス正規化、日付処理など）を共通ライブラリとして提供予定です。各ツールで同じコードを書く必要がなくなります。

```javascript
// 将来のイメージ
import { normalizePath, updateFrontmatter } from "obsiscripta/helpers";
```

### ツール作成ヘルパー

ボイラープレートを減らすためのヘルパー関数を検討中です。

```javascript
// 将来のイメージ
import { defineSimpleTool } from "obsiscripta/define";

export default defineSimpleTool({
  name: "my_tool",
  description: "説明",
  args: { query: "string" },
  handler: async ({ query }) => `結果: ${query}`
});
```

inputSchemaの手書きや、レスポンス形式の組み立てを省略できるようにする予定です。

### Templater連携の改善

現状、Templaterの`tp.create_new_note_from_template()`でテンプレートからノートを作成できますが、テンプレートにパラメータを渡す方法がありません（`tp.user`経由でのパラメータ受け渡しがMCPコンテキストでは動作しない）。

将来的には、MCPツールからTemplaterテンプレートにパラメータを渡せる仕組みを検討中です。

## 注意事項

- **セキュリティ**: スクリプトはサンドボックスなしで動作します。信頼できるスクリプトのみ配置してください。
- **モバイル非対応**: デスクトップ版Obsidianでのみ動作します。
- **認証なし**: ローカルホストのみですが、同一マシン上の他プロセスからはアクセス可能です。

## まとめ

ObsiScripta Bridgeは「決まったツールを使う」のではなく「必要なツールを自分で作る」アプローチを取っています。

Obsidianのvaultをどう操作したいかは人それぞれ。プラグインが提供する固定機能に縛られず、自分のワークフローに最適化したツールを作れるのが強みです。

興味があれば試してみてください。Issue/PRも歓迎です。

https://github.com/daichi-629/obsidian-obsiscripta-mcp
