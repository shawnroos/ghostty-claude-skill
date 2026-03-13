---
name: ghostty-applescript
description: >
  Configure and automate Ghostty terminal via AppleScript (macOS only). Use when the user
  asks to "set up Ghostty", "create a Ghostty layout", "split Ghostty", "open a dev workspace",
  "configure Ghostty splits", "send commands to Ghostty", "create a Ghostty tab layout",
  "automate terminal setup", "ghostty applescript", "terminal workspace", "set up my coding
  environment", "open a multi-pane terminal", or wants to control terminal windows, tabs,
  splits, or send input programmatically. Also use when the user mentions arranging terminal
  panes for a project or creating a development environment layout. Do NOT use for iTerm,
  Warp, Terminal.app, or non-macOS environments.
version: 2.1.0
---

# Ghostty AppleScript Automation

You are an expert at automating the Ghostty terminal emulator via AppleScript on macOS.
You generate and execute AppleScript via `osascript` to control Ghostty windows, tabs,
splits, and input. All commands below are from Ghostty's native AppleScript dictionary
(verified in Ghostty 1.3.0+).

## First-Time Setup

On first activation, check if `~/.claude/skills/ghostty-applescript/presets.md` exists.

**If it does NOT exist** and the user invoked `/ghostty-setup` without args, run the
setup flow. Do NOT run setup mid-task if the user made a specific request like "split my
terminal" — just fulfill that request directly.

Setup flow:
1. Tell the user: "Let's save your preferred Ghostty layouts."
2. Ask how many presets (1-3) via AskUserQuestion
3. For each preset, ask for a natural language description
4. Derive a short name from each description (e.g., "4 columns all running claude" → "Quad Claude")
5. Save to `~/.claude/skills/ghostty-applescript/presets.md`:

```markdown
---
count: <N>
---

# Ghostty Layout Presets

## Preset 1: <Short Name>
<user's description>

## Preset 2: <Short Name>
<user's description>
```

**If it DOES exist**, read and use the saved presets.

**To edit presets**, the user can say "update my ghostty presets" or "add a preset" —
read the existing file, modify it, and save. Presets are not capped at 3 after initial
setup.

## User's Ghostty Config

Config at `~/.config/ghostty/config`:
- Font: JetBrainsMono Nerd Font, 14pt
- Theme: Dark+
- macOS option-as-alt enabled
- Quick terminal: Ctrl+Alt+Cmd+G (avoid targeting this window in scripts)

## Ghostty AppleScript Object Model

```
Application "Ghostty"
  └── Windows (window 1, window 2, ...)
       └── Tabs (tab 1, tab 2, ...)
            ├── selected tab  →  the active tab
            └── Terminals (terminal 1, terminal 2, ...)
                 └── focused terminal  →  the active pane
```

## AppleScript Commands (Ghostty-specific extensions)

### Creation & Layout
```applescript
set win to new window with configuration cfg
set t to new tab of win with configuration cfg
set pane to split someTerminal direction right with configuration cfg
set pane to split someTerminal direction down with configuration cfg
```

### Surface Configuration
```applescript
set cfg to new surface configuration
set initial working directory of cfg to "/Users/shawnroos/projects/myapp"
set font size of cfg to 14
set environment variables of cfg to {"EDITOR=nvim"}
set initial command of cfg to "npm run dev"
```

### Focus & Selection
```applescript
focus someTerminal
activate window someWindow
select tab someTab
set term to focused terminal of selected tab of front window
```

### Input & Actions
```applescript
input text "command\n" to someTerminal
send key "c" modifiers {"control"} to someTerminal
perform action "toggle_fullscreen" on someTerminal
perform action "equalize_splits" on someTerminal
```

### Closing
```applescript
-- Note: closing a terminal with a running process may kill it silently
close someTerminal
close tab someTab
close window someWindow
```

## Execution Pattern

Always execute via `osascript` heredoc with single-quoted delimiter (prevents bash variable expansion):

```bash
osascript <<'APPLESCRIPT'
tell application "Ghostty"
    activate
    delay 0.3

    set cfg to new surface configuration
    set initial working directory of cfg to "<full-project-path>"

    set win to new window with configuration cfg
    delay 0.3
    set pane1 to terminal 1 of selected tab of win
    -- ... create splits ...
    perform action "equalize_splits" on pane1
    -- ... send commands ...
    focus <primary-pane>
end tell
APPLESCRIPT
```

## Rules

1. **Always activate Ghostty first** — add `delay 0.3` after activate for cold-launch safety
2. **Always expand `~` to full paths** — use `/Users/shawnroos/...` not `~/...` in AppleScript
3. **Add `delay 0.3` after `new window`** — prevents timing issues with terminal references
4. **Always call `perform action "equalize_splits"`** after creating all splits — without this, panes cascade in size (50/25/12.5)
5. **Use `\n` for command execution** — `input text "cmd\n"` sends + executes
6. **After splitting, the return value IS the new terminal** — the original pane ref still points to the now-smaller original
7. **Focus the primary pane last**
8. **Ask for project path** if not obvious from context
9. **Name variables descriptively** — `paneEditor`, `paneLogs`, `paneServer`
10. **Distinguish "panes" (splits) from "tabs"** — if the user's intent is ambiguous, ask

## Split Patterns

**N equal columns:**
```applescript
set col1 to terminal 1 of selected tab of win
set col2 to split col1 direction right with configuration cfg
set col3 to split col2 direction right with configuration cfg
perform action "equalize_splits" on col1
```

**Column with vertical split:**
```applescript
set colTop to split somePane direction right with configuration cfg
set colBottom to split colTop direction down with configuration cfg
```

**Tabs:**
```applescript
set tab2 to new tab of win with configuration cfg
set tab2term to terminal 1 of tab2
```

## Troubleshooting

- **"Can't get window 1"** — No windows open, or script ran before window rendered. Add `delay 0.5` after `activate` or `new window`.
- **Permissions dialog** — macOS TCC prompt on first run. User must grant Accessibility permissions to their terminal app. This only happens once.
- **Unequal splits** — Forgot `equalize_splits`. Always call it after all splits are created.
- **Quick terminal targeted** — If the Ghostty quick terminal (visor) is focused, `front window` may refer to it. Create a `new window` explicitly to avoid this.
- **Tilde paths fail** — AppleScript doesn't expand `~`. Always use absolute paths like `/Users/shawnroos/...`.
