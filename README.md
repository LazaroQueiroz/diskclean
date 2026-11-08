# diskclean

A tiered disk cleanup script for Arch Linux. It is **dry-run by default**: every command shows what it would do, and only changes anything when you pass `--run`.

## Features
- **Report**: root filesystem usage, the biggest folders in `$HOME`, and the size of the pacman cache, `~/.cache` and the journal.
- **safe**: clears caches that rebuild themselves: pacman/yay, pip, npm, Go, Gradle build cache, app caches (Spotify, Chromium, Firefox, Discord, Slack, Playwright), unused Docker images and stopped containers (volumes are left alone), and the systemd journal (trimmed to 200M). It also lists orphan packages.
- **compress**: losslessly zstd-compresses `.csv`/`.log`/`.json`/`.txt` files over 50M. The original is deleted only after the archive passes `zstd -t`.
- **artifacts**: removes `node_modules`, `venv`, `.venv` and `build` directories from projects with no other file modified in the last 60 days.
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
diskclean compress ~/data        # list large text files that would be compressed
diskclean compress ~/data --run
diskclean artifacts ~/projects --run
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
