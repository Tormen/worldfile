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
    worldfile -u     # this machine's set-ups
    worldfile -U     # ...run those whose package is installed
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

There is one world file per distribution: `package-list.<os>`, where `<os>` is
`macos` on macOS, else the `ID` of `/etc/os-release` as it stands (`fedora`,
`arch`, `debian`).

### Which machines: the selector

An entry's comment may end with a *selector*, then a *set-up*:

    brew "syncthing"   # file sync  %DESKTOP+@myhost  setup: syncthing-setup --install

A selector is `%TIER` or `@host`, followed by `+TERM` (add) or `-TERM`
(remove), read left to right: `%DESKTOP-@myhost` is every DESKTOP machine but
myhost. No selector = every machine. `@host` is `hostname -s`; this machine's
tier is what `TIER_COMMAND` prints, asked only when an entry names a `%TIER`.
Every command reads only the lines whose selector names this machine. A `%TIER`
with `TIER_COMMAND` empty, or one that `TIERS` does not list, is an error
naming the line -- never a silent miss.

Where another tool already owns the selector -- a host table that knows every
machine -- `SELECT_COMMAND` hands the reading to it: the command prints the
machines a selector names, one per line, and is asked once per selector in the
world file; `TIER_COMMAND` and `TIERS` are then not used.

### The set-up

`setup: <command>` runs with `worldfile -U`, once the entry's package is
installed (an entry that is not is named and skipped; `# manual` ones count
as installed). It runs on every `-U`, so it must change nothing when it is in
place already -- a second `-U` is then a no-op. A command shared by several
entries runs once; one that fails does not stop the rest, and `-U` exits 1.
`worldfile -u` lists them: `ENTRY<tab>COMMAND`.

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
-- brew on macOS, else the `ID` / `ID_LIKE` of `/etc/os-release`. `WORLDFILE`
empty is `/etc/package-list.<os>`; `TIER_COMMAND` (this machine's tier) and
`TIERS` (the tiers a selector may name) are needed only by `%TIER` selectors.

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
