# Maintainer: Quentin Stievenart <quentin.stievenart@gmail.com>

basename=sbcl
pkgname=${basename}-git
pkgver=20130321
pkgrel=1
pkgdesc="Steel Bank Common Lisp - git version"
arch=('i686' 'x86_64')
license=('custom')
depends=('glibc')
provides=('common-lisp' 'cl-asdf' 'sbcl')
makedepends=('git' 'sbcl' 'texinfo')
conflicts=('sbcl')
source=('arch-fixes.lisp')
md5sums=('7ac0c1936547f4278198b8bf7725204d')
url="http://www.sbcl.org/"
install=sbcl.install

_gitroot="git://sbcl.git.sourceforge.net/gitroot/sbcl/${basename}.git"
_gitname="${basename}"

build() {
  cd "${srcdir}"
  msg "Connecting to GIT server...."

  if [ -d $_gitname ] ; then
    cd $_gitname && git pull origin
    msg "The local files are updated."
  else
    git clone $_gitroot $_gitname
  fi

  msg "GIT checkout done or server timeout"
  msg "Starting make..."

  export CFLAGS="${CFLAGS} -fno-omit-frame-pointer -D_GNU_SOURCE"
  #export GNUMAKE="make -e"

  # Clone local repository for the build. Avoid some compilation problems
  if [ -d ${srcdir}/${basename}-build ] ; then
    rm -rf ${srcdir}/${basename}-build
  fi
  git clone ${srcdir}/${basename} ${srcdir}/${basename}-build


  cd ${srcdir}/${basename}-build
  # Make a multi-threaded SBCL, disable LARGEFILE  
  cat >customize-target-features.lisp <<EOF
(lambda (features)
  (flet ((enable (x) (pushnew x features))
         (disable (x) (setf features (remove x features))))
  (enable :sb-thread)
  (disable :largefile)))
EOF

  sh make.sh --prefix=/usr
  mkdir -p ${pkgdir}/usr
  #pushd doc/manual
  #make info || return 1
  #popd 
  unset SBCL_HOME
  INSTALL_ROOT=${pkgdir}/usr sh install.sh

  src/runtime/sbcl --core output/sbcl.core --script ${srcdir}/arch-fixes.lisp
  mv sbcl-new.core ${pkgdir}/usr/lib/sbcl/sbcl.core

# sources
  mkdir -p ${pkgdir}/usr/share/sbcl-source
  cp -R -t ${pkgdir}/usr/share/sbcl-source \
    ${srcdir}/${basename}/{src,contrib}

# drop unwanted files
  find ${pkgdir}/usr/share/sbcl-source -type f \
    -name \*.fasl -or \
    -name \*.o -or \
    -name \*.log -or \
    -name \*.so -or \
    -name a.out -delete

  find ${pkgdir} \( -name Makefile -o -name .cvsignore \) -delete

  #rm ${pkgdir}/usr/share/info/dir
  #gzip -9nf ${pkgdir}/usr/share/info/*

  # license
  install -D -m644 ${srcdir}/${basename}/COPYING \
                   ${pkgdir}/usr/share/licenses/${basename}/license.txt
}
