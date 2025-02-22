# Maintainer: Salamandar <felix@piedallu.me>

pkgname=prusa-slicer-git
pkgver=2.5.0.alpha0.r9.g81e958276
pkgrel=1
pkgdesc='G-code generator for 3D printers (RepRap, Makerbot, Ultimaker etc.)'
arch=('i686' 'x86_64' 'armv6' 'armv6h' 'armv7h')
url='https://github.com/prusa3d/PrusaSlicer'
license=('AGPL3')
makedepends=(
    'cmake'
    'ninja'
    'boost'
    'cereal'
    'eigen'
    'gtest'
)
depends=(
    'boost-libs'
    'cgal'
    'curl'
    'dbus'
    'glew'
    'intel-tbb'
    'libpng'
    'nlopt'
    'openvdb'
    'qhull'
    'wxgtk3'
)

source=(
    "git+${url}"
    'prusa-slicer-boost-placeholders.patch'
)
sha256sums=(
    'SKIP'
    '58cae07a418a797222f4cb10950fa2fd7afb7570519785b082cc7d7e7f407c02'
)
conflicts=('prusa-slicer')

pkgver() {
    git -C "${srcdir}/PrusaSlicer" describe --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/^version_//'
}

prepare() {
    cd "PrusaSlicer"
    # Fix build with Boost 1.76.0
    patch -p1 < "$srcdir/prusa-slicer-boost-placeholders.patch"
    patch CMakeLists.txt ../../prusaslicer_cmakelists.patch
    patch src/CMakeLists.txt ../../prusaslicer_src_cmakelists.patch
    patch src/libslic3r/CMakeLists.txt ../../prusaslicer_src_libslic3r_cmakelists.patch
    patch src/slic3r/CMakeLists.txt ../../prusaslicer_src_slic3r_cmakelists.patch
}

build() {
    cmake -B build -S PrusaSlicer -G Ninja \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_INSTALL_LIBDIR=lib \
        -DSLIC3R_FHS=ON \
        -DSLIC3R_PCH=OFF \
        -DSLIC3R_WX_STABLE=ON \
        -DSLIC3R_GTK=3 \
        -DSLIC3R_STATIC=OFF \
        -DwxWidgets_CONFIG_EXECUTABLE=$(which wx-config-gtk3) \
        -DOPENVDB_FIND_MODULE_PATH=/usr/lib/cmake/OpenVDB

    # This is a trick to workaround RAM issues that kill GCC
    ninja -C build -k0
    ninja -C build -j2
}

check() {
    cd "build"
    ctest -V
}

package () {
    DESTDIR="${pkgdir}" ninja -C "build" install

    # Patch desktop files
    for i in PrusaGcodeviewer PrusaSlicer; do
        sed -i '/^Name=/ s/$/ (Git version)/' "${pkgdir}/usr/share/applications/$i.desktop"
    done
}
