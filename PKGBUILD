# Maintainer: lostmyblood <lostmyblood@gmail.com>
pkgname=ytm-dl
pkgver=0.4.0
pkgrel=1
pkgdesc='Minimal TUI for bulk-downloading YouTube Music albums, EPs and singles (yt-dlp wrapper)'
arch=('any')
license=('MIT')
depends=('bash' 'yt-dlp' 'ffmpeg' 'procps-ng')
optdepends=('deno: JavaScript runtime yt-dlp needs for YouTube (recommended)'
            'nodejs: alternative JavaScript runtime if deno is not installed'
            'python-mutagen: cover art embedding for opus files')
source=('ytm-dl' 'ytm-dl.1' 'LICENSE')
sha256sums=('bbd17ff3575faf212025c8d1746433cf7b148cd67e00fbffd0a6ccd437f57c13'
            '05d7b153b3eca7412efa510ef8e6ade2d95d25aad6d3af91b04a06192fa42a8b'
            '9b96899dd1122ec04ec79bffe14e5171c9c1e20d98b6cd1cc84a0b7a5acea01b')

check() {
  bash -n ytm-dl
}

package() {
  install -Dm755 ytm-dl   "$pkgdir/usr/bin/ytm-dl"
  install -Dm644 ytm-dl.1 "$pkgdir/usr/share/man/man1/ytm-dl.1"
  install -Dm644 LICENSE  "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
