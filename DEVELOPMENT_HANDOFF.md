# Adam's Solitaire — Development Handoff

This document describes the game, the work completed so far, and what another developer needs to continue on a different computer.

## Project at a glance

Adam's Solitaire is a static Progressive Web App (PWA) with three games: Klondike, FreeCell, and Spider. It is implemented with plain HTML, CSS, and JavaScript. There is no framework, build step, package manager, or server-side component.

## Files

- `index.html` — app screens, game board host, settings/panel markup, and script/style links.
- `app.js` — game state, shuffling/dealing, legal moves, drag/tap input, animation, scoring, settings, saved games, and game status.
- `styles.css` — responsive layout, card/table styling, touch controls, and portrait/landscape sizing.
- `manifest.webmanifest` — installed-app name, start URL, display mode, orientation preference, and icon.
- `sw.js` — service worker; precaches the app shell, activates new releases, and removes old caches.
- `icon.svg` — install icon.

## Development history and decisions

The game has been refined through mobile play and visual review. Work to date includes:

- Improved card-face readability, stronger rank/suit contrast, and a long-press card preview.
- Adjusted Spider and FreeCell card/column sizing for narrow portrait screens and landscape phones. Rotation is allowed by the manifest; the board responds to viewport changes.
- Refined card movement animation to move whole stacks together, and made drag/drop more forgiving by snapping to the nearest legal target.
- Added a Settings option for legal drop-target indicators. It is off by default and saved with the user's other settings.
- Added a no-legal-actions check. A position with no legal card moves or usable stock action opens a dialog with Undo (when possible) and End game. Klondike stock/redeal actions and Spider's rule that stock cannot be dealt while a column is empty are included in the check.
- Added repeated-position tracking. Returning to a board position a second time produces a brief warning. On the third visit to the same position, a dialog offers Keep playing, Undo, or End game. This is a loop warning, not a claim that the deal is unsolvable.
- Position tracking supports Spider's board, which has no foundation piles.
- End game discards the active round, returns to the welcome screen, and does not record a top score. Both dialogs require the player to choose an action rather than dismissing them by tapping outside.
- Full deal solvability search and guaranteed-solvable deal generation were deliberately left out. Those remain separate, substantially more complex features.

## Running on another computer

Copy or clone the complete `AdamSol` directory. No dependencies need to be installed to run the game. Since service workers require a secure context, use localhost for local testing rather than opening `index.html` directly as a `file:` URL.

With Python 3 installed:

```sh
cd AdamSol
python3 -m http.server 8000
```

Open `http://localhost:8000/` in a browser. The app is also suitable for static hosting over HTTPS, including GitHub Pages.

## Testing checklist

The workspace has a Playwright test at `../tests/adams-solitaire.spec.js`; verify that its URL points to this directory before using it. Test changes in a current Chrome/Chromium browser at desktop and phone-sized viewports. For game-status changes, test at least:

1. A FreeCell or Klondike position with legal moves remains playable and does not show the stuck dialog.
2. A state with no legal card moves and no available stock action opens the no moves dialog.
3. Undo from that dialog restores the previous position; End game returns to welcome and clears the active round.
4. Return to a position once to see the brief warning, then a second time to see the repeated-position dialog. Keep playing preserves the round; End game clears it.
5. Spider with an empty tableau column does not treat dealing the stock as an available action.

Playwright/Chromium was available in the original development environment, but it is not a project dependency. If it is unavailable on the new computer, install/use a local Playwright setup or test the scenarios manually in Chrome DevTools. Do not add a build system solely to serve these static files.

## PWA, deployment, and saved data

- Publish the contents of `AdamSol` as the site root (or preserve the current GitHub Pages subdirectory layout). App asset URLs and the service worker registration are relative, which supports hosting below a repository path.
- After publishing, verify the live Pages deployment succeeded. On Android, visit the live site in Chrome and refresh, then close and reopen the installed app if it still shows old files.
- When changing cached assets, increment the `CACHE` value in `sw.js`. The install handler precaches the files in `ASSETS`; activation removes caches with older names. Keep that list in sync with the app shell.
- The service worker uses cache-first responses for cached assets. A changed service worker/cache version is how a static release refreshes the installed app's shell.
- User preferences and the active game are stored in browser `localStorage` under `adams-solitaire-v1`. They are scoped to the site's origin; moving to a different domain does not automatically move them. Clearing site data can erase settings, scores, and an in-progress game.
- GitHub Pages must serve the app over HTTPS for installation and service-worker behavior. The GitHub repository page itself is not the game URL.

## Current implementation notes

- Existing preferences are merged with defaults in `app.js`, so newly added settings should have a safe default for existing players.
- Active game state is saved after renders and timer updates. Undo stores snapshots of game state in memory; it is not a persistent undo history across reloads.
- Repeat-position history is stored with the active game and capped at 200 recent position transitions. The position key includes game type, visible/hidden card identities, pile order, stock/waste order, cells, foundations, and Spider completed runs. It excludes score and time so they do not disguise a loop. The dialog appears on the third recorded visit to the same position within that history.
- `hasAnyLegalAction()` relies on the game's existing `canSelect()` and `legal()` rules. If move rules change, update or test this detector alongside them.
- The browser's system auto-rotate setting still controls whether Android rotates the display; the app allows orientation changes and adapts its layout when the viewport changes.
