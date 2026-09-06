# gaokun3 / crDroid 16.0

仅维护设备、自有工具和打过补丁的 fork；其余源码由 crDroid 上游 manifest 管理，不锁定 commit。

- `00-remotes.xml`：声明 gaokun3 和 MindTheGapps 的远端及默认分支。
- `01-removes.xml`：移除需要替换的上游项目。
- `02-gaokun3.xml`：声明本组织的源码仓库，默认跟随 `16.0` 分支；内核只跟随 `linux-rolling-stable`，浅克隆。
- crDroid 未包含的 `device/linaro/dragonboard`（设备树引用其 sepolicy）和 `vendor/gapps`（原项目的 GApps）也在这里声明。前者继承 crDroid 的 AOSP 版本，后者跟随 `baklava`。

参考 [WayDroid-ATV 的拆分方式](https://github.com/WayDroid-ATV/android_vendor_waydroid/tree/lineage-23.2/manifest_scripts/manifests-36)。
`repo` 按文件名字母顺序自动加载这三个 XML：先声明远端，再移除上游项目，最后添加替换项目，无需 `include`。
所有仓库公开，无需 GitHub token。没有独立的设备 vendor tree；`vendor/lineage` 是 crDroid 自身的配置仓库。

## 获取源码

在新的源码目录执行：

```sh
repo init -u https://github.com/crdroidandroid/android.git -b 16.0 --depth=1
git clone -b 16.0 https://github.com/gaokun3/local_manifests.git .repo/local_manifests
repo sync -c --no-tags -j8
```

已有源码请先备份本地修改与旧 local manifest，仅启用本仓库这三个 XML，不要叠加原来的 `local_manifest_gaokun3.xml`。

## 构建

按设备树 `firmware/README.md` 与 `hexagonrpcd-root/README.md` 准备本地固件、传感器配置及 `adb_keys`。这些输入不进源码仓库。

```sh
bash tools/gaokun/build-kernel.sh "$PWD"
bash tools/gaokun/build-android.sh "$PWD"
```

默认并行数 16，可用 `JOBS=8` 调整。产物位于 `out/target/product/gaokun3/`。
GApps 的小型集成补丁由构建准备脚本应用；其他源码补丁已经提交在对应 fork 中。

## 上游与提交

设备项目来自 [vahiru/gaokun-android](https://github.com/vahiru/gaokun-android)。
内核 fork 自 [gregkh/linux](https://github.com/gregkh/linux/tree/linux-rolling-stable)。
Android 源码分别 fork 自 crDroid、LineageOS、linux-msm 和对应 AOSP GitHub 镜像，保留 GitHub 的 fork 关联。
导入补丁的提交正文记录故障原因、修复方式和来源；保留原作者，本次新增的适配单独署名。

`repo sync` 会跟随上述分支更新，不保证不同日期构建的镜像逐字节一致。编译通过也不代替实机启动、音频、GPU、待机验证。
