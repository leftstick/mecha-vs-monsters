# 机甲大战怪兽 · 完整美术升级 — 产品需求文档

## Overview
- **Summary**: 对单文件 Canvas 2D 竖版 STG 游戏（`/workspace/index.html`）进行一次完整的视觉渲染与 UI 升级，引入统一美术配置 `GAME_ART_CONFIG`，覆盖实体描边/阴影、四层视差场景、主体渲染、战斗特效、HUD/UI 重构、可开关后处理与画质档位。
- **Purpose**: 在不改动任何玩法逻辑的前提下，统一机甲末世科幻美术风格，提升竖版 STG 的视觉层次感与可读性。
- **Target Users**: 桌面端键盘玩家与移动端触屏/手柄玩家。

## Goals
- 建立 `GAME_ART_CONFIG` 统一管理所有美术参数、开关、性能档位。
- 提升竖版 STG 视觉优先级：玩家机甲 > 敌方子弹 > 敌方单位 > 特效 > 背景。
- 四层纵向视差场景（深空星云→城市剪影霓虹→建筑废墟→地面杂物），冷灰蓝主调 + 霓虹点缀。
- 实体轮廓描边、动态/固定阴影、子弹三层结构。
- 主体（玩家/怪兽/精英/Boss）渲染差异化与发光特征。
- 分级战斗特效（枪口闪光、命中火花、暴击飘字、普通/精英爆炸、护盾光膜、受击红边+震动）。
- HUD/UI 竖版重构、统一机甲科幻风面板、低血量/低护盾警示。
- 可开关后处理（暗角、胶片颗粒、扫描线、调色）与高/中/低三档画质。

## Non-Goals
- 不改动任何移动、射击、碰撞、AI、手柄、数值、波次、关卡逻辑。
- 不引入外部图片、引擎、构建工具。
- 不拆分单文件结构。
- 不改变输入映射与控制方式。

## Background & Context
- 现有代码为单文件 `index.html`，`GAME_CONFIG` 已含部分视觉参数（UI、粒子、连击、精英预警、子弹配色等）。
- 已存在 `drawPlayer`/`drawMonster`/`drawBoss`/`drawBullets`/`addHitSparks`/`addExplosion`/`addDebris`/`drawBackground`/`drawFarLayer`/`drawGround`/`drawParticles`/`updateHUD` 等函数。
- 经验教训：仅改绘制时，动态效果应通过只读已有状态或渲染侧局部缓存实现，不得向实体对象写入表现字段；`ctx.save()/restore()` 必须成对包裹全局状态变更。

## Functional Requirements

### 一、GAME_ART_CONFIG 配置
- **FR-1**: 新增顶层 `GAME_ART_CONFIG` 对象，集中管理所有视觉参数与开关，包含：轮廓描边、阴影、子弹三层、视差速度、主体发光、特效强度、HUD 样式、后处理开关、画质档位 `quality`（`high`/`medium`/`low`）。
- **FR-2**: `quality` 档位自动影响粒子上限、后处理开关、发光模糊强度、描边粗细等，无需改代码即可降级。

### 二、基础辨识度
- **FR-3**: 玩家机甲 2px 冷青描边；普通怪兽 1.5px 深炭灰描边；精英 2px 暗红描边，锐利无羽化。
- **FR-4**: 玩家动态引擎投影（随喷焰脉动），怪兽固定椭圆冷灰阴影（透明度 0.3）。
- **FR-5**: 子弹三层结构：高亮核心 + 深色内圈 + 光晕；玩家弹冷白/冰蓝，敌方弹橙红/红；大型弹幕（Boss）脉动发光。

### 三、四层纵向视差场景
- **FR-6**: 远景深空星云 → 中景城市剪影霓虹 → 近景建筑废墟 → 地面杂物，四层纵向滚动，速度递增。
- **FR-7**: 冷灰蓝主调，霓虹点缀；纵向雾气空气透视（远景朦胧、近景清晰）。

### 四、主体渲染升级
- **FR-8**: 玩家机甲分层绘制，发光部件呼吸脉动；底部动态引擎喷焰；移动轻微机身倾斜（复用现有 `renderTilt`）。
- **FR-9**: 四类怪兽差异化外观与发光特征；精英带旋转光环（复用现有 `eliteRing`）。
- **FR-10**: 受击核心高亮（基于 `hitFlash` 只读字段，不新增写入）；死亡爆炸拆解为带旋转的碎片（复用现有 `addDebris`/`addExplosion`）。

### 五、分级战斗特效
- **FR-11**: 枪口锥形闪光、命中尖角火花、暴击伤害飘字。
- **FR-12**: 普通/精英分级爆炸（精英更大更久），峰值亮度可控，不遮挡弹幕。
- **FR-13**: 护盾光膜受击扩散（基于 `shieldPulse` 只读字段）；玩家受击边缘闪红 + 轻微震动（基于 `player.invuln` 与 `game.shake` 只读字段）。
- **FR-14**: 全局粒子数量上限（复用 `GAME_CONFIG.particles.max`），自动淘汰旧实例。

### 六、UI 重构（竖版布局）
- **FR-15**: 上下边缘排布 HUD，中间射击区留白；顶部左右分置信息面板，底部血条护盾条。
- **FR-16**: 统一机甲科幻风面板，细霓虹描边；低血量/低护盾警示闪烁。
- **FR-17**: 弹窗、移动端触控控件风格统一；数值变化弹跳动效。

### 七、可开关后处理
- **FR-18**: 弱暗角、极淡胶片颗粒、可选扫描线。
- **FR-19**: 整体调色：高对比度，暗部冷蓝，霓虹高亮。
- **FR-20**: 高/中/低三档画质，自动适配性能。

## Non-Functional Requirements
- **NFR-1**: 所有渲染侧动态效果不得写入实体对象（player/monster/boss/bullet）的属性；只读已有字段或使用渲染侧局部变量。
- **NFR-2**: `ctx.save()/ctx.restore()` 成对使用，`shadowBlur`/`globalAlpha`/`globalCompositeOperation` 等全局状态在使用后必须还原。
- **NFR-3**: 保持 `index.html` 单文件，相对路径，GitHub Pages 可直接部署。
- **NFR-4**: 移动端与桌面端均可运行，画质档位可切换。

## Constraints
- **Technical**: 原生 Canvas 2D；不引入外部资源；不改玩法逻辑。
- **Business**: 增量优化，与现有 `GAME_CONFIG` 中的视觉参数并存（`GAME_ART_CONFIG` 作为独立顶层配置）。
- **Dependencies**: 依赖现有 `game`/`player`/`LEVELS`/`MONSTER_TYPES` 数据结构与字段（`hitFlash`、`shieldPulse`、`invuln`、`shake`、`renderTilt`、`eliteWarning.eliteTypes` 等）。

## Assumptions
- 现有字段 `m.hitFlash`、`player.shieldPulse`、`player.invuln`、`game.shake`、`renderTilt` 在各帧已由原有逻辑维护，渲染侧只读即可。
- `GAME_ART_CONFIG.quality` 初始为 `high`，可通过控制台或设置切换。
- 后处理通过每帧在主画布上叠加全屏图层实现（暗角/颗粒/扫描线），调色通过 `globalCompositeOperation` 实现。

## Acceptance Criteria

### AC-1: GAME_ART_CONFIG 存在且可读取
- **Type**: `rule`
- **Given**: 页面加载完成
- **When**: 在控制台访问 `GAME_ART_CONFIG`
- **Then**: 对象存在，包含 `quality`、`outline`、`shadow`、`bullet`、`parallax`、`fx`、`hud`、`postprocess` 等子配置块
- **Pass Condition**: 所有子块存在且 `quality` ∈ {high, medium, low}
- **Evidence**: 控制台 `Object.keys(GAME_ART_CONFIG)` 输出

### AC-2: 实体描边与阴影
- **Type**: `rule`
- **Given**: 游戏运行中
- **When**: 玩家、普通怪兽、精英怪兽出现在屏幕上
- **Then**: 玩家有冷青描边与动态阴影，普通怪兽有深炭灰描边与椭圆阴影，精英有暗红描边与旋转光环
- **Pass Condition**: 三类实体描边颜色/粗细符合 FR-3，阴影存在且不随实体属性字段新增
- **Evidence**: 代码审查 `drawPlayer`/`drawMonster`，确认仅使用 `ctx.stroke` + 已有字段

### AC-3: 子弹三层结构
- **Type**: `rule`
- **Given**: 玩家射击与敌方射击
- **When**: 子弹在屏幕上飞行
- **Then**: 每层子弹包含高亮核心 + 内圈 + 光晕三层；玩家弹冷色系，敌方弹暖色系；Boss 弹幕脉动
- **Pass Condition**: `drawBullets` 中玩家与敌方子弹均有三层绘制，颜色区分正确
- **Evidence**: 代码审查 `drawBullets`

### AC-4: 四层视差场景
- **Type**: `rubric`
- **Dimension**: 场景层次感与纵深感
- **Scale**: 1-5
- **Anchors**: 1 = 无分层，背景扁平；3 = 有两层视差但无雾气；5 = 四层清晰分层 + 雾气空气透视 + 速度递增
- **Pass Threshold**: >= 4
- **Evidence**: 进入太空关与地面关各观察一次

### AC-5: 主体渲染差异化
- **Type**: `rubric`
- **Dimension**: 玩家/怪兽/精英/Boss 外观辨识度
- **Scale**: 1-5
- **Anchors**: 1 = 无差异；3 = 颜色区分；5 = 形状/发光/动画均有明显差异，精英光环清晰
- **Pass Threshold**: >= 4
- **Evidence**: 各类型怪兽出场观察

### AC-6: 分级战斗特效
- **Type**: `rubric`
- **Dimension**: 枪口闪光、命中火花、爆炸、护盾、受击反馈的丰富度与可读性
- **Scale**: 1-5
- **Anchors**: 1 = 无特效；3 = 有基础粒子；5 = 特效分层清晰、暴击高亮、精英爆炸明显大于普通、受击反馈到位且不遮挡弹幕
- **Pass Threshold**: >= 4
- **Evidence**: 连续战斗 30 秒观察

### AC-7: HUD/UI 竖版布局与风格统一
- **Type**: `rule`
- **Given**: 游戏运行中
- **When**: 观察 HUD 与弹窗
- **Then**: 顶部左右分置信息面板，底部血条护盾条，中间留白；面板统一霓虹描边风格；低血量/低护盾时闪烁
- **Pass Condition**: HUD 元素位置符合 FR-15/16，低血量触发红色闪烁
- **Evidence**: 移动端与桌面端各观察一次

### AC-8: 可开关后处理
- **Type**: `rule`
- **Given**: 游戏运行中
- **When**: 切换 `GAME_ART_CONFIG.postprocess.vignette`/`grain`/`scanlines` 开关
- **Then**: 暗角、胶片颗粒、扫描线可独立开关；高/中/低画质档位自动调整后处理强度与粒子上限
- **Pass Condition**: 三个开关独立生效，quality 切换后粒子上限与发光强度变化
- **Evidence**: 控制台切换开关后画面变化

### AC-9: 玩法逻辑零侵入
- **Type**: `rule`
- **Given**: 美术升级完成
- **When**: 对比升级前后的 `update`/`checkCollisions`/`fireBullet`/`updateMonsters`/`pollGamepad`/数值字段
- **Then**: 这些函数与 `GAME_CONFIG` 中的数值参数未被修改；实体对象未新增表现字段
- **Pass Condition**: `git diff` 显示玩法逻辑函数与数值配置无变更
- **Evidence**: 差异审查

### AC-10: 运行稳定性
- **Type**: `rule`
- **Given**: 游戏运行
- **When**: 连续进行 3 关（含 Boss 战）
- **Then**: 无控制台报错，游戏循环不中断，手柄/键鼠操作正常
- **Pass Condition**: Node 模拟运行无异常，浏览器三关通关无报错
- **Evidence**: 自动化测试 + 手动验证

## Open Questions
- [ ] 画质档位切换入口：是否需要在设置面板增加切换按钮，还是仅控制台切换？（默认仅控制台，后续可加入设置面板）
