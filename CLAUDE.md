# marp.nvim - Claude Code Configuration

このファイルは、marp.nvimプロジェクトの開発をClaude Codeで効率的に行うための設定とガイドラインです。

## プロジェクト概要

marp.nvimは、[Marp](https://marp.app/)（Markdown Presentation Ecosystem）用のNeovimプラグインです。マークダウンファイルからプレゼンテーションを作成し、リアルタイムプレビューや各種形式でのエクスポート機能を提供します。

## ディレクトリ構造

```
marp.nvim/
├── lua/
│   └── marp.lua          # メインのLuaモジュール
├── plugin/
│   └── marp.vim          # Vimコマンド定義
├── doc/
│   ├── marp.txt          # ヘルプドキュメント
│   └── tags              # ヘルプタグ
├── README.md             # プロジェクト説明（日英両対応）
├── LICENSE               # ライセンス
├── Makefile              # ビルド・テスト自動化
└── CLAUDE.md             # このファイル
```

## 開発ワークフロー

### 品質チェックコマンド

開発時は以下のコマンドを使用してコードの品質を保ちます：

```bash
# すべてのチェックを実行
make check

# Luaチェックのみ
make lint

# コードフォーマットチェック
make format-check

# コードフォーマット実行
make format

# テスト実行
make test
```

### CI/CD連携

**重要**: コミット前に必ずローカルで品質チェックを実行してください。

CIでは以下のチェックが自動実行されます：
1. **luacheck**: Lua コードの静的解析（.luacheckrc設定）
2. **stylua**: コードフォーマットチェック（.stylua.toml設定）

ローカルでCIと同じチェックを実行：
```bash
# CI環境と同じチェックを実行
make check

# 個別実行
luacheck lua/ plugin/                              # luacheckのみ
stylua --check lua/ plugin/ --glob '**/*.lua'      # styluaチェックのみ
stylua lua/ plugin/ --glob '**/*.lua'              # stylua自動修正
```

設定ファイル：
- `.luacheckrc`: luacheck設定（Lua 5.1、vim globals、行長制限なし）
- `.stylua.toml`: stylua設定（120文字幅、2スペースインデント、Unix改行）
- `.github/workflows/lint.yml`: CI設定（push/PR時に自動実行）

### コーディング規約

- **Lua**: luacheckとstyluaの設定に従う
- **Vim script**: 最小限に抑え、主要ロジックはLuaで実装
- **コメント**: 機能説明は控えめに、必要な場合のみ記述
- **エラーハンドリング**: vim.notifyを使用してユーザーに適切なフィードバック

## 主要機能

### コアコマンド
- `:MarpWatch` - ライブプレビュー開始
- `:MarpStop` - プレビュー停止  
- `:MarpExport [format]` - エクスポート（html/pdf/pptx/png/jpeg）
- `:MarpTheme [theme]` - テーマ設定
- `:MarpSnippet [name]` - スニペット挿入
- `:MarpDebug` - 診断実行

### 重要な実装詳細

1. **プロセス管理**: `M.active_processes`でバッファごとのMarpプロセスを追跡
2. **自動クリーンアップ**: バッファ削除時にプロセスを自動停止
3. **デュアルモード**: サーバーモード(-s)とウォッチモード(--watch)をサポート
4. **デバッグ支援**: 詳細ログとエラー診断機能

## ファイル固有の注意点

### lua/marp.lua
- メイン実装ファイル（714行）
- 設定管理、プロセス制御、UI統合をすべて担当
- ANSI エスケープシーケンスのクリーニング処理を含む
- デバッグモードの詳細な出力制御

### plugin/marp.vim  
- Vimコマンド定義のみ（42行）
- タブ補完機能を含む
- Luaモジュールへのブリッジ役

## 開発時の推奨アプローチ

1. **機能追加**: まずlua/marp.luaで実装、必要に応じてplugin/marp.vimにコマンド追加
2. **バグ修正**: デバッグモードを活用して問題箇所を特定
3. **リファクタリング**: 品質チェックコマンドで検証しながら実施
4. **テスト**: minimal_init.luaを使用した軽量テスト環境を活用

## 外部依存関係

- **Neovim 0.5+**
- **Marp CLI**: npmパッケージ、npx経由でも利用可能
- **開発ツール**: luacheck, stylua（品質管理用）

## 設定例

```lua
require('marp').setup({
  marp_command = "/opt/homebrew/opt/node/bin/node /opt/homebrew/bin/marp",
  browser = nil,  -- 自動検出
  debug = true,   -- 開発時はtrueを推奨
  server_mode = false,  -- ウォッチモード使用
  show_tips = true,
  auto_copy_path = true,
  html_option = true,  -- ウォッチモードで--htmlオプションを使用（デフォルト: true）
})
```

## トラブルシューティング

- `:MarpDebug`コマンドでMarp CLI環境を診断
- `debug = true`設定で詳細ログを確認
- プロセス状況は`:MarpList`で確認
- HTML生成パスは自動的にクリップボードにコピー

## 開発で得られた知見（v1.1.1）

### MarpWatchの安定性改善

#### 問題と解決策

1. **プロセス管理の不具合**
   - 問題: jobstartで起動したプロセスが適切にクリーンアップされない
   - 解決: プロセス停止時の待機処理とforce killの実装
   ```lua
   -- プロセス停止を確実に待つ
   vim.wait(1000, function()
     return M.active_processes[bufnr] == nil
   end)
   ```

2. **ブラウザの重複起動**
   - 問題: ファイル更新のたびにブラウザが新しく開く
   - 解決: バッファごとにブラウザ起動状態を追跡
   ```lua
   M.metadata.browser_opened[bufnr] = true
   ```

3. **プロセスの異常終了**
   - 問題: Marpプロセスがクラッシュした場合の対応不足
   - 解決: 自動再起動メカニズム（最大3回）
   ```lua
   if exit_code ~= 0 and exit_code ~= 143 then
     -- 自動再起動ロジック
   end
   ```

4. **通知の過剰表示**
   - 問題: HTML更新通知が頻繁に表示される
   - 解決: 1秒間隔でのデバウンス実装

#### ベストプラクティス

1. **メタデータ管理**
   - バッファごとの状態管理にテーブルを活用
   - プロセスライフサイクルに関連する情報を一元管理

2. **エラーハンドリング**
   - vim.fn.jobstartのエラーは適切にpcallでラップ
   - 終了コードごとの処理分岐（正常終了 vs 異常終了）

3. **非同期処理**
   - vim.defer_fnを使用して適切なタイミング制御
   - vim.scheduleで安全なUIアップデート

4. **デバッグ機能**
   - 開発時に有用な状態情報の表示
   - プロセス状態、メタデータ、設定値の確認

### ローカル開発環境での検証

```lua
-- ~/.config/nvim/lua/plugins/init.lua での設定例
{
  "nwiizo/marp.nvim",
  dir = vim.fn.expand "~/ghq/github.com/nwiizo/marp.nvim",
  ft = "markdown",
  cmd = { "MarpWatch", "MarpStop", ... },
  config = function()
    require("marp").setup {
      debug = true,  -- ローカル開発時は必須
    }
  end,
}
```

### リリースプロセス

1. 品質チェック: `make check`でluacheckとstyluaを実行
2. コミット: Conventional Commits形式（fix/feat/docs等）
3. タグ付け: セマンティックバージョニングに従う
4. GitHubリリース: `gh release create`で自動化

### 今後の改善案

- プロセス管理のさらなる最適化（プロセスプールの実装）
- ブラウザとの双方向通信（WebSocket等）
- より詳細なエラーレポート機能
- パフォーマンスメトリクスの収集