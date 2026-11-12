# onboard-conf

Custom [Onboard](https://launchpad.net/onboard) on-screen keyboard setup:

> Created for use with a Lenovo ThinkPad Yoga 11e with the XFCE desktop environment

- **`Small (Custom)` layout**: a US-style layout with `` ` ``, `[ ]`, `; '` and `, . /`, a `\` key under Backspace, a 12?/Ctrl/Win/Alt bottom row, about 20px of side padding, and a half-height top row.
- **Theme `NewGruvNew-Dark`**: a Gruvbox-style colour scheme. The key labels use Maple Mono Bold.
- **Dynamic top row** (`okb.py`): a watcher that changes the half-height row's keys to match the focused app. In terminals it also follows what's running inside (shell, tmux or Neovim). Keys are one-tap shortcuts sent with `xdotool`.

## Layout

```
layouts/     Onboard layout + key geometry SVGs  -> ~/.local/share/onboard/layouts/
themes/      theme + colour scheme               -> ~/.local/share/onboard/themes/
config/      rows.toml (top-row keys per app)    -> ~/.config/onboard-dynamic/
dynamic/     okb.py (generator + watcher)        -> ~/.local/share/onboard/dynamic/
autostart/   starts the watcher at login         -> ~/.config/autostart/
```

## Requirements

- An X11 session. Wayland isn't supported because the watcher uses `xprop` and `xdotool`.
- `onboard`, `xprop`, `xdotool`
- Python 3.11 or later for `tomllib` with PyGObject, Onboard already depends on
- The Maple Mono font, for the key labels

## Install

Run from repo root:

```sh
# 1. Layout, theme
mkdir -p ~/.local/share/onboard/layouts ~/.local/share/onboard/themes
cp layouts/* ~/.local/share/onboard/layouts/
cp themes/*  ~/.local/share/onboard/themes/

gsettings set org.onboard theme "$HOME/.local/share/onboard/themes/NewGruvNew-Dark.theme"
gsettings set org.onboard.theme-settings key-label-font 'Maple Mono Bold'
gsettings set org.onboard layout "$HOME/.local/share/onboard/layouts/Small (Custom).onboard"

# 2. config, script and actions directory for dynamic top row
mkdir -p ~/.config/onboard-dynamic ~/.local/share/onboard/dynamic/actions
cp config/rows.toml ~/.config/onboard-dynamic/
cp dynamic/okb.py   ~/.local/share/onboard/dynamic/
chmod +x ~/.local/share/onboard/dynamic/okb.py

# 3.  Onboard script loading:
#    Onboard loads "script" keys with Python's import, so the actions
#    directory has to be on sys.path. A .pth file in the user site-packages does that.
SITE=$(python3 -m site --user-site)
mkdir -p "$SITE"
echo "$HOME/.local/share/onboard/dynamic/actions" > "$SITE/onboard-dynamic.pth"

# 4. Start watcher and set it to start at login
mkdir -p ~/.config/autostart
cp autostart/onboard-dynamic-row.desktop ~/.config/autostart/
python3 ~/.local/share/onboard/dynamic/okb.py generate
setsid -f python3 ~/.local/share/onboard/dynamic/okb.py watch >> ~/.cache/onboard-dynamic.log 2>&1

# 5. Restart Onboard to detect the .pth file
pkill -x onboard; setsid -f onboard
```


## Editing the top row

In `~/.config/onboard-dynamic/rows.toml`, each profile is a list of `{ label, keys }` entries:

```
```
```toml
```toml
```
[profiles.example]
keys = [
  { label = "+Tab", keys = "ctrl+t" },                 # one chord
  { label = ":w",   keys = "Escape colon w Return" },  # key sequence
  { label = "Sig",  type = "Kind regards," },          # literal text
]
```
```
```

> `keys` uses `xdotool` key names (X11 keysyms). Run `xev` to determine keys.

## Adding an application
To add an app, run `okb.py status` with that app focused to see its WM_CLASS. Then add a `[[rules]]` entry with a `class` regex and a `profile`.


