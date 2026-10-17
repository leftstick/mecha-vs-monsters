# 机甲大战怪兽 · 完整美术升级 — 实施计划

## Task 1: 新增 GAME_ART_CONFIG 统一美术配置
- **Status**: `pending`
- **Priority**: high
- **Depends On**: None
- **Description**:
  - 在 `GAME_CONFIG` 之后新增顶层 `GAME_ART_CONFIG`，包含 `quality`、`outline`、`shadow`、`bullet`、`parallax`、`fx`、`hud`、`postprocess` 子块。
  - `quality` 支持 `high`/`medium`/`low`，并提供 `getArt(prefix)` 辅助或直接在各绘制函数中按档位取参数。
  - 所有后续视觉参数从该配置读取，不硬编码。
- **Acceptance Criteria Addressed**: AC-1, NFR-1
- **Test Requirements**:
  - `rule` TR-1.1: 控制台 `GAME_ART_CONFIG` 存在且 `Object.keys` 包含 quality/outline/shadow/bullet/parallax/fx/hud/postprocess；`quality` 合法。证据：node 语法检查 + 控制台输出。

## Task 2: 实体描边与阴影
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1
- **Description**:
  - `drawPlayer`：机身整体描边（2px 冷青，`shadowColor`+`stroke`，不羽化），底部引擎投影随喷焰脉动（`globalAlpha` 椭圆）。
  - `drawMonster`：在各类型绘制外层统一加 1.5px 深炭灰描边；精英（`eliteWarning.eliteTypes.includes(m.type)`）加 2px 暗红描边；所有怪兽底部加固定冷灰椭圆阴影（alpha 0.3）。
  - 仅用 `ctx.save/restore` 包裹，不向 `m`/`player` 写入新字段。
- **Acceptance Criteria Addressed**: AC-2
- **Test Requirements**:
  - `rule` TR-2.1: drawPlayer/drawMonster 中描边颜色/粗细来自 GAME_ART_CONFIG，未给 player/monster 新增字段。证据：代码审查。
  - `rubric` TR-2.2: 描边锐利度；scale 1-5；anchors 1=无/3=有描边但羽化/5=锐利无羽化；阈值>=4；证据：截图。

## Task 3: 子弹三层结构
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1
- **Description**:
  - `drawBullets`：玩家四种子弹统一为「高亮核心 + 深色内圈 + 光晕」三层；颜色沿用 `bulletColors`（普通冷白、激光冰蓝、霰弹金黄、火箭橙红）。
  - 敌方子弹：橙红/红色调，三层结构；Boss 弹幕根据 `b.pulse` 或 `game.time` 做脉动发光。
  - 不修改子弹数据结构。
- **Acceptance Criteria Addressed**: AC-3
- **Test Requirements**:
  - `rule` TR-3.1: drawBullets 中玩家与敌方子弹均含三层绘制，颜色区分正确，Boss 弹有脉动。证据：代码审查。

## Task 4: 四层纵向视差场景
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1
- **Description**:
  - `drawBackground` 重构为四层：远景深空星云 → 中景城市剪影霓虹 → 近景建筑废墟 → 地面杂物，纵向滚动速度递增。
  - 冷灰蓝主调 + 霓虹点缀；加纵向雾气（顶部/中部半透明渐变遮罩）实现空气透视。
  - 太空关（ground=null）保留星云/行星风格但纳入四层结构；地面关叠加城市/废墟/杂物层。
  - 速度参数来自 `GAME_ART_CONFIG.parallax`。
- **Acceptance Criteria Addressed**: AC-4
- **Test Requirements**:
  - `rubric` TR-4.1: 场景层次感；scale 1-5；anchors 1=扁平/3=两层/5=四层+雾气+速度递增；阈值>=4；证据：太空关与地面关截图。

## Task 5: 主体渲染升级（玩家/怪兽/精英/Boss）
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1, Task 2
- **Description**:
  - `drawPlayer`：发光部件（座舱/翼端/引擎）呼吸脉动（用 `game.time` 计算相位，不写 player）；引擎喷焰动态增强；机身倾斜复用 `renderTilt`。
  - `drawMonster`：四类怪兽（octopus/scorpion/ufo/bug/turret/bomber）差异化发光特征（眼/核心/能量点脉动）；精英旋转光环复用现有 `eliteRing`。
  - `drawBoss`：Boss 核心发光脉动、受击高亮。
  - 受击核心高亮基于只读 `hitFlash` 字段；不新增字段。
- **Acceptance Criteria Addressed**: AC-5, FR-10
- **Test Requirements**:
  - `rubric` TR-5.1: 主体辨识度；scale 1-5；anchors 1=无差异/3=颜色区分/5=形状+发光+动画差异；阈值>=4；证据：各类型出场截图。
  - `rule` TR-5.2: 动态相位用 `game.time`/`performance.now()` 计算，未向实体写入表现字段。证据：代码审查。

## Task 6: 分级战斗特效
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 1
- **Description**:
  - `addHitSparks`：命中尖角火花 + 中心闪光（已有，增强方向扇形与颜色）。
  - 枪口锥形闪光：在 `drawPlayer` 渲染侧根据 `player.fireCD` 刚重置的窗口推断射击瞬间，生成锥形闪光粒子（不写 player、不改 fireBullet）。
  - 爆炸：普通/精英分级（精英 `explosionMult`），峰值亮度受限，不遮挡弹幕。
  - 护盾光膜受击扩散：基于 `player.shieldPulse` 只读字段在玩家周围绘制扩散环。
  - 玩家受击边缘闪红 + 轻微震动：基于 `player.invuln` 与 `game.shake` 只读字段，在屏幕边缘绘制红色内发光（不新增 shake 逻辑）。
  - 暴击飘字复用 `addFloatText` 的 crit 分支。
  - 粒子上限复用 `GAME_CONFIG.particles.max`。
- **Acceptance Criteria Addressed**: AC-6, FR-14
- **Test Requirements**:
  - `rubric` TR-6.1: 特效丰富度与可读性；scale 1-5；anchors 1=无/3=基础粒子/5=分层清晰+精英爆炸更大+受击到位不遮弹幕；阈值>=4；证据：连续战斗截图。
  - `rule` TR-6.2: 枪口闪光通过 fireCD 窗口推断，未修改 fireBullet 与 player 对象。证据：代码审查。

## Task 7: HUD/UI 竖版布局与风格统一
- **Status**: `pending`
- **Priority**: medium
- **Depends On**: Task 1
- **Description**:
  - CSS：顶部左右分置信息面板（左 WAVE+武器，右 SCORE+连击），底部血条/护盾条，中间射击区留白；统一机甲科幻风面板（细霓虹描边、内发光）。
  - 低血量（`hpLowThreshold`）红光闪烁、低护盾警示。
  - 弹窗（结算/暂停/标题）与移动端触控控件风格统一；数值变化弹跳动效（CSS keyframes）。
  - 不改 `updateHUD` 的数值逻辑，仅改样式。
- **Acceptance Criteria Addressed**: AC-7
- **Test Requirements**:
  - `rule` TR-7.1: HUD 布局符合上下边缘排布，低血量触发闪烁，弹窗风格统一。证据：截图。
  - `rule` TR-7.2: updateHUD 数值逻辑未改。证据：diff 审查。

## Task 8: 可开关后处理与画质档位
- **Status**: `pending`
- **Priority**: medium
- **Depends On**: Task 1
- **Description**:
  - 在 `render()` 末尾叠加全屏后处理图层：弱暗角（径向渐变）、极淡胶片颗粒（随机点）、可选扫描线（横线）。
  - 整体调色：通过 `globalCompositeOperation='overlay'/'soft-light'` 叠加冷蓝底色与霓虹高光。
  - `GAME_ART_CONFIG.quality` 三档：high（全开+强发光）、medium（暗角+颗粒，发光减半）、low（关闭后处理，描边/发光减弱，粒子上限降低）。
  - 开关来自 `GAME_ART_CONFIG.postprocess`。
- **Acceptance Criteria Addressed**: AC-8
- **Test Requirements**:
  - `rule` TR-8.1: postprocess 三个开关独立生效，quality 切换后粒子上限与发光强度变化。证据：控制台切换验证。

## Task 9: 稳定性与玩法零侵入验证
- **Status**: `pending`
- **Priority**: high
- **Depends On**: Task 2, Task 3, Task 4, Task 5, Task 6, Task 7, Task 8
- **Description**:
  - Node 模拟环境运行多帧，覆盖玩家射击、怪兽生成、击杀、精英、Boss、粒子溢出，确认无运行时异常。
  - 审查 `update`/`checkCollisions`/`fireBullet`/`updateMonsters`/`pollGamepad` 与 `GAME_CONFIG` 数值字段未被改动。
  - 浏览器手动通关至少 1 关确认无报错。
- **Acceptance Criteria Addressed**: AC-9, AC-10
- **Test Requirements**:
  - `rule` TR-9.1: Node 模拟 300 帧（含精英+Boss）无异常；玩法逻辑函数 diff 无变更。证据：测试输出 + diff。
  - `rule` TR-9.2: 浏览器通关 1 关控制台无报错。证据：浏览器验证。
