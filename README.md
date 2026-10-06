# Rex Workshop

A phone-first web app for planning the [Captain Rex cosplay](https://github.com/EthEcho-UI/captain-rex-cosplay) build and managing its repository. It's an installable web app that also works on desktop.

Open **https://ethecho-ui.github.io/rex-workshop/** and install it ("Add to home screen" on Android, the install icon in the desktop address bar).

- **Blocks:** progress blocks (Helmet, Torso, Blasters…) made of steps that are *To do*, *Doing* or *Done*. Overall and per-block progress, a "Working on now" list, drag to reorder or move steps between blocks. From a step you can upload photos, attach repository files, write a dated **build log** entry (committed to `docs/build-log.md` with the photos), or put it on a board.
- **Board:** boards with lists and cards, like Docket but without the timer and work hours. Tags, priority, pin, due date, checklist, notes and files on each card; finished cards move to the Done list. Long-press to drag on a phone, drag with the mouse on desktop. Cards and steps can be linked, so finishing one finishes the other.
- **Files:** browse the repository, search all files, and open images, Markdown (rendered, with images), code and text, PDFs, audio/video and **STL models in 3D** with their size in millimetres. Upload (button or drag and drop), create files and folders, edit text files, rename or move, select several to move or delete, download, and see recent changes. Every action is one commit.

## How it works

Plain HTML/CSS/JS in `index.html`, no build step. The app talks to the GitHub API directly from the browser with a **fine-grained token** stored only on each device (Settings → GitHub). The token needs access to the project repository with **Contents: Read and write**.

- Boards and blocks live in `workshop/data.json` in the project repository: saved 4 s after a change and when the app is hidden, pulled on start, on return and every 30 s. The newest save wins. A copy stays in the browser, so the app also works offline.
- File changes go through the Git Data API (blobs → tree → commit → move the branch), so a move, a multi-file upload or a folder delete is a single commit. Files up to 100 MB.
- Libraries load only when needed: three.js for STL, marked + DOMPurify for Markdown.
- `sw.js` caches the app shell for offline start; the page is network-first, so a reload always gets the newest version. Bump `VERSION` in `sw.js` when icons, the manifest or `sw.js` change. On activate it deletes only caches starting with `rexws-`: every `*.github.io` site shares this origin's Cache Storage.

## Data model (`workshop/data.json`)

- `blocks[] {id, title, color, desc, collapsed, steps[]}` · `steps[] {id, title, status: todo|doing|done, notes, files[], card, doneAt, created}`
- `boards[] {id, name, color, lists[]}` · `lists[] {id, title, collapsed, isDone, sort, cards[]}` · `cards[] {id, title, desc, done, tags[], due, priority 0-4, pinned, checklist[], files[], step, created}`
- `tags[] {id, name, color}`, `settings {autoMoveDone, logPath, uploadDir, shrink}`, `savedAt`

`files[]` are repository paths; moving or deleting files in the app updates them.

Device-only (localStorage): `rexws.github` (repo, token, branch), `rexws.ui` (theme, current view, board, folder), `rexws.v1` (local copy of the data).
