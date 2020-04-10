# Maintainer: Yardena Cohen <yardenack at gmail dot com>
# tracks: https://projects.archlinux.org/svntogit/packages.git/log/trunk?h=packages/linux

pkgname=linux-linode
_basekernel=5.6.3
_hardenedver=a
pkgrel=1
_kernelname=${pkgname#linux}
_srcname=linux-${_basekernel}
_patchname=linux-hardened
pkgver=${_basekernel}.${_hardenedver}
arch=('x86_64')
url="https://github.com/yardenac/linux-linode"
license=(GPL2)
makedepends=(xmlto docbook-xsl kmod inetutils bc libelf)
options=('!strip')
source=("https://www.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar."{xz,sign}
        "https://github.com/anthraxx/${_patchname}/releases/download/${pkgver}/${_patchname}-${pkgver}.patch"{,.sig}
        'config'
        '08_linux_linode'
        '99-grub-ll.hook'
        'menu.lst'
        'preset')
sha512sums=('d518a6451f8fe35d1baf01f7d831f8ce90665ab1793849c885b4efc873583f50f07d81b98b922c45817f6ed091b06e17cfc9c9dd291fbbc52ca2c544b7d6e089' 'SKIP'
            '985753926628c7072e5f021bbfe9ddc3929525a7f4006ce60ead6871bcd535e39ddc72c702075cf65077989f89da4a75283127438fa41591d4bb4de1fd98cfc8' 'SKIP'
            '02b3887bfd6c647ac7b8608a450442a22dd6632fe12caa2b8646863fb49095bae0e5b2ee48894fc7f5d6857662845ac83784926ebbca6b7ca5d7405eb31641f5'
            '1e901b8894743e9dcb04046a5fa58e14b19095b3295abae679dcbbf309bd79ddf1716dcd07ae8a71e7cdc9361216c0c9da12a76edb45e9388c512b07df7759e7'
            'db9080b2548e4dcd61eaaf20cd7d37cbbc8c204ce85a2e3408d0671f6b26010f77a61affd2c77e809768714eca29d3afb64765a3f2099317a2c928eff3feb4cf'
            '1a17f83747ebd2dbe8d57996a1234f9e72de0754f8907c984477d761c2d99753490b72d80e2c801b85ded705818d530401f6377e3312937d72d1e4052007ce30'
            'a0a78831075336edef0a8faa34fa550986c3c4d89a89f4f39d798da0211129dc90257d162bec2cdefabef2eb5886a710e70c72074b2f3016788861d05d1e2a1f')
sha256sums=('1d13689abbcab59250a8231a5871747be25a1d2fe431cf263395de35780c50aa' 'SKIP'
            'd0d458338b76b4f58903a3f32cf17b901976d65ed83a0727395b1fb2b0f3b32a' 'SKIP'
            '4f64ae3fbc0eff517b4aa4b81b0cf75d70372b4d337e8fed2acb94fe370873d2'
            'a2295cee5ae8c8d9c3efb0483e5e842f6bd9753f851c7433465b242264738546'
            '368fb58e7aa465f597e9a72da4b6eea4183c1a85242173412d54ad18d10d8fb3'
            'df9f20f818bf6c11296e5ac58c5fdd664ba17f3d80cf301eef9309ddb528741d'
            '33d9f913e4aed9f41bf263ff784921ab7d69597732079cbd06ff05b9adb32f44')
md5sums=('42265d19e3eba2b89291cd752830b173' 'SKIP'
         '027681974ab202d9b3f14e98ab5ecde9' 'SKIP'
         'b7108a1217cdb02680e690765ff9e8ff'
         'b24775bcf1d7a04c1d489af73c6e74b0'
         '625481f015365febcd65aa136ee555f9'
         '901c03c685c65119b50666cb31a080be'
         '3d6427cc412dbc226b5d058c82ba7c66')
validpgpkeys=(
              'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linux Torvalds
              '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
              'E240B57E2C4630BA768E2F26FC1B547C8D8172C8' # Levente Polyak
)
pkgdesc="Kernel for Linode servers"
depends=(coreutils linux-firmware kmod mkinitcpio grub)
provides=(linux)
conflicts=(grub-legacy)
backup=(etc/mkinitcpio.d/${pkgname}.preset boot/grub/menu.lst)

prepare() {
  cd "${srcdir}/${_srcname}"
  patch -p1 -i "${srcdir}/${_patchname}-${pkgver}.patch"
  sed -e "/^EXTRAVERSION =/s/=.*/= .${_hardenedver}/" -i Makefile
  scripts/setlocalversion --save-scmversion
  cp "${srcdir}/config" .config
  make oldconfig
  make -s kernelrelease > ../version
}

build() {
  cd "${srcdir}/${_srcname}"
  [[ "$MAKEFLAGS" =~ -j[0-9]* ]] || MAKEFLAGS+=" -j$(nproc)"
  ionice -c 3 nice -n 16 make ${MAKEFLAGS} bzImage modules
}

package() {
  cd "${srcdir}/${_srcname}"
  local _kernver="$(<${srcdir}/version)"
  local modulesdir="$pkgdir"/usr/lib/modules/$_kernver
  mkdir -p "$modulesdir" "${pkgdir}"/boot/grub
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  rm -rf "$modulesdir"/{source,build}
  echo "$pkgname" | install -Dm644 /dev/stdin  "$modulesdir/pkgbase"
  install -D -m644 "$(make -s image_name)"     "$modulesdir/vmlinuz"
  install -D -m644 "${srcdir}/preset"          "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"
  install -D -m755 "${srcdir}/08_linux_linode" "${pkgdir}/etc/grub.d/08_linux_linode"
  install -D -m644 "${srcdir}/99-grub-ll.hook" "${pkgdir}/usr/share/libalpm/hooks/99-grub-ll.hook"
  sed "s/%VER%/${pkgver}-${pkgrel}/ig" "${srcdir}/menu.lst" > "${pkgdir}/boot/grub/menu.lst"
  chmod -Rc u=rwX,go=rX "$pkgdir"
}
