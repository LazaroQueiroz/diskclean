# diskclean

A tiered disk cleanup script for Arch Linux. It is **dry-run by default**: every command shows what it would do, and only changes anything when you pass `--run`, or when you approve an action in interactive mode (`-i`).

## Features
- **Report**: root filesystem usage, the biggest folders in `$HOME`, and the size of the pacman cache, `~/.cache` and the journal.
- **safe**: clears caches that rebuild themselves: pacman/yay, pip, npm, Go, Gradle build cache, app caches (Spotify, Chromium, Firefox, Discord, Slack, Playwright), unused Docker images and stopped containers (volumes are left alone), and the systemd journal (trimmed to 200M). It also lists orphan packages.
- **compress**: losslessly zstd-compresses `.csv`/`.log`/`.json`/`.txt` files over 50M. The original is deleted only after the archive passes `zstd -t`.
- **artifacts**: removes `node_modules`, `venv`, `.venv` and `build` directories from projects with no other file modified in the last 60 days.
- **Interactive mode** (`-i` / `--interactive`): for `safe`, `compress` and `artifacts`, each action is shown with what it modifies (path and size), its impact, and the exact command, then you choose: `y` apply, `n` skip (default), `a` apply this and all remaining, `q` quit. Ends with a summary and the new free space.
- **downloads**: lists items in `~/Downloads` older than 90 days so you can review them by hand. It never deletes anything.

## Requirements
- Bash 4.4+, GNU coreutils/findutils
- Arch Linux (`pacman`, and `sudo` for the pacman and journal steps)
- Optional, each used only if installed: `yay`, `pip`, `npm`, `go`, `docker`, `zstd` (needed for `compress`)

## Installation
```sh
git clone git@github.com:LazaroQueiroz/diskclean.git ~/diskclean
ln -sf ~/diskclean/bin/diskclean ~/.local/bin/diskclean
```
`~/.local/bin` needs to be on your `PATH`.

## Usage
```sh
diskclean                        # usage report, changes nothing
diskclean safe                   # show what would be cleared
diskclean safe --run             # actually clear it
diskclean safe -i                # ask before each action, explaining what it changes
diskclean compress ~/data        # list large text files that would be compressed
diskclean compress ~/data --run
diskclean artifacts ~/projects --run
diskclean artifacts ~/projects -i
diskclean downloads
diskclean --help
```

Example report:
```
Root: 79G used / 112G (74%), 28G free

52G	/home/lazaro
21G	/home/lazaro/Documents
7.3G	/home/lazaro/.cache
...
```

Interactive prompt:
```
▸ Vacuum systemd journal to 200M
  Modifies: /var/log/journal (1.1G)
  Impact:   Oldest logs are deleted; journalctl history becomes shorter. Needs sudo.
  Command:  sudo journalctl --vacuum-size=200M
  Apply? [y/N/a/q]
```

Compressed files can be read with `zstdcat FILE.zst` or restored with `unzstd FILE.zst`.

### Notes
- `safe --run` removes orphan packages with `pacman -Rns` and still asks for confirmation. Check the orphan list from the dry run first.
- `artifacts` skips hidden paths, `~/Downloads` and any `resources/` directory, and does not cross filesystem boundaries (`-xdev`).

## Project Structure
```
bin/diskclean   the script (single file, no dependencies beyond the tools above)
```

## Contributing
Issues and pull requests are welcome. Commits follow [Conventional Commits](https://www.conventionalcommits.org/). Run `bash -n bin/diskclean` (and `shellcheck` if you have it) before submitting.

## License
[MIT](LICENSE)
