# Project Logbook — a project journal for AI agents

A plugin for Claude Code (and Codex). The journal keeps the past chats of Claude Code, Cowork (on
your own computer) and Codex that belong to a project folder, together with the folder's git log.
It searches them, and at the start of a new chat it gives a short brief: where the work stopped,
what was decided, which tasks are open. The agent does not have to rediscover what was already
discussed, and you do not have to retell past chats.

Everything runs on your computer. The journal only reads chat transcripts and git history; nothing
is sent anywhere. A Russian version of this text is in `README.ru.md` in the plugin folder.

## What it does

- **A brief at the start of a chat.** In Claude Code the SessionStart hook brings it. Where hooks do
  not fire, the agent fetches the same brief itself with `journal_brief`; the plugin's skill and the
  MCP server tell it to.
- **The `journal` MCP server, eight tools:**
  - `journal_brief` — what to know at the start of a chat: the latest chats and their commits, the
    documents that describe the project's current state and how old they are, the project's
    languages, open tasks;
  - `journal_search` — search across chats and commits: quotes with the turns around them, changed
    files, commits;
  - `journal_recent` — the project's latest chats;
  - `journal_session` — one whole chat, page by page when it is long;
  - `journal_tasks` — the project's tasks from its files, marked when another chat has taken one and
    since when; with `key`, one task in full and the chats that worked on it;
  - `journal_status` — which folders are connected, how many chats and commits they hold, when the
    journal was last updated;
  - `journal_connect` — connect a project folder (only when the person asks);
  - `journal_open` — build and open the local journal page.
- **Search without dictionaries.** Chats and commits are searched separately: first all words
  together, then word stems, then any word. A stem is a long word without its last letters, for any
  language: "connection" finds "connecting". A commit that matches names the chat that made it. That
  chat ranks higher, and if its own words did not match, it still takes a free slot in the results
  with the person's message before the commit. So a query in the commit's words finds the
  conversation even when it was held in another language. Results stay compact: up to 8 files per
  chat and the commits close in time to the matching turns, plus how many there are in total; the
  full list is in `journal_session`.
- **Tasks from project files.** One task is one file `tasks/<key>.md` with the fields `status`,
  `owner`, `priority`, `group`, `area`, `session`, `created`, `source`, `closed`. The journal reads
  them from the main branch and from every local branch, so a task taken in another branch shows as
  taken. Tasks are read from git, not from the working folder: an uncommitted edit is not visible.
  - Each task shows when its file last changed in the main branch (`touched`) and since when it has
    been marked as taken (`taken.since`). A mark older than 12 hours without new commits is probably
    abandoned: the `journal_tasks` answer and the "Copy task" text suggest checking git and taking
    the task instead of waiting for that chat.
  - Common field and value names in English, German, Russian, Spanish, French and Italian (`wer`,
    `prio`, `приоритет`, `erledigt` …) are understood without a profile. The journal finds a
    project's own folder, field and value names itself when you connect it, or takes them from the
    project profile (see below).
- **The journal page.** One local HTML file that works offline: a control centre (where the work
  stopped, activity, tasks), a map of chats and code areas, and a day-by-day timeline. The page and
  the command line are in English or Russian (see "Languages").
- **The `project-logbook` skill** — when and how the agent should ask the journal.
- **Claude Code commands:** `/project-logbook:connect`, `/project-logbook:view`,
  `/project-logbook:status`.

An example task file:

```markdown
---
status: next
owner: agent
priority: high
group: core
---
# Import: an empty file must not break the upload

What to do and why.
Check: what is not verified yet.
```

Default values: `status` — `now`, `next`, `later`, `waiting`, `in_progress`, `done`; `owner` —
`agent`, `human`, `external`; `priority` — `high`, `medium`, `low`.

## Example prompts

Once a folder is connected, ask in plain words; the agent picks the tool:

- "Where did we stop last time?" — the brief and `journal_recent`.
- "Did we already try caching the import? What came of it?" — `journal_search`, then
  `journal_session` for the whole discussion.
- "Why was the upload limit set to 20 MB?" — the chat that decided it and its commit.
- "Which task can I take next?" — `journal_tasks`, with the tasks other chats have taken marked.
- "Show me the journal page." — `journal_open`.

## How it differs

The journal differs from similar tools in what it remembers and where it takes it from.

- **A map of the work, not of the code.** Code indexes and graphs show which functions exist and who
  calls whom; the agent reads code by itself anyway. The journal shows what the code does not: what
  was discussed, what was decided and why, what was tried and dropped, which chats and commits
  touched this part of the project.
- **Nobody has to keep notes.** Memory plugins usually rely on the agent writing down what matters,
  or ask a model to summarise the session. If the agent forgot to write it down, there is no memory.
  The journal is built from what stays on disk anyway: chat transcripts and git history.
- **Several agents, one history.** Chats of Claude Code, Cowork and Codex in one folder go into a
  shared journal, and each of them sees the work of the others.
- **A chat belongs to a project for a reason.** A chat counts for a project if it was started there,
  changed files there or made a commit there. Merely mentioning the project is not enough.

Also:

- **An archive, not a link.** The journal stores the conversations in its own database, so the
  history survives deletion of the original transcripts.
- **Sources are only read.** The journal does not change agent transcripts or repositories.
- **Everything is local.** Nothing is sent anywhere; the server runs on your computer.
- **No dependencies.** Only Node.js is needed: the database is the SQLite with full-text search that
  is built into Node; nothing is downloaded or installed through npm.
- **One journal for all agents.** The journal server in Claude Code, Cowork and Codex works with the
  same data folder.
- **Passwords and keys are masked in quotes** (passwords, tokens, API keys, private keys). The words
  "password" and "key" are recognised in common languages; add your own in the profile
  (`secretWords`). This protects against an accidental leak; it is not a guarantee.

## Requirements

- **Node.js 22.13 or newer** in `PATH` (the built-in `node:sqlite` is needed). Version 24 is
  recommended. Check with `node --version`.
- **git** in `PATH` — for the commit history. Without it the journal is built from chats only.
- **Claude Code 2.1.139 or newer** — from this version the hook runs without a shell (the `command` +
  `args` form).
- Tested with Claude Code 2.1.257 and Node.js 24.14 on Windows: `claude plugin validate --strict`
  passes, and the MCP server from the plugin connects (`claude mcp get` → Connected). Installation
  in Codex 0.131 on Windows: `codex plugin add` installs the plugin, and `codex mcp list` shows the
  `journal` server.

## Installation

The plugin's repository is a marketplace for Claude Code and Codex: its root holds
`.claude-plugin/marketplace.json` (Claude Code) and `.agents/plugins/marketplace.json` (Codex), and
the plugin itself is in `plugins/project-logbook`. The plugin folder (and the zip) is also a
one-plugin marketplace, for Claude Code only.

Up to 1.1.1 the plugin was called **agent-logbook**. To move over, uninstall `agent-logbook` and
install `project-logbook` as below: the journal moves to its new folder by itself on the first start.

### Claude Code — the main way

From the git repository:

```bash
claude plugin marketplace add <owner>/project-logbook
claude plugin install project-logbook@project-logbook
```

Claude Code keeps its own copy of the repository in `~/.claude/plugins/marketplaces/project-logbook/`
and a copy of the plugin in its cache, so you do not need to keep a folder of your own.

From a folder on disk, first unpack the zip where the folder will stay for good, for example
`C:\Tools\claude-plugins\project-logbook` (the `.claude-plugin` folder must be inside it):

```bash
claude plugin marketplace add "C:\Tools\claude-plugins\project-logbook"
claude plugin install project-logbook@project-logbook
```

- `marketplace add` takes the full path to the unpacked folder, not to the zip file, and without a
  trailing `\` (in PowerShell `"…\"` breaks the path). The error "Invalid marketplace source format"
  means a placeholder such as `<name>` is still there, or a bare folder name is used: write a
  relative path as `./project-logbook`.
- A plugin added from a folder runs right from that folder: do not move or delete it, or Claude Code
  reports "failed to load: cache-miss". So not Downloads and not a temp folder.

Then start a new chat, or run `/reload-plugins` in an open one.

For one session, without installing:

```bash
claude --plugin-dir "<plugin folder>"
```

### Claude Desktop

- **The Code tab** uses Claude Code plugins: the plugin installed above should appear there too.
  [not verified]
- **Upload as a file:** Customize › Plugins › Upload plugin, file `project-logbook-<version>.zip`. The
  app accepts only `.zip` ([#40414](https://github.com/anthropics/claude-code/issues/40414)).
  - An uploaded plugin does not update itself
    ([#93553](https://github.com/anthropics/claude-code/issues/93553)): upload each new version again.
  - Do not use it together with the marketplace installation: the brief and the journal server
    would run twice.

### Codex

Codex installs the skill and the MCP server from the repository's marketplace, without the hook:

```bash
codex plugin marketplace add <owner>/project-logbook
codex plugin add project-logbook@project-logbook
```

Installation is verified (Codex 0.131, Windows): `codex plugin add` installs the plugin, and
`codex mcp list` shows the `journal` server. Work in a live Codex session is not verified, because
it needs an account sign-in. Codex does not accept the plugin folder (zip) as a marketplace: it
needs the repository or a local copy of it, with the plugin in `plugins/project-logbook`.

Without a marketplace you can connect only the MCP server:

```bash
codex mcp add journal -- node "<plugin folder>/server/journal.mjs" mcp
```

If the journal is empty in Codex but full in Claude Code, Codex did not pass the server the
environment variables with the data folder. Then set it explicitly: add
`--env PROJECT_LOGBOOK_HOME=<data folder>` (see "Data and what the journal reads"). Codex has no hook:
the agent fetches the brief itself with `journal_brief`.

## First steps

1. Open a chat in the project folder and run `/project-logbook:connect` (you can name the folder:
   `/project-logbook:connect <folder>`). The journal is built in the background and reads the
   **whole** chat history of this folder, so connect only your own projects.
2. `/project-logbook:status` — whether the journal is ready: how many chats and commits it holds.
3. `/project-logbook:view` — open the journal page in the browser.

You cannot connect a drive root or your home folder: every chat on the computer would end up in the
journal. In agents without commands (Codex) the `journal_connect` tool does the same: ask the agent
to connect the folder.

From then on, every new chat in this folder (and its subfolders) gets the brief and can search the
journal. The journal updates itself: when a chat starts and whenever it is asked.

## Project profile

Usually no profile is needed. When you connect a project, the journal looks for its task folder (a
folder of Markdown files whose front matter has a status field), recognises the field and value
names and the check line. If the project has no profile yet, what was found is written to this
machine's `projects.json` (see below); the journal writes nothing into the project and asks
nothing. The connect answer says what was found and what was not recognised; what was not
recognised stays as it is, and you can add it to the profile.

If the project names its tasks, fields and statuses in a way the journal did not recognise, keeps
its own state documents, or wants to set languages, put a `.project-logbook.json` file in the project
root. Every section and key is optional; anything missing takes the default. An example:

```json
{
  "tasks": {
    "dir": "todo",
    "ref": "main",
    "fields": { "state": "status", "who": "owner", "prio": "priority", "chat": "session" },
    "values": {
      "status": { "doing": "in_progress", "today": "now", "soon": "next", "someday": "later", "blocked": "waiting", "closed": "done" },
      "owner": { "bot": "agent", "me": "human" },
      "priority": { "p1": "high", "p2": "medium", "p3": "low" }
    },
    "checkPrefix": "Verify:",
    "groupOrder": ["core", "docs"],
    "labels": { "group": { "core": "Core", "docs": "Documentation" } }
  },
  "stateDocs": ["docs/STATE.md"],
  "handoffs": { "dirs": ["docs"], "prefix": "HANDOFF-" },
  "dashboard": { "file": "docs/dashboard.json", "sections": { "decisions": "decided" } },
  "languages": ["en", "de"],
  "uiLanguage": "en",
  "secretWords": ["PIN code", "passphrase"]
}
```

- `tasks.dir` — the task folder (default `tasks`); `tasks.ref` — the branch tasks are read from
  (default `main`, then `master`).
- `tasks.fields` — your field name → the standard one; `tasks.values` — your value → the standard
  one for `status`, `owner`, `priority`. The tools answer in standard words, and the hint "how to
  take and close a task" uses the project's words.
- `tasks.checkPrefix` — how the "what to check" line starts (default `Check:`); `tasks.groupOrder` —
  which group matters more; `tasks.labels` — labels on the page for `owner`, `group`, `area`.
- `stateDocs` — where the project describes its current state, in addition to the standard
  `STATUS.md`, `TODO.md`, `HANDOFF.md` (in the root and in `docs/`). `handoffs` — a series of dated
  handoffs `<prefix>YYYY-MM-DD….md` in the folders `dirs`; the newest one goes into the brief.
- `dashboard` — the project's own task panel: a JSON file, or an HTML page whose data sits in
  `const DATA = {…}` (the variable name is the `variable` key). The journal page shows its sections
  `focus`, `waiting`, `queue`, `groups`, `blocks`, `decisions`, `table`, `flags`, `docs`; `sections`
  maps a standard section name → the key in the file, `titles` sets your own headings. The file is
  only read; no code from it runs.
- `languages` — the language codes to search in (`en`, `de`, `ru` …); they replace the languages
  the journal detected itself, in the search hint and in the `languages` field of the
  `journal_search` answer.
- `uiLanguage` — the language of the page and the command line: `en` or `ru` (see "Languages").
- `secretWords` — your own words after which a value in quotes is masked, as after "password"
  (`PIN code: …`). They apply to all connected projects at once, but only to what the journal reads
  after the profile changes: a chat that no longer changes and commits already collected stay as
  they were.
- All paths are inside the project; the journal skips a path that leads outside.

To describe a project on one computer without touching its repository, use the `projects.json` file
in the journal's data folder: the key is the full path of the project folder, the value is the same
profile. It overrides `.project-logbook.json` section by section. The journal also writes what it
found when connecting there.

## Languages

The journal detects by itself which languages the work is in: separately for people's messages,
agents' replies and commit messages, from the latest 300 of each kind. A text's language is
recognised by its script and frequent short words, without dictionaries (22 languages); a language
counts for the project if at least one sixth of the texts are in it. The brief names them in one
line, for example "Languages: people write Russian; commits are in German. Search in each of
Russian, German."; the `journal_search` answer has a `languages` field. So the agent knows which
languages to search in.

Texts for the agent — the brief, the hints and the tool errors — are always in English. The journal
page and the command line are in English or Russian, chosen in this order:

1. `uiLanguage` in the project profile;
2. the environment variable `PROJECT_LOGBOOK_LANG` (`en` or `ru`);
3. the language the project's people write in;
4. the system language;
5. English.

A language without a translation (German, for example) is skipped. The people's language comes
before the system language: on a computer set up in one language people often write in another,
and they are the ones who read the page. Command-line commands that are not about one project (for
example `status`, `sync`, help) take the language only from `PROJECT_LOGBOOK_LANG` and the system.

## Permissions

By default Claude Code asks for permission on every MCP tool call. The six reading tools can be
allowed in advance, with `/permissions` or in `permissions.allow` in the settings:

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_project-logbook_journal__journal_brief",
      "mcp__plugin_project-logbook_journal__journal_search",
      "mcp__plugin_project-logbook_journal__journal_recent",
      "mcp__plugin_project-logbook_journal__journal_session",
      "mcp__plugin_project-logbook_journal__journal_tasks",
      "mcp__plugin_project-logbook_journal__journal_status"
    ]
  }
}
```

Better keep `journal_connect` and `journal_open` behind a confirmation.

## Data and what the journal reads

The data folder:

- Windows: `%LOCALAPPDATA%\ProjectLogbook`;
- macOS and Linux: `~/.project-logbook`;
- elsewhere — the environment variable `PROJECT_LOGBOOK_HOME`.

It holds `journal.sqlite` (the journal itself), `connected.json` (connected folders), `view/*.html`
(pages), `hook.log` and `sync.log` (failures of the hook and of the background refresh, shown by
`journal_status`; the files are only appended to and can be deleted), and
`projects.json` (project profiles: yours, or found when connecting).

What the journal reads (read only), and only for folders you connected yourself:

- Claude Code chats — `~/.claude/projects`;
- local Cowork sessions — Windows `%APPDATA%\Claude\local-agent-mode-sessions`, macOS
  `~/Library/Application Support/Claude/local-agent-mode-sessions`, Linux
  `~/.config/Claude/local-agent-mode-sessions`;
- Codex chats — `~/.codex/sessions` and `~/.codex/archived_sessions`;
- the git log of connected folders.

It does not read claude.ai chats, Claude's memory or anything in the cloud, and it builds no
profile of you: it gives the agent working in a project the history of that same project, on the
same computer.

The journal page is a local file with excerpts from your own chats. Do not publish it or send it to
anyone.

## What runs on your computer

Everything below runs locally under your user account; the plugin makes no network requests.

- **`node server/journal.mjs mcp`** — the MCP server, started by Claude Code (or Codex) for the
  session.
- **`node server/journal.mjs hook-start`** — the SessionStart hook in Claude Code: prints the brief
  (at most 2000 characters) and exits; it starts no other process.
- **`node server/journal.mjs refresh`** and **`connect`** — background processes the MCP server
  starts to read new chats and commits into the journal (when it is older than ten minutes, and
  after a folder is connected); they exit when done.
- **`git`** — read-only commands (`log`, `rev-list`, `rev-parse`, `branch`, `for-each-ref`,
  `ls-tree`, `cat-file`) in connected folders.
- **The system file opener** (`explorer.exe`, `open` or `xdg-open`) — only for `journal_open` and
  `/project-logbook:view`, to show the local journal page.
- **`node server/journal.mjs view --serve`** — only when you run it yourself from a terminal: a
  page server bound to `127.0.0.1:47770`, reachable from this computer only.

## Turning it off and removing it

- **Turn off:** `/plugin` → the `project-logbook` plugin → Disable, or
  `claude plugin disable project-logbook@project-logbook`.
- **Remove the plugin:** `claude plugin uninstall project-logbook@project-logbook`, then, if you no
  longer need the marketplace, `claude plugin marketplace remove project-logbook`.
- **Remove the data:** the journal lives apart from the plugin and stays after the plugin is
  removed. Delete the data folder by hand (see above).
- **Disconnect one folder:** `node "<plugin folder>/server/journal.mjs" disconnect "<project folder>"`.
  What was collected stays in the journal, but the brief and search no longer work for that folder.
  When installed from a folder, `<plugin folder>` is that folder; when installed from a git
  repository, it is the cached copy: `~/.claude/plugins/cache/project-logbook/project-logbook/<version>`
  (Claude Code), `~/.codex/plugins/cache/project-logbook/project-logbook/<version>` (Codex).
- If you connected the journal by hand before (an MCP server or a SessionStart hook in your own
  settings), remove them, or the brief comes twice.

## Limitations and what is not verified

- In the Claude Desktop app SessionStart hooks rarely fire at the moment; the journal still arrives
  through MCP.
- The SessionStart hook in a live Claude Code session is verified only by emulating a shell-less
  launch: the same `node` with the same arguments, from a folder with a space in its path.
- Cowork on your own computer is not verified. Cowork in the cloud and claude.ai chat are not
  supported: the journal lives on your computer.
- Uploading the `.zip` into the app: the file does not update itself
  ([#93553](https://github.com/anthropics/claude-code/issues/93553)), and it cannot be combined with
  the marketplace installation.
- Codex: installation is verified (`codex plugin add`; `codex mcp list` shows `journal` with the
  plugin folder as its working folder); work in a live session is not: whether the server starts
  with this list of environment variables, and whether the MCP server's instructions reach the
  model. Codex has no hook; the agent fetches the brief itself with `journal_brief`.
- macOS and Linux are not tested: the data and Cowork paths, opening the page (`open`, `xdg-open`).
  On macOS an app not started from a terminal may not see `node` from the shell's `PATH`.
- Not verified: whether pre-allowing tools in commands (`allowed-tools`) works, and whether
  `${CLAUDE_PROJECT_DIR}` is substituted in command text. At worst Claude Code asks for permission
  and the agent uses its own working folder.
- The Claude Code minimum of 2.1.139 comes from the documentation (the shell-less hook form); the
  tested version is in "Requirements".

## License

MIT — see `LICENSE`. The page fonts (IBM Plex Sans, IBM Plex Mono, Bricolage Grotesque) are under
the SIL Open Font License 1.1, text in `view/fonts/FONTS-OFL-1.1.txt`. Where the page design comes
from — `view/NOTICE.md`.
