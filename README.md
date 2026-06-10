# AmneziaWG OpenWrt kmod builder

Builds `kmod-amneziawg` for OpenWrt/iStoreOS with GitHub Actions.

This exists because OpenWrt kernel modules must match the exact running kernel ABI. After an iStoreOS update, an older `kmod-amneziawg` package can disappear or stop loading even though `amneziawg-tools` and LuCI remain installed.

Current router target defaults:

```text
iStoreOS/OpenWrt release: 24.10.7
target: x86/64
kernel ABI: 6.6.141-1-50daf8372d971124fb3519e8d87e02ae
```

## Build

Run the `Build kmod-amneziawg` GitHub Actions workflow manually.

The workflow downloads the matching OpenWrt SDK, builds only `kmod-amneziawg`, verifies the package kernel dependency, and uploads the `.ipk` as an artifact.

When iStoreOS updates again, run this on the router to get the new values:

```sh
cat /etc/openwrt_release
cat /etc/opkg/distfeeds.conf
uname -r
```

Use the `openwrt_release`, `target`, and `kernel_abi` workflow inputs to match the new firmware. The kernel ABI is the suffix in the `openwrt_kmods` feed URL, for example:

```text
https://.../targets/x86/64/kmods/6.6.141-1-50daf8372d971124fb3519e8d87e02ae
```

## Install on router

Copy the generated `.ipk` to the router, then install and load it:

```sh
opkg install ./kmod-amneziawg_*.ipk
modprobe amneziawg
lsmod | grep amneziawg
docker restart amnezia-wg-easy
docker logs --tail 80 amnezia-wg-easy
```

If `opkg` reports a kernel dependency mismatch, do not force install it. Rebuild with the exact ABI from the router's current `openwrt_kmods` feed.
