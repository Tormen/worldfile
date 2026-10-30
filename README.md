# worldfile

One list of the packages a machine should have -- the *world file* -- and the
same commands to compare it with what is installed, whatever the package
manager: Homebrew (macOS), dnf (Fedora), pacman/yay (Arch), apt (Debian),
zypper (openSUSE).

    worldfile -m     # in the world file, not installed
    worldfile -M     # ...install them (asks first)
    worldfile -o     # installed on top of the world file
    worldfile -O     # ...remove them (asks first)
    worldfile -l     # the world file's packages
    worldfile -h     # every command

Not every command exists for every package manager yet; one that does not
says so (`NOT YET IMPLEMENTED FOR DISTRO ...`).

## The world file

One package per line (or several, separated by blanks); everything after
`#` is a comment, and a line starting with `###` is a section heading
(`worldfile -s` lists them). On macOS it is a Brewfile (`tap "..."`,
`brew "..."`, `cask "..."`). A commented-out entry is a package that should
NOT be there (`worldfile -c` lists those still installed). `# manual <name>`
records something installed by hand, outside the package manager: it counts
as listed.

## What `-o` compares

- **brew** -- `brew bundle cleanup` against the world file.
- **fedora** -- what dnf records as installed on purpose (`dnf5 repoquery
  --userinstalled`), minus what the image installed (the transactions in
  `FEDORA_IMAGE_TRANSACTIONS`, default `1 2`: a cloud image's own builds --
  `dnf5 history list` shows them), minus the world file. Dependencies do not
  appear: dnf records them as such. Needs no root.
- **arch** -- `pacman -Qtdq`; **debian** -- `deborphan` with the world file
  as its keep list.

## Configuration

Optional: without a config file the defaults apply, and `DISTRO` is detected
-- brew on macOS, else the `ID` / `ID_LIKE` of `/etc/os-release`.

    worldfile --create-config                      # every setting, with its default
    worldfile --create-config /etc/worldfile.conf  # ...written to a file (never over one)

The config is plain shell. It is `$WORLDFILE_CONFIG`, else `--config <FILE>`,
else the first that exists of `/LINKS/default/worldfile.conf` (or
`/LINKS/default/worldfile`), `~/.worldfile.conf`, `/etc/worldfile.conf`,
`/usr/local/etc/worldfile.conf`.

## Tests

    worldfile --run-tests [FILTER]

TAP output (`1..N`, `ok K - name`); the package managers are stubs, so
nothing is installed or removed. Needs `/bin/dash`; runs under any POSIX
`sh` too.
