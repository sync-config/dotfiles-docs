## `bin/ide`: 3-Pane Tmux workspace Setup

The `bin/ide` script automates the creation of a standardized 3-pane development layout within your active `tmux` session.

## Preview

## Key Features

- **Clean Slate Execution:** The script automatically kills other existing in the current window before applying the layout to prevent overlapping or invalid split.
- **Enviroment Validation:** Ensures the script is running inside a valid `tmux` session and checks for the `tmux` binary.
- **Robust Splitting:** Attempts to create panes using pixel-based sizing first, falling back to percentage-base sizing for wider compatibility.
- **Customizable:** Supports runtime overrides for the agent command and pane dimensions.

## 1. Workspace Layout

The script devides your current terminal window into three functional panes:

```text
+------------------------------------------+-----------------------+
| | |
| Master Pane | |
| (Primary Editor / Coding) | Agent Pane |
| | (AI Runtime) |
+------------------------------------------+ |
| | |
| Terminal Pane | |
| (Builds, Tests, Shell Logs) | |
+------------------------------------------+-----------------------+
```

## 2. Usage

You can launch the layout simply by running:

```bash
ide
# Or via the full path
bin/ide
```

**Passing a custom agent command:**

You can override the `AI_AGENT_CMD` on the fly by passing an argument:

```bash
# Run a custom command instead of the default 'opencode'
ide "gh copilot"
```

## 3. Configuration

The layout behavior is controlled by enviroment variable. You can these in your shell config (e.g., `~/.zshrc`) or prefix the command when executing.

| Variable        | Default  | Description                                            |
| --------------- | -------- | ------------------------------------------------------ |
| AI_AGENT_CMD    | opencode | The command to run in the Agent Pane.                  |
| IDE_AGENT_WIDTH | 25       | Percentage/Size allocated to the right Agent Pane.     |
| IDE_TERM_HEIGHT | 30       | Percentage/Size allocated to the bottom Terminal Pane. |

### Example:

```bash
# Start with custom layout size
IDE_AGENT_WIDTH=40 IDE_TERM_HEIGHT=20 ide
```

## 4. Important Notes

- **Window Reset:** Running `ide` will execute `tmux kill-pane -a`, which closes all other panes in the currently active window to establish a fresh, 3-pane layout.Ensure you have no unsavad work or vital running processes in other panes within the same window before running this command.
- **Dependencies:** Requires `tmux` to be installed and avaliable in your system `$PATH`
