# Contributor tomasgroth at yahoo.dk
# Contributor WarheadsSE <max@warheads.net>
# Maintainer CruX <CruX@project-insanity.org>
# Contributor: pepedog at archlinuxarm dot com

buildarch=4

pkgname=xbmc-imx-git
pkgver=15.20151104
pkgrel=1
pkgdesc="A software media player and entertainment hub for digital media for select imx6 systems"
arch=('armv7h')
url="http://xbmc.org"
license=('GPL' 'custom')
depends=('fribidi' 'lzo2' 'smbclient' 'libtiff' 'libpng' 'libcdio' 'yajl' 'libmariadbclient' 'libjpeg-turbo' 'libsamplerate' 'libssh' 'libmicrohttpd' 'sdl_image' 'python2' 'libass' 'libmpeg2' 'libmad' 'libmodplug' 'jasper' 'rtmpdump' 'unzip' 'libbluray' 'libnfs' 'afpfs-ng' 'libshairport' 'avahi' 'bluez-libs' 'tinyxml' 'libplist' 'swig' 'taglib' 'libxslt' 'libfslvpuwrap' 'gpu-viv-bin-mx6q-fb' 'gpu-viv-g2d' 'libcec-imx6' 'firmware-imx')
makedepends=('boost' 'cmake' 'gperf' 'nasm' 'zip' 'udisks' 'git' 'autoconf' 'java-runtime-headless' 'linux-headers-imx6-fsl' 'pkg-config' 'automake' # contains aclocal
'make' 'libtool' 'binutils' # contains strip binary
'libplatform' # Can be found here: https://github.com/zpon/libplatform-arch
'dcadec-git' # Can be found here: https://aur.archlinux.org/packages/dcadec-git/
)
optdepends=(
  'lirc: remote controller support'
  'udisks: automount external drives'
  'unrar: access compressed files without unpacking them'
)
provides=("xbmc")
conflicts=("xbmc")
install="kodi.install"
source=('kodi.service'
        'runkodi'
        'kodi.conf'
        '10-kodi.rules'
        'imx-spdif.conf'
        'imx-hdmi-soc.conf'
        'OpenMaxVideo.cpp.patch')


md5sums=('4c689f3ab03d57ad4f02f57d45d6c923'
         '14300a872c0dbdda478d5cfc71f551ec'
         '8fab4cc5cac44a7090ca7e839e326ec8'
         'c3ad87fc9f278f9530d673be9b1f58f0'
         'df3edfc7269d4a4d6f94d935c9adb0ac'
         '90f401e9f255291ec75414056e0d30c0'
         'c773fa542e067c39b62a3ffd51318cde')

# master branch of xbmc-imx6 organization. Modified by Stephan "wolgar" Rafin, Chris "koying" Browet, Rudi "rudi-warped" Ihle and smallint
_gitname="xbmc"
_gitroot="git://github.com/xbmc"
_gitbranch="master"

_prefix=/usr

prepare() {
  cd "${srcdir}"


  msg2 "Connecting to GIT server..."
  
  if [[ -d "${_gitname}" ]]; then
     cd "${_gitname}" && git pull origin "$_gitbranch"
     msg2 "The local files are updated."
  else
     git clone --branch ${_gitbranch} --depth 1 "${_gitroot}/${_gitname}"
  fi
  
  msg2 "GIT checkout done or server timeout." 

  cd "${srcdir}/${_gitname}"

  # fix lsb_release dependency
  sed -i -e 's:/usr/bin/lsb_release -d:cat /etc/arch-release:' xbmc/utils/SystemInfo.cpp

  # revert OpenVideoMax.cpp
  git checkout xbmc/cores/dvdplayer/DVDCodecs/Video/OpenMaxVideo.cpp
  # patch OpenVideoMax.cpp
  git apply ../../OpenMaxVideo.cpp.patch
}

build() {
  cd "${srcdir}/${_gitname}"

  # Bootstrapping XBMC
  ./bootstrap

  # Configuring XBMC
  export PYTHON_VERSION=2  # external python v2
 
  INCLUDES="-I/opt/fsl/include"
  export CFLAGS="-Ofast -mfloat-abi=hard -mfpu=vfpv3-d16 -mtls-dialect=gnu -march=armv7-a -mtune=cortex-a9 -pipe -fstack-protector --param=ssp-buffer-size=4 -D_FORTIFY_SOURCE=2 $INCLUDES -Wl,-rpath,/opt/fsl/lib -L/opt/fsl/lib"
  export CPPFLAGS="-Ofast -mfloat-abi=hard -mfpu=vfpv3-d16 -mtls-dialect=gnu -march=armv7-a -mtune=cortex-a9 -fstack-protector --param=ssp-buffer-size=4 -D_FORTIFY_SOURCE=2 $INCLUDES -Wl,-rpath,/opt/fsl/lib -L/opt/fsl/lib"
  export CXXFLAGS="-Ofast -mfloat-abi=hard -mfpu=vfpv3-d16 -mtls-dialect=gnu -march=armv7-a -mtune=cortex-a9 -fstack-protector --param=ssp-buffer-size=4 -D_FORTIFY_SOURCE=2 $INCLUDES -Wl,-rpath,/opt/fsl/lib -L/opt/fsl/lib"
  export LDFLAGS="$LDFLAGS -L/opt/fsl/lib"

  ./configure --prefix=$_prefix --exec-prefix=$_prefix  \
    --disable-gl --enable-gles --disable-x11 --disable-sdl \
    --enable-optimizations --disable-external-libraries --disable-goom --disable-hal \
    --disable-pulse --disable-vaapi --disable-vdpau --disable-xrandr --enable-airplay \
    --enable-avahi --enable-libbluray --enable-dvdcss --disable-debug --disable-joystick --disable-mid \
    --enable-nfs --disable-profiling --disable-projectm --enable-rsxs --enable-rtmp --disable-vaapi \
    --disable-external-ffmpeg --enable-optical-drive --enable-codec=imxvpu --enable-libcec

  make
}

package() {
  cd "${srcdir}/${_gitname}"
  # Running make install
  make DESTDIR="${pkgdir}" install

  # run feh with python2
  sed -i -e 's/python/python2/g' ${pkgdir}${_prefix}/bin/xbmc
  sed -i -e 's/python/python2/g' ${pkgdir}${_prefix}/bin/kodi

  # lsb_release fix
  sed -i -e 's/which lsb_release > \/dev\/null/\[ -f \/etc\/arch-release ]/g' "${pkgdir}${_prefix}/bin/xbmc"
  sed -i -e 's/which lsb_release > \/dev\/null/\[ -f \/etc\/arch-release ]/g' "${pkgdir}${_prefix}/bin/kodi"
  sed -i -e "s/lsb_release -a 2> \/dev\/null | sed -e 's\/\^\/    \/'/cat \/etc\/arch-release/g" "${pkgdir}${_prefix}/bin/xbmc"
  sed -i -e "s/lsb_release -a 2> \/dev\/null | sed -e 's\/\^\/    \/'/cat \/etc\/arch-release/g" "${pkgdir}${_prefix}/bin/kodi"

  # Tools
  # TexturePacker has been moved, this step fails
  #install -D -m 0755 "${srcdir}/${_gitname}/tools/TexturePacker/TexturePacker" "${pkgdir}${_prefix}/share/kodi/"

  # Licenses
  install -d -m 0755 "${pkgdir}${_prefix}/share/licenses/${pkgname}"
  for licensef in LICENSE.GPL copying.txt; do
    mv "${pkgdir}${_prefix}/share/doc/kodi/${licensef}" "${pkgdir}${_prefix}/share/licenses/${pkgname}"
  done

  # systemd stuff
  install -Dm0644 $srcdir/kodi.service $pkgdir/usr/lib/systemd/system/kodi.service
  install -Dm0755 $srcdir/runkodi $pkgdir/usr/bin/runkodi
  install -Dm0644 $srcdir/kodi.conf $pkgdir/etc/tmpfiles.d/kodi.conf
  install -Dm0644 $srcdir/10-kodi.rules $pkgdir/etc/polkit-1/rules.d/10-kodi.rules
  chmod 700 $pkgdir/etc/polkit-1/rules.d

  # imx6-specific alsa conf
  install -Dm0644 $srcdir/imx-spdif.conf $pkgdir/usr/share/alsa/cards/imx-spdif.conf
  install -Dm0644 $srcdir/imx-hdmi-soc.conf $pkgdir/usr/share/alsa/cards/imx-hdmi-soc.conf
}
