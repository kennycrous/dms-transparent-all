# set-opacity

One command to set background transparency across a DankMaterialShell (DMS) desktop on **Niri** or **Hyprland**: compositor window rules, GTK, terminals, Firefox and Zen Browser.

```sh
set-opacity 0.85
```

It's a single dependency-free Python 3 script. Anything that isn't installed (no config directory) is skipped.

## Usage

```sh
set-opacity 0.85            # decimal, 0.0 – 1.0
set-opacity 85%             # percentage
set-opacity translucent     # preset
set-opacity 0.80 --dry-run  # print what would be written, change nothing
set-opacity --status        # show the current values
set-opacity --install       # copy to ~/.local/bin/set-opacity
```

| Preset | Opacity |
|---|---|
| `opaque`, `solid` | 1.00 |
| `high` | 0.90 |
| `medium` | 0.85 |
| `translucent` | 0.75 |
| `low` | 0.65 |
| `transparent` | 0.50 |

The script file isn't executable in the repo. Run it with `python3 set-opacity …`, or use `--install`, which installs an executable copy.

## What it changes

| Target | File(s) | Reload |
|---|---|---|
| Niri | `~/.config/niri/dms/windowrules.kdl`, included from `config.kdl` | `niri msg action load-config-file` |
| Hyprland | `~/.config/hypr/dms/opacity.lua`, required from `hyprland.lua` | Automatic on config change |
| GTK 3 / 4 | `~/.config/gtk-3.0/gtk.css`, `~/.config/gtk-4.0/gtk.css` | Restart the app |
| VS Code / VSCodium / Cursor | `settings.json`: removes any `workbench.colorCustomizations` override so the colour theme is kept; transparency comes from the compositor rule | Restart the app |
| Firefox (incl. LibreWolf, Floorp, Flatpak, Snap) | Each profile's `chrome/userChrome.css` and `user.js` | Restart the browser |
| Zen Browser | Each profile's `chrome/userChrome.css` and `user.js` | Restart the browser |
| Kitty | `~/.config/kitty/opacity.conf`, included from `kitty.conf` | `SIGUSR1` sent automatically |
| Foot | `~/.config/foot/foot.ini` | `SIGUSR1` sent automatically |
| Alacritty | `~/.config/alacritty/alacritty.toml` (`[window] opacity`) | Live reload by Alacritty |
| Ghostty | `~/.config/ghostty/config` (`background-opacity`) | Restart Ghostty |
| WezTerm | `~/.config/wezterm/wezterm.lua` (`window_background_opacity`) | Live reload by WezTerm |

Browsers get a dark translucent background, `rgba(20, 20, 30, <opacity>)`. The script also enables `toolkit.legacyUserProfileCustomizations.stylesheets` so Firefox and Zen load `userChrome.css`.

### Compositor rules

**Niri:** a global `window-rule` with `opacity <value>` and one `exclude` line per excluded app. VS Code gets its own rule with the same opacity and `draw-border-with-background false`.

**Hyprland** (Lua config):

```lua
-- BEGIN DMS TRANSPARENCY
hl.window_rule({ match = { class = ".*" }, opacity = "0.85" })
-- Excluded apps stay opaque (last matching rule wins)
hl.window_rule({ match = { class = "^firefox$|^zen$|…" }, opacity = "1.0" })
-- END DMS TRANSPARENCY
```

To blur the DMS bar, popouts and notifications, turn on DMS Settings → Theme & Colors → **Background Blur**. DMS requests the blur itself, so the script doesn't add a rule for it.

`require("dms.opacity")` is inserted *above* the DMS window rules in `hyprland.lua`. Hyprland applies the last matching rule, so an opacity rule of your own for a specific app still takes priority.

The compositor opacity *multiplies* with an app's own transparency. A terminal set to 0.85 that also gets the 0.85 window rule ends up at about 0.72.

### Excluded apps

These are never made transparent by the compositor rule, on either compositor. The list is `EXCLUDED_APP_PATTERNS` at the top of the script; add a regex there to exclude more apps.

- **Browsers:** Firefox, Zen, LibreWolf, Floorp, Chrome (including Chrome web apps), Chromium, Brave, Vivaldi
- **Game launchers and games:** Steam, Steam games (`steam_app_\d+`), Lutris, Heroic, Bottles, Faugus, Prism Launcher
- **DMS:** the DMS settings window (`com.danklinux.dms`)

Patterns are matched against the Niri `app-id` or Hyprland `class`. To find a window's ID, use `niri msg windows` or `hyprctl clients`.

## Safety

- **Marker blocks.** Generated content goes between `BEGIN DMS TRANSPARENCY` / `END DMS TRANSPARENCY` markers, and later runs replace only that block. Everything else in the file is left alone.
- **Backups.** The first time a file is modified, a `.bak` copy is saved next to it. Later runs never overwrite that backup.
- **DMS-managed files.** DMS overwrites files like `hypr/dms/windowrules.lua`, so the Hyprland rules go in a separate `dms/opacity.lua` that DMS doesn't manage.
- **Previews.** `--dry-run` prints every file the script would write.

## Requirements

- Python 3.8+
- Niri or Hyprland 0.55+ (Lua config) with DMS. Either is optional; the GTK, terminal and browser handlers work on their own.
