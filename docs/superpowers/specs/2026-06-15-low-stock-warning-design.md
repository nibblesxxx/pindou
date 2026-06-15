# 低库存预警功能设计

**日期**: 2026-06-15
**状态**: 已批准

## 概述

在色号仓库顶部增加低库存预警条，当某个色号库存低于用户设定阈值（默认 200 颗）**且库存 > 0** 时显示预警。库存为 0 的色号视为未持有，不预警。阈值可在设置弹窗中调整。

## 功能需求

### 1. 预警阈值设置

- 默认阈值：200 颗
- 在现有 GitHub 设置弹窗（`#setup-modal`）中增加"低库存预警阈值"输入字段
- 阈值存储到 localStorage（key: `bead_warning_threshold`）
- 阈值变更后即时刷新预警条

### 2. 顶部预警条

- 位置：搜索栏下方、色号网格上方
- 无预警色号时完全隐藏
- 只预警库存 > 0 且 < 阈值的色号（库存为 0 表示未持有，不预警）
- 显示低库存色号芯片列表，分两级：
  - **严重不足**（红色背景 `#ffe0e0`）：库存 < 阈值的 50%
  - **偏低**（黄色背景 `#fff3cd`）：库存 < 阈值但 ≥ 50%
- 每个芯片显示：色号颜色方块 + 色号代码 + 当前库存数
- 芯片可点击，点击后滚动到对应色号卡片并短暂高亮

### 3. 预警条交互

- **可折叠**：右侧有收起/展开按钮，折叠状态存储到 localStorage
- **搜索联动**：搜索过滤时，预警只统计当前可见（匹配搜索）的色号
- **动态更新**：编辑库存、导入CSV、云端加载、扣除后自动刷新预警

### 4. 色号卡片更新

- 低库存色号卡片增加 `.low-stock` 样式标志
- 卡片上库存数字用橙色/红色显示（复用现有 `.low` 和 `.negative` 类，扩展到 200 阈值区间）

## 技术实现

### 新增函数

| 函数 | 职责 |
|------|------|
| `getWarningThreshold()` | 从 localStorage 读取阈值，默认 200 |
| `getLowStockColors(threshold)` | 筛选 0 < 库存 < 阈值的色号，按库存升序排列 |
| `renderWarningBar()` | 渲染/更新预警条 DOM |
| `scrollToColor(code)` | 滚动到指定色号卡片并高亮闪烁 |

### 修改函数

| 函数 | 改动 |
|------|------|
| `renderWarehouse()` | 在网格前插入预警条容器，更新卡片预警样式 |
| `showSetupModal()` | 预填阈值字段 |
| `saveSetup()` | 保存阈值到 localStorage |
| `init()` | 初始化阈值默认值 |

### 新增 CSS

```css
/* 预警条容器 */
.warning-bar { ... }
.warning-bar.collapsed .warning-chips { display: none; }

/* 预警芯片 */
.warning-chip { display: inline-flex; align-items: center; gap: 4px; padding: 4px 10px; border-radius: 14px; cursor: pointer; }
.warning-chip.severe { background: #ffe0e0; border: 1px solid #e8c0c0; }
.warning-chip.low { background: #fff3cd; border: 1px solid #e8d080; }

/* 卡片高亮闪烁 */
.color-card.highlight { animation: flash 0.6s ease 3; border-color: var(--primary); }
@keyframes flash { 50% { background: #e8f0ff; } }
```

## 数据流

```
localStorage('bead_warning_threshold') → getWarningThreshold()
  → getLowStockColors(threshold) → renderWarningBar()
  → 用户点击芯片 → scrollToColor(code) → 卡片高亮
```

## 影响范围

- 仅修改 `index.html`（单文件应用）
- 不涉及 CSV 格式变更
- 不影响 GitHub 同步逻辑
- 向后兼容：无阈值设置时默认 200
