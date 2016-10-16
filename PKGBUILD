# Maintainer: yhfudev <yhfudev gmail>
# Contributor: Valentin Churavy <v.churavy@gmail.com>
# Contributor: Romain Reignier <rom.reignier@gmail.com> (ARM support)
# Contributor: Fabien Dubosson <fabien.dubosson@gmail.com>
# Contributor: David Manouchehri <david@davidmanouchehri.com>
# Contributor: CHEN Xing <cxcxcxcx@gmail.com>
# Contributor: Martin Imobersteg <martin.imobersteg@gmail.com>
# Contributor: Artyom Smirnov <smirnoffjr@gmail.com>
# Also largely inspired by `opencv` in extra, so including contributors too:
# Contributor: Ray Rashif <schiv@archlinux.org>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>

pkgname="opencv-git"
pkgver=3.1.0.r1522.g1ae27eb
pkgrel=1
pkgdesc="Open Source Computer Vision Library compiled with extra modules(opencv_contrib) and CUDA"
url="http://opencv.org/"
license=('BSD')
arch=('i686' 'x86_64' 'armv7h' 'armv6h')
depends=('gstreamer0.10-base' 'openexr'
         'xine-lib' 'libdc1394' 'gtkglext'
         'nvidia-utils' 'hdf5-cpp-fortran' # The following variables are used in this project, but they are set to NOTFOUND : CUDA_CUDA_LIBRARY, HDF5_hdf5_cpp_LIBRARY
         )
depends_x86_64=('intel-tbb')
depends_i686=('intel-tbb')
depends_armv7h=('intel-tbb')
makedepends=('git' 'cmake' 'python2-numpy' 'python-numpy' 'mesa' 'eigen'
             'ceres-solver' # for opencv_contrib's sfm
             'cuda' 'libcl'
    )
optdepends=('eigen'
            #'libcl: For coding with OpenCL'
            'python-numpy: Python 3 interface'
            'python2-numpy: Python 2 interface')
options=('staticlibs')
provides=("${pkgname%-git}")
conflicts=("${pkgname%-git}")
changelog="ChangeLog"
source=("${pkgname%-git}::git+http://github.com/Itseez/opencv.git"
        "${pkgname%-git}_contrib::git+https://github.com/opencv/opencv_contrib.git"
        'opencv_contrib_sfm_cmake.patch'
        'opencv_gcc6_pch.patch'
        'fsh.patch'
    )
md5sums=('SKIP'
         'SKIP'
         '6497098ac0f9d52e3c988f5413af22c8'
         '9c69b76b8f1b781e201c70d21e3d175e'
         '752652494b42fd1532f52b789a85947f'
    )

_cmakeopts=('-D WITH_OPENCL=ON'
            '-D WITH_OPENGL=ON'
            '-D WITH_TBB=ON'
            '-D WITH_XINE=ON'
            '-D BUILD_WITH_DEBUG_INFO=OFF'
            '-D BUILD_TESTS=OFF'
            '-D BUILD_PERF_TESTS=OFF'
            '-D BUILD_EXAMPLES=OFF'
            '-D INSTALL_C_EXAMPLES=OFF'
            '-D INSTALL_PYTHON_EXAMPLES=OFF'
            '-DBUILD_opencv_python2=ON',
            '-DBUILD_opencv_python3=ON',
            '-D CMAKE_BUILD_TYPE=Release'
            '-D CMAKE_INSTALL_PREFIX=/usr'
            '-D CMAKE_SKIP_RPATH=ON'
            '-D BUILD_NEW_PYTHON_SUPPORT=ON'
            '-D WITH_NVCUVID=ON'
            '-D WITH_CUDA=ON'
            '-D ENABLE_FAST_MATH=ON'
            '-D CUDA_FAST_MATH=ON'
            '-D WITH_CUBLAS=ON'
            #'-D CMAKE_CXX_FLAGS=-std=c++11'
            '-D CMAKE_CXX_FLAGS=-std=c++98'
            #'-D SFM_DEPS_OK=ON' # opencv_contrib sfm problem: https://github.com/opencv/opencv_contrib/issues/500
# Settings for neural network module'
            '-D BUILD_opencv_dnn=ON'
            '-D BUILD_LIBPROTOBUF_FROM_SOURCES=ON'
    )

# SSE only available from Pentium 3 onwards (i686 is way older)
[[ "$CARCH" = 'i686' ]] && \
    _cmakeopts+=('-D ENABLE_SSE=OFF'
                 '-D ENABLE_SSE2=OFF'
                 '-D ENABLE_SSE3=OFF')

# all x64 CPUs support SSE2 but not SSE3
[[ "$CARCH" = 'x86_64' ]] && _cmakeopts+=('-D ENABLE_SSE3=OFF')

# NEON support only for armv7h
[[ "$CARCH" = 'armv7h' ]] && _cmakeopts+=('-D ENABLE_NEON=ON')

# intel-tbb not available for armv6h
[[ "$CARCH" = 'armv6h' ]] && _cmakeopts+=('-D WITH_TBB=OFF')

pkgver() {
    cd "${srcdir}/${pkgname%-git}"
    git describe --long | sed -r 's/([^-]*-g)/r\1/;s/-/./g'
}

prepare() {
    cd "${srcdir}/${pkgname%-git}"
    patch -p1 -i "${srcdir}/opencv_gcc6_pch.patch"

    # hack-fix folder naming inconsistency that they won't fix
    # see http://code.opencv.org/issues/2512
    # and https://bugs.archlinux.org/task/32342
    # patch -p1 -i "${srcdir}/fsh.patch"
    sed 's/OpenCV\/doc/doc\/opencv/' -i CMakeLists.txt
    sed 's/share\/OpenCV/share\/opencv/' -i CMakeLists.txt
    sed 's/share\/OpenCV/share\/opencv/' -i cmake/templates/opencv_run_all_tests_unix.sh.in

    cd "${srcdir}/${pkgname%-git}_contrib"
    patch -p1 -i "${srcdir}/opencv_contrib_sfm_cmake.patch"
}

build() {
    cd "${srcdir}/${pkgname%-git}"

    #-D CUDA_NVCC_FLAGS='-std=c++11 -Xcompiler -D__CORRECT_ISO_CPP11_MATH_H_PROTO'
    cmake ${_cmakeopts[@]} \
        -D OPENCV_EXTRA_MODULES_PATH=$srcdir/${pkgname%-git}_contrib/modules \
        .

    make
}

package() {
    cd "${srcdir}/${pkgname%-git}"

    make DESTDIR="${pkgdir}" install

    # install LICENSE file
    install -Dm644 "LICENSE" "${pkgdir}/usr/share/licenses/${pkgname%-git}/LICENSE"
}

# vim:set ts=4 sw=4 et:
