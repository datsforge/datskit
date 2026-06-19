# notify

A small bash script that plays a notification sound on demand. It does
**not** detect when a task finishes on its own — you chain it onto a
command with `&&` so it plays once that command completes:

```bash
./build.sh && notify
rsync -av big_folder/ remote: && notify 5
```

Ten sound "slots" (0-9) are available. Ships with 10 default sounds, but
every slot can be remapped to your own audio file. The "done" and "bye"
messages printed alongside the sound can also be customized.

## Requirements

- `bash`
- `aplay` (part of `alsa-utils` on Debian/Ubuntu)
- `file` (for validating sound files you add — usually preinstalled)

## Setup

No installation required. Just keep the `notify` script next to its
`assets/` folder (the bundled default sounds), and run it directly:

```bash
./notify
```

### Running `notify` from anywhere

To type `notify` from any directory instead of `./notify`, put it
somewhere on your `PATH`. Two ways to do this:

**Option 1 — symlink into `~/.local/bin`** (recommended, keeps the repo
in one place and the script stays editable there):

```bash
chmod +x notify
mkdir -p ~/.local/bin
ln -s "$(pwd)/notify" ~/.local/bin/notify
```

Make sure `~/.local/bin` is on your `PATH`. Check with:

```bash
echo "$PATH" | tr ':' '\n' | grep "\.local/bin"
```

If nothing prints, add this line to `~/.bashrc` (or `~/.zshrc` if you use
zsh), then restart your terminal or run `source ~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**Option 2 — add the project folder itself to `PATH`:**

```bash
export PATH="/path/to/this/project:$PATH"
```

Add that line to `~/.bashrc` to make it permanent.

Either way works because the script locates its own real path at runtime
(`readlink -f "$0"`) and finds `assets/` relative to that — so a symlink
in `~/.local/bin` still correctly finds the bundled sounds back in the
project folder. Just don't move the `notify` script away from its
`assets/` folder, or move them together if you do.

Once set up:

```bash
notify        # works from any directory
```

## Usage

```
notify [NUMBER]
notify -a, --add NUMBER FILE
notify -d, --remove NUMBER
notify -l, --list
notify -R, --reset-sounds
notify -m, --message done|bye [TEXT]
notify -m, --message reset done|bye
notify -h, --help
```

### Playing a sound

```bash
notify        # play the default sound (slot 1)
notify 5      # play slot 5
```

While a sound is playing, press any key to stop it early.

If you give an invalid slot (anything other than a single digit 0-9),
`notify` prints an error and exits without playing anything — it never
silently falls back to a different sound.

### Mapping your own sounds

```bash
notify -a 5 ~/Music/mysound.wav   # slot 5 now plays your file
notify -d 5                       # remove the override, back to default
notify -l                         # see what every slot currently plays
notify -R                         # reset ALL slots back to defaults
```

Your own sound files are referenced **in place** — they aren't copied
anywhere. If a mapped file is later moved or deleted, `notify -l` will
flag it as `[MISSING]`, and playing that slot will print a warning and
fall back to the bundled default sound for that slot instead of failing.

### Customizing messages

Two messages are shown: one when a sound finishes playing normally
("done"), and one when you stop a sound early or it finishes ("bye").

```bash
notify -m done "Build complete!"   # set the "done" message
notify -m bye "Catch you later"    # set the "bye" message
notify -m done                     # show the current "done" message
notify -m reset done                # revert "done" message to default
```

> **Note on `!` in messages:** if your message contains an exclamation
> mark and you use double quotes, interactive bash may try to expand it
> as a history reference and error with `event not found`:
> ```bash
> notify -m done "Task done!"
> bash: !: event not found
> ```
> Use single quotes instead, which bash does not expand:
> ```bash
> notify -m done 'Task done!'
> ```

## Configuration

All overrides (sound mappings and custom messages) are stored in:

```
~/.config/datsforge/notify/config
```

This is a plain text file, one `key=value` setting per line. You can edit
it by hand if you prefer, but the `-a`, `-d`, `-R`, and `-m` flags are the
safer way to manage it since they validate input before writing.

## Default sounds

| Slot | File |
|------|------|
| 0 | 634441__sophieciruela__notification.wav |
| 1 | 634450__sophieciruela__success.wav (default slot) |
| 2 | 751862__def__success-sound.wav |
| 3 | 751863__def__failure-notification-sound.wav |
| 4 | 751890__def__monophonic-synth-fanfare-sound.wav |
| 5 | 276607__mickleness__ringtone-foofaraw.wav |
| 6 | 276608__mickleness__ringtone-fanta.wav |
| 7 | 276614__mickleness__ludic.wav |
| 8 | 482653__joao_janz__bonus-points-1_1.wav |
| 9 | 504784__joao_janz__reward-notification-3_1.wav |

See `CREDITS.md` for sound attribution.

