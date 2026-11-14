## [18:20] - 功能实现: 三战机外观 / 初始机制 / 攻击形态拉开差距

- **文件**: `index.html`
- **外观**: `SHIP_TYPES[].visual` 新增 `scale / shieldR / aura / muzzleScale`：天狼 1.00/48/瞄准环，壁垒 1.18/62/六边形重盾，游隼 0.88/40/速度残影；机库卡片、机体识别光环、炮口闪光同步按机体缩放。
- **初始机制**: `fireBullet()` normal 分支三分：天狼单发标准弹；壁垒三向重炮（单发 0.52 伤害、`heavy`）；游隼双联针弹（单发 0.40 伤害、`needle`）。
- **拾取物形态**: `shipWeaponMod` 分派 — 激光 天狼标准 / 壁垒 1.7 倍宽 + 暖色 / 游隼双联细针绿色；霰弹 5 / 7 / 3 发；火箭 3 / 3 / 2 发。
- **护盾交互**: 壁垒子弹对护盾兵 1.8 倍破盾；游隼针弹直接穿透护盾兵护盾。
- **HUD**: `applyShipTheme` 写入 `--ship-accent`，波次面板显示关卡名、波次格与机体色点。
- **验证**: 差异化探针 27/27；三机同机位截图（含 2.2 倍局部）轮廓、护盾圈、僚机数量可区分。

## [18:45] - 功能实现: 14 Boss 专属弹型 / 移动签名 / 安全走廊

- **文件**: `index.html`
- **签名表**: 新增 `BOSS_SIGNATURES`，14 个 Boss 各自持有 `color / shape / move / gap / gapSize / spacing / speed / spiral`。
- **弹型**: `traceEnemyBulletShape()` 支持 11 种轮廓（diamond/shard/needle/fang/seed/rock/hex/tentacle/void/bubble/orb），敌方子弹三层描边全部走该函数；圆形脉冲与分裂环保留圆形以保证判读。
- **移动语法**: 每个 Boss 自动生成 `sigSpiral / sigMove / sigMoveAlt` 三招，颜色、弹型、旋臂参数、缺口位置均不同；`sideSweep` 新增左右边缘压来的纵向弹墙，只留一条横向走廊，与 `sweep` 的纵向压墙 + 横向缺口互补。
- **公平性**: 所有移动招式带真实缺口（探针实测最小缺口 101.6px、最大 385.9px，缺口内 0 弹）；`hsafe` 新预警整条走廊提前高亮；一阶段在作者编排之后追加签名招式，不打断开场节奏。
- **验证**: 28 个移动招式全部 shaped + 有预警（27/27 探针）；Sprint 5B 25/25；Sprint 5 16/16；P0/P1/P3 全通过。
- **截图**: voidlord 菱形弹 + 安全走廊、dragon 碎片弹墙 + 通道预警、kraken 触手星弹、singularity 虚空弹、helios 日冕弹。

## [19:00] - 功能实现: 关卡背景前景层 + 空气层 + HUD 层次

- **文件**: `index.html`
- **近景**: 新增 `BIOME_FRAMES` 14 套边缘剪影（wreck/coral/dune/rock/organic/crystal/hex/shard）与 `drawBiomeFrame()`，每侧 2–4 组、带轻微浮动，补足“只有远景没有前景”的层次缺口。
- **空气层**: `AMBIENT_PRESETS` 14 套 + `drawAmbientOverlay()`：关卡色雾、2–4 道斜向体积光、尘埃/余烬（第 13 关上升）、边缘速度线、暗角；水下焦散、极光带、磁暴弧光、相位缝隙等专属元素。
- **验证**: 14 关背景全部渲染无异常；深海遗迹 / 土星极光环 / 日冕矿带 / 相位回廊截图层次可辨。
- **构建**: `npx vite build` 通过 780.03 kB（gzip 212.46 kB）。
- **遗留**: 未提交，等待用户确认。
