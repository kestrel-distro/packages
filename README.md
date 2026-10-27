# Kestrel packages

The signed binary package repository for Kestrel's own packages, served as a
pacman `[kestrel]` overlay from GitHub Pages:

```
[kestrel]
SigLevel = Required
Server = https://kestrel-distro.github.io/packages/
```

Everything here is built by Kestrel from a Kestrel source repository. Arch's
`core`/`extra`/`multilib` remain the base for the rest of the system.

## What is served

**Desktop**

| Package | Built from | Notes |
|---|---|---|
| `hyprland-kestrel` | `kestrel-distro/hyprland` | `Provides`/`conflicts`/`replaces` `hyprland` |
| `quickshell-kestrel` | `kestrel-distro/quickshell` | `Provides`/`conflicts`/`replaces` `quickshell` |
| `kestrel-desktop` | `kestrel-distro/desktop` | Metapackage for the matched desktop set |
| `aquamarine-kestrel` | `desktop` | DRM backend library |
| `hyprutils-kestrel` | `desktop` | |
| `hyprlang-kestrel` | `desktop` | |
| `hyprgraphics-kestrel` | `desktop` | |
| `hyprcursor-kestrel` | `desktop` | |
| `hyprwire-kestrel` | `desktop` | |
| `qt6-base-kestrel` | `desktop` | Qt base with Wayland explicit sync from `kestrel-distro/qtbase` |

The library packages keep upstream versions exactly, use distinct `-kestrel`
names, `conflicts`/`replaces` the Arch package and `provides` its soname, so
Arch cannot overwrite them and consumers such as `hyprlock` stay satisfied.
The desktop is one matched set: Hyprland, Quickshell and this closure are built
with the same `x86-64-v3`/ThinLTO codegen.

**Kernel bridge and scheduler**

| Package | Built from | Notes |
|---|---|---|
| `linux-kestrel` | `kestrel-distro/kernel` | Default kernel, hint ABI, bootable image + mkinitcpio preset |
| `linux-kestrel-headers` | `kernel` | DKMS and module builds |
| `libkestrel-hint` | `kestrel-distro/kestrel` | `sched_setattr` hint library and `kestrel-hint` tool |
| `kestrel-broker` | `kestrel` | Process placement into the Kestrel cgroups |
| `scx-kestrel` | `kestrel-distro/scx` | `scx_kestrel` (scx_layered fork, baked layer set) |
| `scx-tools-kestrel` | `kestrel-distro/scx-loader` | Loader fork with `SupportedSched::Kestrel`; `provides`/`conflicts` `scx-tools` |

The kernel is skipped on desktop-only publishes; the existing kernel packages
are retained so the repository always contains a bootable kernel.

## Publishing

`.github/workflows/publish.yml` builds every package from its repository, signs
the packages and the database, assembles `kestrel.db`, and deploys GitHub Pages.
It runs:

- on a daily schedule,
- on `workflow_dispatch` (with a `build_kernel` toggle),
- on `repository_dispatch` types `upstream-release` and `desktop-release`.

`validate-packages.yml` checks the workflows and packaging before a publish.

### Required secrets

| Secret | Meaning |
|---|---|
| `KESTREL_GPG_PRIVATE_KEY` | Armored export of the passphrase-less Kestrel packaging key |
| `KESTREL_GPG_KEY_ID` | Signing key id or fingerprint |

Generate a **dedicated** packaging key, never a personal one:

```bash
gpg --batch --quick-gen-key "Kestrel Packaging <packaging@kestrel.org>" rsa4096 sign never
gpg --armor --export-secret-keys packaging@kestrel.org |
  gh secret set KESTREL_GPG_PRIVATE_KEY --repo kestrel-distro/packages
gpg --list-secret-keys --keyid-format LONG packaging@kestrel.org   # KESTREL_GPG_KEY_ID
```

The public half ships in the distribution so installed systems trust `[kestrel]`
(`pacman-key --add` + `pacman-key --lsign-key`). The private key lives only in
the repository secret.

## Client configuration

The distribution enables the repository in
`kestrel/default/pacman/pacman-*.conf` with `SigLevel = Required`, and
`kestrel/install/config/kestrel-repo.sh` registers and locally signs the key on
an installed system. To add it manually:

```bash
sudo pacman-key --add /usr/share/pacman/keyrings/kestrel.gpg
sudo pacman-key --lsign-key <key-id>
sudo pacman -Sy
```

## Verifying a publish

- `kestrel.db` and every `.pkg.tar.zst` have matching `.sig` files.
- `pacman -Si <package>` resolves from `[kestrel]` with the Kestrel packager.
- `pacman -Qkk <package>` and `pacman -Qo` confirm ownership after install.
- `deps.txt` on Pages records the Arch versions the build was compiled against.

## Layout

| Path | Contents |
|---|---|
| `.github/workflows/publish.yml` | build, sign, assemble and deploy |
| `.github/workflows/validate-packages.yml` | pre-publish validation |
| `README.md` | this file |

Package sources live in their own repositories (`kestrel`, `kernel`, `hyprland`,
`quickshell`, `desktop`, `scx`, `scx-loader`); this repository only builds,
signs and serves them.

## License

The repository metadata is MIT; each package keeps its upstream license.
