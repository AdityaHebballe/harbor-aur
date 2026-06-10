# harbor-stremio-git AUR packaging

Testing AUR package for Harbor from upstream git.

The package uses Tauri's existing `externalBin` setup and creates symlinks during
`prepare()` so the build sees Arch's system binaries as Linux sidecars:

- `/usr/bin/ffmpeg`
- `/usr/bin/ffprobe`
- `/usr/bin/yt-dlp`

This keeps the downstream patch surface small while upstream Linux support is
still in testing.

## Versioning

`pkgver()` reads the app version from upstream `package.json`, then appends the
git revision count and short commit:

```text
0.9.4.r123.gabcdef0
```

For `-git` packages this is computed by `makepkg`; no GitHub workflow is needed
just to update the package version.

## Build

```bash
makepkg -si
```

Regenerate `.SRCINFO` before publishing to AUR:

```bash
makepkg --printsrcinfo > .SRCINFO
```
