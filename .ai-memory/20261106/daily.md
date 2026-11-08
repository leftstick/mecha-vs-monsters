## [20:20] - 性能优化: 移动端 P0 渲染优化（DPR 上限 / 60fps / 画质档 / 颗粒改造）

- **文件**: index.html
- **决策**: 按性能分析结论落地 P0 四项：① 移动端 DPR 上限 1.5（393×873 后备缓冲 137 万 → 77 万像素，-44%）；② 主循环按 60fps 目标时间轴调度，兼容 60/90/120/144Hz（模拟平均均为 60fps，掉帧不追帧）；③ 移动端默认 `quality: 'medium'`，降低发光模糊、粒子上限与后处理强度；④ 胶片颗粒由每帧约 1.37 万次 1px `fillRect` 改为预生成 128×128 噪声贴图 + `createPattern` 一次全屏填充（后处理 fillRect 实测 13700+ → 3 次）。
- **验证**: 内联脚本 `node --check` 通过；`vite build` 通过（494.67 kB / gzip 133.60 kB）；CDP 魅族 22 UA + 393×873 + DPR2：`isMobile=true`、`quality=medium`、`DPR=1.5`、canvas 589×1309、`postProcessFillRects=3`、颗粒 pattern 就绪；桌面 1280×800 DPR2：`quality=high`、`DPR=2`，未误触发降级；0/5/11 关 smoke 与 430×932 resize 后 errors: []；移动端标题/战斗截图正常，颗粒无明显平铺痕迹。

## [20:56] - 性能优化: 微信内置浏览器运行时自适应画质

- **文件**: index.html
- **决策**: ① 用 `MicroMessenger` UA 识别微信，微信内直接以 `low` 画质 + DPR1.25 起步（普通移动端 medium/1.5、桌面 high/2 不变）；② 新增渲染帧间隔采样（45 帧/窗口，连续 2 个差窗口降档、6 个好窗口升档、4s 冷却、升档后 10s 内再掉帧则封锁升档 60s）；③ 低画质下通过属性覆盖屏蔽 Canvas `shadowBlur`/`shadowColor` 并把 `lighter` 降为 `source-over`，关闭全屏调色/暗角/颗粒、粒子上限降至 40%，CSS 关闭 `#scan` 混合、HUD/弹层 `backdrop-filter`、标题滤镜动画与血条扫描动画；④ 微信 DPR 只在 1.25/1.0 间调整，避免画质与分辨率双抖动。
- **验证**: 内联脚本 `node --check` 通过；`vite build` 通过（501.93 kB / gzip 136.60 kB）；CDP 微信 UA + 873×393 + DPR2：`quality=low`、`DPR=1.25`、canvas 1091×491、body `perf-wechat perf-low`、`#scan` display:none、`backdropFilter=none`、colorGrade=0、particleCap=160、shadowBlur 强制 0、lighter→source-over；模拟差帧自动降到 DPR1.0、恢复后回到 1.25，升档失败后 60s 封锁期内不再升档；微信 0/5/11/12 关 smoke errors: []；同场景 headless 软件渲染下 low 档帧率约为 medium/high 的 3~4 倍（绝对值受无 GPU 环境限制，仅作相对参考）；标题与战斗截图正常。
