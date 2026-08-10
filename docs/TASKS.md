# Chronicle 日記アプリ タスクリスト

最終更新: 2026-08-10(フェーズ2完了)
関連ドキュメント: [DESIGN.md](./DESIGN.md)

## 進行中の取り組み: 公開用日記データの導入

設計方針は [DESIGN.md 2.2](./DESIGN.md) の「公開用ノート方式」を採用。

### フェーズ1: データ設計 ✅完了(2026-08-10)

- [x] `public_entries.json` のデータ構造を確定する → 現行の日記データ構造(date, title, content, weather, images)をそのまま踏襲(DESIGN.md 2.3)
- [x] 画像の扱いを決定する → 画像も含める
- [x] 公開マークの配置場所を決定する → 編集画面・一覧(サイドバー)の両方に配置

### フェーズ2: アプリ機能追加(index.html) ✅完了(2026-08-10)

- [x] 日記編集画面に「この日記を公開する」チェックボックスを追加(新規作成・編集フォーム両方)
- [x] サイドバーの一覧(INDEX)にも公開マークの切替UI(🌐ボタン)を追加(編集画面側と同期)
- [x] エントリ保存処理(`saveNewDiary` / `saveEditDiary`)に `isPublic` フィールドを反映
- [x] 公開用データ書き出し関数 `exportPublicData()` を実装(Managementエリアに「export public entries」ボタンを追加)
- [x] ~~起動時処理に`public_entries.json`のfetch処理・「公開日記」プレビュータブを追加~~ → 2026-08-10に撤去(下記フェーズ2.5参照)

### フェーズ2.5: UI見直し(2026-08-10、ユーザーフィードバックにより追加)

- [x] 「公開日記」プレビュータブを撤去。`file://`でのローカル閲覧時にfetchがブロックされ、公開されているのに0件と表示される誤解を招くため
- [x] 代わりに、現在のノート内の全エントリを一括で公開/非公開に切り替える「すべて公開にする」「すべて非公開にする」ボタンをサイドバーに追加(`bulkSetPublic()`)
- [x] 一括操作後も、個別エントリを🌐ボタンで個別にオン/オフし直せることを確認(仕様として維持)

### フェーズ2.6: 閲覧専用デモモード(2026-08-10、ユーザーフィードバックにより追加)

- [x] `location.hostname === 'ocoe-puipui.github.io'` の場合のみ `initReadOnlyDemo()` を実行するよう `window.onload` を分岐
- [x] デモモードでは新規作成・編集・削除・公開切替(個別/一括)・インポート・エクスポート・ノート管理UIを全て非表示にする
- [x] デモモードでは `public_entries.json` を読み込み、Index一覧+本文表示のみの読み取り専用ビューを表示する(`renderReadOnlyEntry()`)
- [x] コードをダウンロードして別ドメイン/ローカルで実行した場合は `isLiveDemo` が false になり、通常通りフル機能で動作することを確認(構文チェック・ロジック確認)

JavaScript構文チェック済み(Node.jsで検証、エラーなし)。実際に`https://ocoe-puipui.github.io/diary/`へpushしてからの表示確認はフェーズ4/5で実施。

### フェーズ3: リポジトリ設定

- [x] `.gitignore` に `!public_entries.json` の例外を追加(2026-08-10)
- [ ] 初回の `public_entries.json` を作成・配置(puipuiさんの作業: 「export public entries」→`diary`フォルダに配置→コミット)

### フェーズ4: 動作確認

- [x] ローカルブラウザ(file://)で、公開マークを付けたエントリが正常に管理できることを確認
- [x] 個人日記(IndexedDB)側の既存データに影響がないことを確認
- [ ] `public_entries.json` を配置・pushした後、本番URL(`https://ocoe-puipui.github.io/diary/`)の「公開日記(閲覧専用)」に正しく表示されることを確認

### フェーズ5: 公開・反映

- [ ] `index.html` の変更をコミット
- [ ] `public_entries.json` をコミット
- [ ] GitHubへpush
- [ ] GitHub Pages反映後、公開URLで表示確認
- [ ] CHANGELOG.md を更新

## 保留・検討事項(未着手)

- [ ] AIIMAGEVIEWERとの連携方式の具体化(進捗投稿、画像アップロード)
- [ ] 公開エントリへの `source` タグ付け(手動 / AIIMAGEVIEWER由来)の要否検討
- [ ] 画像ホスティング方法の検討(base64埋め込み vs 外部ホスティング)

## 完了

(まだありません)
