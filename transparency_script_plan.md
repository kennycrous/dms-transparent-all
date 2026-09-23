# Unified Window Background Transparency Script Plan

## Objective
Create a unified CLI script (`set-opacity`) designed for CachyOS running the **Niri** compositor (with XRay blur enabled) and **DMS** (Dank Material Shell). The script will set the background transparency/opacity level consistently across GTK, Qt, terminal emulators, and DMS UI components while supporting live reloading.

---

## 1. Scope & Targeted Applications

The script will automatically detect and manage transparency for:

1. **GTK 3 & GTK 4 Applications**:
   - Targets `~/.config/gtk-3.0/gtk.css` and `~/.config/gtk-4.0/gtk.css`.
   - Modifies background opacity for window elements (`window`, `.background`, `headerbar`, `dialog`) using `rgba(..., opacity)` while preserving text/icon opacity.

2. **Terminal Emulators** (Auto-detected):
   - **Kitty**: Modifies `~/.config/kitty/opacity.conf` (included via `kitty.conf`) and optionally sends `kitty @ set-colors` / SIGHUP for dynamic reload.
   - **Alacritty**: Modifies `~/.config/alacritty/opacity.toml` (included via `alacritty.toml`) setting `window.opacity`.
   - **Foot**: Modifies `~/.config/foot/opacity.ini` setting `[main] alpha=...` and sends `SIGUSR1` to reload.
   - **Ghostty**: Modifies `~/.config/ghostty/config` or included opacity file setting `background-opacity = ...`.
   - **WezTerm**: Modifies `~/.config/wezterm/opacity.lua` setting `window_background_opacity = ...`.

3. **Firefox & Zen Browser**:
   - **Firefox**: Targets `~/.mozilla/firefox/<profile>/chrome/userChrome.css` (as well as Flatpak and Snap profile paths). Enables `toolkit.legacyUserProfileCustomizations.stylesheets` in `user.js` and applies translucent UI background rules (`--lwt-accent-color`, `--lwt-frame`, `#main-window`, `#browser`, `#navigator-toolbox`).
   - **Zen Browser**: Targets `~/.config/zen/<profile>/chrome/userChrome.css` (or `~/.zen/<profile>/chrome/userChrome.css`). Overrides `--zen-main-browser-background` with `rgba(20, 20, 30, opacity) !important;` to prevent the browser from becoming 100% transparent/invisible, maintaining consistent translucent background styling across the UI.

4. **DMS & Desktop UI**:
   - Integrates with DMS theme CSS or configuration files in `~/.config/dms/` to ensure shell panels and popups match window transparency.

5. **Qt Applications**:
   - Ensures Qt5/Qt6 apps respect GTK transparency via `qt6ct`/`qt5ct` or Kvantum theme opacity settings.

---

## 2. Safety & Config Modification Strategy

To avoid corrupting or overwriting user configurations:
- **Config Inclusion**: Where supported (Kitty, Alacritty, Foot), dedicated opacity snippet files are modified (`include opacity.conf`).
- **Comment Markers**: For CSS and KDL files, code is injected strictly between `/* BEGIN DMS TRANSPARENCY */` / `// BEGIN DMS TRANSPARENCY` markers.
- **Niri App Exclusions**: Dynamically detects all applications with custom window rules in Niri configurations and adds `exclude app-id="..."` directives to the global opacity window rule so specific app configurations are never overridden.
- **Automatic Backups**: Creates `.bak` files prior to any first modification.

---

## 3. CLI Features & Interface

- **Command Usage**:
  - `set-opacity 0.85` (sets opacity to 85%)
  - `set-opacity 85%` (percentage support)
  - `set-opacity opaque` (preset: 1.0)
  - `set-opacity medium` (preset: 0.85)
  - `set-opacity translucent` (preset: 0.70)
  - `set-opacity --status` (queries current configured opacity)
  - `set-opacity 0.80 --dry-run` (previews changes without writing)

- **Live Reload Signals**:
  - Automatically sends signals (`SIGHUP`, `SIGUSR1`, IPC commands) to running applications so changes take effect immediately without requiring logouts.

---

## 4. Implementation Details (Python Script)

The script will be implemented as a clean, dependency-free Python 3 executable (`~/.local/bin/set-opacity` or local workspace executable):
- Modular design with individual handlers for GTK, Kitty, Alacritty, Foot, Ghostty, WezTerm, and DMS.
- Error handling to gracefully skip uninstalled tools or missing config directories.

---

## Next Steps
1. Review the plan above.
2. Confirm to proceed with generating the Python script and configuration hooks.
