### 构建选项
# 将接下来的两个变量设置为非空的任意值，以启用它们

# 通过 nconfig 在构建前调整内核选项
: "${_makenconfig:=""}"

# 仅编译活动模块，以便大大减少构建的模块数和构建时间。

# 为了跟踪您的特定系统/硬件所需的模块，尝试使用 module_db：https://aur.archlinux.org/packages/modprobed-db 
# 如果存在，则此 PKGBUILD 会读取保存的数据库
#
# 此wiki页面上的更多信息 ---> https://wiki.archlinux.org/index.php/Modprobed-db
: "${_localmodcfg:=""}"

# 可选择按编号选择一个子架构，或者留空，在构建期间需要用户交互。注意，通用（默认）选项是 40。
#
#  1. AMD Opteron/Athlon64/Hammer/K8 (MK8)
#  2. AMD Opteron/Athlon64/Hammer/K8 with SSE3 (MK8SSE3)
#  3. AMD 61xx/7x50/PhenomX3/X4/II/K10 (MK10)
#  4. AMD Barcelona (MBARCELONA)
#  5. AMD Bobcat (MBOBCAT)
#  6. AMD Jaguar (MJAGUAR)
#  7. AMD Bulldozer (MBULLDOZER)
#  8. AMD Piledriver (MPILEDRIVER)
#  9. AMD Steamroller (MSTEAMROLLER)
#  10. AMD Excavator (MEXCAVATOR)
#  11. AMD Zen (MZEN)
#  12. AMD Zen 2 (MZEN2)
#  13. AMD Zen 3 (MZEN3)
#  14. AMD Zen 4 (MZEN4)
#  15. Intel P4 / older Netburst based Xeon (MPSC)
#  16. Intel Core 2 (MCORE2)
#  17. Intel Atom (MATOM)
#  18. Intel Nehalem (MNEHALEM)
#  19. Intel Westmere (MWESTMERE)
#  20. Intel Silvermont (MSILVERMONT)
#  21. Intel Goldmont (MGOLDMONT)
#  22. Intel Goldmont Plus (MGOLDMONTPLUS)
#  23. Intel Sandy Bridge (MSANDYBRIDGE)
#  24. Intel Ivy Bridge (MIVYBRIDGE)
#  25. Intel Haswell (MHASWELL)
#  26. Intel Broadwell (MBROADWELL)
#  27. Intel Skylake (MSKYLAKE)
#  28. Intel Skylake X (MSKYLAKEX)
#  29. Intel Cannon Lake (MCANNONLAKE)
#  30. Intel Ice Lake (MICELAKE)
#  31. Intel Cascade Lake (MCASCADELAKE)
#  32. Intel Cooper Lake (MCOOPERLAKE)
#  33. Intel Tiger Lake (MTIGERLAKE)
#  34. Intel Sapphire Rapids (MSAPPHIRERAPIDS)
#  35. Intel Rocket Lake (MROCKETLAKE)
#  36. Intel Alder Lake (MALDERLAKE)
#  37. Intel Raptor Lake (MRAPTORLAKE)
#  38. Intel Meteor Lake (MMETEORLAKE)
#  39. Intel Emerald Rapids (MEMERALDRAPIDS)
#  40. Generic-x86-64 (GENERIC_CPU)
#  41. Generic-x86-64-v2 (GENERIC_CPU2)
#  42. Generic-x86-64-v3 (GENERIC_CPU3)
#  43. Generic-x86-64-v4 (GENERIC_CPU4)
#  44. Intel-Native optimizations autodetected by the compiler (MNATIVE_INTEL)
#  45. AMD-Native optimizations autodetected by the compiler (MNATIVE_AMD)
: "${_subarch:=""}"

# 使用当前内核的 .config 文件 启用此选项将使用正在运行的内核的 .config，而不是 ARCH 的默认值。
# 当软件包得到更新而你已经经历了定制配置选项的麻烦时，此选项很有用。
# 当新的内核发布时不推荐使用，但对于软件包的更新依然很方便。
: "${_use_current:=""}"

# 启用 LLVM 编译
: "${_use_llvm_lto:=""}"

# 启用/禁用调试选项
# 如果已在您的 .config 文件中启用调试选项，请将其设置为“y”以启用，设置为“n”以强制禁用，如果为空则忽略调试选项。
: "${_debug:=""}"

### 重要提示：除非您确切知道自己在做什么，否则请不要编辑以下内容。

_major=6.8
_minor=7
_srcname=linux-${_major}
_clr=${_major}.6-1426
_gcc_more_v='20240221.2'
pkgbase=linux-clear-sun
pkgver=${_major}.${_minor}
pkgrel=1
pkgdesc='Clear Linux'
arch=('x86_64')
url="https://github.com/clearlinux-pkgs/linux"
license=(GPL-2.0-only)
makedepends=('bc' 'cpio' 'git' 'libelf' 'pahole' 'xmlto')
if [ -n "$_use_llvm_lto" ]; then
  makedepends+=(clang llvm lld python)
fi
options=(!debug !strip)
if [ "$_debug" == "y" ]; then
    options=(!strip)
fi
source=(
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${_major}.tar.xz"
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${_major}.tar.sign"
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/patch-${pkgver}.xz"
  "$pkgbase::git+https://github.com/clearlinux-pkgs/linux.git#tag=${_clr}"
  "more-uarches-$_gcc_more_v.tar.gz::https://github.com/graysky2/kernel_compiler_patch/archive/$_gcc_more_v.tar.gz"
  "git+https://github.com/openzfs/zfs.git#branch=zfs-2.2-release"
)

if [ -n "$_use_llvm_lto" ]; then
  BUILD_FLAGS=(
    LLVM=1
    LLVM_IAS=1
  )
fi

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
    # 这个函数负责准备构建环境

    cd ${_srcname}

    ### 添加上游补丁
    echo "Add upstream patches"
    patch -Np1 -i ../patch-${pkgver}

    ### 设置版本
    echo "Setting version..."
    echo "-$pkgrel" > localversion.10-pkgrel
    echo "${pkgbase#linux}" > localversion.20-pkgname

    ### 添加 Clearlinux 修补程序
    for i in $(grep '^Patch' ${srcdir}/$pkgbase/linux.spec |\
     grep -Ev '^Patch0132|^Patch0109|^Patch0118|^Patch0113|^Patch0138|^Patch0139|^Patch0147' | sed -n 's/.*: //p'); do
        if [ -n "$_use_llvm_lto" ]; then
            if [ "${i}" == "0162-extra-optmization-flags.patch" ] ; then
                continue
            fi
        fi
        echo "Applying patch ${i}..."
        patch -Np1 -i "$srcdir/$pkgbase/${i}"
    done

    ### 添加zfs补丁
    cd ${srcdir}/"zfs"
    ./autogen.sh
    ./configure CC=gcc --prefix=/usr --sysconfdir=/etc --sbindir=/usr/bin --libdir=/usr/lib \
            --datadir=/usr/share --includedir=/usr/include --with-udevdir=/lib/udev \
            --libexecdir=/usr/lib/zfs --with-config=kernel \
            --enable-linux-builtin \
            --with-linux=${srcdir}/$_srcname
    ./copy-builtin ${srcdir}/${_srcname}

    ### 开启zfs选项
    cd ${srcdir}/${_srcname}
    scripts/config -e CONFIG_ZFS

    ### 设置配置
    echo "Setting config..."
    cp -Tf $srcdir/$pkgbase/config ./.config

    ### 启用额外选项
    echo "Enable extra options..."

    # 常规设置
    scripts/config --set-str DEFAULT_HOSTNAME archlinux \
                   -e IKCONFIG \
                   -e IKCONFIG_PROC \
                   -u RT_GROUP_SCHED

    # 电源管理和 ACPI 选项
    scripts/config -e ACPI_REV_OVERRIDE_POSSIBLE \
                   -e ACPI_TABLE_UPGRADE

    # 虚拟化
    scripts/config -e KVM_SMM

    # 取决于架构的一般选项
    scripts/config -e KPROBES

    # 启用可加载模块支持
    scripts/config -u MODULE_SIG_FORCE \
                   -e MODULE_COMPRESS_ZSTD

    # 网络支持
    scripts/config -e NETFILTER_INGRESS

    # 设备驱动程序
    scripts/config -e FRAMEBUFFER_CONSOLE_DEFERRED_TAKEOVER \
                   -e DELL_SMBIOS_SMM \
                   -m PATA_JMICRON \
                   -E SOUND SOUND_OSS_CORE \
                   -e SND_OSSEMUL \
                   -M SND_OSSEMUL SND_MIXER_OSS \
                   -M SND_MIXER_OSS SND_PCM_OSS \
                   -E SND_PCM_OSS SND_PCM_OSS_PLUGINS \
                   -m AGP -M AGP AGP_INTEL -M AGP_INTEL AGP_VIA

    # 内核调试 -> 编译时检查和编译器选项 -> 使节 mismatch 错误变为非致命
    scripts/config -e SECTION_MISMATCH_WARN_ONLY

    # 文件系统
    scripts/config -m NTFS3_FS \
                   -e NTFS3_LZX_XPRESS \
                   -e NTFS3_FS_POSIX_ACL

    scripts/config -m SMB_SERVER \
                   -e SMB_SERVER_SMBDIRECT \
                   -e SMB_SERVER_CHECK_CAP_NET_ADMIN \
                   -e SMB_SERVER_KERBEROS5

    # 安全选项
    scripts/config -e SECURITY_SELINUX \
                   -e SECURITY_SELINUX_BOOTPARAM \
                   -e SECURITY_SMACK \
                   -e SECURITY_SMACK_BRINGUP \
                   -e SECURITY_SMACK_NETFILTER \
                   -e SECURITY_SMACK_APPEND_SIGNALS \
                   -e SECURITY_TOMOYO \
                   -e SECURITY_APPARMOR \
                   -e SECURITY_YAMA

    # 库子程序
    scripts/config -k -e FONT_TER16x32

    if [ -n "$_use_llvm_lto" ]; then
        scripts/config -d LTO_NONE \
                       -e LTO \
                       -e LTO_CLANG \
                       -e ARCH_SUPPORTS_LTO_CLANG \
                       -e ARCH_SUPPORTS_LTO_CLANG_THIN \
                       -e HAS_LTO_CLANG \
                       -e LTO_CLANG_THIN \
                       -e HAVE_GCC_PLUGINS
    fi

    if [ "$_debug" == "y" ]; then
        scripts/config -e DEBUG_INFO \
                       -e DEBUG_INFO_BTF \
                       -e DEBUG_INFO_DWARF4 \
                       -e PAHOLE_HAS_SPLIT_BTF \
                       -e DEBUG_INFO_BTF_MODULES
    elif [ "$_debug" == "n" ]; then
        scripts/config -d DEBUG_INFO \
                       -d DEBUG_INFO_BTF \
                       -d DEBUG_INFO_DWARF4 \
                       -d PAHOLE_HAS_SPLIT_BTF \
                       -d DEBUG_INFO_BTF_MODULES
    fi

    make ${BUILD_FLAGS[*]} olddefconfig
    diff -u $srcdir/$pkgbase/config .config || :

    # https://github.com/graysky2/kernel_compiler_patch
    # 确保在 olddefconfig 之后应用，以便下一部分
    echo "Patching to enable GCC optimization for other uarchs..."
    patch -Np1 -i "$srcdir/kernel_compiler_patch-$_gcc_more_v/more-uarches-for-kernel-6.8-rc4+.patch"

    if [ -n "$_subarch" ]; then
        # 用户想要一个子级，因此通过 "是 "交互式应用上面定义的选择
        yes "$_subarch" | make ${BUILD_FLAGS[*]} oldconfig
    else
        # 未定义 subarch，因此允许用户选择一个
        make ${BUILD_FLAGS[*]} oldconfig
    fi

    ### 可选地使用运行内核的配置
    # 最初由 nous 编写的代码; http://aur.archlinux.org/packages.php?ID=40191
    if [ -n "$_use_current" ]; then
        if [[ -s /proc/config.gz ]]; then
            echo "Extracting config from /proc/config.gz..."
            # modprobe 配置
            zcat /proc/config.gz > ./.config
        else
            warning "Your kernel was not compiled with IKCONFIG_PROC!"
            warning "You cannot read the current config!"
            warning "Aborting!"
            exit
        fi
    fi

    ### 为 make localmodconfig 加载所需的模块
    # See https://aur.archlinux.org/packages/modprobed-db
    if [ -n "$_localmodcfg" ]; then
        if [ -e $HOME/.config/modprobed.db ]; then
            echo "Running Steven Rostedt's make localmodconfig now"
            make ${BUILD_FLAGS[*]} LSMOD=$HOME/.config/modprobed.db localmodconfig
        else
            echo "No modprobed.db data found"
            exit
        fi
    fi

    make -s kernelrelease > version
    echo "Prepared $pkgbase version $(<version)"

    [[ -z "$_makenconfig" ]] || make ${BUILD_FLAGS[*]} nconfig

    ### 保存配置以供以后重复使用
    cp -Tf ./.config "${startdir}/config-${pkgver}-${pkgrel}${pkgbase#linux}"
}

build() {
    # 这个函数负责构建内核

    cd ${_srcname}
    make ${BUILD_FLAGS[*]} all
}

_package() {
    # 这个函数负责打包内核

    pkgdesc="The $pkgdesc kernel and modules"
    depends=('coreutils' 'kmod' 'initramfs')
    optdepends=('wireless-regdb: to set the correct wireless channels of your country'
                'linux-firmware: firmware images needed for some devices'
                'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
    provides=(VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE KSMBD-MODULE)
    install=linux.install

    cd $_srcname

    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    echo "Installing boot image..."
    # systemd期望这里找到内核以便允许休眠
    # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
    install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

    # mkinitcpio 用来命名内核
    echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

    echo "Installing modules..."
    ZSTD_CLEVEL=19 make ${BUILD_FLAGS[*]} INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
        DEPMOD=/doesnt/exist modules_install  # Suppress depmod

    # remove build link
    rm "$modulesdir"/build
}

_package-headers() {
    # 这个函数负责打包内核头文件

    pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
    depends=(pahole)

    cd ${_srcname}
    local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

    echo "Installing build files..."
    install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
        localversion.* version vmlinux
    install -Dt "$builddir/kernel" -m644 kernel/Makefile
    install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
    cp -t "$builddir" -a scripts

    # 启用 STACK_VALIDATION 时需要
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool

    # 启用 DEBUG_INFO_BTF_MODULES 时必须使用
    if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
        install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
    fi

    echo "Installing headers..."
    cp -t "$builddir" -a include
    cp -t "$builddir/arch/x86" -a arch/x86/include
    install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

    install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
    install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

    # https://bugs.archlinux.org/task/13146
    install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

    # https://bugs.archlinux.org/task/20402
    install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
    install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

    # https://bugs.archlinux.org/task/71392
    install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

    echo "Installing KConfig files..."
    find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

    echo "Removing unneeded architectures..."
    local arch
    for arch in "$builddir"/arch/*/; do
        [[ $arch = */x86/ ]] && continue
        echo "Removing $(basename "$arch")"
        rm -r "$arch"
    done

    echo "Removing documentation..."
    rm -r "$builddir/Documentation"

    echo "Removing broken symlinks..."
    find -L "$builddir" -type l -printf 'Removing %P\n' -delete

    echo "Removing loose objects..."
    find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

    echo "Stripping build tools..."
    local file
    while read -rd '' file; do
        case "$(file -Sib "$file")" in
            application/x-sharedlib\;*)      # Libraries (.so)
                strip -v $STRIP_SHARED "$file" ;;
            application/x-archive\;*)        # Libraries (.a)
                strip -v $STRIP_STATIC "$file" ;;
            application/x-executable\;*)     # Binaries
                strip -v $STRIP_BINARIES "$file" ;;
            application/x-pie-executable\;*) # Relocatable binaries
                strip -v $STRIP_SHARED "$file" ;;
        esac
    done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

    echo "Stripping vmlinux..."
    strip -v $STRIP_STATIC "$builddir/vmlinux"

    echo "Adding symlink..."
    mkdir -p "$pkgdir/usr/src"
    ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

sha256sums=('c969dea4e8bb6be991bbf7c010ba0e0a5643a3a8d8fb0a2aaa053406f1e965f3'
            'SKIP'
            '6ef359c3497d90b0a687d1dc2050d0c662f6a72224c8bf6122847002e145ce11'
            'SKIP'
            '1d3ac3e581cbc5108f882fcdc75d74f7f069654c71bad65febe5ba15a7a3a14f'
            'SKIP')

validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
