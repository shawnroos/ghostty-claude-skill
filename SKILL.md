---
name: ghostty-applescript
description: >
  Configure and automate Ghostty terminal via AppleScript. Use when the user asks to
  "set up Ghostty", "create a Ghostty layout", "split Ghostty", "open a dev workspace",
  "configure Ghostty splits", "send commands to Ghostty", "create a Ghostty tab layout",
  "automate terminal setup", "ghostty applescript", "terminal workspace", or wants to
  control Ghostty windows, tabs, splits, or send input programmatically. Also use when
  the user mentions arranging terminal panes for a project or creating a development
  environment layout.
version: 2.0.0
---

# Ghostty AppleScript Automation

You are an expert at automating the Ghostty terminal emulator via AppleScript on macOS.
You generate and execute AppleScript via `osascript` to control Ghostty windows, tabs,
splits, and input.

## First-Time Setup

On first activation, check if `~/.claude/skills/ghostty-applescript/presets.md` exists.

**If it does NOT exist**, run the setup flow before doing anything else:

1. Tell the user: "This is your first time using ghostty-setup. Let's configure your preferred layouts."

2. Use AskUserQuestion to ask: "How many preset layouts do you want to save? (You can always add more later)" with options: 1, 2, 3

3. For each preset, ask the user to describe it in natural language. Use AskUserQuestion with a free-text prompt like: "Describe preset 1 (e.g. '4 equal columns all running claude')"

4. After collecting all descriptions, save them to `~/.claude/skills/ghostty-applescript/presets.md` in this format:

```markdown
---
count: <N>
---

# Ghostty Layout Presets

## Preset 1: <short name>
<user's description>

## Preset 2: <short name>
<user's description>

## Preset 3: <short name>
<user's description>
```

5. Then proceed with the user's original request.

**If it DOES exist**, read it and use the saved presets. The `/ghostty-setup` command (with no args) will present these as the picker options.

## User's Ghostty Config

The user's config is at `~/.config/ghostty/config`:
- Font: JetBrainsMono Nerd Font, 14pt
- Theme: Dark+
- macOS option-as-alt enabled
- Quick terminal: Ctrl+Alt+Cmd+G

## Ghostty AppleScript Object Model

```
Application "Ghostty"
  └── Windows (window 1, window 2, ...)
       └── Tabs (tab 1, tab 2, ...)
            ├── selected tab  →  the active tab
            └── Terminals (terminal 1, terminal 2, ...)
                 └── focused terminal  →  the active pane
```

## AppleScript Commands

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
set initial working directory of cfg to "/path/to/project"
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
close someTerminal
close tab someTab
close window someWindow
```

## Execution Pattern

Always execute via `osascript` heredoc:

```bash
osascript <<'APPLESCRIPT'
tell application "Ghostty"
    activate
    set cfg to new surface configuration
    set initial working directory of cfg to "<project-dir>"
    -- ... create layout ...
    perform action "equalize_splits" on <any-pane>
    -- ... send commands ...
    focus <primary-pane>
end tell
APPLESCRIPT
```

## Rules

1. **Always activate Ghostty first**
2. **Always call `perform action "equalize_splits"`** after creating all splits — without this, panes cascade in size (50/25/12.5)
3. **Use `\n` for command execution** — `input text "cmd\n"` sends + executes
4. **After splitting, the return value IS the new terminal** — the original pane ref still points to the now-smaller original
5. **Focus the primary pane last**
6. **Ask for project path** if not obvious from context
7. **Name variables descriptively** — `paneEditor`, `paneLogs`, `paneServer`

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

- **"Can't get window 1"** — No windows open. Create one first.
- **Permissions dialog** — macOS TCC prompt on first run. User must approve.
- **Unequal splits** — Forgot `equalize_splits`. Always call it after all splits are created.
