# m2468 rootfs（postmarketOS）

魅族 21 Note 上跑的那套系统的清单和配置。

系统本体是 **postmarketOS v26.06**（Alpine 系，apk 包管理）+ phosh 桌面，
内核换成了我们自己的 mainline 7.3.0。

三个仓库的分工：

- [m2468-mainline](https://github.com/leluo123/m2468-mainline) — 内核改动、踩坑记录、实验证据
- [m2468-uefi](https://github.com/leluo123/m2468-uefi) — UEFI 引导（Mu）
- [m2468-system](https://github.com/leluo123/m2468-system) — systemd 单元、udev 规则、基带 daemon
- **本仓库** — 系统里装了哪些包

## packages.txt

`packages.txt` 是设备上 `apk info -v` 的完整输出，1447 个包，带版本号。照这份清单装出来的
系统和我们跑的基本一致。

里面跟这个移植直接相关的：

```
greetd / greetd-phrog         登录和会话管理
phosh / phoc                  GNOME 移动桌面
wlroots                       phoc 的底层
mesa / freedreno              GPU
seatd                         seat 管理
qrtr / libqmi / libqrtr-glib  modem 通信
rmtfs / tqftpserv             EFS 和固件服务
pipewire                      音频
```

## 装完之后还要做什么

包只是基础。让系统真正能用的是 [m2468-system](https://github.com/leluo123/m2468-system)
里那些配置和脚本：

- `seatd`、`fix-shm-perm`、`m2468-cpufreq`、`m2468-charge-limit` 四个服务
- `m2468-data` 和 `m2468-rf-keeper` 两个基带 daemon
- `99-m2468-charge.rules` 这条 udev 规则
- `phoc.ini` 的屏幕参数（mode 必须带 `Hz` 后缀）
- NetworkManager 的 connectivity 检测换成国内的 204 端点

照着那边的 README 放回对应路径，`systemctl daemon-reload` 之后启用就行。

## 内核

**不要用 pmOS 自带的内核。** 那个内核没有 m2468 的面板驱动、触摸（QUP-GPI）支持
和基带相关改动，装上去屏幕不亮、触摸没反应。

要么自己编（主仓库的 `scripts/build_cxfloor.sh`），要么直接下
[Release 里的现成镜像](https://github.com/leluo123/m2468-mainline/releases)。

## 引导链

```
ABL → boot_a/boot_b 里的 Mu-m2468.img（UEFI）
      → ESP 里的 EFI/BOOT/BOOTAA64.EFI（EFI zboot + zstd 内核）
      → initramfs
      → rootfs（这份清单装出来的系统）
```

内核启动时会打印 `rootfs FOUND on /dev/sdaXX`，找不到就停在 initramfs。

## 现成的镜像

不想自己装的话，[Releases](https://github.com/leluo123/m2468-rootfs/releases) 里有打好的
`m2468-rootfs-postmarketOS-20261106.tar.zst`（1.87 GB，解开 4.90 GB）。

已经清理过：machine-id 删除、root 锁定、`user` 密码重置为 `147147`、WiFi 配置和 SSH key 全部剔除。

用法见 Release 说明。**注意解压要带 `-p` 保留权限**，不然 sudo 会坏。

## 为什么不放整盘镜像

这台机器的 rootfs 分区是 449 GB（只用了 5 GB 左右），整盘 dump 没意义。
上面那个 Release 是**文件系统层面的 tar 包**，解到你自己的根分区就行，跟分区多大无关。

## 致谢

postmarketOS 把整套打包和启动流程都做好了，我们只是在这上面加了自己机型的部分。
还有 [Phosh](https://gitlab.gnome.org/World/Phosh/phosh)、
[wlroots](https://github.com/swaywm/wlroots)、
[linux-msm](https://github.com/linux-msm) 那些项目，见主仓库的 CREDITS.md。
