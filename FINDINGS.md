# AmneziaWG iStoreOS Findings

## Router issue

The `amnezia-wg-easy` Docker container stopped working after the iStoreOS update.

Container log symptom:

```text
Command failed: awg show wg0 dump
Unable to access interface: Protocol not supported
```

Container status showed it was running but unhealthy/starting:

```text
amnezia-wg-easy   ghcr.io/wg-easy/wg-easy:15   Up ... (health: starting)
```

The container is configured to force AmneziaWG mode:

```text
EXPERIMENTAL_AWG=true
OVERRIDE_AUTO_AWG=awg
WG_HOST=istoreos.soulripper13.online
WG_PORT=51820
```

## Root cause

The router update changed the running kernel from the previously working kernel to:

```text
6.6.141
```

The previous working AmneziaWG kernel package metadata remains on the router:

```text
/usr/lib/opkg/info/kmod-amneziawg.control
/etc/modules.d/amneziawg
```

That old package was built for:

```text
kernel (=6.6.127~50daf8372d971124fb3519e8d87e02ae-r1)
```

After the iStoreOS update, the actual module is no longer present:

```text
/lib/modules/6.6.141/amneziawg.ko   # missing
```

The router only has the standard WireGuard module:

```text
/lib/modules/6.6.141/wireguard.ko
```

Module load checks failed:

```text
modprobe amneziawg
failed to find a module named amneziawg

modprobe awg
failed to find a module named awg
```

Direct interface creation also failed:

```text
ip link add dev wgtest type amneziawg
Error: Unknown device type.
```

So the container is not failing because of Docker networking or the web UI. It is failing because the AmneziaWG kernel module for the current iStoreOS kernel is missing.

## Package feed state

After `opkg update`, the current feeds provide:

```text
amneziawg-tools
luci-app-amneziawg
kmod-wireguard
wireguard-tools
```

They do not provide the required package:

```text
opkg install kmod-amneziawg
Unknown package 'kmod-amneziawg'.
```

The current OpenWrt/iStoreOS kmods feed is:

```text
https://mirrors.sustech.edu.cn/openwrt/releases/24.10.7/targets/x86/64/kmods/6.6.141-1-50daf8372d971124fb3519e8d87e02ae
```

The exact needed build target is:

```text
iStoreOS/OpenWrt release: 24.10.7
target: x86/64
kernel ABI: 6.6.141-1-50daf8372d971124fb3519e8d87e02ae
```

## GitHub builder repo

Created repo:

```text
https://github.com/soulripper13/amneziawg-openwrt-kmod-builder
```

Local path:

```text
/Users/katoaroosultan/Documents/GitHub/amneziawg-openwrt-kmod-builder
```

Current commit:

```text
8908dfe Add OpenWrt AmneziaWG kmod builder
```

Workflow:

```text
Build kmod-amneziawg
```

Workflow run started:

```text
https://github.com/soulripper13/amneziawg-openwrt-kmod-builder/actions/runs/27253014379
```

Inputs used:

```text
openwrt_release=24.10.7
target=x86/64
kernel_abi=6.6.141-1-50daf8372d971124fb3519e8d87e02ae
amneziawg_ref=master
publish_release=false
```

At the time of handoff, the GitHub Actions run was still in the `Build kmod-amneziawg` compile step.

## Resume steps

Check the workflow result:

```sh
gh run view 27253014379 -R soulripper13/amneziawg-openwrt-kmod-builder
```

If it failed, fetch failed logs:

```sh
gh run view 27253014379 -R soulripper13/amneziawg-openwrt-kmod-builder --log-failed
```

If it succeeded, download the artifact:

```sh
gh run download 27253014379 -R soulripper13/amneziawg-openwrt-kmod-builder
```

Copy the generated `.ipk` to the router and install:

```sh
opkg install ./kmod-amneziawg_*.ipk
modprobe amneziawg
lsmod | grep amneziawg
docker restart amnezia-wg-easy
docker logs --tail 80 amnezia-wg-easy
```

Do not force-install the package if `opkg` reports a kernel dependency mismatch. Rebuild with the exact ABI from the router's current `openwrt_kmods` feed.
