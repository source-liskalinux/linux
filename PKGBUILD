pkgbase=linux
pkgname=('linux' 'linux-headers')
_kernel="-liska"
_hostname="liskalinux"
pkgver=7.1.4
pkgrel=1
arch=('x86_64')
url="https://www.kernel.org"
license=('GPL-2.0-only')
makedepends=('bc' 'libelf' 'pahole' 'resolvconf' 'systemd-tools' 'kmod' 'inetutils' 'xmlto' 'docbook-xsl' 'kconfig' 'zstd')
options=('strip' '!debug')
source=(
  "https://cdn.kernel.org/pub/linux/kernel/v7.x/linux-${pkgver}.tar.xz"
  "https://gitlab.archlinux.org/archlinux/packaging/packages/linux/-/raw/main/config.x86_64"
)
sha256sums=('SKIP' 'SKIP')

prepare() {
  cd "linux-${pkgver}"
  echo "===> [LOG]: Setting up base configuration...."
  cp ../config.x86_64 .config
  echo "===> [LOG]: Injecting Liska Linux kernel identity...."
  ./scripts/config --file .config --set-str CONFIG_LOCALVERSION "${_kernel}"
  ./scripts/config --file .config --set-str CONFIG_DEFAULT_HOSTNAME "${_hostname}"
  echo "===> [LOG]: Initializing total purge of all debug info...."
  ./scripts/config --file .config --disable CONFIG_DEBUG_INFO
  ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_BTF
  ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT
  ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF4
  ./scripts/config --file .config --disable CONFIG_DEBUG_INFO_DWARF5
  ./scripts/config --file .config --enable CONFIG_DEBUG_INFO_NONE
  echo "===> [LOG]: Forcing ZSTD compression on kernel modules...."
  ./scripts/config --file .config --enable CONFIG_MODULE_COMPRESS
  ./scripts/config --file .config --enable CONFIG_MODULE_COMPRESS_ZSTD
  ./scripts/config --file .config --disable CONFIG_MODULE_COMPRESS_NONE
  echo "===> [LOG]: Hardcoding all core storage, USB, network, and filesystem drivers into Kernel...."
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
  ./scripts/config --file .config --enable CONFIG_NETDEVICES
  ./scripts/config --file .config --enable CONFIG_ETHERNET
  ./scripts/config --file .config --enable CONFIG_NET_CORE
  ./scripts/config --file .config --enable CONFIG_R8169
  ./scripts/config --file .config --enable CONFIG_E1000E
  ./scripts/config --file .config --enable CONFIG_IGC
  ./scripts/config --file .config --enable CONFIG_TIGON3
  ./scripts/config --file .config --enable CONFIG_ATL1E
  ./scripts/config --file .config --enable CONFIG_ALX
  ./scripts/config --file .config --enable CONFIG_ISO9660_FS
  ./scripts/config --file .config --enable CONFIG_JOLIET
  ./scripts/config --file .config --enable CONFIG_ZISOFS
  ./scripts/config --file .config --enable CONFIG_SQUASHFS
  ./scripts/config --file .config --enable CONFIG_SQUASHFS_ZSTD
  ./scripts/config --file .config --enable CONFIG_OVERLAY_FS
  ./scripts/config --file .config --enable CONFIG_FAT_FS
  ./scripts/config --file .config --enable CONFIG_VFAT_FS
  ./scripts/config --file .config --enable CONFIG_EXT4_FS
  echo "===> [LOG]: Resolving dependencies and finalizing .config...."
  make olddefconfig
  make clean
}

build() {
  cd "linux-${pkgver}"
  make -j$(nproc) all
}

package_linux() {
  pkgdesc="Liska Linux Main Kernel"
  depends=('coreutils' 'kmod')
  optdepends=('linux-firmware')
  provides=("linux=${pkgver}")
  cd "linux-${pkgver}"
  echo "===> [INFO]: Installing kernel modules into package directory...."
  make INSTALL_MOD_PATH="${pkgdir}" modules_install
  rm -f "${pkgdir}"/lib/modules/${pkgver}${_kernel}/source
  rm -f "${pkgdir}"/lib/modules/${pkgver}${_kernel}/build
  if [ -d "${pkgdir}/lib/modules/" ]; then
     find "${pkgdir}/lib/modules/" -name "*.log" -type f -delete
  fi
  echo "===> [INFO]: Installing kernel image..."
  install -Dm644 "$(make -s image_name)" "${pkgdir}/boot/vmlinuz-${pkgbase}"
  install -Dm644 COPYING "${pkgdir}/usr/share/licenses/linux/GPL2.txt"
  rm -f "${pkgdir}/boot/vmlinux" 2>/dev/null || true
  echo "===> [INFO]: Compressing remaining uncompressed modules with ZSTD...."
  find "${pkgdir}/lib/modules/" -type f -name "*.ko" -exec zstd -19 --rm -f {} + 2>/dev/null || true
}			

package_linux-headers() {
  pkgdesc="Liska Linux Headers Kernel"
  provides=("linux-headers=${pkgver}")  
  cd "linux-${pkgver}"
  echo "===> [INFO]: Populating kernel headers infrastructure...."
  install -Dt "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build" -m644 .config Makefile Module.symvers
  install -Dt "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build/kernel" -m644 kernel/Makefile
  cp -t "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build" -a include scripts
  echo "===> [INFO]: Sweeping build residues out of headers...."
  find "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build" -name "*.o" -type f -delete
  find "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build" -name "*.log" -type f -delete
  find "${pkgdir}/usr/lib/modules/${pkgver}${_kernel}/build" -name "*.a" -type f -delete
}
