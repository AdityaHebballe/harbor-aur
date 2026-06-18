# Maintainer: Aditya Hebballe <adityahebballe@proton.me>

pkgname=harbor-stremio-git
_pkgname=harbor
pkgver=0.9.11.r61.gfb6118e
pkgrel=1
pkgdesc='A Stremio client built for adventure'
arch=('x86_64')
url='https://github.com/harborstremio/harbor'
license=('MIT')
options=('!lto')

depends=(
  'ffmpeg'
  'gst-libav'
  'gst-plugins-bad'
  'gst-plugins-good'
  'gtk3'
  'libayatana-appindicator'
  'mpv'
  'webkit2gtk-4.1'
  'yt-dlp'
)

makedepends=(
  'cargo'
  'git'
  'libarchive'
  'nodejs'
  'pnpm'
  'pkgconf'
  'rust'
)

provides=('harbor-stremio' 'harbor')
conflicts=('harbor-stremio' 'harbor')

source=(
  "$_pkgname::git+https://github.com/harborstremio/harbor.git"
  'fix-gtk-raw-pointer-inference.patch'
  'fix-linux-mpv-render-runtime.patch'
)
sha256sums=(
  'SKIP'
  'e3436c7bab81dbe80d7e747b4034ac8ddf3f617300d04684b41ce132f16c9092'
  'dece91982901fa05138111e79d215ac44ba9d49096dbdd3b3e89079c065405b8'
)

pkgver() {
  cd "$srcdir/$_pkgname"

  local version revision commit
  version="$(node -p "require('./package.json').version")"
  revision="$(git rev-list --count HEAD)"
  commit="$(git rev-parse --short HEAD)"

  printf '%s.r%s.g%s' "$version" "$revision" "$commit"
}

prepare() {
  cd "$srcdir/$_pkgname"

  patch -Np1 -i "$srcdir/fix-gtk-raw-pointer-inference.patch"
  patch -Np1 -i "$srcdir/fix-linux-mpv-render-runtime.patch"

  node <<'EOF'
const fs = require('fs');
const path = 'src-tauri/tauri.conf.json';
const config = JSON.parse(fs.readFileSync(path, 'utf8'));
config.bundle.createUpdaterArtifacts = false;
fs.writeFileSync(path, `${JSON.stringify(config, null, 2)}\n`);
EOF

  mkdir -p src-tauri/binaries
  ln -sf /usr/bin/ffmpeg src-tauri/binaries/ffmpeg-x86_64-unknown-linux-gnu
  ln -sf /usr/bin/ffprobe src-tauri/binaries/ffprobe-x86_64-unknown-linux-gnu
  ln -sf /usr/bin/yt-dlp src-tauri/binaries/yt-dlp-x86_64-unknown-linux-gnu

  cat > pnpm-workspace.yaml <<'EOF'
allowBuilds:
  esbuild: true
EOF

  pnpm install --frozen-lockfile
}

build() {
  cd "$srcdir/$_pkgname"

  pnpm tauri build --bundles deb
}

package() {
  cd "$srcdir/$_pkgname"

  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  local deb data_archive
  deb="$(find src-tauri/target/release/bundle/deb -type f -name '*.deb' | head -n 1)"

  if [[ -z "$deb" ]]; then
    echo "Debian bundle not found" >&2
    return 1
  fi

  rm -rf "$srcdir/deb-extract"
  mkdir -p "$srcdir/deb-extract"
  bsdtar -xf "$deb" -C "$srcdir/deb-extract"

  data_archive="$(find "$srcdir/deb-extract" -maxdepth 1 -type f -name 'data.tar.*' | head -n 1)"
  if [[ -z "$data_archive" ]]; then
    echo "Debian data archive not found" >&2
    return 1
  fi

  bsdtar -xf "$data_archive" -C "$pkgdir"

  rm -f "$pkgdir/usr/bin/ffmpeg" \
        "$pkgdir/usr/bin/ffprobe" \
        "$pkgdir/usr/bin/yt-dlp"

  if [[ -x "$pkgdir/usr/bin/harbor" && ! -e "$pkgdir/usr/bin/harbor-stremio" ]]; then
    ln -s harbor "$pkgdir/usr/bin/harbor-stremio"
  fi
}
