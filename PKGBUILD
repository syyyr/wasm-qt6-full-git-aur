# Maintainer: Václav Kubernát <sir.venceslas@gmail.com>
# Contributor: João Figueiredo <jf.mundox@gmail.com>
# Contributor: Antonio Rojas <arojas@archlinux.org>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Andrea Scarpino <andrea@archlinux.org>

pkgname=qt6-full-git
pkgver=6.0.0_r5244.g5b22f8ec
pkgrel=1
arch=($CARCH)
url='https://www.qt.io'
license=(GPL3 LGPL3 FDL custom)
pkgdesc='A cross-platform application and UI framework'
depends=(libjpeg-turbo xcb-util-keysyms xcb-util-renderutil libgl fontconfig xdg-utils
         shared-mime-info xcb-util-wm libxrender libxi sqlite xcb-util-image mesa
         tslib libinput libxkbcommon-x11 libproxy libcups double-conversion md4c brotli)
makedepends=(cmake libfbclient mariadb-libs unixodbc postgresql-libs alsa-lib gst-plugins-base-libs
             gtk3 libpulse cups freetds vulkan-headers git)
optdepends=('postgresql-libs: PostgreSQL driver'
            'mariadb-libs: MariaDB driver'
            'unixodbc: ODBC driver'
            'libfbclient: Firebird/iBase driver'
            'freetds: MS SQL driver'
            'gtk3: GTK platform plugin'
            'perl: for fixqt4headers and syncqt')
conflicts=(${pkgname%-git} qt6-multimedia-ffmpeg)
provides=(${pkgname%-git} qt6-multimedia-ffmpeg)
_modules=(
    qtbase
    qtmultimedia
    qtserialport
    qtshadertools
    qtsvg
    qtdeclarative
    qttools
    qtwebsockets
  )

groups=(qt6)
source=(
    qt6::git+https://code.qt.io/qt/qt5.git#branch=dev
    )
  sha256sums=(
      'SKIP'
      )

for module in "${_modules[@]}"; do
  source+=("git+https://code.qt.io/qt/${module}.git#branch=dev")
  conflicts+=(qt6-"${module#qt}")
  provides+=(qt6-"${module#qt}")
  sha256sums+=('SKIP')
done

pkgver() {
  cd qt6
  _ver="$(git describe | sed 's/^v//;s/-.*//')"
  echo "${_ver}_r$(git rev-list --count HEAD).g$(git rev-parse --short HEAD)"
}

build() {
  pushd qt6
  git submodule init "${_modules[@]}"
  for module in "${_modules[@]}"; do
    git config submodule.${module}.url "$srcdir/$module"
  done
  git -c protocol.file.allow=always submodule update "${_modules[@]}"
  popd
  cmake -G Ninja -B build -S qt6 \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DINSTALL_BINDIR=lib/qt6/bin \
    -DINSTALL_DOCDIR=share/doc/qt6 \
    -DINSTALL_ARCHDATADIR=lib/qt6 \
    -DINSTALL_DATADIR=share/qt6 \
    -DINSTALL_INCLUDEDIR=include/qt6 \
    -DINSTALL_MKSPECSDIR=lib/qt6/mkspecs \
    -DINSTALL_EXAMPLESDIR=share/doc/qt6/examples \
    -DQT_FEATURE_journald=ON \
    -DQT_BUILD_EXAMPLES=OFF \
    -DQT_BUILD_TESTS=OFF \
    -DFEATURE_zstd=OFF \
    -DQT_USE_CCACHE=ON \
    -DQT_FEATURE_assistant=OFF \
    -DQT_FEATURE_testlib=OFF
  VERBOSE=1 cmake --build build
}

package() {
  DESTDIR="$pkgdir" cmake --install build

  install -Dm644 qtbase/LICENSE* -t "$pkgdir"/usr/share/licenses/$pkgbase

  # Symlinks for backwards compatibility
  mkdir -p "$pkgdir"/usr/bin
  for _b in uic rcc qmake; do
    ln -rs "$pkgdir"/usr/lib/qt6/bin/$_b "$pkgdir"/usr/bin/$_b-qt6
  done
}
