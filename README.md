# Build Companion — a Claude Code skill

**A live, plain-language build board for non-developers.** Open it in a browser
window and always know what exists, what Claude is doing *right now*, what is
finished but not yet in front of real users, what hasn't been checked, and what
needs your approval — without ever hearing the words "branch", "commit", or
"merge".

Built for people who build with Claude Code but don't come from software:
founders, designers, operators, students.

It is a **reverse issue tracker**. Nobody writes tickets up front. Claude writes
them down as it builds, so one page tells you where everything actually stands.

## The one idea that makes it work

Every piece of work carries two facts, and the board refuses to confuse them:

| | |
|---|---|
| **how far it has got** | Idea → Planning → Building → Testing → Done |
| **where it actually runs** | in prod · on a preview link · nowhere yet |

Something is routinely **finished and not deployed**. That gap is the single
most useful column on the board, and it is the thing every other status tool
quietly hides by calling both of them "done".

## What it looks like

A warm, quiet, full-width board showing, per project:

- **A count across the top** — how many user stories exist, how many are live in
  prod, how many are built but not deployed, how many are being worked on, how
  many were discussed and never built.
- **Needs you** — the only loud element. "Ready to go to prod — reply in the
  chat to approve." Nothing is deployed without you.
- **Right now** — a live line of what Claude is doing this second ("Rewriting how
  large PDFs load…"), updating as it works.
- **Five columns of user stories** — each one a thing a *person* can do
  ("Reset a forgotten password"), not a thing the code has. Stories move
  backward without drama: "Back in Building — large files were too slow.
  Nothing is lost."
- **Not yet checked** — honest uncertainty, per story and per feature: "Works
  with sample files — not yet tested over 50MB."
- **Deploy history** — every time something reached real users, in plain words.
- **Parking lot** — everything that was discussed and never built, *with the
  reason*. Half of what gets talked about while vibe-coding never gets built and
  the reason is forgotten within a day. This is the cheapest thing on the board
  and the one people thank you for.

Everything renders from one human-readable file, `build-status.json`, that lives
in your project and that you own.

## How it works

```
┌─────────────────────┐        ┌──────────────────────────┐
│  Claude Code (chat)  │ writes │   build-status.json      │
│  = the workspace     │ ─────▶ │  (in your project root)  │
└─────────────────────┘        └───────────┬──────────────┘
                                     reads every 2s
                               ┌───────────▼──────────────┐
                               │  dashboard.html           │
                               │  (the board, in a browser)│
                               └──────────────────────────┘
```

- **`SKILL.md`** teaches Claude Code to keep the status file updated as a natural
  part of working: adding stories, moving them along, recording decisions,
  flagging uncertainty, parking dropped ideas, and asking before anything is
  deployed.
- **`dashboard.html`** is a single static file, zero dependencies, that polls the
  JSON and renders the board. No build step, no framework, no server logic.
- The board is **read-only by design**: decisions happen in the conversation, the
  board reflects them. That keeps the architecture honest and dead simple.

## Install

Requires [Claude Code](https://code.claude.com/docs) and Python 3 (only for the
one-line local file server).

Inside Claude Code, add this repo as a marketplace and install the plugin:

```
/plugin marketplace add ulrichmonthe/build-companion
/plugin install build-companion@build-companion
```

If the install summary says `Run /reload-plugins to activate.`, run that. Install
to **user scope** and it works in every project on your machine.

You get updates with `/plugin marketplace update build-companion`, and you can
remove it cleanly with `/plugin uninstall build-companion@build-companion`.

<details>
<summary>Or install it by hand, without the plugin system</summary>

The repository root <em>is</em> the skill, so cloning it into your skills folder
works too:

```bash
git clone https://github.com/ulrichmonthe/build-companion.git ~/.claude/skills/build-companion
```

Restart Claude Code. You won't get updates or a clean uninstall this way, but
nothing else differs.
</details>

## Use

In any project, just start building, or say:

> "Set up the build companion for this project."

Claude creates `build-status.json` and copies `dashboard.html` into the project.
Then serve the project folder from a second terminal:

```bash
python3 -m http.server 4321
```

Open http://localhost:4321/dashboard.html and give it a full browser window — the
board is wide, not a sidebar. From then on it updates itself as you and Claude
talk.

There is also a launcher script that serves the folder and opens the browser for
you, `scripts/dashboard.sh`. Its path depends on how you installed, so just ask
Claude to start the board and it will use the right one.

Try the demo without a real project: copy `assets/build-status.example.json` to a
folder as `build-status.json`, copy `assets/dashboard.html` beside it, run the
script.

## The status file is yours

`build-status.json` is plain, documented JSON (see
[`references/STATUS-FORMAT.md`](references/STATUS-FORMAT.md)). Open it, edit it,
commit it to git, diff it between sessions. The board is just a view; the file is
the source of truth. If you delete the board, the record survives.

Timestamps are ISO 8601 and rendered as relative time when you look at the board,
so nothing in the file goes stale and starts lying.

## Forking & customizing

This repo is meant to be forked. Common tweaks, all easy:

- **Rename the columns** — edit `STATES` and `LABEL` in `dashboard.html` and the
  state table in `SKILL.md`, keeping them in sync.
- **Change the look** — every color and font is a CSS variable at the top of
  `dashboard.html`. `--prod` and `--held` are the two that carry meaning: in prod,
  and built-but-not-deployed.
- **Change the voice** — the plain-language rules live in one section of
  `SKILL.md` ("Language rules"). Translate them, make them stricter, make them
  yours.
- **Share with a team** — put the skill in a repo's `.claude/skills/` folder
  instead of `~/.claude/skills/` and commit it; everyone who clones the project
  gets it.

If you build something good — different state models, other languages, a mobile
layout — PRs are welcome.

## Repo layout

```
build-companion/
├── .claude-plugin/
│   └── marketplace.json            # makes the repo installable as a plugin
├── SKILL.md                        # instructions Claude Code follows
├── README.md
├── LICENSE                         # MIT
├── assets/
│   ├── dashboard.html              # the board (single static file)
│   └── build-status.example.json   # demo data + init template
├── references/
│   └── STATUS-FORMAT.md            # full field-by-field schema
└── scripts/
    └── dashboard.sh                # serve + open the board
```

The repository root doubles as the skill folder, which is why both install
methods above work from the same layout: the marketplace entry uses
`"source": "./"` with `"skills": ["./"]`, and a manual clone lands `SKILL.md`
exactly where Claude Code looks for it.

## FAQ

**Does the board need internet?** No. Everything is local: your file, your
browser, a localhost server.

**Can the board's buttons control Claude?** No — approvals happen by replying in
the chat, and the board reflects them. Read-only is a feature: one source of
truth, no magic.

**Why is everything "built, not deployed"?** Because that is usually the truth,
and the board will not pretend otherwise. `env` only changes when a deploy
actually happens.

**Multiple projects at once?** Each project has its own file and board; run the
script on a different port per project (`bash dashboard.sh 4322`).

**What if Claude forgets to update the file?** Just ask "update the build status"
— the skill covers it. Power users can add a Claude Code
[hook](https://code.claude.com/docs/en/hooks-guide) (e.g. on `Stop`) to make
updates automatic every turn.

**I have an older `build-status.json`.** It still renders — the board maps the old
`pieces` and `stage` fields onto stories. Claude migrates a feature the next time
it touches it.

## License

MIT — see [LICENSE](LICENSE).
