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
- [x] 起動時処理(`window.onload`)に `public_entries.json` の fetch 処理を追加(ファイル未配置時は空扱いでエラーにならない)
- [x] 「公開日記」タブ(`renderPublicView()`、読み取り専用ビュー)のUIを追加
- [x] 公開日記タブには編集・削除ボタンを実装していない(読み取り専用)

JavaScript構文チェック済み(Node.jsで検証、エラーなし)。ブラウザでの実機動作確認はフェーズ4で実施。

### フェーズ3: リポジトリ設定

- [ ] `.gitignore` に `!public_entries.json` の例外を追加
- [ ] 初回の `public_entries.json` を作成・配置

### フェーズ4: 動作確認

- [ ] ローカルブラウザで、公開マークを付けたエントリのみが公開日記タブに表示されることを確認
- [ ] 個人日記(IndexedDB)側の既存データに影響がないことを確認
- [ ] 公開日記タブが読み取り専用であることを確認(編集・削除できないこと)

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
