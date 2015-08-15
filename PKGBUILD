# Maintainer: Daniel Micay <danielmicay@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox-oss
pkgver=12.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
depends=('gtk2' 'mozilla-common' 'nss>=3.13.3' 'libxt' 'hunspell' 'startup-notification' 'mime-types' 'dbus-glib' 'alsa-lib' 'sqlite' 'libnotify' 'desktop-file-utils' 'libvpx' 'libevent' 'hicolor-icon-theme')
makedepends=('unzip' 'zip' 'diffutils' 'python2' 'wireless_tools' 'yasm' 'mesa' 'autoconf2.13' 'libidl2' 'xorg-server-xvfb' 'imake')
url="http://www.mozilla.org/projects/firefox"
install=firefox.install
source=(ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.bz2
        mozconfig firefox.desktop firefox-install-dir.patch vendor.js
        gcc47.patch)
md5sums=('80c3e5927274de7f181fb5f931ac5fd4'
         'db6d2bd4671831859566f403a086fdf2'
         'bdeb0380c7fae30dd0ead6d2d3bc5873'
         '594523c536a9be951b52c90dd85aff18'
         '0d053487907de4376d67d8f499c5502b'
         '79e471e0bd03b1d4e84fd3a6776bd619')

build() {
  cd "mozilla-release"

  # switch to the OSS audio backend
  sed -i 's/sydney_audio_alsa/sydney_audio_oss/' media/libsydneyaudio/src/Makefile.in

  cp "$srcdir/mozconfig" .mozconfig
  patch -Np1 -i "$srcdir/firefox-install-dir.patch"
  patch -Np1 -i "$srcdir/gcc47.patch"

  # Fix PRE_RELEASE_SUFFIX
  sed -i '/^PRE_RELEASE_SUFFIX := ""/s/ ""//' \
    browser/base/Makefile.in

  export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/firefox"
  export PYTHON="/usr/bin/python2"

  LD_PRELOAD="" /usr/bin/Xvfb -nolisten tcp -extension GLX -screen 0 1280x1024x24 :99 &
  LD_PRELOAD="" DISPLAY=:99 make -j1 -f client.mk profiledbuild MOZ_MAKE_FLAGS="$MAKEFLAGS"
  kill $! || true
}

package() {
  cd "mozilla-release"
  make -j1 -f client.mk DESTDIR="$pkgdir" install

  install -m644 "$srcdir"/vendor.js "$pkgdir/usr/lib/firefox/defaults/pref"

  for i in 16x16 22x22 24x24 32x32 48x48 256x256; do
      install -Dm644 browser/branding/official/default${i/x*/}.png \
        "$pkgdir/usr/share/icons/hicolor/$i/apps/firefox.png"
  done

  install -Dm644 "$srcdir/firefox.desktop" \
    "$pkgdir/usr/share/applications/firefox.desktop"

  rm -rf "$pkgdir"/usr/lib/firefox/{dictionaries,hyphenation}
  ln -sf /usr/share/hunspell "$pkgdir/usr/lib/firefox/dictionaries"
  ln -sf /usr/share/hyphen "$pkgdir/usr/lib/firefox/hyphenation"


  # We don't want the development stuff
  rm -r "$pkgdir"/usr/{include,lib/firefox-devel,share/idl}

  #workaround for now
  #https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -sf firefox "$pkgdir/usr/lib/firefox/firefox-bin"
}
