# mobile — 本线路实测精简源

对多个聚合源库做**本地实测筛选**后的订阅源，针对「云南移动」线路优化。
每条源都经过「拉 m3u8 取 HTTP 状态与 TTFB」+「解析 ts 分片实测 3 秒真实下载速率」两阶段验证。

## ⚠️ 影视仓 / TVBox 用户先看这一节

这类 App（影视仓、TVBox 及各种分支）的配置体系分两套，**填错栏一定报错**：

| App 里那一栏 | 要什么格式 | 填错会怎样 |
|---|---|---|
| **配置地址**（接口地址 / 仓库地址） | **JSON** | 填 m3u/txt → `JSON解析失败 Value http of type java.lang.String cannot be converted to JSONObject` |
| **直播地址**（直播源） | **txt 优先，m3u 次之** | — |

### 直播源：为什么优先用 txt

TVBox 系对 m3u 通常只认每个频道的第一条，备用线路全部作废。txt 是每频道独立成行，所有线路都能用。
另外 —— **精简版只有 12 KB，全量版 92 KB**。本线路走 jsDelivr 的吞吐只有 20–40 KB/s，
拉全量列表冷缓存要 5 s 以上，播放器容易转圈超时；精简版 1 s 内就能拿到。

**所以除非你要找冷门地方台，一律用精简版。**

## 地址

前缀统一为 `https://fastly.jsdelivr.net/gh/eye1130/iptv`

| 用途 | 地址 | 体积 | 频道数 |
|---|---|---|---|
| **直播源 · 精简版（推荐）** | `@latest/mobile/iptv4_mobile_lite.txt` | ~12 KB | 91 |
| 直播源 · 全量 | `@latest/mobile/iptv4_mobile.txt` | ~92 KB | 912 |
| 直播源 · 精简版 m3u | `@latest/mobile/iptv4_mobile_lite.m3u` | — | 91 |
| 直播源 · 全量 m3u | `@latest/mobile/iptv4_mobile.m3u` | — | 912 |
| 配置地址（JSON，影视仓专用） | `@latest/mobile/tvbox.json` | — | 两档都在 |

完整写法举例：

```
https://fastly.jsdelivr.net/gh/eye1130/iptv@latest/mobile/iptv4_mobile_lite.txt
```

`@latest` 跟随默认分支 HEAD，**地址永远不用改**。

> ### ⚠️ 关于 `@latest` 的 12 小时缓存（务必读）
>
> jsDelivr 会把 `@latest` **解析成一个具体 commit 后缓存约 12 小时**。实测行为：
>
> | 情况 | 表现 |
> |---|---|
> | 仓库**新增**文件 | 立即能取到（旧 commit 里没有，缓存穿透） |
> | 仓库**修改**已有文件 | 最多滞后 12 小时才拿到新内容 |
>
> 加 `?v=xxx` 查询参数、调用 `purge.jsdelivr.net` 都**绕不过**（实测无效，解析缓存在 CDN 之外）。
>
> **对使用的影响**：源每天更新一次，所以最坏情况你拿到的是 12 小时前那批。源的生命周期通常有几天，
> 这个滞后可以接受。**但如果某天发现源大面积为 0，别急着骂源失效——很可能只是缓存还没轮到**。
>
> **要立刻拿最新版**：用当天的 tag 引用（tag 是全新缓存键，立即生效）：
>
> ```
> https://fastly.jsdelivr.net/gh/eye1130/iptv@v20261024/mobile/iptv4_mobile_lite.txt
> ```
>
> 日常不用管这个，固定地址就行。

> ⚠️ 本线路实测只有 `fastly.jsdelivr.net` 与 `testingcf.jsdelivr.net` 可用。
> `cdn.jsdelivr.net`（TTFB 4 s+）、`gcore.jsdelivr.net`（TLS 握手直接失败）、
> `raw.githubusercontent.com`、各类 gh-proxy 加速在本线路全部不可达。
>
> ⚠️ 换地址后播放器要**手动刷新订阅**——播放器自身也有 6–24 小时缓存。

### 精简版包含什么

| 分组 | 内容 |
|---|---|
| 央视频道 | CCTV1–17 全套 + CCTV5+ + CGTN 各语种 + 央视付费频道（风云足球/剧场/台球等） |
| 卫视频道 | 全国 40+ 卫视，含凤凰、香港、澳门、大湾区 |
| 云南频道 | 云南卫视、云南都市、云南娱乐 |

每频道保留**最优 2 条**源，共 149 条。

### tvbox.json 说明

`sites` 为空数组（不含点播源），**只提供直播**。填进配置地址后会新增一个仓库，
选中即可看到「直播·精简版」和「直播·全量」两个直播源。点播页是空的，别当主配置用。

## 数据说明（2026-10-24 实测）

| 项 | 值 |
|---|---|
| 生成时间 | 2026-10-24 21:36 |
| 输入源库 | `iptv4`（zbds/上游）、`bbxx365_lite`、`YanG-1989/m3u`、`iptv-org/iptv` |
| 候选唯一源 | 7452 |
| 连通 | 2215 |
| 优质（≥2 Mbps 且 TTFB ≤2 s） | 460 |
| 保留频道 | **912** |
| 分组数 | 38 |

## 已修复的坑（别再踩）

| 坑 | 现象 | 修法 |
|---|---|---|
| **熔断阈值过激** | 同一台服务器上几条死链就把整机拉黑，连带好源一起消失。实测 CCTV5 全量只剩 1 条 | 阈值 3 → 8，且只有连接层失败才计数；404 / 非 HLS 不熔断 |
| **音频流冒充直播源** | 央视系把广播版挂在 `/audio/` 下（如 `…/audio/cctv5_2.m3u8`），握手拉流测速全部正常，就是**没画面**。CCTV5 曾因此只剩这一条 | 探测阶段识别 `/audio/` `/radio/` 直接判不合格 |
| **tvbox.json 钉死旧 commit** | JSON 里的直播源地址写死在某个 commit 上，仓库怎么更新它都不变 —— 用配置地址的设备永远吃旧源 | 改回 `@latest` |
| **推送文件名写死日期** | `push_mobile.py` 里硬编码 `20261005`，隔天就推旧文件 | 改成按日期自动取最新 |
| **晚高峰测速失真** | 21:00–23:00 扫，连接超时 3.5 s 会把慢但活的源判死。同源库深夜扫优质 1107、晚高峰扫只剩 460 | 定时任务必须放凌晨跑 |

## 已知短板

- 移动咪咕源（`cmvideo.cn`）与省级运营商源（`39.130.x.x` 云南移动）返回 302 或不可达，**需 IPTV 专网**，公网访问不到。
- 大量优质源是各省 IPTV 转出（`key=txiptv`、`:9901/tsfile/`），生命周期短，**建议每周至少重跑一次**。
- CCTV5 / CCTV13 这类版权严的频道，公网可用源本来就少，晚高峰速率衰减明显。
- 测速在晚间完成，白天与深夜的码率会更好。

## 重新生成

```bash
# 1. 拉最新源（gh api 比第三方镜像快，且内容最新）
gh api repos/kimwang1978/collect-tv-txt/contents/bbxx365_lite.m3u \
  -H "Accept: application/vnd.github.raw" > data/raw/bbxx_new.m3u
gh api repos/eye1130/iptv/contents/tv/iptv4.m3u \
  -H "Accept: application/vnd.github.raw" > data/raw/iptv4_gh.m3u

# 2. 全量扫描
python scripts/scan_sources.py \
  --in data/raw/iptv4_gh.m3u data/raw/bbxx_new.m3u data/raw/yang-gather.m3u \
       data/raw/yang-migu.m3u data/raw/iptv-org-cctv.m3u \
  --out data/out --workers 64 --top 3 --min-mbps 2.0 --max-ttfb 2.0

# 3. 出精简版
python scripts/make_lite.py \
  --in data/out/iptv4_mobile_$(date +%Y%m%d).txt \
  --m3u-src data/out/iptv4_mobile_$(date +%Y%m%d).m3u \
  --out data/out/iptv4_mobile_lite_$(date +%Y%m%d).txt --top 2

# 4. 推送（一次原子提交，避免被上游 force push 吃掉文件）
python scripts/push_mobile.py --tag v$(date +%Y%m%d)
```

单频道体检：

```bash
python scripts/check_channel.py CCTV5
```

## 注意

本目录位于上游同步仓库中。上游 workflow 采用 `fetch + merge + force push`，不会删除本目录；
但若上游将来新增同名路径，请以本目录为准并检查同步结果。
