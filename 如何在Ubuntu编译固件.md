

### 一、部署编译环境

1. **更新系统**  
   确保你的系统软件包是最新的，并执行全面升级：
   
   ```bash
   sudo apt -y update
   sudo apt -y full-upgrade
   ```

2. **安装编译环境所需的工具和依赖**  
   安装OpenWRT编译所需的依赖包和工具：
   
   ```bash
   sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
   bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib \
   git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev \
   libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libreadline-dev libssl-dev libtool lrzsz \
   mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python3 python3-pyelftools \
   libpython3-dev qemu-utils rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip \
   vim wget xmlto xxd zlib1g-dev python3-setuptools
   ```

### 二、下载OpenWrt及其软件包

1. **下载官方OpenWrt源代码**  
   使用`git`命令下载OpenWrt源代码，并切换到23.05分支：
   
   ```bash
   git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
   cd openwrt
   ```

2. **更新OpenWrt的feeds**  
   使用以下命令更新所有feeds：
   
   ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

3. **下载防检测软件包**  
   下载UA2F、rkp-IPID等防检测相关的第三方软件包：
   
   ```bash
   git clone https://github.com/Zxilly/UA2F.git package/UA2F
   git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
   git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon
   git clone https://github.com/jerrykuku/luci-app-argon-config.git package/luci-app-argon-config
   git clone https://github.com/linkease/istore.git package/istore
   ```

4. **再次更新feeds**  
   下载完这些软件包后，执行以下命令更新feeds：
   
   ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

### 三、更改内核MD5标识

1. **创建并编写`vermagic`文件**  
   在OpenWrt根目录下创建`vermagic`文件，内容为官方23.05版的MD5标识。使用以下命令：
   
   ```bash
   echo "59d1431675acc6823a33c7eb2323daeb" > vermagic
   ```

2. **修改`kernel-defaults.mk`文件**  
   打开并修改`openwrt/include/kernel-defaults.mk`文件，替换默认MD5生成规则为使用自定义的`vermagic`文件：
   
   - 将下面的代码：
     
     ```makefile
     grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | mkhash md5 > $(LINUX_DIR)/.vermagic
     ```
   
   - 替换为：
     
     ```makefile
     # grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | mkhash md5 > $(LINUX_DIR)/.vermagic
     cp $(TOPDIR)/vermagic $(LINUX_DIR)/.vermagic
     ```

3. **修改`package/kernel/linux/Makefile`**  
   修改依赖`vermagic`文件的规则：
   
   - 找到并替换：
     
     ```makefile
     STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | mkhash md5)
     ```
   
   - 替换为：
     
     ```makefile
     # STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | mkhash md5)
     STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
     ```

### 四、配置OpenWrt固件

1. **进入菜单配置**  
   使用`make menuconfig`配置编译选项：
   
   ```bash
   make menuconfig
   ```
   
   - 设置硬件平台为`x86_64`:
     - `Target System`: x86
     - `Subtarget`: x86_64
     - `Target Profile`: Generic x86/64
   - 设置内存大小：
     - `Target Images`的两个值设置为`256`和`2048`。

2. **配置rkp-IPID和UA2F模块**：
   
   - 选择`Kernel modules -> Other modules -> kmod-rkp-ipid`。
   - 选择`Network -> Routing and Redirection -> ua2f`。

3. **配置iptables和防火墙模块**：
   
   - 在`Network -> Firewall`中选择以下模块：
     - `iptables-mod-filter`
     - `iptables-mod-ipopt`
     - `iptables-mod-u32`
   - 在`Kernel modules -> Netfilter Extensions`中确保选择：
     - `kmod-ipt-ipopt`
     - `kmod-ipt-u32`

4. **配置界面和其他模块**：
   
   - 选择`LuCI -> Applications -> luci-app-argon-config`和`luci-app-store`。
   - 选择`LuCI -> Themes -> luci-theme-argon`，设置管理界面主题。

### 五、编译OpenWrt

1. **下载依赖文件**  
   在编译前，下载所有编译所需的文件：
   
   ```bash
   make download -j$(nproc) V=cs
   ```

2. **开始编译**  
   使用多线程编译OpenWrt固件：
   
   ```bash
   make -j$(nproc) V=cs
   ```
