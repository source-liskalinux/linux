# PKGBUILD For Liska Linux Kernel (Combined Kernel + Headers)

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=linux
pkgbase=linux
_kernel="-liska"
_hostname="liskalinux"
# _kernelver drives ONLY the source tarball URL and srcdir path below. Kept
# separate from pkgver because kernel.org's tarball naming is inconsistent:
# X.Y.0 releases are published as linux-X.Y.tar.xz (no trailing .0), while
# X.Y.Z releases with Z>0 are linux-X.Y.Z.tar.xz. pkgver, on the other hand
# follows the kernel's actual X.Y.Z version scheme (matching KERNELRELEASE/
# `uname -r`), so it stays "7.2.0" here even though the tarball is fetched
# as "linux-7.2.tar.xz". The /lib/modules/<...> directory name is derived
# independently in package() via `make -s kernelrelease`, so it's correct
# regardless of what either of these two variables is set to.
_kernelver=7.2
pkgver=7.2.0
pkgrel=1
pkgdesc="Liska Linux Kernel and Headers"
arch=('x86_64')
url="https://www.kernel.org"
license=('GPL-2.0-only')
depends=('coreutils' 'kmod')
optdepends=('linux-firmware')
makedepends=('bc' 'elfutils' 'libelf' 'pahole' 'kmod' 'inetutils' 'zstd')
options=('strip' '!debug')
source=(
  "https://cdn.kernel.org/pub/linux/kernel/v7.x/linux-${_kernelver}.tar.xz"
  "https://gitlab.archlinux.org/archlinux/packaging/packages/linux/-/raw/main/config.x86_64"
)
sha256sums=('SKIP' 'SKIP')

prepare() {
    cd "${srcdir}/linux-${_kernelver}"
    echo "--> [PREPARE] Setting up base configuration...."
    cp ../config.x86_64 .config
    echo "--> [PREPARE] Injecting Liska Linux kernel identity...."
    ./scripts/config --file .config --set-str CONFIG_LOCALVERSION "${_kernel}"
    ./scripts/config --file .config --set-str CONFIG_DEFAULT_HOSTNAME "${_hostname}"
    echo "--> [PREPARE] Initializing total purge of all debug info...."
    ./scripts/config --file .config --disable CONFIG_DEBUG_INFO
    ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_BTF
    ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT
    ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF4
    ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF5
    ./scripts/config --file .config --enable CONFIG_DEBUG_INFO_NONE
    echo "--> [PREPARE] Forcing zstd compression on kernel modules...."
    ./scripts/config --file .config --enable CONFIG_MODULE_COMPRESS
    ./scripts/config --file .config --enable CONFIG_MODULE_COMPRESS_ZSTD
    ./scripts/config --file .config --disable CONFIG_MODULE_COMPRESS_NONE
    echo "--> [PREPARE] Hardcoding all core drivers into Kernel...."
    ./scripts/config --file .config --enable CONFIG_BLK_DEV_SR
    ./scripts/config --file .config --enable CONFIG_CHR_DEV_SG
    ./scripts/config --file .config --enable CONFIG_BLK_DEV_LOOP
    ./scripts/config --file .config --enable CONFIG_BLK_DEV_NVME
    ./scripts/config --file .config --enable CONFIG_ATA
    ./scripts/config --file .config --enable CONFIG_SATA_AHCI
    ./scripts/config --file .config --enable CONFIG_USB
    ./scripts/config --file .config --enable CONFIG_USB_SUPPORT
    ./scripts/config --file .config --enable CONFIG_USB_XHCI_HCD
    ./scripts/config --file .config --enable CONFIG_USB_EHCI_HCD
    ./scripts/config --file .config --enable CONFIG_USB_OHCI_HCD
    ./scripts/config --file .config --enable CONFIG_USB_STORAGE
    ./scripts/config --file .config --enable CONFIG_USB_UAS
    ./scripts/config --file .config --enable CONFIG_ISO9660_FS
    ./scripts/config --file .config --enable CONFIG_JOLIET
    ./scripts/config --file .config --enable CONFIG_ZISOFS
    ./scripts/config --file .config --enable CONFIG_SQUASHFS
    ./scripts/config --file .config --enable CONFIG_SQUASHFS_ZSTD
    ./scripts/config --file .config --enable CONFIG_OVERLAY_FS
    ./scripts/config --file .config --enable CONFIG_FAT_FS
    ./scripts/config --file .config --enable CONFIG_VFAT_FS
    ./scripts/config --file .config --enable CONFIG_EXT4_FS
    make olddefconfig
    make clean
}

# lkmake calls this automatically right after prepare() and uses its
# output as the final pkgver, this is the same idea as package()'s local
# _kernver: ask the kernel itself for its real X.Y.Z release instead of
# hardcoding/guessing it. Needs to run after prepare() because CONFIG_LOCALVERSION
# (the "-liska" suffix) only exists in .config once prepare() has written it.
# The suffix is stripped because pkgver format disallows "-".
pkgver() {
    cd "${srcdir}/linux-${_kernelver}"
    make -s kernelrelease | sed 's/-.*//'
}

build() {
    cd "${srcdir}/linux-${_kernelver}"
    make -j$(nproc) all
}

package() {
    cd "${srcdir}/linux-${_kernelver}"
    # kernel.org drops the trailing ".0" sublevel from the tarball name for
    # X.Y.0 releases (e.g. linux-7.2.tar.xz), but KERNELRELEASE always keeps
    # all three numbers (7.2.0-liska). Neither pkgver nor _kernelver reliably
    # matches the module directory `make modules_install` actually creates,
    # so ask the kernel itself for its real release string instead of
    # gluing version variables together manually.
    local _kernver="$(make -s kernelrelease)"
    echo "--> [PACKAGE] Installing kernel modules for ${_kernver}...."
    make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
    if [ ! -d "${pkgdir}/lib" ]; then
        ln -s usr/lib "${pkgdir}/lib"
    fi
    echo "--> [PACKAGE] Installing kernel image as /boot/vmlinuz-linux...."
    install -Dm644 "$(make -s image_name)" "${pkgdir}/boot/vmlinuz-linux"
    install -Dm644 COPYING "${pkgdir}/usr/share/licenses/${pkgname}/GPL2.txt"
    echo "--> [PACKAGE] Populating kernel headers infrastructure...."
    local builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"
    rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build"
    rm -f "${pkgdir}/usr/lib/modules/${_kernver}/source"
    mkdir -p "${builddir}"
    install -Dt "${builddir}" -m644 .config Makefile Module.symvers
    install -Dt "${builddir}/kernel" -m644 kernel/Makefile
    cp -t "${builddir}" -a include scripts
    echo "--> [PACKAGE] Sweeping build residues out of headers...."
    find "${builddir}" -name "*.o" -type f -delete
    find "${builddir}" -name "*.log" -type f -delete
    find "${builddir}" -name "*.a" -type f -delete
    echo "--> [PACKAGE] Compressing remaining uncompressed modules with zstd...."
    find "${pkgdir}/usr/lib/modules/" -type f -name "*.ko" -exec zstd -19 --rm -f {} + 2>/dev/null || true
    rm -f "${pkgdir}/lib"
}
