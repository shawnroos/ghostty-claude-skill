---
name: ghostty-setup
description: Set up a Ghostty workspace layout from a description or preset
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion
args: layout description (e.g. "4 columns all running claude in ~/projects/Slate")
---

# Ghostty Setup

Set up a Ghostty terminal workspace from a description or saved preset.
Requires macOS with Ghostty 1.3.0+ and Accessibility permissions granted.

## With args: `/ghostty-setup <description>`

Parse the description and build the layout directly. Examples:
- `/ghostty-setup 4 columns all running claude in ~/projects/Slate`
- `/ghostty-setup 2 panes: left running micro, right running claude`
- `/ghostty-setup 3 cols, first two with claude, third is clean terminal`

If ambiguous whether user means splits or tabs, default to splits (panes).

## Without args: `/ghostty-setup`

Read presets from `~/.claude/skills/ghostty-applescript/presets.md`.

**If presets file doesn't exist**, run first-time setup:
1. Tell the user you'll save their preferred layouts
2. Ask how many presets (1-3) via AskUserQuestion
3. For each preset, ask for a natural language description
4. Derive a short name from each (e.g., "4 columns all running claude" → "Quad Claude")
5. Save to `~/.claude/skills/ghostty-applescript/presets.md`
6. Then present the picker with those presets

**If presets file exists**, present saved presets via AskUserQuestion with preview
diagrams. Example preview format:

```
╭──────────┬──────────┬──────────┬──────────╮
│  claude  │  claude  │  claude  │  claude  │
╰──────────┴──────────┴──────────┴──────────╯
```

After selection, ask for project directory if not obvious, then execute.

## Execution

Follow the AppleScript rules from the `ghostty-applescript` skill. Key rules:

- Always `activate` Ghostty first, then `delay 0.3`
- **Expand `~` to full paths** — use `/Users/shawnroos/...` in AppleScript, never `~/`
- Add `delay 0.3` after `new window` before referencing terminals
- **Always call `perform action "equalize_splits"` after all splits**
- Use `input text "command\n"` to run commands
- Focus primary pane last
- Use single-quoted heredoc `<<'APPLESCRIPT'` to prevent bash variable expansion

### Template
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
    -- ... splits ...
    perform action "equalize_splits" on pane1
    -- ... commands ...
    focus <primary-pane>
end tell
APPLESCRIPT
```

## After execution

Confirm with a brief summary of what was created.
