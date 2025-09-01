# YAAWG: Yet another AmneziaWG variation for OpenWrt

This project is aimed to update sources of the initial AmneziaWG and make them as close to four upstream projects ([luci-proto-wireguard](https://github.com/openwrt/luci/tree/master/protocols/luci-proto-wireguard), [amneziawg-tools](https://github.com/amnezia-vpn/amneziawg-tools/), [amneziawg-linux-kernel-module](https://github.com/amnezia-vpn/amneziawg-linux-kernel-module), [amneziawg-go](https://github.com/amnezia-vpn/amneziawg-go)) as possible.

Why? Because it looks like the original repository is abandoned: more than half a year has passed since the last commit and no bugs were fixed while two upstream projects receive updates on a regular basis (at least from the community).

The main differences and objectives:
1. `luci-proto-amneziawg` has been aligned in accordance with [luci-proto-wireguard](https://github.com/openwrt/luci/tree/master/protocols/luci-proto-wireguard):
   - AmneziaWG settings tab now have placeholders (default values, if chosen, will mimic classic Wireguard protocol).
   - Fixed bug with QR code generation. Please note that the generated QR code will contain AmneziaWG specific information (remove it manually to make it compatible with classic Wireguard).
   - Added checkboxes to enable/disable peers.
   - Took `luci-proto-wireguard` as the codebase.
   - Added the correct icon for the interface.
   - Added support for v1.5 protocol parameters: S3-S4, I1-I5, J1-J3, ITIME.
3. `amneziawg-tools` has been aligned  in accordance with the upstream repo [amneziawg-tools](https://github.com/amnezia-vpn/amneziawg-tools/):
   - The package is now compiled based on the upstream repo. `feature/awg-fix` branch has been chosen as a reference.
   - Fixed bug with non-existent `proto_amneziawg_check_installed` method.
   - Changed temp folders and files to match the protocol name.
   - Refactored scripts a bit to make them look more `amneziish`.
   - Fixed bug with incorrect path when using `amneziawg-go`.
   - Added support for v1.5 protocol parameters: S3-S4, I1-I5, J1-J3, ITIME.
4. `kmod-amneziawg` is now compiled totally based on the upstream [amneziawg-linux-kernel-module](https://github.com/amnezia-vpn/amneziawg-linux-kernel-module) repo. Master branch has been chosen as a reference.
5. `amneziawg-go` has been introduced in version `1.0.20250721` as a replacement for `kmod-amneziawg` because it seems that kernel module has been discontinued. Please check [this section](#kmod-amneziawg-vs-amneziawg-go) for more information. The Go implementation is also totally based on the upstream project [amneziawg-go](https://github.com/amnezia-vpn/amneziawg-go). `v0.2.14-beta-awg-1.5-1` tag has been chosen as a reference.

# `kmod-amneziawg` vs `amneziawg-go`
It looks like that kernel module (`kmod-amneziawg`) has been discontinued while the userspace implementation written in Go language (amneziawg-go) receives updates on a regular basis. Thus I decided to include the userspace implementation in the repo. It is now possible to choose:
1. Use `kmod-amneziawg`: it provides an older yet still working version of the protocol, but requires a less powerful device to run and consumes less space.
1. Use `amneziawg-go`: it provides a newer version of the protocol (v1.5), but requires a more powerful device to run and consumes more space.

Please choose and install only one implementation. If both implementations have been installed, `kmod-amneziawg` will be used by default.

# Results

Everything seems to work fine. No major problems have been detected or reported so far.

# How to build and use

Please note that this repository is primarily intended for compiling packages during the firmware build process. To do this follow the steps:
1. Clone the OpenWrt repo by running `git clone https://github.com/openwrt/openwrt.git` command.
2. Add line `src-git awgopenwrt https://github.com/this-username-has-been-taken/amneziawg-openwrt.git` to the `feeds.conf.default` file.
3. Update package feeds by running `{path to openwrt dir}/scripts/feeds update -a` command.
4. Install packages by running `{path to openwrt dir}/scripts/feeds install -a` command.
5. Choose target, settings, other packages and AmneziaWG packages (`amneziawg-go` or `kmod-amneziawg` + `amneziawg-tools` + `luci-proto-amneziawg`) in the menuconfig by running `make -C {path to openwrt dir} menuconfig` command and save configuration.
6. Make defconfig: `make -C {path to openwrt dir} defconfig`.
7. Build the firmware: `make -C openwrt -j$(nproc) V=sc`.

Nevertheless you can compile the packages without building the firmware.

Kudos to another fork and its Author (@defanator) for the build pipelines: https://github.com/defanator/amneziawg-openwrt
Please refer to the original manual as well.

Side note: the build process is not hard at all, but requires some attention and basic knowledge.

General steps:
1. Get parameters for your router:
     - OpenWRT version: `SNAPSHOT` or release (e.x. `23.05.5`). Can be found on the `Status -> Overview` page (`Firmware Version` parameter value).
     - Package manager you are using: `apk` (the newer one) or `opkg` (the legacy one). If you are running a stable version of OpenWRT then it's more likely you use `opkg`. `apk` is used only in the `main`/`master` branch so far. To make sure run the following commands: `apk -h` and `opkg -h`. The one that's won't fail (`command not found` message is shown) shows the package manager that you use.
     - CPU/package architecture: run `apk info kernel` or `opkg info kernel` (depending on your package manager) in console and check the `Architecture` value (e.x. `aarch64_cortex-a53`) or consult [OpenWRT router database](https://openwrt.org/toh/start).
     - Target: can be found on the `Status -> Overview` page. The first part (before the slash) of the `Target Platform` value.
     - Subtarget: can be found on the `Status -> Overview` page. You guessed it! The second part (after the slash) of the `Target Platform` value.
     - Vermagic: run `apk info kernel` or `opkg info kernel` (depending on your package manager) in console check hash after the kernel version of the `Version` value. E.x. if the value equals to `6.6.52~f58afd3748410d3b1baa06a466d6682-r1` then vermagic equals to `f58afd3748410d3b1baa06a466d6682`. You can also choose:
         - `auto`: the script will get vermagic value from the OpenWrt site.
         - `any`: the script will not check the variable.
3. Make a fork of this repo.
4. Optional: update/change commit hashes (`PKG_SOURCE_VERSION` variable) of the upstream repos in `amneziawg-tools/Makefile`, `kmod-amneziawg/Makefile` and `amneziawg-go/Makefile` files. Remember that `amneziawg-tools` features should match `amneziawg-linux-kernel-module` features, i.e. choose two corresponding commits in both repos.
5. Go to Actions (enable them if not available).
6. Choose `Build OpenWrt toolchain cache`, put your router parameters (from step 1) in the `Run workflow` menu and run the job.
7. It will take ~2-2.5 hours to build the cache. So get some cookies, tea, your favorite book and wait.
8. After the cache has been created, choose `Build AmneziaWG from cache` job, put the same parameters in the `Run workflow` menu and run it.
9. It will take ~10-15 minutes to build the binaries. After the process is finished you can download them in the job's artifacts section (bottom of the page).
10. Unpack the archive, and install:
   - Via WebInterface (LuCi):
       - Go to `System -> Software` menu.
       - Press `Upload Package...`
       - Select `kmod-amneziawg` or `amneziawg-go` .ipk file.
       - Confirm installation.
       - Repeat those steps for `amneziawg-tools` .ipk file and then `luci-proto-amneziawg` .ipk file.
   - Via console:
       - Transfer files into the router.
       - Run `apk install {path to the kmod-amneziawg or amneziawg-go .ipk}` or `opkg install {path to the kmod-amneziawg or amneziawg-go .ipk}` depending on your package manager.
       - Run `apk install {path to the amneziawg-tools .ipk}` or `opkg install {path to the amneziawg-tools .ipk}` depending on your package manager.
       - Run `apk install {path to the luci-proto-amneziawg .ipk}` or `opkg install {path to the luci-proto-amneziawg .ipk}` depending on your package manager.
11. Reboot router or run `/etc/init.d/network restart` command in the console.
12. Congratulations - you now have AmneziaWG installed on your router. Go to `Network -> Interfaces` page, press `Add new interface..` and select `AmneziaWG` as protocol.

*Sometimes is is required to clean the browser's cache to see the new protocol available in the OpenWRT.
