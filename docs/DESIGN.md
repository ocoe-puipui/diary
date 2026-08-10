# Chronicle 日記アプリ 設計シート

最終更新: 2026-08-10

## 1. 現状分析(As-Is)

### 1.1 概要

- リポジトリ: `ocoe-puipui/diary`(GitHub Pages公開中: https://ocoe-puipui.github.io/diary/)
- 構成ファイル: `index.html`(単一ファイル、約1,350行)、`README.md`、`.gitignore`
- 技術スタック: Vanilla JavaScript、HTML5、CSS3(フレームワーク不使用)
- ストレージ: IndexedDB(利用不可環境ではlocalStorageに自動フォールバック)

### 1.2 データ構造(現状)

```javascript
// ノート(notebook)
{
  id: 'nb_default',
  name: '日記1',
  theme: 'theme-classic',   // 10種類のテーマから選択
  isVertical: false,        // 横書き / 縦書き
  entries: []
}

// 日記エントリ(entry)
{
  id: 'entry_1',
  date: '2026-07-12',
  title: 'タイトル',
  content: '本文内容',
  weather: '晴天',
  images: ['base64_image_data_1', ...]
}
```

保存キー: IndexedDB `ChronicleAppDB` / objectStore `app_state`(キー: `notebooks`, `current_nb_id`, `edit_history`)。localStorageモード時は `chronicle_v3_` 接頭辞を使用。旧バージョン(`chronicle_notebooks`等)からの自動移行処理あり。

### 1.3 主な機能一覧

| 分類 | 機能 |
|---|---|
| 日記 | 日付・タイトル・本文の記録、複数画像添付、天気/備考メモ、編集履歴 |
| 表示 | 10種類のカラーテーマ、横書き/縦書き切替、レスポンシブ |
| ノート管理 | 複数ノート作成・切替・名称編集、ノートごとに独立したテーマ/書字方向 |
| カレンダー | 月間ナビゲーション、日記の有無をハイライト、今日へジャンプ |
| データ管理 | JSON形式でのエクスポート/インポート、自動保存 |

### 1.4 主要関数(index.html内)

初期化系: `initDB`, `getDBValue`, `setDBValue`, `clearDB`
データ入出力: `exportData`, `importData`, `saveAllData`
エントリ操作: `saveNewDiary`, `saveEditDiary`, `deleteEntry`, `viewDiary`, `showNewForm`, `showEditForm`
ノート操作: `addNotebook`, `switchNotebook`, `changeNotebookTheme`, `changeNotebookWritingMode`, `manageNotebookPrompt`
表示系: `applyTheme`, `renderTabs`, `renderCalendar`, `renderHistory`, `refreshSidebar`, `updateNotebookHeader`, `updateLabels`
カレンダー: `moveCalendar`, `jumpToDiary`, `jumpToToday`, `syncCalendarMonth`
画像: `handleFileSelectMulti`, `renderFormImagesGrid`, `removeFormImage`

### 1.5 現状の重要な性質(設計上の前提)

- **公開ページと個人データは完全に分離している。** GitHub / GitHub Pagesにpushされるのはコード(`index.html`)のみで、日記データはブラウザのローカルストレージにしか存在しない。
- そのため、別端末・別ブラウザで開くと日記データは空の状態になる(バックアップJSONをインポートしない限り)。
- `.gitignore` は `*.json` を一括除外しており、`!sample_diary.json` という例外指定の書き方が既に用意されている(未使用)。

## 2. 今後の設計(To-Be) — 公開用日記データの導入

### 2.1 目的

ローカルで書いた日記のうち、公開してよいものだけを選び、GitHub Pages上の公開ページにも表示できるようにする。個人の全データを晒すのではなく、選択制・手動反映を前提とする。

### 2.2 採用方式: 公開用ノート方式(別データファイル方式)

個人日記(IndexedDB)とは別に、`docs`とは別の**公開用JSONファイル**(例: `public_entries.json`)をリポジトリに追加し、アプリ起動時にfetchして読み取り専用の「公開日記」タブとして表示する。

理由: 継続的な運用(何度も選んで公開・更新していく)に向いており、個人日記と公開作品を明確に分離できる。将来のAIIMAGEVIEWER連携(作品発表の場)とも相性が良い。

### 2.3 データ構造(公開用ファイル案)

```javascript
// public_entries.json
{
  "version": "1.0",
  "updatedAt": "2026-08-10",
  "entries": [
    {
      "id": "pub_0001",
      "date": "2026-08-09",
      "title": "タイトル",
      "content": "本文内容",
      "weather": "晴天",
      "images": []   // 画像を含める場合はbase64 or 外部URL。運用ルールは要決定
    }
  ]
}
```

備考: 個人日記の`entry`構造とほぼ互換にし、変換コストを抑える。`id`は個人日記内部IDと衝突しないよう`pub_`接頭辞を付与する案。

### 2.4 必要な機能追加(index.html側)

1. **公開マーク機能**: 日記エントリ編集画面 or サイドバーに「この日記を公開する」チェックボックスを追加。エントリオブジェクトに `isPublic: true/false` フィールドを追加(個人データ側の拡張)。
2. **公開用データ書き出し機能**: `exportData()`と同様のパターンで、`isPublic === true` のエントリのみを抽出し `public_entries.json` としてダウンロードするボタンを追加(例: `exportPublicData()`)。
3. **公開データ読み込み・表示機能**: アプリ起動時(`window.onload`内)に `fetch('public_entries.json')` を実行し、成功した場合は「公開日記」専用タブ(読み取り専用ビュー)として表示する。個人ノートの編集UIとは分離する。
4. **UIの追加**: タブバーに「公開日記」を追加。公開日記側には編集・削除ボタンを表示しない(読み取り専用)。

### 2.5 運用フロー(To-Be)

1. ローカルで日記を書く/編集する
2. 公開したいエントリに公開マークを付ける
3. 「公開用データを書き出す」ボタンで `public_entries.json` をダウンロード
4. ダウンロードしたファイルを `diary/public_entries.json` に配置
5. `.gitignore` に `!public_entries.json` の例外を追加(初回のみ)
6. `git add` → `git commit` → `git push`
7. GitHub Pages反映後、公開URLで表示確認

### 2.6 留意点・未決事項

- 画像を公開データに含める場合、base64埋め込みはリポジトリ肥大化につながる。枚数制限・圧縮・またはCDN/外部ホスティングの検討が必要(AIIMAGEVIEWER連携時に画像ホスティングの仕組みを流用できる可能性あり)。
- 自動公開(書いたら即公開)は行わず、明示的な選択→書き出し→手動push というワンクッションを維持し、誤公開を防ぐ。
- `public_entries.json` は読み取り専用データとして扱い、公開ページ上での編集機能は持たせない(編集はローカル側でのみ行う)。
- 将来的にAIIMAGEVIEWERからの画像取り込み・進捗更新投稿を行う場合、公開エントリのデータ構造に `source: "manual" | "aiimageviewer"` のようなタグを追加する拡張も検討可能。
