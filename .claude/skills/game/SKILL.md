---
name: game
description: Create a new browser game in the class repo peelparel-prehistorie, start the dev server with auto-reload and open the game in the browser. Use for "/game", "new game", "make a game for class", or when a game is about to be built in the classroom.
user-invocable: true
args: "[name]"
---

# New game for class

All games live in this repo, each in its own folder with a single `index.html`. The overview of all games is online at https://peelparel-games.site/ and is updated automatically by `bin/new-game`.

## How to answer

The kids making the game are reading along. So write in plain language about the game, not about the code. Tell them what happens now when they play and which key or click goes with it, and keep it to a few lines: they want to play, not read.

Leave out of your answer: function names, file names, line numbers, commit hashes, color codes, pixel sizes, names of techniques and reports of what you tested. "The cat now jumps higher when you hold the space bar longer" is good. "I raised JUMP_FORCE to 470 and added EXTRA_JUMP_FORCE in the update loop" is not.

The code itself can be as technical as it needs to be, they don't look at it. Only the way you explain it has to be simple.

Do say so when there's something they need to know to keep going: a new key, something that works differently than they expected, or something you didn't manage to do.

## Step 1: Pick a name

Pick a short name in kebab-case together with the user (lowercase letters, digits, hyphens), for example `dino-run` or `mammoth-hunt`. If an argument was passed, use that. Check that the folder doesn't exist yet.

## Step 2: Create the game

```bash
cd "$(git rev-parse --show-toplevel)"
bin/new-game <name>
```

This copies `template/index.html` (a canvas with an empty game loop) to `<name>/index.html` and adds the game to the overview in `index.html`. A new game starts as an empty canvas: don't add anything the user didn't ask for, no example scene, no styling around the canvas. Every step comes from a prompt in class.

## Step 3: Start the dev server and open the browser

Check whether the server is already running, otherwise start it in the background, and open the game:

```bash
cd "$(git rev-parse --show-toplevel)"
curl -s -o /dev/null http://localhost:8000/ || (nohup bin/serve >/dev/null 2>&1 &)
sleep 1
xdg-open http://localhost:8000/<name>/
```

Check that a browser window really appears (for example with `hyprctl clients | grep -i chrom`). If `xdg-open` opens nothing without an error, start the browser by hand in the terminal (`chromium http://localhost:8000/<name>/`) so the real error shows up, and fix it.

The page reloads itself as soon as a file in the game folder changes. So every change you save is visible right away, nothing needs to be refreshed by hand.

## Step 4: Build

From here on, work in `<name>/index.html`. Rules:

- Everything in one `index.html` with inline JavaScript and the `<canvas>`, no frameworks, no build tools, no npm. Images or sounds may go in the game folder as separate files.
- Keep `<script src="../livereload.js"></script>` at the bottom of the body, otherwise auto-reload won't work.
- Use Dutch names in the code and add short comments, so you can still find your way around later. The code can get as technical as needed: the students look at the game, not at the file. Do make one clear change per prompt, so they see the effect in the browser right away.

## Step 4a: Check briefly, don't test extensively

The class is waiting to play, so keep checking to a few seconds. A syntax check on the script in the page is enough: then you know the game won't get stuck on a black screen.

```bash
cd "$(git rev-parse --show-toplevel)"
python3 -c "import io,re; s=io.open('<name>/index.html',encoding='utf-8').read(); io.open('/tmp/check.js','w').write(re.search(r'<script>(.*?)</script>',s,re.S).group(1))"
node --check /tmp/check.js
```

They test the rest themselves: the game is already open with livereload, so every change is visible immediately. Don't build test setups to play through the game: no bots that run a level, no rebuilt physics, no headless browser, no screenshots. That takes minutes and you don't have them.

If you change something that might make a level impossible (jump distances, floating platforms, a new route), it's better to just ask: "can you make it to the top?" They'll see it in one try, and meanwhile they get to play.

## Step 4b: Commit after every prompt

Commit after every prompt from the user, without asking. The commits are a log of the class: later you can read back which question led to which change. Only push when the user asks for it (step 6).

The commit message is a short title with the game name, and below it the user's literal prompt:

```bash
cd "$(git rev-parse --show-toplevel)"
git add -A && git commit -F - <<'PROMPT'
<name>: <short summary of the change>

Prompt:
<the user's prompt, word for word, not shortened or rewritten>
PROMPT
```

If the prompt itself contains a line with just `PROMPT`, use a different end word for the heredoc.

## Step 5: Rename

You often only know the final name at the end. Feel free to start with a working name and rename later:

```bash
cd "$(git rev-parse --show-toplevel)"
bin/rename-game <old-name> <new-name>
```

This moves the folder with `git mv`, updates the `<title>` and refreshes the overview. Then open http://localhost:8000/<new-name>/ in the browser and commit the rename following step 4b.

## Step 6: Put it online

When the user wants to share the game: push to `main`. GitHub Pages publishes it within a minute at https://peelparel-games.site/<name>/ and the overview at the root shows the new game.

```bash
cd "$(git rev-parse --show-toplevel)"
git push
```
