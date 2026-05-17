# SPDX-License-Identifier: AGPL-3.0

#    -----------------------------------------------------
#    Copyright © 2024, 2025, 2026  Pellegrino Prevete
#
#    All rights reserved
#    -----------------------------------------------------
#
#    This program is free software: you can redistribute
#    it and/or modify it under the terms of the
#    GNU Affero General Public License as published by
#    the Free Software Foundation, either version 3 of
#    the License, or (at your option) any later version.
#
#    This program is distributed in the hope that it
#    will be useful, but WITHOUT ANY WARRANTY;
#    without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
#    See the GNU Affero General Public License for
#    more details.
#
#    You should have received a copy of the
#    GNU Affero General Public License
#    along with this program.
#    If not, see <https://www.gnu.org/licenses/>.

# Maintainers:
#   Truocolo
#     <truocolo@aol.com>
#     <truocolo@0x6E5163fC4BFc1511Dbe06bB605cc14a3e462332b>
#   Pellegrino Prevete (dvorak)
#     <pellegrinoprevete@gmail.com>
#     <dvorak@0x87003Bd6C074C713783df04f36517451fF34CBEf>
#   a821 at (nospam) mail de
# Contributors:
#   Daniel Milde
#     <daniel@milde.cz>
#   Chih-Hsuan Yen
#     <yan12125@gmail.com>
# Modified from extra/python;
#   original contributors:
#   Angel Velasquez
#     <angvp@archlinux.org>
#   Felix Yan
#     <felixonmars@archlinux.org>
#   Stéphane Gaudreault
#     <stephane@archlinux.org>
#   Allan McRae
#     <allan@archlinux.org>
#   Jason Chu
#     <jason@archlinux.org>

shopt \
  -s \
  "extglob"

_os="$(
  uname \
    -o)"
if [[ ! -v "_c_compiler" ]]; then
  if [[ "${_os}" == "GNU/Linux" ]]; then
    _c_compiler="gcc"
  elif [[ "${_os}" == "Android" ]]; then
    _c_compiler="llvm"
  elif [[ "${_os}" == "Msys" ]]; then
    _c_compiler="gcc"
  fi
fi
if [[ ! -v "_git" ]]; then
  _git="true"
fi
if [[ ! -v "_evmfs" ]]; then
  _evmfs="false"
fi
if [[ ! -v "_tag_name" ]]; then
  _tag_name="branch"
fi
if [[ ! -v "_archive_format" ]]; then
  if [[ "${_evmfs}" == "false" ]]; then
    _archive_format="git"
  elif [[ "${_evmfs}" == "true" ]]; then
    _archive_format="bundle"
  fi
fi
_py=python
_pkg="${_py}"
_Pkg="c${_pkg}"
pkgbase="${_py}-git"
pkgname=(
  "${pkgbase}"
)
_branch="main"
pkgver=3.15.0b1.r113.gacefff95eab
pkgrel=1
_pkgdesc=(
  "The Python programming language"
)
pkgdesc="${_pkgdesc[*]}"
arch=(
  "aarch64"
  "arm"
  "armv7l"
  "armv8l"
  "i686"
  "mips"
  "powerpc"
  "pentium4"
  'x86_64'
)
license=(
  'PSF-2.0'
)
url="https://www.${_py}.org/"
depends=(
  'bzip2'
  'expat'
  'gdbm'
  'libffi'
  'libnsl'
  'libxcrypt'
  'openssl'
  'zlib'
  'tzdata'
  'mpdecimal'
  'zstd'
)
makedepends=(
  'bluez-libs'
  "${_c_compiler}"
  'sqlite'
  'gdb'
  'xorg-server-xvfb'
  'tk'
  'ttf-font'
)
if [[ "${_git}" == "true" ]]; then
  makedepends+=(
    "git"
  )
fi
# provides=(
#   "${_py}=${_pymajver}"
# )
# conflicts=(
#   "${_py}"
# )
_http="https://github.com"
_ns="${_py}"
_url="${_http}/${_ns}/${_Pkg}"
if [[ "${_tag_name}" == "branch" ]]; then
  _tag="${_branch}"
fi
_tarname="${_Pkg}-${_tag}"
_tarfile="${_tarname}.${_archive_format}"
if [[ "${_git}" == "true" ]]; then
  _uri="git+${_url}#${_tag_name}=${_tag}"
  _src="${_tarname}::${_uri}"
fi
source=(
  "${_src}"
)
sha256sums=(
  'SKIP'
)

pkgver() {
  cd \
    "${_tarname}"
  git \
    describe \
    --long \
    --tags |
    sed \
      's/^v//;s/-/.r/;s/-/./g'
}

prepare() {
  cd \
    "${_tarname}"
  # Ensure that we are using the
  # system copy of various libraries (expat),
  # rather than copies shipped in the tarball
  rm \
    -r \
    "Modules/expat"
}

build() {
  local \
    _configure_opts=()
  _configure_opts+=(
    --prefix="/usr"
    --enable-shared
    --with-computed-gotos
    --enable-optimizations
    --with-lto
    --enable-ipv6
    --with-system-expat
    --with-dbmliborder="gdbm:ndbm"
    --with-system-ffi
    --with-system-libmpdec
    --enable-loadable-sqlite-extensions
    --without-ensurepip
    --with-tzpath="/usr/share/zoneinfo"
  )
  cd \
    "${_tarname}"
  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3} -ffat-lto-objects"
  # Disable bundled pip & setuptools
  ./configure \
    "${_configure_opts[@]}"
  # Obtain next free server number
  # for xvfb-run;
  # this even works in a chroot environment.
  export \
    servernum=99
  while ! xvfb-run \
            -a \
            -n \
              "${servernum}" \
            "/bin/true" \
            2>"/dev/null"; do \
  servernum=$((
    servernum + 1 )); \
  done
  LC_CTYPE="en_US.UTF-8" \
    xvfb-run \
      -s \
        "-screen 0 1920x1080x16 -ac +extension GLX" \
      -a \
      -n \
        "${servernum}" \
      make \
        EXTRA_CFLAGS="${CFLAGS}"
}

package() {
  local \
    _make_opts=() \
    _pybasever
  _make_opts+=(
    DESTDIR="${pkgdir}"
    EXTRA_CFLAGS="$CFLAGS"
    ENSUREPIP="install"
  )
  optdepends=(
    'sqlite'
    'mpdecimal: for decimal'
    'xz: for lzma'
    'tk: for tkinter'
  )
  cd \
    "${_tarname}"
  # Hack to avoid building again
  sed \
    's/^all:.*$/all: build_all/' \
    -i \
    "Makefile"
  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3}"
  make \
    "${_make_opts[@]}" \
    altinstall \
    maninstall
  # Work around a conflict with the 'python' package.
  rm \
    "${pkgdir}/usr/lib/libpython3.so" \
    "${pkgdir}/usr/share/man/man1/python3.1"
  _pybasever="$(
    sed \
      -n \
      's/^VERSION=//p' \
      "configure")"
  # some useful "stuff" FS#46146
  install \
    -vdm755 \
    "${pkgdir}/usr/lib/python${_pybasever}/Tools/"{"i18n","scripts"}
  install \
    -vm755 \
    "Tools/i18n/"{"msgfmt","pygettext"}".py" \
    "${pkgdir}/usr/lib/python${_pybasever}/Tools/i18n/"
  install \
    -vm755 \
    "Tools/scripts/"{"README",*"py"} \
    "${pkgdir}/usr/lib/python${_pybasever}/Tools/scripts/"
}
