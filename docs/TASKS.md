# Chronicle 日記アプリ タスクリスト

最終更新: 2026-08-10
関連ドキュメント: [DESIGN.md](./DESIGN.md)

## 進行中の取り組み: 公開用日記データの導入

設計方針は [DESIGN.md 2.2](./DESIGN.md) の「公開用ノート方式」を採用。

### フェーズ1: データ設計

- [ ] `public_entries.json` のデータ構造を確定する(DESIGN.md 2.3を叩き台として調整)
- [ ] 画像の扱いを決定する(含める/含めない、含める場合の形式・上限)
- [ ] 個人日記エントリに追加する `isPublic` フィールドの仕様を決定する

### フェーズ2: アプリ機能追加(index.html)

- [ ] 日記編集画面に「この日記を公開する」チェックボックスを追加
- [ ] エントリ保存処理(`saveNewDiary` / `saveEditDiary`)に `isPublic` フィールドを反映
- [ ] 公開用データ書き出し関数 `exportPublicData()` を実装(`exportData()` を参考に)
- [ ] 起動時処理(`window.onload`)に `public_entries.json` の fetch 処理を追加
- [ ] 「公開日記」タブ(読み取り専用ビュー)のUIを追加
- [ ] 公開日記タブでは編集・削除ボタンを非表示にする

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
