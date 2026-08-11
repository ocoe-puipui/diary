🌐 日本語版 (Japanese version): [README.ja-JP.md](./README.ja-JP.md)

---

# Diary App

A browser-based diary app. All data is stored locally in the browser (IndexedDB/localStorage) and is never sent to any server.

🔗 **Live demo**: https://ocoe-puipui.github.io/diary/
(This URL runs in a read-only demo mode. See [About the Live Demo](#-about-the-live-demo) for details.)

📄 **License**: [MIT License](./LICENSE)

---

## 📖 About the Live Demo

The live demo URL above runs in a **read-only** mode that only shows diary entries the author (puipui) has chosen to publish as samples. Creating, editing, deleting, and toggling public/private status are all disabled.

This restriction exists so that visitors can't accidentally modify or delete the sample content being shown. Read-only features such as switching between multiple notebooks, the calendar, and Recent Activity are still fully available to try.

**To try the full feature set (create, edit, delete, toggle public/private, etc.), please follow the "Fork or download to use" section below and open `index.html` in your own environment.** In your own downloaded copy, this restriction does not apply and the app runs with full functionality.

---

## Features

### 📝 Diary
- **Record date, title, and body text**
- **Multiple images** - attach multiple images to a single entry
- **Weather / notes field** - add metadata such as weather or mood
- **Edit history**

### 🎨 10 color themes

- **Horizontal and vertical writing support**
- **Responsive design**

### 📚 Notebook management
- **Create multiple notebooks** - organize diary entries into separate categories
  - Each notebook has its own independent theme and writing direction
  - Notebook names can be customized

### 📅 Calendar
- **Visualize which days have entries** - dates with entries are highlighted
- **Month navigation** - move between months easily
- **Jump to today** - jump to today's date with one click

### 🌐 Public diary feature
- **Public flag** - toggle "public/private" per entry (from both the edit screen and the list)
- **Bulk toggle** - mark all entries in a notebook as public or private at once
- **Export public data** - export only the entries marked public as `public_entries.json`, which can be published via a separate content repository and hosted on GitHub Pages, etc.

### 💾 Data management
- **Auto-save** - all changes are saved automatically
- **Backup & restore** - export/import data as JSON
- **Local storage only** - all data stays in the browser (nothing is sent to a server)

---

## Usage

1. **Open in a browser**
   ```
   Just open index.html in your browser to start.
   ```

2. **First use**
   - A default notebook "日記1" is created automatically
   - Click "Write a Diary" to create your first entry

3. **Write an entry**
   - Enter the date, title, and body text
   - Click to add images
   - Check "Publish this entry" if you want it public
   - Click "Save to Chronicle" to save

### Managing notebooks

- **Add a notebook** - click "+ ADD" in the tab bar
- **Switch notebooks** - select a tab at the top
- **Rename a notebook** - click the 📔 icon on a notebook tab

### Theme and display settings

- **Change color theme** - use the "COLOR THEME" dropdown in the header
- **Change writing direction** - use "WRITING DIRECTION" to switch between horizontal and vertical writing

### Data backup

- **Export** - click "export (backup)" in the left sidebar
- **Import** - select a backup file from "import (restore)" to restore it

### Publishing entries (for use with GitHub Pages, etc.)

The public data file (`public_entries.json`) is managed in a separate, dedicated repository from the app's code (for this project, [`diary-content`](https://github.com/ocoe-puipui/diary-content)). This separation ensures that anyone who forks or downloads the code repository does not also receive puipui's personal diary content.

1. Mark the entries you want to publish using the 🌐 icon in the edit screen or sidebar list
2. To set several at once, use "Publish all" / "Unpublish all" in the sidebar
3. Use "export public entries" in Management to export `public_entries.json`
4. Commit and push the exported file to the content repository (`diary-content`) — do not commit it to the app's code repository
5. Once published, only the entries you marked public will appear on the public page

If you fork this project for your own use, update the `PUBLIC_CONTENT_URL` constant in `index.html` (used to fetch the announcement/public data) to point to your own content repository's raw URL.

---

## 🍴 Fork or download to use

This repository is released under the MIT License, and you're free to fork or download it for your own use.

### Getting started

1. Download or fork this repository
2. Open `index.html` in a browser (that's all — no server or build step required)
3. Start using it right away as your own diary

### ⚠️ One thing worth checking

Near the top of the script in `index.html`, you'll find the following:

```javascript
const LIVE_DEMO_HOSTNAME = 'ocoe-puipui.github.io';
const isLiveDemo = (location.hostname === LIVE_DEMO_HOSTNAME);
```

This is a **setting specific to the author's (puipui's) own public page**. It switches the app into a "read-only demo mode" (which hides all editing features) only when accessed from that exact hostname.

When you run this in your own environment, leaving this line as-is causes no problems (since your hostname won't match, the app will always run with full functionality). However, to avoid confusion, one of the following is recommended:

- **If you're just using it for yourself** → feel free to remove the `isLiveDemo` check entirely
- **If you also want to "publish only some entries while keeping the rest editable"** → change `LIVE_DEMO_HOSTNAME` to your own public domain

### About the license notice

You're free to redistribute or modify this code and documentation under the terms of the MIT License, as long as you retain the copyright notice in the [LICENSE file](./LICENSE) (commercial use, modification, and redistribution are all permitted).

---

## 💻 Technical details

### Frontend
- **HTML5**
- **CSS3** - Flexbox/Grid layout, custom properties
- **Vanilla JavaScript** - no framework dependencies

### Storage

#### IndexedDB
- Efficient management of structured data

#### localStorage
- Automatic fallback for environments where IndexedDB is unavailable

### Data structure

```javascript
// Notebook
{
  id: 'nb_default',
  name: '日記1',
  theme: 'theme-classic',
  isVertical: false,
  entries: []
}

// Diary entry
{
  id: 'entry_1',
  date: '2026-07-12',
  title: 'Title',
  content: 'Body text',
  weather: 'Sunny',
  images: ['base64_image_data_1', ...],
  isPublic: false
}
```

### Public data structure (public_entries.json)

```javascript
{
  "version": "2.0",
  "updatedAt": "2026-08-10",
  "notebooks": [
    {
      "id": "pub_nb_default",
      "name": "日記1",
      "theme": "theme-classic",
      "isVertical": false,
      "entries": [
        { "id": "pub_entry_1", "date": "2026-07-12", "title": "Title", "content": "Body text", "weather": "Sunny", "images": [] }
      ]
    }
  ]
}
```

---

### Requirements
- A web browser (recent version recommended)
- Internet connection: not required (works fully offline)

---

## 📖 Glossary

- **Notebook** - a unit for organizing diary entries; you can create multiple notebooks
- **Entry** - a single diary post
- **Theme** - the app's color scheme setting
- **Export** - saving data to a file
- **Import** - restoring data from a saved file
- **Public flag** - a flag indicating whether an entry is included when exporting `public_entries.json`
- **Read-only demo mode** - a display mode, active only on the author's public URL, that hides all editing features

---

## Tips & Troubleshooting

### Data isn't being saved
- Check that local storage is enabled in your browser settings
- Try clearing your browser cache and trying again
- Try a different browser

### Images aren't showing up
- Check the browser console (F12) for errors
- Check whether the image file is too large

### Theme changes aren't applied
- Reload the browser (Ctrl+F5)
- Try a different browser

### Changes aren't showing up on the public page
- Confirm that after exporting `public_entries.json`, you actually committed and pushed it to the content repository (`diary-content`) — not the app's code repository
- Confirm that `PUBLIC_CONTENT_URL` in `index.html` points to the correct content repository's raw URL
- It can take a few minutes for GitHub Pages to reflect changes after pushing

---

## 📄 License

This project is released under the [MIT License](./LICENSE).
Copyright (c) 2026 puipui
