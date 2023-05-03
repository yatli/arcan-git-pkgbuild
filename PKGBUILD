# Maintainer: Adrián Pérez de Castro <aperez@igalia.com>
pkgdesc='Game Engine meets a Display Server meets a Multimedia Framework'
pkgname='arcan-git'
pkgver=0.6.2.1.r73.g42ec5d86
pkgrel=1
license=('GPL2' 'LGPL' 'custom:BSD')
arch=(aarch64 'x86_64')
depends=('freetype2' 'harfbuzz' 'harfbuzz-icu' 'mesa' 'luajit' 'sqlite'
         'libxkbcommon' 'libvncserver' 'libusb' 'openal' 'ffmpeg' 'apr' 'wayland-protocols')
makedepends=('cmake' 'ruby' 'git')
provides=('arcan')
conflicts=('arcan')
url='https://arcan-fe.com/'
source=("${pkgname}::git+https://github.com/letoram/arcan.git"
        "0001-fix-build-werror.patch")
sha512sums=('SKIP' 'SKIP')

pkgver () {
	cd "${pkgname}"
	(
		set -o pipefail
		git describe --long 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
		printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
	)
}

prepare () {
  cd "${srcdir}"
  patch -Np1 -i 0001-fix-build-werror.patch
}

build () {
	pushd "${pkgname}/doc" &> /dev/null
	ruby docgen.rb mangen
	popd &> /dev/null

	rm -rf "${pkgname}/build"
	mkdir "${pkgname}/build"
	cd "${pkgname}/build"
	cmake \
		-DCMAKE_BUILD_TYPE=Release \
		-DCMAKE_INSTALL_PREFIX=/usr \
		-DVIDEO_PLATFORM=egl-dri \
		-DHYBRID_SDL=ON \
		-DENABLE_LWA=ON \
		-DENABLE_LTO=ON \
		../src
	make
}

package () {
	cd "${pkgname}/build"
	DESTDIR="${pkgdir}" make install

	# Fix location of manpages.
	if [[ -d ${pkgdir}/usr/man ]] ; then
		mv "${pkgdir}/usr/man" "${pkgdir}/usr/share"
	fi

	# Ditto for CMake files.
	if [[ -d ${pkgdir}/usr/CMake ]] ; then
		for path in "${pkgdir}/usr/CMake"/*.cmake ; do
			local basename=${path##*/}
			if [[ ${basename} = *-config.cmake ]] ; then
				install -vDm644 "${path}" \
					"${pkgdir}/usr/share/cmake/${basename:0:-13}/${basename}"
			elif [[ ${basename} = *Config.cmake ]] ; then
				install -vDm644 "${path}" \
					"${pkgdir}/usr/share/cmake/${basename:0:-12}/${basename}"
			fi
		done
		rm -rf "${pkgdir}/usr/CMake"
	fi
}
