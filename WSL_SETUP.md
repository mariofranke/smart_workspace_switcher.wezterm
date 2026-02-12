# WSL Support for Smart Workspace Switcher

This plugin now supports using zoxide from within WSL while running WezTerm on Windows!

## Features

- Query zoxide database from WSL
- Automatic path translation from WSL to Windows UNC paths
- Support for both WSL paths (`/home/user/...`) and mounted Windows drives (`/mnt/c/...`)
- Configurable WSL distribution

## Configuration

### Basic WSL Setup

In your WezTerm configuration file (e.g., `.wezterm.lua`):

```lua
local wezterm = require("wezterm")
local workspace_switcher = wezterm.plugin.require("https://github.com/MLFlexer/smart_workspace_switcher.wezterm")

-- Enable WSL mode
workspace_switcher.use_wsl = true

-- (Optional) Specify your WSL distro name (default: "Ubuntu")
workspace_switcher.wsl_distro = "Ubuntu-22.04"  -- or "Debian", "Ubuntu", etc.

local config = wezterm.config_builder()
workspace_switcher.apply_to_config(config)

return config
```

### How It Works

1. **Path Translation**: WSL paths are automatically converted to Windows UNC paths:
   - `/home/user/projects/myapp` → `\\wsl$\Ubuntu\home\user\projects\myapp`
   - `/mnt/c/Users/user/code` → `C:\Users\user\code`

2. **Display Format**: Paths are shown in a user-friendly format:
   - `\\wsl$\Ubuntu\home\user\projects\myapp` → `~/projects/myapp`
   - `C:\Users\user\code` → `~\code` (if under home directory)

3. **Zoxide Integration**: When you select a workspace, zoxide scores are updated in the WSL environment

## Requirements

- Windows 10/11 with WSL2
- WezTerm running on Windows (not inside WSL)
- Zoxide installed **inside WSL** (not on Windows)
- Your WSL distro name must match `workspace_switcher.wsl_distro`

### Installing Zoxide in WSL

```bash
# In your WSL terminal
curl -sS https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | bash

# Add to your ~/.bashrc or ~/.zshrc
eval "$(zoxide init bash)"  # or zsh, fish, etc.
```

## Finding Your WSL Distro Name

Run this in PowerShell or CMD:

```powershell
wsl -l -v
```

Use the exact name shown in the output (e.g., "Ubuntu", "Ubuntu-22.04", "Debian").

## Example Complete Configuration

```lua
local wezterm = require("wezterm")
local workspace_switcher = wezterm.plugin.require("https://github.com/MLFlexer/smart_workspace_switcher.wezterm")

-- WSL Configuration
workspace_switcher.use_wsl = true
workspace_switcher.wsl_distro = "Ubuntu"

local config = wezterm.config_builder()

-- Apply workspace switcher with default keybinds
workspace_switcher.apply_to_config(config)

-- Or use custom keybinds
config.keys = {
  {
    key = "s",
    mods = "ALT",
    action = workspace_switcher.switch_workspace(),
  },
}

return config
```

## Troubleshooting

### "Command not found: zoxide"
- Ensure zoxide is installed in WSL, not Windows
- Verify zoxide is in your PATH in WSL: `wsl -d Ubuntu -e which zoxide`

### Wrong distro name
- Check your distro name: `wsl -l -v`
- Update `workspace_switcher.wsl_distro` to match exactly

### Paths not working
- Ensure WSL2 is being used (not WSL1)
- Verify the UNC path works: Try opening `\\wsl$\Ubuntu` in Windows Explorer

### Performance issues
- Zoxide queries might be slightly slower due to WSL overhead
- Consider using `extra_args` to filter results if you have many paths

## Disabling WSL Mode

To go back to using Windows zoxide:

```lua
workspace_switcher.use_wsl = false
```

Or simply remove/comment out the `use_wsl` line (defaults to `false`).
