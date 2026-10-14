# Kestrel packages

The binary package repository for Kestrel's own packages, served as a pacman
overlay from GitHub Pages:

```
Server = https://kestrel-distro.github.io/packages/
```

It serves only packages Kestrel builds itself:

- `hyprland-kestrel` (conflicts/replaces/provides `hyprland`)
- `quickshell-kestrel` (conflicts/replaces/provides `quickshell`)
- `linux-kestrel`, `linux-kestrel-headers`

Arch's `core`/`extra`/`multilib` remain the base for everything else: because
the Kestrel packages use distinct names and conflict with the Arch ones, Arch's
stock `hyprland`/`quickshell` can never install or overwrite them. Their updates
come only from this repository.

## Publishing

`.github/workflows/publish.yml` builds each package from its fork, signs them
and the database, and deploys the result to GitHub Pages. It runs:

- on a daily schedule,
- on `workflow_dispatch`,
- on `repository_dispatch` with type `upstream-release` (the forks can fire
  this when they sync an upstream release).

### Required secrets

| Secret | Meaning |
|---|---|
| `KESTREL_GPG_PRIVATE_KEY` | Armored export of the passphrase-less Kestrel packaging key |
| `KESTREL_GPG_KEY_ID` | Signing key id or fingerprint |

Generate a **dedicated** packaging key (not a personal one):

```
gpg --batch --quick-gen-key "Kestrel Packaging <packaging@kestrel.org>" rsa4096 sign never
gpg --armor --export-secret-keys packaging@kestrel.org | gh secret set KESTREL_GPG_PRIVATE_KEY --repo kestrel-distro/packages
gpg --list-secret-keys --keyid-format LONG packaging@kestrel.org   # -> KESTREL_GPG_KEY_ID
```

The matching **public** key is shipped in the distribution so installed systems
trust `[kestrel]` (`pacman-key --add` + `pacman-key --lsign-key`).

## Client configuration

The distribution enables this repository in
`default/pacman/pacman-*.conf`:

```
[kestrel]
SigLevel = Required
Server = https://kestrel-distro.github.io/packages/
```
