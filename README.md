# Warring States: Grid Hegemony Simulator

> A pure front-end strategy sandbox simulator of the Warring States period. Seven powers clash on a 30×30 grid, with AI engaging in autonomous gameplay while players can intervene to alter the course of history.

## Project Overview

*Warring States* is a grid-based strategy simulator set in the ancient Chinese Warring States era. The seven states of Qin, Zhao, Yan, Qi, Chu, Han, and Wei engage in autonomous gameplay on a 30×30 grid sandbox: internal administration, diplomatic alliances, military conquests, state collapse and partition, and reunification—all driven by AI decision-making. Players observe the simulation from a "Mandate of Heaven" perspective and can click on grid cells to deploy troops and intervene in the unfolding conflict.

The entire project is built as a **single HTML file using vanilla JavaScript and Canvas**, with zero dependencies. Simply open it in a browser to play.

---

## Core Features

### 1. Deep AI Strategy

AI decision-making does not rely on a single metric but uses a 5-dimensional scoring system to select primary targets:

| Dimension | Weight | Description |
|------|------|------|
| Power Gap | ×1.0 | Prioritize attacking weaker states |
| Geopolitical Value | ×25 | Weighted strategic value of passes, granaries, cities, and capitals |
| Border Length | ×3 | Longer shared borders increase invasion likelihood |
| Vulnerable Zones | ×8 | Enemy anchor points far from their frontlines indicate weak defense |
| Allied Coordination | ×80 | Prioritize joining a siege if an ally has already declared war on the target |

- **Single-Front AI**: Non-dominant powers concentrate attacks on a single front; dominant powers can advance on multiple fronts.
- **Primary Attack Anchor**: Once a target is selected, an anchor is set at the army's rally point. The AI actively attacks within a 4-cell radius, while other borders only have a 25% chance of self-defense.
- **Stalemate Breaking**: Forces an attack during prolonged periods of no warfare; late-game fatigue coefficients accelerate decisive battles.
- **General Command**: Influences military power; new generals can be recruited every 100 turns.

### 2. Diplomatic System

- **Multiple Alliances**: Each state can have up to 2 allies, with alliance durations ranging from 150 to 300 turns (Vertical Alliance: 250-300; Horizontal Alliance: 150-220).
- **Neutrality**: Neither allied nor at war; neutral states have a 90% chance of not attacking each other.
- **Coordinated Sieges**: Dominant powers rally allies to declare war on the same target.
- **Defense Requests**: Attacked states notify allies; if an ally shares a border with the attacker, they may send reinforcements based on their diplomatic persona probability.
- **Dynamic Hatred**: +12 for occupying territory, +25 for non-war targets; hatred decays over time.

### 3. Military System

- **Troop Visualization**: Darker shades of the same color indicate stronger troop concentrations, based on the distance to the frontline (0 = darkest, ≥6 = lightest), with moderate darkening around capitals.
- **March Delay + Trajectory Animation**: Troop deployment is not instantaneous; it takes N turns to arrive (1 turn per 3 cells). The dynamic layer draws dashed trajectories, moving troop dots, and flowing particles in real-time.
- **City Victory Formula**: `P = (A/B)^1.5 / ((A/B)^1.5 + 1)`, amplifying the power gap; overwhelming national power leads to direct crushing.
- **Rout Mechanism**: Losing 2 consecutive cells triggers a rout; weaker states have lower thresholds and higher probabilities.
- **Three-Dimensional National Power**: Economic Power (Territory + Vassal Cities + Grain) × 0.4 + Military Power (Troops × Commander) × 0.5 + Morale × 0.1.

### 4. State Collapse and Reunification

- **Smart Capital Relocation**: Triggered when the capital is threatened (Enemy distance ≤1 and Enemy count ≥2, or Power Ratio <0.6).
  - 4-Dimensional Candidate Scoring: Distance from enemy ×3 + Distance from frontline ×2 + Plains hinterland ×1.5 + Moderate distance from old capital ×0.5.
  - 30-turn cooldown, costs 150 grain, -8 morale, and 2/3 of the garrison relocates.
- **Partition on Collapse**: Upon the capital's fall, remaining territories are clustered into 1-3 minor states using k-means, with 5 naming prefixes (Later/Puppet/South/North/Remnant) randomly shuffled.
  - Partitioned states can only be split once (they do not split further if destroyed again).
- **Reunification Mechanism**: If a partitioned minor state recaptures the original capital, it consolidates all同源 (same-origin) partitioned states and re-emerges as a major power.

### 5. Dynamic City Building & Historical Events

- **Dynamic City Building**: Every 100 turns, each state can build one vassal city (fortress + defense bonus). It is automatically removed if captured.
- **Historical Events** (60% trigger rate every 50 turns):
  - Shang Yang's Reforms (Economy +30%, Morale -20%)
  - Natural Disaster (Grain -200, Morale -15%)
  - Enlightened Monarch (Morale +25, Troops +100)
  - Bumper Harvest (Grain +400)
  - Renowned General Arrives (Command +70 to 90)
  - Internal Rebellion (Morale -30, Troops -80)

### 6. Terrain & Strategic Elements

- **Plains**: Base economic +10.
- **Mountains**: Occupiable, +50% defense.
- **Lakes**: Impassable.
- **Capital ★**: Loss of the capital means state collapse.
- **Vassal City ●**: Economic bonus; removed upon capture.
- **Granary ▲**: Yields 300 grain upon capture.
- **Passes**: Terrain with defense bonuses.

### 7. Player Intervention

- **Click Cell to Deploy Troops (+1000)**: Adds troops to the owning state of the cell (syncs `troops` and `garrison`).
- **Hover for Details**: Displays real-time terrain, ownership, garrison, and combat status.
- **State Details Panel**: Shows national power, morale, frontlines, troop movements, alliances, hatred, and strategic objectives.

---

## Technical Architecture

### Rendering Layer (Dual Canvas)

- **Static Layer**: Terrain + national colors (troop depth) + city/capital/granary markers + grid lines + national borders.
  - Dirty-cell incremental updates + neighbor redraw mechanism (avoids border overlapping).
  - Unified `drawGridLines` single-pass rendering with 0.5px pixel alignment.
- **Dynamic Layer**: Combat flashing + alliance dashed lines + march trajectories + particles; redrawn every frame.
- **Alliance Lines**: Short dashed lines for adjacent borders; Bezier curve offsets for long-distance to avoid overlapping; red flashing when expiring.

### ES6+ Class Architecture

```
BattleEngine     - Combat simulation / AI decision-making / march management
DiplomacyManager - Diplomatic evaluation / alliances / coordinated sieges
Renderer         - Dual-layer rendering / animations
UIPanel          - Power rankings / battle logs / details
App              - Entry point / event binding / game loop
```

### Performance Optimization

- Dual-layer rendering + dirty-cell incremental updates.
- Particle ring buffer (max 280, overwriting reuse).
- rAF throttled rendering loop.
- Old rendering loop auto-stops (after reset, verified via renderer reference comparison).
- BFS distance-to-frontline calculation with a radius-6 scan for approximate enemy distance.

---

## Gameplay Instructions

1. **▶ Start**: Begin automatic simulation.
2. **⏭ Next Turn**: Manually advance one turn (for observing details).
3. **↻ Reset**: Randomly regenerate the starting positions of the seven states (every reset creates a new game).
4. **Speed Slider**: Exponential curve mapping from 30ms to 2000ms.
5. **Click Cell**: Deploy +1000 troops for that state (intervene in the war).
6. **Hover**: View information for any cell; automatically selects the owning state for detailed viewing.

---

## Engineering Highlights

- **Single File**: Pure HTML + JS + Canvas, zero dependencies, fully offline-capable.
- **Historical Aesthetic**: Dark gold theme, Song-typeface state names, classical Chinese battle logs, ink-wash sandbox style.
- **Dynamic Balance**: Late-game fatigue coefficients (≤4 states: 1.4 / ≤3 states: 1.8 / ≤2 states: 2.5) force decisive battles and prevent stalemates.
- **Meticulous Details**: Pixel-aligned grid lines, anti-aliasing, dirty neighbor redrawing, and isolated rendering side effects.

---

## Seven States Configuration

The seven states spawn randomly within their respective historical regional zones. They may spawn closer to the Central Plains (dangerous but advantageous for expansion) or further back (safe but marginalized):

| State | Region |
|----|------|
| Qin | Northwest (Guanzhong → Central Plains) |
| Zhao | North (Dai Region → Central Plains) |
| Yan | Northeast (Ji-Liao → Central Plains) |
| Qi | East (Qi-Lu → Central Plains) |
| Chu | Southeast (Jing-Chu → Central Plains) |
| Han | South (Han Territory → Central Plains) |
| Wei | Southwest (Wei-Liang → Central Plains) |

---

## Tech Stack

- HTML5 Canvas (Dual-layer rendering)
- Vanilla JavaScript ES6+ (Class architecture)
- Zero third-party dependencies

---

## Use Cases

- Strategy game prototype design reference
- AI decision-making system teaching example
- Canvas performance optimization practice
- Warring States historical sandbox visualization

---

Open `index.html` to begin the struggle for hegemony. Unite the realm, and forge your empire.


# 战国七雄 · 网格争霸模拟器

> 一个基于纯前端实现的战国策略沙盘模拟器，30×30 网格上七国争锋，AI 自主博弈，玩家可干预战局。

## 项目简介

《战国七雄》是一款以战国时代为背景的网格化策略模拟器。秦、赵、燕、齐、楚、韩、魏七国在 30×30 的网格沙盘上自主博弈：内政发展、外交结盟、军事征伐、亡国分裂、复国一统——全部由 AI 决策，玩家以"天命"视角观察推演，并可点击格子增兵以干预战局。

整个项目为**单文件 HTML + 原生 JavaScript + Canvas**，无任何依赖，浏览器打开即用。

---

## 核心特色

### 一、深度 AI 战略

AI 决策不依赖单一指标，而是综合 5 维评分选择主攻目标：

| 维度 | 权重 | 说明 |
|------|------|------|
| 国力差距 | ×1.0 | 优先攻弱国 |
| 地缘价值 | ×25 | 关隘/粮仓/城池/首都战略价值加权 |
| 接壤长度 | ×3 | 接壤越长越易入侵 |
| 薄弱区 | ×8 | 敌方锚点远离己方前线 → 防御薄弱 |
| 盟友协同 | ×80 | 盟友已对此国宣战 → 优先加入围攻 |

- **单线作战 AI**：非强国只集中攻一个方向，强国可多线推进
- **主攻锚点机制**：选定目标后在大军集结处设锚点，半径 4 格内主动进攻，其余边境仅 25% 概率自卫
- **僵局打破**：长期无战事时强制进攻；后期疲劳系数加速决战
- **将领统帅力**：影响军事力，每 100 回合可招募新将

### 二、外交系统

- **多盟约**：每国最多 2 个盟友，联盟时效 150-300 回合（合纵 250-300，连横 150-220）
- **中立关系**：既不结盟也不征战，中立国间 90% 概率不互攻
- **盟国协同围攻**：强国号召盟国对同一目标宣战
- **防御求援**：被攻击方通知盟友，盟友与攻击者接壤时按 persona.diplo 概率出兵支援
- **仇恨动态**：侵占领土 +12，非战争目标 +25，仇恨衰减

### 三、军事系统

- **兵力可视化**：同色越深兵力越强，基于距前线距离（0=最深，≥6=最浅），国都周边适度加深
- **行军延迟+轨迹动画**：调兵非瞬间，需 N 回合抵达（每 3 格 1 回合）。动态层实时绘制虚线轨迹 + 移动兵力点 + 流动小点
- **城池战胜率公式**：`P = (A/B)^1.5 / ((A/B)^1.5 + 1)`，扩大国力差距体现；压倒性国力直接碾压
- **溃败机制**：连失 2 格触发溃败，弱国阈值更低、概率更高
- **三维国力**：经济力（领土+别城+粮草）×0.4 + 军事力（兵力×统帅）×0.5 + 民心×0.1

### 四、亡国与复国

- **智能迁都**：首都受威胁时（距敌 ≤1 且敌方 ≥2 或国力比 <0.6）触发迁都评估
  - 4 维候选评分：距敌远×3 + 距前线远×2 + 平原腹地×1.5 + 距旧都适中×0.5
  - 30 回合冷却，消耗 150 粮草，民心 -8，2/3 驻军迁移
- **灭国分裂**：首都陷落后剩余领土按 k-means 聚为 1-3 个小国，5 种命名前缀（后/伪/南/北/残）随机洗牌
  - 分裂国最多迭代一次（被灭后不再分裂）
- **复国机制**：分裂小国夺回原国都 → 整合同源分裂国，重新成为大国

### 五、动态建城与历史事件

- **动态建城**：每 100 回合每国可建一座别城（要塞+防御加成），被攻占后自动消除
- **历史事件**（每 50 回合 60% 触发）：
  - 商鞅变法（经济+30%，民心-20%）
  - 天灾（粮草-200，民心-15%）
  - 明君（民心+25，兵力+100）
  - 丰收（粮草+400）
  - 名将投奔（统帅力+70-90）
  - 内乱（民心-30，兵力-80）

### 六、地形与战略要素

- **平原**：基础经济 +10
- **山脉**：可占领，防御 +50%
- **湖泊**：不可通行
- **都城 ★**：首都陷落即亡国
- **别城 ●**：经济加成，被占后消除
- **粮仓 ▲**：占领得粮三百石
- **关隘**：防御加成地

### 七、玩家干预

- **点击格子增兵 +1000**：为该格归属国增兵（`troops` 和 `garrison` 同步增加）
- **hover 查看详情**：实时显示地形、归属、驻军、交战状态
- **诸侯详情面板**：国力、民心、前线、行军、盟约、仇恨、战略目标等

---

## 技术架构

### 渲染层（双层 Canvas）

- **静态层**：地形 + 国家色（兵力深浅）+ 城市/首都/粮仓标记 + 网格线 + 国家边界
  - dirty 格增量更新 + 邻居重画机制（避免边界叠加）
  - 统一 `drawGridLines` 单次绘制，0.5px 像素对齐
- **动态层**：交战闪烁 + 联盟虚线 + 行军轨迹 + 粒子，每帧重画
- **联盟线**：接壤=短虚线，远距离=贝塞尔曲线偏移避免重叠；即将过期=红色闪烁

### ES6+ Class 架构

```
BattleEngine   - 战斗推演/AI 决策/行军管理
DiplomacyManager - 外交评估/盟约/协同围攻
Renderer       - 双层渲染/动画
UIPanel        - 国力榜/战报/详情
App            - 入口/事件绑定/游戏循环
```

### 性能优化

- 双层渲染 + dirty 格增量更新
- 粒子环形缓冲（最大 280，覆盖式复用）
- rAF 节流渲染循环
- 旧渲染循环自动停止（reset 后通过 renderer 引用比较）
- BFS 距前线距离计算，半径 6 扫描近似敌距

---

## 玩法说明

1. **▶ 启动**：开始自动推演
2. **⏭ 单回合**：手动推进一回合（便于观察细节）
3. **↻ 重置**：重新随机生成七国出生点（每次重置都是新棋局）
4. **速度滑块**：30ms-2000ms 指数曲线映射
5. **点击格子**：为该国增兵 +1000（干预战局）
6. **hover**：查看任意格子信息，自动选中归属国查看详情

---

## 工程亮点

- **单文件**：纯 HTML + JS + Canvas，零依赖，离线可用
- **历史风格**：暗黑金主题、宋体国号、文言战报、墨色沙盘
- **动态平衡**：后期疲劳系数（≤4国1.4 / ≤3国1.8 / ≤2国2.5）强制决战，避免僵局
- **细节考究**：网格线像素对齐、抗锯齿处理、dirty 邻居重画、渲染副作用隔离

---

## 七国配置

七国在各自历史方向的大区域内随机出生，可能靠前（近中原，危险但便于扩张）或靠后（远中原，安全但被边缘化）：

| 国 | 区域 |
|----|------|
| 秦 | 西北（关中→中原） |
| 赵 | 北方（代地→中原） |
| 燕 | 东北（蓟辽→中原） |
| 齐 | 东方（齐鲁→中原） |
| 楚 | 东南（荆楚→中原） |
| 韩 | 南方（韩地→中原） |
| 魏 | 西南（魏梁→中原） |

---

## 技术栈

- HTML5 Canvas（双层渲染）
- 原生 JavaScript ES6+（Class 架构）
- 无任何第三方依赖

---

## 适用场景

- 策略游戏原型设计参考
- AI 决策系统教学示例
- Canvas 性能优化实践
- 战国历史沙盘可视化

---

打开 `index.html` 即可开始争霸。一统天下，霸业可成。
