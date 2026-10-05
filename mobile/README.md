# mobile — 本线路实测精简源

对多个聚合源库做**本地实测筛选**后的精简订阅源，针对「云南移动」线路优化。

## ⚠️ 影视仓 / TVBox 用户先看这一节

这类 App（影视仓、TVBox 及其各种分支）的配置体系是分两套的，**填错栏一定报错**：

| App 里那一栏 | 要什么格式 | 填错会怎样 |
|---|---|---|
| **配置地址**（接口地址 / 仓库地址） | **JSON** | 填 m3u/txt → `JSON解析失败 Value http of type java.lang.String cannot be converted to JSONObject` |
| **直播地址**（直播源） | **txt 优先，m3u 次之** | — |

### 直播源：为什么必须用 txt，不用 m3u

同一个频道在本目录的 m3u 里写了 3 条线路（`#EXTINF` 重复 3 次）。**TVBox 系对 m3u 通常只认每个频道的第一条，后面两条备用线路全部作废。**

txt 格式则是每个频道独立成行，3 条线路全部保留：

```
CCTV5,http://113.90.154.189:9901/tsfile/live/0005_1.m3u8?key=txiptv&playlive=1&authid=0
CCTV5,http://27.129.145.110:4433/hls/5/index.m3u8
CCTV5,http://221.226.51.220:50081/newlive/live/hls/6/live.m3u8
```

**所以给影视仓/TVBox 用，一律填 txt 那个地址。**

### 两条配置路径

**A 方案（推荐）** —— 填「直播地址 / 直播源」栏：

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile.txt
```

**B 方案** —— 有些版本直播源入口不好找，可以把下面这个 JSON 填进「配置地址」栏：

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/tvbox.json
```

> ⚠️ B 方案的 JSON 里 `sites` 是空数组（不含点播源），所以它**只提供直播**。填进去会新增一个仓库，选中后点播页是空的——别把它当主配置用，看完直播切回原仓库即可。

## 地址状态（2026-10-05 23:50 实测）

`@latest` 与 `@master` 这类**分支引用**会被 jsDelivr 缓存约 12 小时，实测当前仍吐出**修复前的旧内容**（466 个 `#genre#`，分组解析会乱）。所以下面区分两栏：

| 用途 | 分支引用（长期，滞后 ≤12h） | 钉版引用（**立即生效**） |
|---|---|---|
| 直播源（txt） | `…@latest/mobile/iptv4_mobile.txt` | `…@4283239/mobile/iptv4_mobile.txt` |
| 直播源（m3u） | `…@latest/mobile/iptv4_mobile.m3u` | `…@4283239/mobile/iptv4_mobile.m3u` |
| 配置地址（JSON） | `…@latest/mobile/tvbox.json` | `…@c3e04d2/mobile/tvbox.json` |

前缀统一为 `https://fastly.jsdelivr.net/gh/eye1130/iptv`。

**判断内容新旧**：txt 里 `#genre#` 出现 **32 次**是修复版；**466 次**是旧版。

`tvbox.json` 内部的直播源地址已钉在修复版 commit 上，所以即使外层 JSON 走的是滞后的 `@latest`，拉到的直播列表仍是修复版。

> ⚠️ **不要用 `@master`**。语义与 `@latest` 相同，但该缓存键被缓存了更旧的内容（497 条，TTFB 36.9s）。

## 订阅地址（通用播放器）

走 jsDelivr 的 **Fastly** 节点（本线路直连实测可用）。

### 主地址（固定不变）

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile.m3u
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile.txt
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/tvbox.json
```

`@latest` 跟随默认分支 HEAD，**地址永远不用改**。代价是 jsDelivr 对分支引用有约 12 小时 CDN 缓存，源更新后最多滞后半天自动同步。

### 立即取到最新版（commit / 标签引用）

分支引用有 12h 缓存，要立刻拿到新内容，用 **commit sha** 或 **版本标签**：

```
# commit sha（把 4283239 换成最新 commit 前 7 位）
https://fastly.jsdelivr.net/gh/eye1130/iptv@4283239/mobile/iptv4_mobile.txt

# 版本标签
https://fastly.jsdelivr.net/gh/eye1130/iptv@v20261007/mobile/iptv4_mobile.txt
```

> ⚠️ **不要用 `@master`**。`@master` 与 `@latest` 语义相同，但实测该缓存键被 CDN 缓存了旧内容（只有 497 条，且 TTFB 高达 36.9s）。用 `@latest` 或具体标签。
>
> ⚠️ 标签名**不要带小段号**（`v20261005.1` 会被 jsDelivr 判 404）。用纯 `vYYYYMMDD`。
>
> ⚠️ 一个引用一旦被请求过就被 CDN 记住。**标签删了又重建指向别的 commit，会继续吐旧内容**（实测 `v20261006` 如此）。要换内容就换新引用名，别复用旧名。
>
> ⚠️ 实测只有 `fastly.jsdelivr.net` 可用。`cdn.jsdelivr.net`、`gcore.jsdelivr.net`、`raw.githubusercontent.com`、`gh-proxy` / `ghfast.top` / `gitmirror` 系加速在本线路**全部不可达**。
>
> ⚠️ 换地址后播放器要**手动刷新订阅**——播放器自身也有 6–24 小时缓存。

## 数据说明

| 项 | 值 |
|---|---|
| 生成时间 | 2026-10-05 |
| 输入源库 | `live.zbds.top`、`kimwang1978/collect-tv-txt`（bbxx365_lite）、`YanG-1989/m3u`、`iptv-org/iptv` |
| 候选唯一源 | 7760 |
| 连通 | 2658 |
| 优质（≥2Mbps 且 TTFB≤2s） | 1107 |
| 保留 | **947 频道** |
| 覆盖 | 央视频道全、卫视频道全、地方/港澳台/纪录/体育等 32 个分组 |

筛选口径：两阶段实测——先拉 m3u8 取 HTTP 状态与 TTFB，再解析 ts 分片实测 3 秒真实下载速率。每频道保留最优 3 个源。

txt 采用标准 TVBox 分组格式：`分组名,#genre#` 每组只声明一次，同组频道连续排列。

## 效果对比

| 项 | 换库前 | 换库后 |
|---|---|---|
| CCTV5 | 0.94 Mbps（唯一可用源） | **32.85 Mbps** |
| 卫视数 | 21 | **47** |
| 频道总数 | 430 | **947** |

> 关键教训：**源库的选择决定成品下限**。小合集里 CCTV5 只有一个 0.94Mbps 的源；大合集（bbxx365_lite，7380 条）里同一频道有 101 个候选、最好 50Mbps。

## 已知短板

- 移动咪咕源（`cmvideo.cn`）与省级运营商源（`39.130.x.x` 云南移动）返回 302 或不可达，**需 IPTV 专网**，公网访问不到。
- 大量优质源是各省 IPTV 转出（`key=txiptv`、`:9901/tsfile/`），生命周期短，**建议每周重跑一次**。
- 测速于 2026-10-05 深夜完成，晚间高峰（20:00–22:00）码率会衰减。

## 重新生成

```bash
python scripts/scan_sources.py \
  --in data/raw/iptv4.m3u data/raw/bbxx-lite.m3u data/raw/yang-gather.m3u data/raw/yang-migu.m3u data/raw/iptv-org-cctv.m3u \
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
