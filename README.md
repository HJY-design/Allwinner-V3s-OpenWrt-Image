# Allwinner-V3s-OpenWrt-Image
全志 V3s开发板硬件设计与 OpenWrt 系统集成方案，包含 SDIO Wi-Fi 驱动加载与固件修复指南。 / Hardware design and OpenWrt integration for Allwinner V3s with RTL8723BS module.
本项目适用于https://oshwhub.com/a-ordinary-people/project_lywsnqgn 的开源设计
## 📦 镜像下载与使用

请前往本仓库的 [Releases 页面](../../releases) 下载最新编译的系统镜像压缩包。

* **默认 IP**: `192.168.1.1` *(请根据实际情况修改)*
* **默认账号**: `root`
* **默认密码**: *(为空)*
* **特性**: 
  * 大部分外设能够正常工作
  * 需要手动拉取旧版RTL8723BS系统固件,openwrt内部闭源固件不可用。

系统镜像的使用:在压缩包里有2个文件一个是系统img镜像,另一个是设备树文件。你需要使用rufus这样的磁盘烧写工具将系统镜像烧写到SD卡里,接着点开SD卡,一般有2个分区。点击的系统分区空间小的那一个,只显示一个磁盘的,就点击那个磁盘就行。接着你会在磁盘空间里看到3个文件其中有一个.dtb文件,将压缩包里的那个.dtb文件拖入空间,将它命名为原来在空间的那个.dtb文件名称,删除原来的dtb文件。就可以退出了，将SD卡插入卡槽接上串口就可以，正式进入系统了。


## 🐛 深度排障记录 (Troubleshooting)

在调试该组合时，系统日志极其容易出现以下几种致命报错。以下是详细的错误成因与物理层面的解决办法。

### 坑位一：SDIO 初始化超时 `error -110`

**现象**：
开机初期日志出现：`mmc1: error -110 whilst initialising SDIO card`。

**推敲与解决**：
首先需要你手动打上旧版的RTL8723BS的驱动，当你成功刷写完成系统后，请借助完好的有线以太网登录上网络，你需要配置好DHCH和DNS服务器，这样就能正常访问公网。接着在终端下执行以下命令：
# 1.进入指定目录
cd /lib/firmware/rtlwifi/
rm -f rtl8723bs_nic.bin

# 2. 从 Linux Kernel 官方库拉取早期的固定 Commit 版本（绝对官方的早期版本）
wget --no-check-certificate -O rtl8723bs_nic.bin "https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtlwifi/rtl8723bs_nic.bin?id=e6b9001e91110c654573b8f8e2db6155d10d3b57"

# 3. 校验大小 (正常应为 14.2KB / 14216 字节左右)
ls -lh rtl8723bs_nic.bin

校验完成后,需要重新加载驱动,执行以下命令:
rmmod rtl8723bs
modprobe rtl8723bs
dmesg | tail -n 20
若执行完以上的操作依旧无法识别网卡,除了常规的硬件虚焊，这通常是由**热启动 (Warm Boot) 陷阱**引发的。当执行 `reboot` 时，虽然 V3s 芯片复位，但 3.3V 主电源未断开，RTL8723BS 内部状态机可能处于卡死状态，导致对 SDIO 命令无响应。
 **对策**：彻底断电（拔掉电源线），等待电容放电后重新上电（Cold Boot）。若冷启动后恢复为 `new high speed SDIO card`，则硬件正常。

目前关于系统配置就写这么多，关于我如何详细的配置系统，后期再慢慢补充。
感谢你能看到这里。
---
