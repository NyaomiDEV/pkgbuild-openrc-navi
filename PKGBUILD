# Maintainer: artoo <artoo@artixlinux.org>
# Maintainer: Chris Cromer <cromer@artixlinux.org>
# Contributor: williamh <williamh@gentoo.org>

_url=https://gitea.artixlinux.org/artix
_extras=1.2 # git rev-parse ${_extras} #79d369b5089c1af77289ebe1e2cf711f6f7a5e28
_alpm=1.4 # git rev-parse ${_alpm} #83961019292a041e1d2c07389d639065632e3f1f

pkgname=openrc
pkgver=0.50
pkgrel=1
pkgdesc="Gentoo's universal init system"
arch=('x86_64')
url="https://github.com/OpenRC/openrc"
license=('BSD2')
makedepends=('git' 'meson')
depends=('bash' 'glibc' 'inetutils' 'libcap' 'libcap.so'
         'ncurses' 'libncursesw.so' 'netifrc' 'pam' 'libpam.so' 'psmisc' 'perl')
optdepends=('networkmanager-openrc: networkmanager init script'
            'elogind-openrc: elogind init script')
provides=('init-rc' 'svc-manager' 'librc.so' 'libeinfo.so')
conflicts=('init-rc' 'svc-manager')
replaces=(openrc-{deptree2dot,{bash,zsh}-completions})
backup=('etc/rc.conf'
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
        'etc/conf.d/agetty.tty'{1,2,3,4,5,6})
source=("${pkgname}-${pkgver}.tar.gz::${url}/archive/refs/tags/${pkgver}.tar.gz"
        'openrc.logrotate'
        'sysctl.conf'
        "artix-rc-conf.patch::${_url}/openrc/commit/e2acda84ddd772594fb968f03eacbc129cc7d65e.patch"
        "artix-meson.patch::${_url}/openrc/commit/05b1fd974c71041265a862ca3a2ba4fc79e797cc.patch"
        "git+${_url}/openrc-extra.git#tag=${_extras}"
        "git+${_url}/alpm-hooks.git#tag=${_alpm}")
sha256sums=('8d9bb3a68a491d5d4e0f0af1515e00f27e4463acc0c256930aded26c7c8a834b'
            '0b44210db9770588bd491cd6c0ac9412d99124c6be4c9d3f7d31ec8746072f5c'
            '874e50bd217fef3a2e3d0a18eb316b9b3ddb109b93f3cbf45407170c5bec1d6d'
            '9420304937ca075714fa3d5deefda5b2bd51ee7398f90ba8ca49594f07baef7a'
            '3924bfe28ef14f2d20c03675f246ffb4fdc83f6a5b80f4b3bda0d5e7a14303ef'
            'SKIP'
            'SKIP')

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
    local meson_options=(
        -Dbranding="\"Artix Linux\""
        -Dos=Linux
        -Drootprefix=/usr
        -Dshell=/bin/bash
        -Dpam=true
        -Dsysvinit=true
        -Dpkgconfig=true
        -Dtermcap=ncurses
        -Dbash-completions=true
        -Dzsh-completions=true
        -Dsplit-usr=true
        -Dnewnet=false
        -Daudit=disabled
        -Dselinux=disabled
        -Dlibrcdir=openrc
    )

    artix-meson "${pkgname}-${pkgver}" build "${meson_options[@]}"

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
