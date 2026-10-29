# bili-cdn-pin

**海外看B站卡顿的自动化解决方案：测出最快CDN节点并钉住它。**

中文 | [English](README.md)

## ⭐ 强烈推荐：BTR（Bilibili 线程撕裂者）

实测表明，冷门/老视频卡顿的真正原因是海外边缘节点没缓存、每个分片都要回源大陆拉取，这在 DNS / hosts 层面无解，钉哪个 IP 都一样。

如果你用了本项目还是卡，请直接换这个更聪明的方案：

**[Bilibili-thread-ripper（BTR）](https://github.com/MrTangLuyao/Bilibili-thread-ripper)** —— 免费开源油猴脚本（MIT，1.6k+ star）。它把视频切成小块，多线程**直连大陆 CDN 并发下载**，绕开「海外边缘没缓存」这个死结，冷门视频和 4K/8K/杜比视界都能流畅播放；内置智能调度器，持续测速各 CDN 并动态分配节点和线程，支持断点续传。

- 📺 视频介绍：[仅需一个免费开源插件解决海外B站卡顿](https://www.bilibili.com/video/BV1Teec6BE3s/)
- 🖥️ Windows 官方客户端版：[Bilibili-thread-ripper-desktop](https://github.com/MrTangLuyao/Bilibili-thread-ripper-desktop)，一行命令安装、可一键还原

本项目保留作为 CDN 卡顿类型的**诊断参考**（想弄清自己属于哪种卡顿，见[诊断指南](docs/diagnosis.md)）。


## 它解决什么问题

B站给海外用户发视频主要走两个域名：

```
upos-hz-mirrorakam.akamaized.net    (Akamai)
upos-sz-mirroraliov.bilivideo.com   (阿里云海外)
```

这两家CDN全球都有大量节点，但调度经常把你解析到远端/超载/死掉的节点——哪怕你所在城市就有3ms的本地节点（真实案例：澳洲千兆宽带默认源全线超时0 Mbps，钉住本地节点后571 Mbps）。先确认你的卡顿属于这种类型：见 [诊断指南](docs/diagnosis.md)。

本项目对候选节点逐个做 TCP 443 连接测速，把最快的IP通过 hosts / DNS重写，并定期复查：**当前节点延迟仍 ≤60ms 就不动，变差或死掉才自动换新**。只影响这两个精确域名，不影响任何其他网站。

## 快速开始

### 本机模式（hosts，只管这台电脑）

卡了重跑一次即可，无需定时：

```powershell
# Windows：右键"使用 PowerShell 运行"（自动请求管理员）；撤销加 -Restore
windows\bili-cdn-fix.ps1
```

```bash
# macOS；撤销加 --restore
sudo bash macos/bili-cdn-fix.sh
```

### 服务器模式（全家生效，推荐）

家里任何一台常开设备跑 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 当全家DNS，脚本每周定时测速并通过API自动更新DNS重写，手机/平板/电视全部受益：

1. 常开设备装 AdGuard Home，上游DNS填 `1.1.1.1`
2. 路由器 DHCP 的 DNS 指向这台设备；设备自身DNS填 `1.1.1.1`（防环路）
3. 部署更新脚本（改开头的地址/账号/密码），加每周日 04:30 定时任务：
   - 群晖：DSM 任务计划，root 运行 `bash /path/to/server/bili-agh-update.sh >> /path/to/bili-agh.log 2>&1`
   - Mac mini / Debian：`sudo crontab -e` 加 `30 4 * * 0 /bin/bash /path/to/server/bili-agh-update.sh >> /path/to/bili-agh.log 2>&1`（macOS记得关睡眠）
   - Windows常驻机：任务计划程序运行 `windows\bili-agh-update.ps1`
4. 验证：`nslookup upos-hz-mirrorakam.akamaized.net 设备IP` 应返回钉住的IP

### 路由器模式（OpenWrt/梅林，最轻量）

dnsmasq 把路由器 `/etc/hosts` 的记录答复给全局域网，无需AdGuard：

```sh
opkg install curl
sh openwrt/bili-openwrt-update.sh
# crontab -e 加：30 4 * * 0 /root/bili-openwrt-update.sh >> /root/bili-cdn.log 2>&1
```

## 目录结构

```
windows/bili-cdn-fix.ps1        本机模式（hosts）
windows/bili-agh-update.ps1     服务器模式（AdGuard API，Windows常驻机）
macos/bili-cdn-fix.sh           本机模式（hosts）
server/bili-agh-update.sh       服务器模式（AdGuard API，群晖/macOS/Debian通用）
openwrt/bili-openwrt-update.sh  路由器模式（dnsmasq hosts）
docs/                           诊断指南 / FAQ / 排坑指南
```

## 文档

- [诊断指南](docs/diagnosis.md)——先分清你是"CDN家族选错"还是"节点分配错"，两种问题及解决方案
- [FAQ](docs/faq.md)——更新频率、影响范围、手机App、节点失效等常见问题
- [排坑指南](docs/troubleshooting.md)——副DNS轮换、RDNSS漏网、IPv6误区、DoH绕过、油猴不生效、逐层验证方法

## 致谢

- IP种子列表与思路来源：[miyouzi/akamTester](https://github.com/miyouzi/akamTester)
- CDN域名切换方案：[Kanda-Akihito-Kun/ccb](https://github.com/Kanda-Akihito-Kun/ccb)

## License

MIT
