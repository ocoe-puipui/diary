# Chronicle 日記アプリ 設計シート

最終更新: 2026-08-10(フェーズ1決定事項を反映)

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

### 2.3 データ構造(公開用ファイル・確定版)

方針決定(2026-08-10): 画像はデータに含める。項目は現行の日記データ構造(date, title, content, weather, images)をそのまま踏襲する。

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
      "images": ["base64_image_data_1", ...]   // 画像を含める(方針確定)
    }
  ]
}
```

備考: 個人日記の`entry`構造とほぼ互換にし、変換コストを抑える。`id`は個人日記内部IDと衝突しないよう`pub_`接頭辞を付与する案。画像を含めるため、リポジトリ容量の増加は許容する前提(枚数・サイズの制限は運用しながら調整、2.6参照)。

### 2.4 必要な機能追加(index.html側)

1. **公開マーク機能**: 日記エントリ編集画面と、サイドバーの一覧(INDEX)の両方に「この日記を公開する」チェックボックス/トグルを配置(方針確定)。エントリオブジェクトに `isPublic: true/false` フィールドを追加(個人データ側の拡張)。一覧側で切り替えた場合も編集画面側の状態と同期させる。
2. **一括操作機能**: 現在開いているノート(タブ)内の全エントリを「すべて公開にする」「すべて非公開にする」で一括切り替えできるボタンをサイドバーに配置。一括操作後も、個別エントリは🌐ボタンで個別にオン/オフし直せる。
3. **公開用データ書き出し機能**: `exportData()`と同様のパターンで、`isPublic === true` のエントリ(画像含む)を抽出し `public_entries.json` としてダウンロードするボタンを追加(`exportPublicData()`)。

**変更履歴(2026-08-10・1回目)**: 当初「公開日記」専用タブ(`fetch`で`public_entries.json`を読み込み、アプリ内でプレビュー表示する機能)を実装したが、`file://`でローカル閲覧した際にブラウザのセキュリティ制限で正しく読み込めず、「公開されているはずなのに0件と表示される」という誤解を招く挙動になったため撤去した。代わりに、公開/非公開の管理はノート単位の一括操作ボタンと個別の🌐トグルのみで行う方式に変更。

**変更履歴(2026-08-10・2回目、閲覧専用デモモードの追加)**: このアプリはログイン機能を持たないため、GitHub Pagesで公開している本番URL(`https://ocoe-puipui.github.io/diary/`)にアクセスした人は誰でも、新規作成・編集・削除・公開/非公開の切り替えといった全機能を(自分のブラウザのローカルデータに対してではあるが)操作できてしまう状態だった。データそのものが荒らされる実害はないが、「サンプルとして公開している日記の見た目が誰でも操作できてしまう」ことへの懸念から、以下の方針を追加した。

- アプリコード自体はオープンソースのまま維持し、誰でもダウンロード・自由に実行できるようにする
- ただし `location.hostname` が公式公開ドメイン(`ocoe-puipui.github.io`)と一致する場合のみ「閲覧専用デモモード」に切り替える
- 閲覧専用デモモードでは、新規作成ボタン・編集/削除ボタン・公開マーク切り替え・一括操作・インポート/エクスポート・ノート管理など編集系UIを全て非表示にし、`public_entries.json` の内容のみを読み取り専用で一覧表示する
- コードをダウンロードして自分の環境(別ドメイン、ローカルファイルなど)で実行した場合は、`location.hostname`が一致しないため通常通りフル機能(編集・削除・公開管理など)が使える

実装は `index.html` 内の `isLiveDemo` 判定と `initReadOnlyDemo()` / `renderReadOnlyEntry()` 関数。

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
