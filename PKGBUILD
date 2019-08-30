# Maintainer: Jingbei Li <i@jingbei.li>
# Contributor: Grey Christoforo <first name at last name dot net>

## This PKGBUILD creates an Arch Linux package for the proprietary MATLAB application. A license from The MathWorks is needed in order to both build the package and to run MATLAB once the package is installed. In order to build the package the user must supply a plain text file installation, the software and a license file. The tar archive file can be generated from an ISO downloaded from The MathWorks, generated from the official DVD, or created by using the interactive installer to download the toolboxes (installation can be made to a temporary directory and canceled once the toolboxes are downloaded). The contents of the tar archive must include: ./archives/ ./bin/ ./etc/ ./help/ ./java/ /sys ./activate.ini ./install ./installer_input.txt

pkgbase='matlab'
pkgname=('matlab' 'matlab-licenses')
pkgver=9.6.0.1072779
pkgrel=1
pkgdesc='A high-level language for numerical computation and visualization'
arch=('x86_64')
url='http://www.mathworks.com'
license=(custom)
makedepends=('gendesk')
depends=('gconf'
         'glu'
         'gstreamer0.10-base'
         'gtk2'
         'libunwind'
         'libxp'
         'libxpm'
         'libxss'
         'libxtst'
         'nss'
         'gcc6'
         'portaudio'
         'python2'
         'qt5-svg'
         'qt5-webkit'
         'qt5-websockets'
         'qt5-x11extras'
         'xerces-c')
source=("file://matlab.tar"
        "file://matlab.fik"
        "file://license.dat")
md5sums=('SKIP' 'SKIP' 'SKIP')

prepare() {
  msg2 'Creating desktop file'
  gendesk -f -n --pkgname "${pkgname}" --pkgdesc "${pkgdesc}" --categories "Development;Education;Science;Mathematics;IDE" > /dev/null
  sed -i "/^Exec=/ s,$, -desktop," "${srcdir}/${pkgname}.desktop"

  msg2 'Extracting file installation key'
  _fik=$(grep -o [0-9-]* ${pkgname}.fik)

  msg2 'Modifying the installer settings'
  sed -i "s,^# destinationFolder=,destinationFolder=${pkgdir}/opt/tmw/${pkgname}/," "${srcdir}/installer_input.txt"
  sed -i "s,^# agreeToLicense=,agreeToLicense=yes," "${srcdir}/installer_input.txt"
  sed -i "s,^# mode=,mode=silent," "${srcdir}/installer_input.txt"
  sed -i "s,^# fileInstallationKey=,fileInstallationKey=${_fik}," "${srcdir}/installer_input.txt"
  sed -i "s,^# licensePath=,licensePath=${srcdir}/license.dat," "${srcdir}/installer_input.txt"
}

package_matlab() {
  msg2 'Starting MATLAB installer'
  "${srcdir}/install" -t -inputFile "${srcdir}/installer_input.txt" -mode silent

  msg2 'Installing license'
  install -D -m644 "${pkgdir}/opt/tmw/${pkgname}/license_agreement.txt" "${pkgdir}/usr/share/licenses/tmw/${pkgname}/LICENSE"

  msg2 'Creating links for license'
  rm -rf "${srcdir}/licenses"
  mv "${pkgdir}/opt/tmw/${pkgname}/licenses" "${srcdir}/licenses"
  mkdir -p "${pkgdir}/opt/tmw/${pkgname}/licenses"

  msg2 'Creating links for executables'
  install -d -m755 "${pkgdir}/usr/bin/"
  for _executable in deploytool matlab mbuild mcc; do
    ln -s "/opt/tmw/${pkgname}/bin/${_executable}" "${pkgdir}/usr/bin/${_executable}"
  done
  ln -s "/opt/tmw/${pkgname}/bin/mex" "${pkgdir}/usr/bin/mex-$pkgbase"

  msg2 'Installing desktop files'
  install -D -m644 "${pkgname}.desktop" "${pkgdir}/usr/share/applications/${pkgname}.desktop"
  install -D -m644 "${pkgdir}/opt/tmw/${pkgname}/help/matlab/matlab_env/matlab_desktop_icon.png" "${pkgdir}/usr/share/pixmaps/${pkgname}.png"

  msg2 'Configuring mex options'
  sed -i "s/gcc/gcc-6/g" "${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/mexopts/gcc_glnxa64.xml"
  sed -i "s/g++/g++-6/g" "${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/mexopts/g++_glnxa64.xml"
  sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/mexopts/gfortran.xml"
  sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/mexopts/gfortran6.xml"
  sed -i "s/gfortran6-/gfortran-6/g" "${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/mexopts/gfortran6.xml"

  # See $MATLABROOT/sys/os/glnxa64/README.libstdc++
  msg2 'Removing unused library files'
  rm ${pkgdir}/opt/tmw/${pkgname}/sys/os/glnxa64/{libstdc++.so.6.0.22,libstdc++.so.6,libgcc_s.so.1,libgfortran.so.3.0.0,libgfortran.so.3,libquadmath.so.0.0.0,libquadmath.so.0}

  # https://bbs.archlinux.org/viewtopic.php?id=236821
  rm ${pkgdir}/opt/tmw/${pkgname}/bin/glnxa64/libfreetype.*

  # make sure MATLAB can find libgfortran.so.3
  sed -i 's,LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`",LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`:/usr/lib/gcc/x86_64-pc-linux-gnu/'$(pacman -Q gcc6 | awk '{print $2}' | cut -d- -f1)'",g' "${pkgdir}/opt/tmw/matlab/bin/matlab"
}

package_matlab-licenses() {
  depends=("matlab=$pkgver")
  mkdir -p "${pkgdir}/opt/tmw/${pkgbase}"
  mv "${srcdir}/licenses" "${pkgdir}/opt/tmw/${pkgbase}/licenses"
}
