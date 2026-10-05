# mobile — 本线路实测精简源

对多个聚合源库做**本地实测筛选**后的精简订阅源，针对「云南移动」线路优化。

## 订阅地址

走 jsDelivr 的 **Fastly** 节点（本线路实测可直连）：

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@master/mobile/iptv4_mobile.m3u
https://fastly.jsdelivr.net/gh/eye1130/iptv@master/mobile/iptv4_mobile.txt
```

> ⚠️ 实测结论：`cdn.jsdelivr.net`、`gcore.jsdelivr.net`、`raw.githubusercontent.com`、`gh-proxy` / `ghfast.top` / `gitmirror` 系加速，在本线路**全部不可达**。只有 `fastly.jsdelivr.net` 通（200，TTFB 0.515s）。

## 数据说明

| 项 | 值 |
|---|---|
| 生成时间 | 2026-10-05 |
| 输入源库 | `live.zbds.top`、`YanG-1989/m3u`、`iptv-org/iptv`（仅取 CCTV/CGTN） |
| 候选唯一源 | 865 |
| 连通 | 545 |
| 优质（≥2Mbps 且 TTFB≤2s） | 374 |
| 保留 | 430 频道 / 497 源 |
| 覆盖 | 央视 17/18（缺 CCTV6）、卫视 21 个 |

筛选口径：两阶段实测——先拉 m3u8 取 HTTP 状态与 TTFB，再解析 ts 分片实测 3 秒真实下载速率。
不达标频道回退到"仅可连通"的源，保证频道不丢。

## 已知短板

- **CCTV5 只有 0.94 Mbps**（`gmxw.7766.org`），全网找不到更好的公网源。看体育直播会卡。
- 移动咪咕源（`cmvideo.cn`）返回 302，跳转后需 **IPTV 专网**，公网不可达。
- 熔断名单里的 `223.110.245.x`、`183.207.x.x`、`39.134/39.135.x` 均为移动 IPTV 专网段，普通宽访问不到。
- 测速于 2026-10-05 深夜完成，晚间高峰（20:00–22:00）码率会衰减。

## 重新生成

```bash
python scripts/scan_sources.py \
  --in data/raw/iptv4.m3u data/raw/yang-gather.m3u data/raw/iptv-org-cctv.m3u \
  --txt data/raw/iptv4.txt \
  --out data/out --workers 48 --top 2 --min-mbps 2.0 --max-ttfb 2.0
```

单频道体检：

```bash
python scripts/check_channel.py CCTV5
```

## 注意

本目录位于上游同步仓库中。上游 workflow 采用 `fetch + merge + force push`，不会删除本目录；
但若上游将来新增同名路径，请以本目录为准并检查同步结果。
