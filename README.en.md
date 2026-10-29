# bili-cdn-pin

**An automated fix for Bilibili lag overseas: measure the fastest CDN node and pin it.**

English (AU) | [中文](README.md)

## ⭐ Strongly recommended: BTR (Bilibili Thread Ripper)

If you're still buffering with this project, switch to this much better one:

**[Bilibili-thread-ripper (BTR)](https://github.com/MrTangLuyao/Bilibili-thread-ripper)** — a free open-source userscript (MIT, 1.6k+ stars). It splits the video into chunks and downloads them **multi-threaded, straight from mainland CDN nodes**, sidestepping the "empty overseas edge cache" problem entirely — cold/unpopular videos and 4K/8K/Dolby Vision all play smoothly. A built-in smart scheduler keeps probing the CDNs and dynamically assigns nodes and threads, with resumable downloads.

- 📺 Video intro (Chinese): [One free open-source plugin fixes overseas Bilibili lag](https://www.bilibili.com/video/BV1Teec6BE3s/)
- 🖥️ Windows official-client version: [Bilibili-thread-ripper-desktop](https://github.com/MrTangLuyao/Bilibili-thread-ripper-desktop) — one-line install, one-click restore

This repo lives on as a **diagnosis reference** for CDN-related lag types (to work out which kind of lag you have, see the [Diagnosis Guide](docs/diagnosis.md), in Chinese).

## What problem does it solve

Bilibili serves videos to overseas users mainly through two domains:

```
upos-hz-mirrorakam.akamaized.net    (Akamai)
upos-sz-mirroraliov.bilivideo.com   (Alibaba Cloud overseas)
```

Both CDNs have plenty of nodes around the world, but the scheduler often resolves you onto a far / overloaded / dead node — even when a 3ms local node sits right in your city. (Real case: an Aussie gigabit fibre line returned 0 Mbps across the board on the default source; after pinning a local node it hit 571 Mbps.) First confirm your lag is actually this type: see the [Diagnosis Guide](docs/diagnosis.md) (in Chinese).

This project runs a TCP 443 connection speed test against each candidate node, pins the fastest IP via hosts / DNS rewrite, and re-checks it periodically: **if the current node's latency is still ≤60ms it stays put; only when it degrades or dies does it auto-switch**. It only affects these two exact domains — nothing else on the internet.

## Quick start

### Local mode (hosts, this PC only)

Just run it again whenever things stall — no schedule needed:

```powershell
# Windows: right-click → "Run with PowerShell" (it auto-prompts for admin); to undo add -Restore
windows\bili-cdn-fix.ps1
```

```bash
# macOS; to undo add --restore
sudo bash macos/bili-cdn-fix.sh
```

### Server mode (whole household, recommended)

Run [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) on any always-on device as the household DNS. The script tests speed weekly and updates DNS rewrites via the API, so phones / tablets / TVs all benefit:

1. Install AdGuard Home on an always-on device; set the upstream DNS to `1.1.1.1`
2. Point the router's DHCP DNS at this device; set the device's own DNS to `1.1.1.1` (to prevent a loop)
3. Deploy the update script (edit the address / username / password at the top), then add a weekly Sunday 04:30 scheduled task:
   - Synology: DSM Task Scheduler, run as root `bash /path/to/server/bili-agh-update.sh >> /path/to/bili-agh.log 2>&1`
   - Mac mini / Debian: `sudo crontab -e` and add `30 4 * * 0 /bin/bash /path/to/server/bili-agh-update.sh >> /path/to/bili-agh.log 2>&1` (remember to disable sleep on macOS)
   - Always-on Windows machine: Task Scheduler to run `windows\bili-agh-update.ps1`
4. Verify: `nslookup upos-hz-mirrorakam.akamaized.net <device IP>` should return the pinned IP

### Router mode (OpenWrt / Merlin, the lightest)

dnsmasq replies to the whole LAN from the router's `/etc/hosts` — no AdGuard needed:

```sh
opkg install curl
sh openwrt/bili-openwrt-update.sh
# crontab -e add: 30 4 * * 0 /root/bili-openwrt-update.sh >> /root/bili-cdn.log 2>&1
```

## Directory structure

```
windows/bili-cdn-fix.ps1        Local mode (hosts)
windows/bili-agh-update.ps1     Server mode (AdGuard API, always-on Windows machine)
macos/bili-cdn-fix.sh           Local mode (hosts)
server/bili-agh-update.sh       Server mode (AdGuard API, works on Synology / macOS / Debian)
openwrt/bili-openwrt-update.sh  Router mode (dnsmasq hosts)
docs/                           Diagnosis guide / FAQ / troubleshooting guide (in Chinese)
```

## Documentation

- [Diagnosis Guide](docs/diagnosis.md) — first work out whether you've got "the wrong CDN family" or "the wrong node" (in Chinese)
- [FAQ](docs/faq.md) — update frequency, scope of impact, mobile apps, node failure and other common questions (in Chinese)
- [Troubleshooting Guide](docs/troubleshooting.md) — secondary DNS round-robin, RDNSS leak, IPv6 misconceptions, DoH bypass, Tampermonkey not taking effect, layer-by-layer verification (in Chinese)

## Acknowledgements

- IP seed list and the original idea: [miyouzi/akamTester](https://github.com/miyouzi/akamTester)
- CDN domain switching approach: [Kanda-Akihito-Kun/ccb](https://github.com/Kanda-Akihito-Kun/ccb)

## License

MIT
