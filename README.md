# Habit Tracker

A mobile-first habit tracker (PWA) inspired by a handwritten notebook tracker: daily check-offs, monthly grid report, streaks, live graphs, PDF and Excel export, light/dark themes and file backup. No server, no build step. Data stays on the device.

## Features
- 14 default habits (Yes/No and Hours types); add or remove your own
- Edit today and the previous 2 days only
- Month-only streaks (reset on the 1st)
- Monthly report grid with frozen header row and Day column, totals row, two live graphs
- Menu with expandable month history
- Themes: Auto / Light / Dark plus 5 accent colours
- Report download: PDF (grid + graphs) or Excel (editable data, SUM formulas)
- Backup to a `.json` file and restore from file
- Works offline once opened; installable on iPhone / Android

## Deploy on GitHub Pages
1. Create a new GitHub repository (public).
2. Upload everything in this folder to the repository root (`index.html`, `sw.js`, `manifest.webmanifest`, `icons/`, `.nojekyll`, `README.md`, `AI_HANDOFF.md`).
3. Go to **Settings -> Pages -> Build and deployment -> Source: Deploy from a branch**, choose `main` and `/ (root)`, then Save.
4. After about a minute the app is live at `https://<your-username>.github.io/<repo-name>/`.

## Install on iPhone
Open the link in **Safari**, tap Share, then **Add to Home Screen**.

## Notes
- Data is saved in the browser's local storage on each device (no sync). Use **Settings -> Create backup file** regularly. iOS can clear website data for sites unused for a long time; the Home Screen app is safer.
- PDF/Excel use jsPDF and SheetJS from cdnjs. The first export needs internet; afterwards they are cached for offline use.
- When you change any file, bump `VERSION` in `sw.js` so phones pick up the update.

See `AI_HANDOFF.md` for the full logic, history and source code. 
