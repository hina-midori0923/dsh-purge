<p align="center">
  <img src="docs/banner.svg" alt="dsh-purge" width="720">
</p>

<h1 align="center">dsh-purge — 破甲 for DeepSeek Harness (maintained fork)</h1>

<p align="center">
  <strong>Version 1.1.73</strong> · DSH 0.2.x · maintained by
  <a href="https://github.com/hina-midori0923">hina-midori0923</a>
</p>

> **This is a maintained fork.** The original repository `YuJunZhiXue/dsh-purge` and its npm
> package are gone (verified 2026-10-22). This fork carries the DSH 0.2.1 compatibility fix
> (the host no longer registers the `{{cwd}}` prompt variable, which used to break prompt
> assembly) and is the install/update source from now on. Attribution and backup history:
> [MIRROR-NOTICE.md](MIRROR-NOTICE.md).

---

## What this is

`dsh-purge` is a local plugin for the official **DeepSeek Harness**. It does not change the
model. It changes what the host puts in front of the model: the system prompt, the identity
section, the refusal/approval wording, and a set of default policies that ship with DSH.

After **Apply**, the harness runs with your prompt instead of its own: the official identity
is stripped, the plugin's operator prompt is injected on every assembly, and optional
code-level patches adjust sandbox/approval/permission defaults. The host's tools, skills and
MCP connections stay available.

Supported hosts: official **desktop EXE** (`0.2.1-rc.2` tested) and official **`dsh web`**.
Install and apply them separately — a Web install does not reach the desktop app.

## What you get

| Area | What it does |
|---|---|
| **dsh-purge dock** | Button beside the session title; opens the right dock with **Clean** and **Drill** pages |
| **Clean** | Patch status by group, **Apply** / **Restore** / **Uninstall**, prompt editor, multiple rule sets |
| **Prompt inject** | Your prompt (prompt-inject.md / rule set) is injected at every assembly; official identity and refusal phrasing are stripped |
| **Host policy** | Optional code-level patches for sandbox, approval, permission and tool-nudge defaults |
| **Drill** | Red-team workflow: presets, role prompts, skills and an asset store, for local/offline or written-authorized targets |
| **On start** | Checks and reapplies patches; optional auto-retry when the model starts refusing, optional auto-continue |

The plugin never reads the host's prompt file. The identity comes from the encrypted prompt
shipped in the plugin, the one the prompt box shows by default.

## Requirements

- DeepSeek Harness **0.2.x** (desktop `0.2.1-rc.2` or official `dsh web`). Older `0.1.x` hosts
  need the old plugin line; do not apply this one.
- Node.js **≥ 18** for the CLI/patch helpers. Windows no-flash needs **Node ≥ 22**.
- `$DSH_HOME` (usually `~/.dsh`). No hardcoded drive letters.

## Install (this repository)

```sh
# official dsh web
dsh plugin --profile web add github:hina-midori0923/dsh-purge

# official desktop EXE
dsh plugin --profile desktop add github:hina-midori0923/dsh-purge
```

If `dsh` is not on PATH, use the CLI shipped with DSH, or install manually:

1. Clone this repository to `$DSH_HOME/plugins/dsh-purge` (the folder name must be `dsh-purge`;
   an existing clone is fine):
   ```sh
   git clone https://github.com/hina-midori0923/dsh-purge.git "$DSH_HOME/plugins/dsh-purge"
   ```
2. Add it to the profile you use (`$DSH_HOME/profiles/web/package.json` or
   `.../desktop/package.json`): add `"dsh-purge": "file:../../plugins/dsh-purge"` to
   `dependencies`, and `"dsh-purge"` at the end of `dsh.profile.bundles`.
3. Run `pnpm install` inside that profile only.
4. Restart DSH.

Tarball alternative when GitHub fetch is unreliable:
`https://github.com/hina-midori0923/dsh-purge/archive/refs/heads/master.tar.gz`.

## First run

1. Fully quit and reopen DSH (desktop: quit the tray, then reopen the EXE).
2. Open any session, click **dsh-purge** beside the session title.
3. On **Clean**, click **Apply**. A successful apply restarts the host; an unfinished apply
   does not.
4. After the restart, start a **new** chat — that is when the inject enters the session.

If the dock does not show up, the plugin is not loaded from this repository; re-check the
profile's `dependencies` / `dsh.profile.bundles`.

## Update

The plugin's own update check looks at this fork's tags (`lib/update.js` → `hina-midori0923/dsh-purge`).

- From the plugin: **Clean** page → check update.
- From the CLI: `dsh plugin --profile desktop update dsh-purge` (same for `web`), then restart.
- Release flow for maintainers: bump `version` in `package.json`, push `master`, then push a
  `vX.Y.Z` tag. Installed copies resolve the git spec to a commit, so an update re-resolves
  `master` — bumping the version makes the change obvious in the UI.

## Restore and uninstall

- **Restore** on the Clean page reverts the patched host files from the backups the plugin made.
- **Uninstall** removes the plugin from the profile and re-applies the originals.
- Agent tool `purge_revert` does the same restore; `purge_status` shows patch/shim status;
  `purge_apply` applies the patch set.
- `$DSH_HOME/prompt-inject.md` is user data and survives restore/uninstall.

## Compatibility notes

- Host patches target the DSH 0.2.x file layout. After a DSH upgrade, patches whose original
  text no longer matches show as **skipped** and are left untouched; Apply still completes.
- The host is patched in its unpacked `app` directory. If the installation still ships
  `app.asar`, the status page reports that the running process reads the archive instead of
  the patched files — prompt injection (runtime section) still works; code-level patches
  follow the status page's hint.
- pnpm/npm may overwrite `node_modules` on upgrade; the plugin re-applies on start so you do
  not need to hand-edit installed files.
- No hardcoded paths: it resolves `$DSH_HOME`, `.dsh` next to the launcher, then `~/.dsh`.
- It only touches the official `@deepseek-ai` packages and local config on your machine. It is
  not a public scanner and does not ship payloads aimed at third-party sites.

## Credits and license

- Original project: **dsh-purge** by **YuJunZhiXue** (original repository
  `YuJunZhiXue/dsh-purge`, deleted 2026-10-22).
- Backup/mirror record and the differences made by this fork: [MIRROR-NOTICE.md](MIRROR-NOTICE.md).
- License: **MIT** (see [LICENSE](LICENSE), kept unchanged). The original author is credited
  here; if you redistribute or build on this fork, keep the license and the credit.
