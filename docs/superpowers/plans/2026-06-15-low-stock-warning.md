# 低库存预警 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在色号仓库顶部增加可配置阈值的低库存预警条，库存 > 0 且 < 阈值的色号分两级显示预警。

**Architecture:** 纯前端单文件改动，在现有 `index.html` 中增加 CSS、预警渲染函数、设置弹窗阈值字段。预警条位于搜索栏和色号网格之间，数据从 warehouse 对象实时计算，阈值从 localStorage 读取。

**Tech Stack:** 纯 HTML + CSS + JS（无框架）

---

### Task 1: 新增 CSS 样式

**Files:**
- Modify: `index.html` — 在现有 `</style>` 前插入新 CSS

- [ ] **Step 1: 在 `.color-card .qty.low` 样式后面追加预警相关 CSS**

找到 `index.html` 中约第 280 行 `.color-card .qty.low` 规则，在其后插入：

```css
/* Warning bar */
.warning-bar {
  background: linear-gradient(135deg, #fff9e6, #fff3cd);
  border: 1px solid var(--warning);
  border-radius: var(--radius);
  padding: 10px 14px;
  margin-bottom: 10px;
  transition: all var(--transition);
}
.warning-bar-header {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}
.warning-bar-title {
  font-size: 13px;
  font-weight: 700;
}
.warning-bar-summary {
  font-size: 11px;
  color: #856404;
}
.warning-bar-toggle {
  margin-left: auto;
  font-size: 11px;
  color: #999;
  cursor: pointer;
  user-select: none;
  background: none;
  border: none;
  font-family: inherit;
}
.warning-bar-toggle:hover { color: #666; }
.warning-chips {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  margin-top: 6px;
}
.warning-bar.collapsed .warning-chips {
  display: none;
}
.warning-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  border-radius: 14px;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all var(--transition);
  white-space: nowrap;
}
.warning-chip:hover {
  transform: scale(1.05);
  box-shadow: var(--shadow);
}
.warning-chip.severe {
  background: #ffe0e0;
  border-color: #e8c0c0;
}
.warning-chip.low-warn {
  background: #fff3cd;
  border-color: #e8d080;
}
.warning-chip .wc-swatch {
  width: 12px;
  height: 12px;
  border-radius: 3px;
  border: 1px solid rgba(0,0,0,0.1);
  flex-shrink: 0;
}

/* Card highlight flash for scroll-to */
.color-card.highlight {
  animation: card-flash 0.5s ease 3;
}
@keyframes card-flash {
  50% { background: #d0e4ff; border-color: var(--primary); box-shadow: 0 0 8px rgba(74,144,217,0.4); }
}
```

- [ ] **Step 2: 验证 CSS 无语法错误**

在浏览器中打开 `index.html`，检查开发者工具 Console 无 CSS 报错。

---

### Task 2: 新增 JS 辅助函数

**Files:**
- Modify: `index.html` — 在 `<script>` 标签内、`init()` 函数之前插入新函数

- [ ] **Step 1: 在 `lastSnapshot` 变量声明附近新增阈值 key 常量**

在 `const STORAGE_KEY_REPO = 'bead_github_repo';` 后面添加：

```js
const STORAGE_KEY_WARNING_THRESHOLD = 'bead_warning_threshold';
```

- [ ] **Step 2: 在 `findNearbyColors` 函数后面插入预警相关函数**

```js
// ===== LOW STOCK WARNING =====
function getWarningThreshold() {
  const v = parseInt(localStorage.getItem(STORAGE_KEY_WARNING_THRESHOLD));
  return (v > 0) ? v : 200;
}

function getLowStockColors(threshold) {
  const results = [];
  for (const c of MARD_COLORS) {
    const qty = warehouse[c.code] || 0;
    if (qty > 0 && qty < threshold) {
      results.push({ code: c.code, r: c.r, g: c.g, b: c.b, qty });
    }
  }
  results.sort((a, b) => a.qty - b.qty);
  return results;
}

function renderWarningBar() {
  const panel = document.getElementById('panel-warehouse');
  let bar = document.getElementById('warning-bar');
  const search = document.getElementById('warehouse-search').value.toLowerCase();

  const threshold = getWarningThreshold();
  let lowStock = getLowStockColors(threshold);

  // Filter by search if active
  if (search) {
    lowStock = lowStock.filter(c => c.code.toLowerCase().includes(search));
  }

  // Remove old bar if no warnings
  if (lowStock.length === 0) {
    if (bar) bar.remove();
    return;
  }

  // Create bar if not exists
  if (!bar) {
    bar = document.createElement('div');
    bar.id = 'warning-bar';
    bar.className = 'warning-bar';
    // Restore collapsed state
    if (localStorage.getItem('bead_warning_collapsed') === '1') {
      bar.classList.add('collapsed');
    }
    // Insert after controls, before grid
    const grid = document.getElementById('warehouse-grid');
    grid.parentNode.insertBefore(bar, grid);
  }

  const severeThreshold = Math.floor(threshold * 0.5);
  const collapsed = bar.classList.contains('collapsed');

  bar.innerHTML = `
    <div class="warning-bar-header">
      <span class="warning-bar-title">⚠ 低库存预警</span>
      <span class="warning-bar-summary">（阈值: ${threshold}颗，共 ${lowStock.length} 个色号不足）</span>
      <button class="warning-bar-toggle" onclick="toggleWarningBar()">${collapsed ? '▼ 展开' : '▲ 收起'}</button>
    </div>
    <div class="warning-chips">
      ${lowStock.map(c => {
        const cls = c.qty < severeThreshold ? 'severe' : 'low-warn';
        return `<span class="warning-chip ${cls}" onclick="scrollToColor('${c.code}')" title="${c.code} 仅剩 ${c.qty} 颗">
          <span class="wc-swatch" style="background:rgb(${c.r},${c.g},${c.b})"></span>
          ${c.code} (${c.qty})
        </span>`;
      }).join('')}
    </div>
  `;
}

function toggleWarningBar() {
  const bar = document.getElementById('warning-bar');
  if (!bar) return;
  bar.classList.toggle('collapsed');
  localStorage.setItem('bead_warning_collapsed', bar.classList.contains('collapsed') ? '1' : '0');
  renderWarningBar(); // refresh toggle text
}

function scrollToColor(code) {
  // Switch to warehouse tab if not already
  switchTab('warehouse');

  // Find the card element
  const cards = document.querySelectorAll('#warehouse-grid .color-card');
  for (const card of cards) {
    if (card.querySelector('.code')?.textContent === code) {
      card.scrollIntoView({ behavior: 'smooth', block: 'center' });
      card.classList.add('highlight');
      setTimeout(() => card.classList.remove('highlight'), 1500);
      break;
    }
  }
}
```

- [ ] **Step 3: 检查语法，确认函数无报错**

在浏览器 Console 中执行 `getWarningThreshold()` 应返回 200。

---

### Task 3: 修改 `renderWarehouse()` 调用预警渲染

**Files:**
- Modify: `index.html` — `renderWarehouse()` 函数

- [ ] **Step 1: 在 `renderWarehouse` 函数末尾，`grid.innerHTML = ...` 之后加上预警渲染调用**

找到 `renderWarehouse()` 函数（约第 1352 行），在函数末尾 `grid.innerHTML = filtered.map(...).join('');` 之后添加：

```js
  renderWarningBar();
```

完整修改后的函数结尾：

```js
function renderWarehouse() {
  const grid = document.getElementById('warehouse-grid');
  const search = document.getElementById('warehouse-search').value.toLowerCase();

  const filtered = MARD_COLORS.filter(c => {
    if (!search) return true;
    return c.code.toLowerCase().includes(search);
  });

  grid.innerHTML = filtered.map(c => {
    const qty = warehouse[c.code] || 0;
    let qtyClass = '';
    if (qty < 0) qtyClass = 'negative';
    else if (qty > 0 && qty < 20) qtyClass = 'low';

    return `
      <div class="color-card" onclick="openEditModal('${c.code}')" title="${c.code} - 库存: ${qty}">
        <div class="swatch" style="background:rgb(${c.r},${c.g},${c.b})"></div>
        <div class="code">${c.code}</div>
        <div class="qty ${qtyClass}">${qty}</div>
      </div>
    `;
  }).join('');

  renderWarningBar();
}
```

- [ ] **Step 2: 验证：修改库存后预警条自动刷新**

在浏览器中打开页面，编辑某个色号库存为 50，确认预警条出现该色号。

---

### Task 4: 修改设置弹窗，增加阈值字段

**Files:**
- Modify: `index.html` — `#setup-modal` HTML 和 `showSetupModal()`、`saveSetup()` 函数

- [ ] **Step 1: 在 setup-modal 的 HTML 中，标题和 GitHub Token 标签之间插入阈值字段**

找到 `#setup-modal` 的 HTML（约第 508 行），将：

```html
    <h2>⚙️ 配置 GitHub 同步</h2>
    <p>需要 GitHub Personal Access Token 来读写云端仓库中的库存数据。</p>
```

改为：

```html
    <h2>⚙️ 配置</h2>
    <p>GitHub 同步设置和库存预警阈值。</p>
```

并在 `<p>...</p>` 说明文字后面、`<ol>` 前面插入预警阈值输入区：

```html
    <label>⚠ 低库存预警阈值</label>
    <div style="display:flex;align-items:center;gap:6px;">
      <input type="number" id="setup-threshold" value="200" min="1" max="99999" style="width:100px;">
      <span style="font-size:14px;">颗</span>
    </div>
    <p style="font-size:11px;color:var(--text-secondary);margin-top:2px;">库存低于此数量且不为 0 的色号将显示预警</p>
```

放在 `<ol>` 前面，形成：
```
标题 + 说明 → 阈值字段 → GitHub 设置说明 → Token → 仓库 → 按钮
```

- [ ] **Step 2: 修改 `showSetupModal()` 预填阈值**

在 `showSetupModal()` 函数（约第 881 行）中，在现有赋值语句后添加：

```js
  document.getElementById('setup-threshold').value = getWarningThreshold();
```

完整函数：

```js
function showSetupModal() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN) || '';
  const repo = localStorage.getItem(STORAGE_KEY_REPO) || '';
  document.getElementById('setup-token').value = token;
  document.getElementById('setup-repo').value = repo;
  document.getElementById('setup-threshold').value = getWarningThreshold();
  document.getElementById('setup-modal').classList.add('show');
}
```

- [ ] **Step 3: 修改 `saveSetup()` 保存阈值**

在 `saveSetup()` 函数中，在现有 `localStorage.setItem` 调用后添加阈值保存：

```js
  const threshold = parseInt(document.getElementById('setup-threshold').value) || 200;
  localStorage.setItem(STORAGE_KEY_WARNING_THRESHOLD, threshold);
```

完整函数：

```js
function saveSetup() {
  const token = document.getElementById('setup-token').value.trim();
  const repo = document.getElementById('setup-repo').value.trim();

  if (!token || !repo) {
    alert('请填写 Token 和仓库路径，或点击"跳过"使用本地模式');
    return;
  }

  localStorage.setItem(STORAGE_KEY_TOKEN, token);
  localStorage.setItem(STORAGE_KEY_REPO, repo);

  const threshold = parseInt(document.getElementById('setup-threshold').value) || 200;
  localStorage.setItem(STORAGE_KEY_WARNING_THRESHOLD, threshold);

  document.getElementById('setup-modal').classList.remove('show');
  updateStatusBar();
  renderWarehouse(); // refresh warning bar with new threshold
  toast('设置已保存！');
}
```

- [ ] **Step 4: 修改 `skipSetup()` 同样保存阈值**

```js
function skipSetup() {
  const threshold = parseInt(document.getElementById('setup-threshold').value) || 200;
  localStorage.setItem(STORAGE_KEY_WARNING_THRESHOLD, threshold);
  document.getElementById('setup-modal').classList.remove('show');
  updateStatusBar();
  renderWarehouse();
}
```

- [ ] **Step 5: 验证设置弹窗**

打开页面 → 点击设置 → 阈值默认 200 → 改为 100 → 保存。编辑某色号库存为 50，确认预警条按阈值 100 显示。

---

### Task 5: 端到端验证 + 提交

- [ ] **Step 1: 打开 `index.html` 在浏览器中完整测试**

测试清单：
1. 首次加载：阈值默认 200，验证 `getWarningThreshold()` 返回 200
2. 预警条显示：编辑某色号为 50，预警条出现，显示严重（红色芯片）
3. 预警条不显示 0：库存为 0 的色号不出现在预警中
4. 预警条分级：50 颗以下红色，100~199 黄色
5. 点击芯片：滚动到对应色号卡片，卡片闪烁高亮
6. 收起/展开：点击收起，刷新页面后保持折叠状态
7. 搜索联动：搜索"A1"，预警只显示匹配的 A1 相关色号
8. 设置阈值：设为 50，预警条按新阈值刷新
9. 无预警时隐藏：所有色号库存 > 阈值时预警条消失
10. 云端加载/CSV导入/扣除后预警自动刷新

- [ ] **Step 2: 提交代码**

```bash
git add index.html
git commit -m "feat: add low-stock warning bar with configurable threshold"
```
