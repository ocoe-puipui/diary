# Chronicle Diary 設計ドキュメント(現状まとめ)

最終更新: 2026-08-11

このドキュメントは過去の変更履歴ではなく、**現時点の仕様**をまとめたものです。
開発経緯やユーザー向けの説明は[README.md](../README.md)を参照してください。

## 1. 概要

- リポジトリ: `ocoe-puipui/diary`(GitHub Pages公開: https://ocoe-puipui.github.io/diary/)
- 構成: `index.html`(単一ファイル。HTML/CSS/JS完結、ビルド不要)、`public_entries.json`(公開データ)、`README.md`、`LICENSE`
- 技術スタック: Vanilla JavaScript / HTML5 / CSS3(フレームワーク不使用)
- ストレージ: IndexedDB(`ChronicleAppDB` / `app_state`)。利用不可環境ではlocalStorageに自動フォールバック
- ライセンス: MIT License

## 2. データ構造

### 2.1 個人日記データ(ブラウザ内のみ、非公開)

```javascript
// ノート(notebook)
{
  id: 'nb_default',
  name: '日記1',
  theme: 'theme-classic',   // 10種類のテーマから選択
  isVertical: false,        // 横書き / 縦書き
  fontSize: 'medium',       // 'small' | 'medium' | 'large'(既定: medium)
  entries: []
}

// 日記エントリ(entry)
{
  id: 1234567890,
  date: '2026-07-12',
  title: 'タイトル',
  content: '本文内容',
  weather: '晴天',
  images: ['base64_image_data_1', ...],
  isPublic: false
}
```

保存キー: `notebooks`, `current_nb_id`, `edit_history`, `site_announcement`。

### 2.2 お知らせ(サイト全体の告知)

```javascript
{
  content: '<p>HTML文字列(リンク・画像を含めてよい)</p>',
  isPublic: false
}
```

個々の日記エントリーとは独立した、公開ページ上部に常時表示される告知欄。生のHTMLをそのまま入力・表示する。画像は編集画面の「画像を選択して挿入（１）〜（３）」ボタン(最大3枚、均等幅で配置)から追加でき、日記の画像添付と同様にbase64化してHTML内に直接埋め込む(外部URLに依存しないため公開ページでも確実に表示される)。

### 2.3 公開用データ(`public_entries.json`、リポジトリにコミットする)

```javascript
{
  "version": "2.1",
  "updatedAt": "2026-08-10",
  "announcement": { "content": "..." },  // isPublicがtrueの場合のみ含む。無ければnull
  "notebooks": [
    {
      "id": "pub_nb_default",
      "name": "日記1",
      "theme": "theme-classic",
      "isVertical": false,
      "fontSize": "medium",
      "entries": [
        { "id": "pub_entry_1", "date": "2026-07-12", "title": "タイトル", "content": "本文内容", "weather": "晴天", "images": [] }
      ]
    }
  ]
}
```

## 3. 機能一覧

| 分類 | 機能 |
|---|---|
| 日記 | 日付・タイトル・本文の記録、複数画像添付、天気/備考メモ、編集履歴 |
| 表示 | 10種類のカラーテーマ、横書き/縦書き切替、文字サイズ(小/中/大)、レスポンシブ |
| ノート管理 | 複数ノート作成・切替・名称編集・削除、ノートごとに独立したテーマ/書字方向/文字サイズ |
| カレンダー | 月間ナビゲーション、日記の有無をハイライト、今日へジャンプ |
| INDEX一覧 | 表示件数(3/5/10件、既定5件)選択、ページ送り |
| 表示モード | 個別表示(1件詳細)/まとめて表示(feed、ページ単位、既定) |
| 公開機能 | エントリ単位の公開マーク、ノート内一括公開/非公開、お知らせ(サイト告知)、公開用データの書き出し |
| データ管理 | JSON形式でのフルバックアップ・復元、ノート単位のエクスポート/追加インポート、自動保存 |
| 多言語UI | 縦書きノート=日本語表記、横書きノート=英語表記(`updateLabels`で切替) |

## 4. 主要関数(`index.html`)

- 初期化・DB: `initDB`, `getDBValue`, `setDBValue`, `saveAllData`
- エントリ操作: `saveNewDiary`, `saveEditDiary`, `deleteEntry`, `viewDiary`, `showNewForm`, `showEditForm`
- ノート操作: `addNotebook`, `switchNotebook`, `manageNotebookPrompt`, `changeNotebookTheme`, `changeNotebookWritingMode`, `changeNotebookFontSize`
- 表示系: `applyTheme`, `renderTabs`, `renderCalendar`, `renderHistory`, `refreshSidebar`, `updateNotebookHeader`, `updateLabels`, `renderEntryFeed`, `toggleMainViewMode`
- INDEX ページング: `changeIndexPageSize`, `moveIndexPage`, `renderIndexPageNav`, `ensurePageContainsEntry`
- 公開機能: `toggleEntryPublic`, `bulkSetPublic`, `exportPublicData`, `showAnnouncementForm`, `saveAnnouncement`, `renderAnnouncementPreview`
- エクスポート/インポート: `exportData`, `importData`, `exportNotebookData`, `importNotebookData`
- 閲覧専用デモモード: `initReadOnlyDemo`, `renderDemoTabs`, `renderDemoCalendar`, `renderDemoActivity`, `switchDemoNotebook`, `viewDemoEntry`

## 5. 公開の仕組み

### 5.1 データの分離

個人日記(IndexedDB)と公開データ(`public_entries.json`)は完全に分離されている。GitHubにpushされるのはコードと`public_entries.json`のみで、個人の全データが晒されることはない。

### 5.2 公開までの運用フロー

1. ローカルで日記を書く/編集する
2. 公開したいエントリに公開マーク(🌐)を付ける、または「お知らせ」を編集・公開設定にする
3. Managementの「export public entries」で`public_entries.json`を書き出す
4. `diary/public_entries.json`に配置し、`git add` → `git commit` → `git push`
5. GitHub Pages反映後、公開URLで表示を確認

### 5.3 閲覧専用デモモード

- `index.html`内の`LIVE_DEMO_HOSTNAME`(値: `ocoe-puipui.github.io`)と`location.hostname`が一致する場合のみ`initReadOnlyDemo()`が実行される
- デモモードでは新規作成・編集・削除・公開切替・インポート/エクスポート・ノート管理(SETTINGパネル全体)など編集系UIをすべて非表示にし、`public_entries.json`の内容のみ読み取り専用で表示する
- ノート切替・カレンダー・Recent Activityは機能紹介として残す
- INDEX一覧のページ送り、まとめて表示(feed)モードも公開ページに適用される(規定値: 表示件数5件・まとめて表示)。表示件数(SHOW)の変更UIのみ非表示
- コードをダウンロードして別ドメイン/ローカルで実行した場合は`isLiveDemo`がfalseになり、フル機能で動作する

## 6. 今後の検討事項

- AIIMAGEVIEWERとの連携方式の具体化(進捗投稿、画像アップロード)
- 公開エントリへの`source`タグ付け(手動 / AIIMAGEVIEWER由来)の要否
- 画像ホスティング方法(base64埋め込み vs 外部ホスティング)の見直し(リポジトリ容量増加への対応)
