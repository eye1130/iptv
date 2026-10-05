# mobile — 本线路实测精简源

对多个聚合源库做**本地实测筛选**后的精简订阅源，针对「云南移动」线路优化。

## 订阅地址

走 jsDelivr 的 **Fastly** 节点（本线路直连实测可用）。

### 主地址（固定不变，推荐）

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile.m3u
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile.txt
```

`@latest` 跟随默认分支 HEAD，**地址永远不用改**。代价是 jsDelivr 对分支引用有约 12 小时 CDN 缓存，所以源更新后最多滞后半天自动同步。

### 立即取到最新版（版本标签）

每次重新扫描会打一个新标签，标签地址**立即生效且永久缓存**（速度快）：

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@v20261005/mobile/iptv4_mobile.m3u
```

> ⚠️ **不要用 `@master`**。`@master` 与 `@latest` 语义相同，但实测该缓存键被 CDN 缓存了旧内容（只有 497 条，且 TTFB 高达 36.9s）。用 `@latest` 或具体标签。

> ⚠️ 实测结论：`cdn.jsdelivr.net`、`gcore.jsdelivr.net`、`raw.githubusercontent.com`、`gh-proxy` / `ghfast.top` / `gitmirror` 系加速，在本线路**全部不可达**。只有 `fastly.jsdelivr.net` 通。

> ⚠️ 播放器端通常也有自己的订阅刷新周期（常见 6–24 小时）。换新地址后在播放器里**手动刷新订阅**，否则会继续播旧缓存列表。

## 数据说明

| 项 | 值 |
|---|---|
| 生成时间 | 2026-10-05 |
| 输入源库 | `live.zbds.top`、`kimwang1978/collect-tv-txt`（bbxx365_lite）、`YanG-1989/m3u`、`iptv-org/iptv` |
| 候选唯一源 | 7760 |
| 连通 | 2624 |
| 优质（≥2Mbps 且 TTFB≤2s） | 1244 |
| 保留 | **955 频道 / 1308 源** |
| 覆盖 | 央视 18/18（含 CCTV6）、央视系 42 个、卫视 **47** 个 |

筛选口径：两阶段实测——先拉 m3u8 取 HTTP 状态与 TTFB，再解析 ts 分片实测 3 秒真实下载速率。每频道保留最优 3 个源，单源失效时仍有备用。

## 效果对比

| 项 | 换库前 | 换库后 |
|---|---|---|
| CCTV5 | 0.94 Mbps（唯一可用源） | **32.85 Mbps** |
| 卫视数 | 21 | **47** |
| 频道总数 | 430 | **955** |

> 关键教训：**源库的选择决定成品下限**。小合集里 CCTV5 只有一个 0.94Mbps 的源；大合集（bbxx365_lite，7381 条）里同一频道有 101 个候选、最好 50Mbps。

## 已知短板

- 移动咪咕源（`cmvideo.cn`）与省级运营商源（`39.130.x.x` 云南移动）返回 302 或不可达，**需 IPTV 专网**，公网访问不到。
- 大量优质源是各省 IPTV 转出（`key=txiptv`、`:9901/tsfile/`），生命周期短，**建议每周重跑一次**。
- 测速于 2026-10-05 深夜完成，晚间高峰（20:00–22:00）码率会衰减。

## 重新生成

```bash
python scripts/scan_sources.py \
  --in data/raw/iptv4.m3u data/raw/bbxx-lite.m3u data/raw/yang-gather.m3u data/raw/iptv-org-cctv.m3u \
  --txt data/raw/iptv4.txt \
  --out data/out --workers 64 --top 3 --min-mbps 2.0 --max-ttfb 2.0
```

单频道体检：

```bash
python scripts/check_channel.py CCTV5
```

## 注意

本目录位于上游同步仓库中。上游 workflow 采用 `fetch + merge + force push`，不会删除本目录；
但若上游将来新增同名路径，请以本目录为准并检查同步结果。
