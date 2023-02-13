# Maintainer: Václav Kubernát <sir.venceslas@gmail.com>
# Contributor: João Figueiredo <jf.mundox@gmail.com>
# Contributor: Antonio Rojas <arojas@archlinux.org>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Andrea Scarpino <andrea@archlinux.org>

pkgname=wasm-qt6-full-git
pkgver=6.0.0_r5250.gb9c25127
pkgrel=1
arch=($CARCH)
url='https://www.qt.io'
license=(GPL3 LGPL3 FDL custom)
options=('!strip' '!makeflags' '!buildflags' 'staticlibs')
pkgdesc='A cross-platform application and UI framework'
depends=(libjpeg-turbo xcb-util-keysyms xcb-util-renderutil libgl fontconfig xdg-utils
         shared-mime-info xcb-util-wm libxrender libxi sqlite xcb-util-image mesa
         tslib libinput libxkbcommon-x11 libproxy libcups double-conversion md4c brotli)
makedepends=(cmake libfbclient mariadb-libs unixodbc postgresql-libs alsa-lib gst-plugins-base-libs
             gtk3 libpulse cups freetds vulkan-headers git)
optdepends=()
conflicts=(${pkgname%-git} wasm-qt6-multimedia-ffmpeg)
provides=(${pkgname%-git} wasm-qt6-multimedia-ffmpeg)
_modules=(
    qtbase
    qtmultimedia
    qtshadertools
    qtsvg
    qtdeclarative
    qttools
    qtwebsockets
  )

source=(
    git+https://github.com/emscripten-core/emsdk.git
    qt6::git+https://code.qt.io/qt/qt5.git#branch=dev
    )
sha256sums=(
    'SKIP'
    'SKIP'
    )

for module in "${_modules[@]}"; do
  source+=("git+https://code.qt.io/qt/${module}.git#branch=dev")
  conflicts+=(wasm-qt6-"${module#qt}")
  provides+=(wasm-qt6-"${module#qt}")
  sha256sums+=('SKIP')
done

prepare() {
  ./emsdk/emsdk install 3.1.25
  ./emsdk/emsdk activate 3.1.25
}

pkgver() {
  cd qt6
  _ver="$(git describe | sed 's/^v//;s/-.*//')"
  local version="${_ver}_r$(git rev-list --count HEAD).g$(git rev-parse --short HEAD)"
  depends+=(qt6-full-git="$version")
  echo "$version"
}

build() {
  source "./emsdk/emsdk_env.sh"
  pushd qt6
  git submodule init "${_modules[@]}"
  for module in "${_modules[@]}"; do
    git config submodule.${module}.url "$srcdir/$module"
  done
  git -c protocol.file.allow=always submodule update "${_modules[@]}"
  popd
  emcmake cmake -G Ninja -S qt6 -B build \
    -DCMAKE_INSTALL_PREFIX=/usr/wasm_32 \
    -DQT_HOST_PATH=/usr \
    -DBUILD_SHARED_LIBS=OFF \
    -DFEATURE_static_runtime=ON \
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

  install -Dm644 qtbase/LICENSES/* -t "$pkgdir"/usr/wasm_32/share/licenses/$pkgbase
}
