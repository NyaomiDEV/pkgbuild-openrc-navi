# Maintainer: artoo <artoo@artixlinux.org>
# Maintainer: Chris Cromer <cromer@artixlinux.org>
# Contributor: williamh <williamh@gentoo.org>

_url=https://gitea.artixlinux.org/artix
_extras=1.2
_alpm=1.4

pkgname=openrc
pkgver=0.54
pkgrel=1
pkgdesc="Gentoo's universal init system"
arch=('x86_64')
url="https://github.com/OpenRC/openrc"
license=('BSD-2-Clause')
makedepends=('git' 'meson')
depends=(
    'bash'
    'glibc'
    'inetutils'
    'libcap' 'libcap.so'
    'netifrc'
    'pam' 'libpam.so'
    'psmisc'
    'perl'
)
optdepends=(
    'networkmanager-openrc: networkmanager init script'
    'elogind-openrc: elogind init script'
)
provides=(
    'init-rc'
    'svc-manager'
    'librc.so'
    'libeinfo.so'
)
conflicts=('init-rc' 'svc-manager')
replaces=(openrc-{deptree2dot,{bash,zsh}-completions})
backup=(
    'etc/rc.conf'
    'etc/conf.d/consolefont'
    'etc/conf.d/keymaps'
    'etc/conf.d/hostname'
    'etc/conf.d/modules'
    'etc/conf.d/hwclock'
    'etc/conf.d/etmpfiles-dev'
    'etc/conf.d/etmpfiles-setup'
    'etc/conf.d/udev'
    'etc/conf.d/udev-trigger'
    'etc/conf.d/udev-settle'
    'etc/conf.d/agetty.tty'{1,2,3,4,5,6}
)
source=(
    "${pkgname}-${pkgver}.tar.gz::${url}/archive/refs/tags/${pkgver}.tar.gz"
    'openrc.logrotate'
    'sysctl.conf'
    "rc-conf-artix.patch::${_url}/openrc/commit/6f9e4c6b4bebebad2f00d1c19bf1f93c707d9a09.patch"
    "artix-meson.patch::${_url}/openrc/commit/05b1fd974c71041265a862ca3a2ba4fc79e797cc.patch"
    "git+${_url}/openrc-extra.git#tag=${_extras}"
    "git+${_url}/alpm-hooks.git#tag=${_alpm}"
)
sha256sums=('c84ff1d8e468c043fe136d11d3d34d6bb28328267d1352526a5d18cdf4c60fb0'
            '0b44210db9770588bd491cd6c0ac9412d99124c6be4c9d3f7d31ec8746072f5c'
            '874e50bd217fef3a2e3d0a18eb316b9b3ddb109b93f3cbf45407170c5bec1d6d'
            'e8f5374e4efd64db07a8f352a10da065e9393761faf64b7f26aba1928d4286af'
            '3924bfe28ef14f2d20c03675f246ffb4fdc83f6a5b80f4b3bda0d5e7a14303ef'
            '88c2ddad5ac5d347962ce9805a0ed7a4f1737aaafa3d6a8c0a7a55009ce5fef1'
            '8cd1cb0f89c4afe85cd286a10647f18e4443faed58f663ec3da39fa4cd807512')

prepare() {
    cd "${pkgname}-${pkgver}"
    # apply patch from the source array (should be a pacman feature)
    local src
    for src in "${source[@]}"; do
        src="${src%%::*}"
        src="${src##*/}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src"
    done
}

check(){
    meson test -C build --print-errorlogs
}

build(){
    local _meson_options=()
    _meson_options+=(
        -Dbranding="\"Artix Linux\""
        -Dos=Linux
        -Drootprefix=/usr
        -Dshell=/bin/bash
        -Dpam=true
        -Dsysvinit=true
        -Dpkgconfig=true
        -Dbash-completions=true
        -Dzsh-completions=true
        -Dnewnet=false
        -Daudit=disabled
        -Dselinux=disabled
        -Dlibrcdir=openrc
    )

    artix-meson "${pkgname}-${pkgver}" build "${_meson_options[@]}"

    meson compile -C build
}

package() {
    meson install -C build --destdir "${pkgdir}"

    install -Dm644 "${srcdir}/${pkgname}".logrotate "${pkgdir}"/etc/logrotate.d/"${pkgname}"

    install -d "${pkgdir}"/usr/lib/{openrc/cache,binfmt.d,sysctl.d}

    # sysctl defaults
    install -m755 "${srcdir}"/sysctl.conf "${pkgdir}"/usr/lib/sysctl.d/50-default.conf

    # license
    install -Dm644 "${pkgname}-${pkgver}"/LICENSE "${pkgdir}"/usr/share/licenses/"${pkgname}"/LICENSE

    # openrc extra; agetty,kmod,udev,tmpfiles,sysusers
    make -C "${pkgname}"-extra DESTDIR="${pkgdir}" install

    # pacman hooks
    make -C alpm-hooks DESTDIR="${pkgdir}" install_openrc

    # remove suport dir
    rm -r "${pkgdir}"/usr/share/openrc

    # remove init symlink
    rm -v "${pkgdir}"/usr/bin/init

    install -m755 "${pkgname}-${pkgver}"/support/deptree2dot/deptree2dot "${pkgdir}"/usr/bin/deptree2dot
}
