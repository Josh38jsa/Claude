# My LifeOS install

A pre-built [LifeOS](https://github.com/danielmiessler/LifeOS) **v7.40.4** core install (upstream commit `5e2f2e8`), ready to drop into Claude Code on any machine.

| Folder | What it is | Goes to |
|---|---|---|
| `claude-config/` | LifeOS core: the LifeOS skill + full skill library, the `LIFEOS/` runtime (Algorithm, system prompt, tools, Pulse, statusline) | your Claude config dir — `~/.claude` (macOS/Linux) or `%USERPROFILE%\.claude` (Windows) |
| `USER/` | Your personal tree (identity, TELOS, contacts, projects) — currently blank templates | `~/.config/LIFEOS/USER`, linked into `LIFEOS/USER` |
| `LICENSE-LifeOS` | Upstream MIT license | — |

**Not included (on purpose):** hooks and LifeOS's `settings.json`. Those change Claude Code's permissions (they auto-allow many shell commands, set the default mode to `auto`, and skip the dangerous-mode prompt), so they're opt-in on your own machine — see step 4.

> ⚠️ This repo is **public**. Don't commit filled-in `USER/` files (TELOS, identity, finances, health) here — make the repo private first, or keep `USER/` in a private repo.

## Install on your computer

Prereqs: [Claude Code](https://docs.claude.com/claude-code), git, and [bun](https://bun.sh).

### 1. Install bun

- macOS / Linux: `curl -fsSL https://bun.sh/install | bash`
- Windows (PowerShell): `powershell -c "irm bun.sh/install.ps1 | iex"`

### 2. Copy the core into your Claude config (additive — never overwrites existing files)

macOS / Linux:
```bash
git clone https://github.com/Josh38jsa/Claude ~/lifeos-install
mkdir -p ~/.claude ~/.config/LIFEOS/USER
cp -Rn ~/lifeos-install/claude-config/. ~/.claude/
cp -Rn ~/lifeos-install/USER/. ~/.config/LIFEOS/USER/
```

Windows (PowerShell):
```powershell
git clone https://github.com/Josh38jsa/Claude $HOME\lifeos-install
# /XC /XN /XO = skip any file that already exists
robocopy $HOME\lifeos-install\claude-config $HOME\.claude /E /XC /XN /XO
robocopy $HOME\lifeos-install\USER $HOME\.config\LIFEOS\USER /E /XC /XN /XO
```

### 3. Install dependencies and link your USER tree

From your config dir (`~/.claude`, or `$HOME\.claude` on Windows):
```bash
cd ~/.claude
for d in . LIFEOS/TOOLS LIFEOS/TOOLS/TokenXray LIFEOS/PULSE LIFEOS/PULSE/Observability \
         skills/Apify skills/Art/Tools skills/Evals skills/Remotion/Tools \
         skills/Telos/ReportTemplate skills/Telos/DashboardTemplate skills/Prompting/Templates/Tools; do
  bun install --cwd "$d"
done
bun skills/LifeOS/Tools/LinkUser.ts            # dry run — shows the link
bun skills/LifeOS/Tools/LinkUser.ts --apply    # symlink (macOS/Linux) or directory junction (Windows)
```
On Windows, run the same `bun install --cwd <dir>` for each folder above in PowerShell.

### 4. Optional: hooks (always-on behavior)

Hooks give you the response format, memory loop, and per-turn context injection. They live in the upstream payload, so run them from a clone of upstream. **Read the dry-run output before applying** — it merges LifeOS's settings (including the broad permission rules above) into your `settings.json`, backing it up first.
```bash
git clone --depth 1 https://github.com/danielmiessler/LifeOS ~/LifeOS-upstream
cd ~/LifeOS-upstream/LifeOS
bun Tools/InstallHooks.ts              # dry run
bun Tools/InstallHooks.ts --apply
bun Tools/ActivateImports.ts --apply
```

### 5. Add the `lifeos` launch command

The LifeOS system prompt ("constitution") loads only when Claude is launched through the LifeOS launcher; plain `claude` stays vanilla.

- zsh / bash — add to `~/.zshrc` or `~/.bashrc`:
  ```bash
  alias lifeos='bun ~/.claude/LIFEOS/TOOLS/lifeos.ts -s ~/.claude/LIFEOS/LIFEOS_SYSTEM_PROMPT.md'
  ```
- Windows PowerShell — add to your `$PROFILE`:
  ```powershell
  function lifeos { bun "$HOME\.claude\LIFEOS\TOOLS\lifeos.ts" -s "$HOME\.claude\LIFEOS\LIFEOS_SYSTEM_PROMPT.md" @args }
  ```

### 6. Check capabilities and finish setup

```bash
bun ~/.claude/LIFEOS/TOOLS/Doctor.ts
```
Then start `lifeos` and ask it to run the **Interview** to name your assistant and capture your TELOS (current → ideal state).

## Windows notes

Claude Code on Windows gets full LifeOS (skills, USER data, hooks). The USER tree links as a directory junction (no admin needed). Pulse's background service and the worksweep/derivedsync jobs use launchd/systemd, so they're skipped on Windows; the menu-bar app is macOS-only.
