---
name: ghostty-setup
description: Set up a Ghostty workspace layout from a description or preset
allowed-tools: Bash, Read, Write, AskUserQuestion
args: layout description (e.g. "4 columns all running claude in ~/projects/Slate")
---

# Ghostty Setup

Set up a Ghostty terminal workspace from a description or saved preset.

## With args: `/ghostty-setup <description>`

Parse the description and build the layout directly. Examples:
- `/ghostty-setup 4 columns all running claude in ~/projects/Slate`
- `/ghostty-setup 2 panes: left running micro, right running claude`
- `/ghostty-setup 3 cols, first two with claude, third is clean terminal`

## Without args: `/ghostty-setup`

Read presets from `~/.claude/skills/ghostty-applescript/presets.md`.

**If presets file doesn't exist**, run first-time setup:
1. Tell the user this is their first time and you'll save their preferred layouts
2. Ask how many presets (1-3) via AskUserQuestion
3. For each preset, ask for a natural language description via AskUserQuestion
4. Save to `~/.claude/skills/ghostty-applescript/presets.md`
5. Then present the picker with those presets

**If presets file exists**, present saved presets via AskUserQuestion with preview diagrams showing the layout using rounded box-drawing characters (╭╮╰╯│─┬┴├┤┼).

After selection, ask for project directory if not obvious, then execute.

## Execution

Generate and run AppleScript via `osascript` heredoc. Follow these rules:

- Always `activate` Ghostty first
- Use `new surface configuration` with working directory
- Use `split <terminal> direction right` for columns
- Use `split <terminal> direction down` for rows
- **Always call `perform action "equalize_splits"` after all splits**
- Use `input text "command\n"` to run commands
- Focus primary pane last

### Template
```bash
osascript <<'APPLESCRIPT'
tell application "Ghostty"
    activate
    set cfg to new surface configuration
    set initial working directory of cfg to "<project-dir>"

    set win to new window with configuration cfg
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
