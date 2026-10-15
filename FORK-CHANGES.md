# 本 fork 与原版的区别和更新

**本仓库**：`HYR-blip/Rhine-Music-Windows`（个人分支）
**上游**：`DuoJiuBetter/Rhine-Music-Windows` → 基于 `RonaldDeng/Rhine-Music-Demo` → 界面基于 `LBEILC/RhineLabUI`

本 fork 不是重写，而是在上游基础上**新增两组改动**，并保留原有的入口、资源、主题、署名与第三方许可。原许可证（MIT）与 NOTICE 一字未改。

---

## 更新记录

### 2026-10-15 · 播放功能（3 项）

| 功能 | 入口 | 说明 |
| --- | --- | --- |
| 播放顺序 | 顶栏播放控件左侧四个图标按钮 | 顺序 / 循环 / **单曲循环** / 随机；随机按一轮洗牌，退出时精确还原且不丢播放中新增的曲目 |
| 自定义快捷键 | 设置 → 个性化 → 自定义快捷键 | 12 项动作逐项绑定；**默认 `1` = 上一首，`5` = 单曲循环**；一键只属一个动作，空格/Enter/Tab/Esc 保留给界面 |
| 歌词切换动画 | 设置 → 个性化 → 歌词切换动画 | 无动画 / 淡入淡出 / 上滑淡入 / 缩放聚焦 × 慢/标准/快，带实时预览；只过渡透明度与位移，不动字号 |

细节与验证：[docs/WINDOWS-PLAYBACK-FEATURES.md](docs/WINDOWS-PLAYBACK-FEATURES.md)

### 2026-10-15 · 性能优化（5 项，面向所有设备）

| 优化 | 做法 | 关键数据 |
| --- | --- | --- |
| 封面图集上传合批 | 每块封面一次纹理拷贝 → 按行内连续段合并 | 铺满图集 **432 次拷贝 → 27 次**，逐像素不变 |
| 遮挡面积分析让帧 | 诊断采样移出 60fps 运动帧（带饥饿上限） | 消掉 8.6ms 的周期性 CPU 尖峰 |
| 闭环自适应画质 | 用实测 GPU/帧时间逐项调节，替代静态显卡字符串启发式 | 只在自动模式生效，从不写偏好；按实测下调并精确恢复 |
| 解码缓存分档 | 12M 像素/192 张硬编码 → 按画质档位 12M/16M/24M | 性能档 48 MiB / 独显增强档 96 MiB |
| 基准脚本修复 | `benchmark-windows.mjs` 三处导致无法运行的问题 | 修好后基准才能跑 |

细节与验证：[docs/WINDOWS-PERF22.md](docs/WINDOWS-PERF22.md)

---

## 与上游的具体差异

**新增文件**

```
src/music-playback-order.ts     播放顺序定义、切歌策略（纯函数）、图标              ← 播放功能
src/music-shortcuts.ts          快捷键绑定表、按键匹配、捕获状态机、设置标记        ← 播放功能
src/music-lyric-motion.ts       歌词动画选择、持久化形状、设置标记与预览           ← 播放功能
src/adaptive-quality.ts         自适应画质闭环控制器                                ← 性能
scripts/check-music-playback-order.mjs / check-music-shortcuts.mjs / check-music-lyric-motion.mjs  ← 播放功能
scripts/check-adaptive-quality.mjs / check-glass-visibility.mjs                    ← 性能
docs/WINDOWS-PLAYBACK-FEATURES.md / docs/WINDOWS-PERF22.md / FORK-CHANGES.md
```

**修改的既有文件**

| 文件 | 改动 |
| --- | --- |
| `src/music-player.ts` | 新增 `playbackOrder` 状态与 `setPlaybackOrder()`；把原先**四处**硬编码的 `queue[currentIndex + 1]`（`next` / `previous` / 浏览器 `ended` / 无缝 `prepareSuccessor` 两条路径）统一为同一套顺序策略 |
| `src/music-app.ts` | 顶栏插入四个顺序按钮；设置 → 个性化插入两个板块；键盘处理接入自定义快捷键（捕获优先）；偏好新增 `playbackOrder` / `shortcuts` / `lyricMotion`；自适应画质接入 1.5 秒诊断节拍 |
| `src/music-shell.css` | 顶栏顺序按钮样式（窄窗口 ≤1100px 隐藏以保住播放控件） |
| `src/music-settings.css` | 快捷键按键框、分段控件、歌词动画预览样式 |
| `src/music-song-view.css` | 歌词行过渡规则（4 种效果 × 3 档时长，经 `--music-motion-scale` 缩放） |
| `src/cover-atlas.ts` | 上行带暂存 + 连续段合批上传；解码预算参数化与档位化 |
| `src/glass-visibility-pass.ts` | 面积分析按运动状态推迟，新增 `analysisDeferred` / `analysisFrame` 诊断 |
| `src/scene.ts` | 解码预算跟随画质档位 |
| `scripts/benchmark-windows.mjs` | 修复 fixture 双斜杠 404、`waitForFunction` 选项位置、缺少软件光栅器 WebGL 开关 |
| `package.json` | 新增脚本 `check:shortcuts`、`check:glass`、`check:adaptive` |

**行为层面的差异（用户可感知）**

- 顶栏多出四个播放顺序按钮，激活态取主题强调色；
- 设置 → 个性化多出「自定义快捷键」与「歌词切换动画」两个板块；
- 播放顺序不再只有"顺序"一种：单曲循环在曲终重播当前曲目，随机播放会重排队列（退出时还原）；
- 歌词在句与句之间有了过渡，并在设置里可调可关；
- 自动画质不再只看显卡型号：会按实测帧时间在既有档位内逐项微调（手动模式不受影响）。

**刻意不改变的**

主题与配色、动效速度、减少动态效果、可见玻璃预算（`arrayBudget`）、物理玻璃路径、抗锯齿、图集纹理质量、既有快捷键（空格 / ←→ / ↑↓ / `/` / `v` / `Enter` / `Esc`）、账号与会话处理、本地 HTTP 边界、许可与署名。任何本地曲库、账号凭据、个人路径与日志都没有进入仓库。

---

## 如何验证

```bash
npm ci
npx tsc --noEmit                 # 类型检查
npm run build                    # 生产构建（tsc + vite + PWA）
npm run check:music              # 既有播放/场景/模型检查
npm run check:shortcuts          # 新增：快捷键 / 播放顺序 / 歌词动画（21 项）
npm run check:glass              # 性能：遮挡采样让帧
npm run check:adaptive           # 性能：自适应画质闭环
npm run check:gpu-budget         # 既有画质迁移与预算检查
```

**未验证**（如实记录）：真实显卡上的帧时间与 GPU 分段数据（开发机无 GPU 加速的无头浏览器）；真实账号在线歌单下跑完整的随机一轮；真实设备听感（切歌点、无缝衔接）；实机截图归档。

## 与上游同步 / 回退

- 本 fork 不改写已公开历史；`main` 在上游提交之上追加，未改动上游任何提交。
- 想回到纯上游状态：`git checkout -B main upstream/main`（或直接使用上游仓库的 Release 包）。
- 两组改动也可分别取用：播放功能在 `feat/playback-order-shortcuts-lyrics`，性能在 `perf/all-devices-frame-budget`；对应上游 PR 为 [#2](https://github.com/DuoJiuBetter/Rhine-Music-Windows/pull/2) 与 [#1](https://github.com/DuoJiuBetter/Rhine-Music-Windows/pull/1)。
