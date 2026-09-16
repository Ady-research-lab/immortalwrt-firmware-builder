# 钛星人 M3 ImmortalWrt 云编译

[English](README.md) | 简体中文

这是一个面向钛星人 M3 的实验性 ImmortalWrt 构建项目。它使用 GitHub
Actions 检出官方 immortalwrt/immortalwrt 源码，默认固定到
v24.10.6，无需在本地准备完整编译环境。

项目以当前已验证可运行的斐讯 K2P 配置为基础，只做与 M3 相关的必要改动：

- 保留 phicomm,k2p 设备名称和现有升级兼容性。
- 保留已在实机正常工作的 K2P Flash 分区、网口、无线校准和 LED 配置。
- 启用 MT7621 原生 SDHCI 控制器，以支持 M3 的 TF 卡槽。
- 预装 kmod-mmc-mtk、block-mount、EXT4、FAT32 和 UTF-8 支持。
- 将 K2P 标记为可迁移来源，使现有 K2P 固件可以先用 sysupgrade -T
  检查 M3 镜像兼容性。

> [!WARNING]
> 这是根据实机运行状态和 MT7621 同类设备推导出的首个测试版本，不是
> ImmortalWrt 官方支持机型。首次刷写前必须备份 Flash，并确认可以通过
> Breed 或串口恢复。若 sysupgrade -T 检查失败，不要使用 -F 强刷。

## GitHub Actions 使用方法

1. 将本仓库推送到你自己的 GitHub 仓库。
2. 打开仓库的 **Actions** 页面。
3. 选择 **Build Taixingren M3 firmware**。
4. 点击 **Run workflow**。
5. 保持 immortalwrt_ref 为 v24.10.6。
6. 如需额外预装包，在 extra_packages 中输入以空格分隔的包名。
7. 构建完成后，从该次运行的 **Artifacts** 下载固件。

默认会生成并上传：

- phicomm_k2p 的 squashfs-sysupgrade.bin
- manifest 和构建信息
- 实际展开后的差异配置
- SHA-256 校验文件

只有主动勾选 publish_release 时，工作流才会额外创建 GitHub Release。

## 软件包组成

ImmortalWrt 会自动加入完整路由器默认包，包括基础系统、netifd、
dnsmasq-full、firewall4、nftables、Dropbear SSH、LuCI、PPPoE、IPv6 和
wpad-openssl。K2P profile 继续包含现有固件使用的 kmod-mt7615-firmware。

首版另外加入识别和挂载 TF 卡所需组件：

~~~text
kmod-mmc-mtk
block-mount
kmod-fs-ext4
kmod-fs-vfat
kmod-nls-utf8
~~~

kmod-mmc-mtk 会自动带入 MMC 核心模块。不要同时加入
kmod-sdhci-mt7620，两套驱动在 ImmortalWrt 中声明为冲突。

固件还预装 OpenClash 和 luci-app-nikki 所需的内核模块：

~~~text
kmod-inet-diag
kmod-nft-socket
kmod-nft-tproxy
kmod-tun
kmod-dummy
~~~

kmod-nft-socket 和 kmod-nft-tproxy 会自动带入匹配同一次构建内核 ABI 的
kmod-nf-socket、kmod-nf-tproxy 等依赖。代理程序、规则和数据库仍建议安装
到 TF 卡 Extroot，避免占用内部 Flash。

可按需通过 extra_packages 添加维护工具：

~~~text
e2fsprogs dosfstools fdisk
~~~

luci-app-diskman 也可以加入，但依赖较多；M3 只有 16MB Flash，建议先
完成 TF 卡识别测试，再决定是否加入。

## 仓库结构

~~~text
.github/workflows/build-firmware.yml
taixingren-m3/
├── seed.config
└── patches/
    └── 0001-enable-tf-card-for-k2p-profile.patch
~~~

- seed.config：目标机型及额外预装软件包清单。
- patches/*.patch：修改 K2P 设备树与 profile，启用 SDHCI 并加入驱动。
- Actions 工作流：检出官方源码、应用上述改动、编译并上传固件。

## 刷写前检查

在现有固件中先备份至少以下 MTD 分区：

~~~sh
dd if=/dev/mtd0 of=/tmp/mtd0-u-boot.bin
dd if=/dev/mtd1 of=/tmp/mtd1-u-boot-env.bin
dd if=/dev/mtd2 of=/tmp/mtd2-factory.bin
dd if=/dev/mtd3 of=/tmp/mtd3-permanent_config.bin
~~~

备份文件必须复制到路由器之外保存。上传新镜像后先检查：

~~~sh
sysupgrade -T /tmp/immortalwrt-*-phicomm_k2p-squashfs-sysupgrade.bin
~~~

只有返回兼容且已确认恢复手段后，才进入实际刷写测试。

## 来源

工作流结构基于
[coachpo/immortalwrt-firmware-builder](https://github.com/coachpo/immortalwrt-firmware-builder)，
固件源码来自
[immortalwrt/immortalwrt](https://github.com/immortalwrt/immortalwrt)。

## 许可证

构建项目使用 [MIT License](LICENSE)。ImmortalWrt 及其设备树、内核和软件包
分别遵循各自的上游许可证。
