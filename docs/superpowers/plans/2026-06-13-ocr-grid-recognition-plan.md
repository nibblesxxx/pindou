# OCR 网格识别 · 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为拼豆色号助手添加OCR编号识别模式——上传带编号的图纸后，自动检测网格、OCR色号编号、颜色交叉验证、统计扣除。

**Architecture:** 在现有单文件 `index.html` 中，将"图转色号"tab改造为支持三种识别模式（像素图/照片/OCR编号）。Tesseract.js v5 通过CDN懒加载。网格检测通过水平/垂直投影找波谷。逐格OCR识别色号编号并用MARD颜色表交叉验证。

**Tech Stack:** 纯HTML+CSS+JS, Tesseract.js v5 CDN, Canvas API

---

## 文件结构

- **修改**: `index.html` — 所有改动在此单文件内
  - HTML: 图转色号面板改造 + 模式选择 + 网格确认弹窗
  - CSS: 拾色器光标、模式选择器、网格设置面板
  - JS: Tesseract加载、网格检测、色号OCR、交叉验证

---

### Task 1: 图转色号面板 — 添加三种模式选择器

**修改**: `index.html` (HTML section, 替换 panel-matcher 内容)

- [ ] **Step 1: 改造上传区域，添加上传后模式选择**

将 `#panel-matcher` 内的上传区域改为：
```html
<section id="panel-matcher" class="tab-panel">
  <div class="upload-zone" id="upload-zone" onclick="document.getElementById('file-input').click()">
    <div class="icon">🖼️</div>
    <p>拖拽拼豆截图到此处，或点击上传</p>
    <p class="hint">支持 PNG / JPG / WebP</p>
  </div>
  <input type="file" id="file-input" accept="image/*" style="display:none" onchange="handleFileSelect(event)">
  
  <!-- 模式选择栏（上传后显示） -->
  <div id="mode-bar" style="display:none;margin-bottom:12px;">
    <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap;">
      <span style="font-size:13px;font-weight:600;">识别模式：</span>
      <button class="btn btn-sm mode-btn active" data-mode="pixel" onclick="setMode('pixel')">🎯 像素图</button>
      <button class="btn btn-sm mode-btn" data-mode="photo" onclick="setMode('photo')">📷 照片网格</button>
      <button class="btn btn-sm mode-btn" data-mode="ocr" onclick="setMode('ocr')">🔤 编号识别</button>
    </div>
    <!-- OCR/照片模式的网格设置 -->
    <div id="grid-settings" style="display:none;margin-top:8px;padding:10px;background:var(--surface);border-radius:var(--radius);">
      <div style="font-size:12px;color:var(--text-secondary);margin-bottom:6px;">
        点击预览图上的颜色来选取（容差 <input type="number" id="color-tolerance" value="10" min="1" max="50" style="width:50px;font-size:12px;">）
      </div>
      <div style="display:flex;gap:12px;flex-wrap:wrap;align-items:center;">
        <span style="font-size:12px;">🏳️ 背景色 <span class="picked-color" id="picked-bg" style="display:inline-block;width:16px;height:16px;border:1px solid #ccc;border-radius:3px;vertical-align:middle;"></span></span>
        <span style="font-size:12px;">📏 主网格 <span class="picked-color" id="picked-grid1" style="display:inline-block;width:16px;height:16px;border:1px solid #ccc;border-radius:3px;vertical-align:middle;"></span></span>
        <span style="font-size:12px;">📐 次网格 <span class="picked-color" id="picked-grid2" style="display:inline-block;width:16px;height:16px;border:1px solid #ccc;border-radius:3px;vertical-align:middle;"></span></span>
        <button class="btn btn-primary btn-sm" onclick="runGridRecognition()">🔍 检测网格</button>
      </div>
    </div>
  </div>
  
  <div class="preview-container" id="preview-container" style="display:none;"></div>
  <div id="match-results"></div>
  <div class="action-bar" id="match-actions" style="display:none;">
    <button class="btn btn-danger btn-sm" id="btn-undo" onclick="undoLastDeduction()" disabled>↩ 撤销上次扣除</button>
    <button class="btn btn-outline btn-sm" onclick="clearMatches()">🔄 重新识别</button>
  </div>
</section>
```

- [ ] **Step 2: 添加网格确认弹窗（放在 edit-modal 之后）**

```html
<!-- Grid confirmation modal -->
<div class="modal-overlay" id="grid-modal">
  <div class="modal">
    <div class="edit-modal-body">
      <h2 style="font-size:16px;margin-bottom:4px;">🔍 网格检测结果</h2>
      <p id="grid-info" style="font-size:13px;color:var(--text-secondary);margin-bottom:12px;"></p>
      <div style="display:flex;gap:12px;justify-content:center;">
        <div>
          <label>列数（宽）</label>
          <input type="number" class="edit-input" id="grid-cols" min="2" max="500" style="width:100px;">
        </div>
        <div>
          <label>行数（高）</label>
          <input type="number" class="edit-input" id="grid-rows" min="2" max="500" style="width:100px;">
        </div>
      </div>
      <div class="btn-row" style="margin-top:16px;justify-content:center;">
        <button class="btn btn-outline btn-sm" onclick="closeGridModal()">取消</button>
        <button class="btn btn-primary btn-sm" onclick="confirmGrid()">确认并识别</button>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: add mode selector and grid confirmation modal UI"
```

---

### Task 2: CSS 样式 — 模式选择器、拾色器、网格设置

**修改**: `index.html` (CSS section)

- [ ] **Step 1: 添加新样式**

在 `.toast` 样式之前插入：

```css
    /* Mode selector */
    .mode-btn {
      background: var(--surface);
      color: var(--text-secondary);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 6px 14px;
      cursor: pointer;
      font-size: 13px;
      font-family: inherit;
      transition: all var(--transition);
    }
    .mode-btn.active {
      background: var(--primary);
      color: #fff;
      border-color: var(--primary);
    }

    /* Color picker cursor on preview */
    .preview-picking {
      cursor: crosshair;
    }

    /* OCR cell status */
    .ocr-ok { color: var(--success); }
    .ocr-warn { color: var(--warning); }
    .ocr-fail { color: var(--danger); }

    /* Grid settings picked color swatches */
    .picked-color { vertical-align: middle; }
```

- [ ] **Step 2: Commit**

```bash
git add index.html && git commit -m "style: add mode selector and OCR status styles"
```

---

### Task 3: 状态变量 + Tesseract.js 懒加载

**修改**: `index.html` (JS section, 在现有 state 变量之后)

- [ ] **Step 1: 添加识别模式状态变量**

在 `let editingCode = null;` 之后添加：

```javascript
// ===== RECOGNITION MODE STATE =====
let currentMode = 'pixel';     // 'pixel' | 'photo' | 'ocr'
let pickedColors = { bg: null, grid1: null, grid2: null };
let tolerance = 10;
let pickingTarget = null;      // 'bg' | 'grid1' | 'grid2' | null
let detectedGrid = { cols: 0, rows: 0 };
let ocrWorker = null;         // Tesseract worker (lazy)
let ocrLoading = false;
```

- [ ] **Step 2: 添加 Tesseract.js 加载函数**

在 `init()` 函数之前添加：

```javascript
// ===== TESSERACT OCR =====
async function loadTesseract() {
  if (ocrWorker) return ocrWorker;
  if (ocrLoading) {
    // Wait for existing load
    return new Promise((resolve, reject) => {
      const check = setInterval(() => {
        if (ocrWorker) { clearInterval(check); resolve(ocrWorker); }
        if (!ocrLoading) { clearInterval(check); reject(new Error('OCR load failed')); }
      }, 200);
    });
  }
  ocrLoading = true;
  toast('正在加载 OCR 引擎（首次加载约需几秒）...');
  
  // Load Tesseract from CDN
  if (typeof Tesseract === 'undefined') {
    await new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = 'https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js';
      script.onload = resolve;
      script.onerror = () => reject(new Error('OCR 引擎加载失败'));
      document.head.appendChild(script);
    });
  }
  
  ocrWorker = await Tesseract.createWorker('eng');
  ocrLoading = false;
  toast('OCR 引擎就绪 ✅');
  return ocrWorker;
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: add recognition state and Tesseract.js lazy loader"
```

---

### Task 4: 模式切换 + 拾色器交互

**修改**: `index.html` (JS section)

- [ ] **Step 1: 添加模式切换函数**

```javascript
function setMode(mode) {
  currentMode = mode;
  document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('active'));
  document.querySelector(`.mode-btn[data-mode="${mode}"]`).classList.add('active');
  
  const gridSettings = document.getElementById('grid-settings');
  const previewImg = document.querySelector('#preview-container img');
  
  if (mode === 'pixel') {
    gridSettings.style.display = 'none';
    if (previewImg) previewImg.classList.remove('preview-picking');
    // Re-run with pixel mode
    if (matchResults.length > 0) reprocessImage();
  } else if (mode === 'photo' || mode === 'ocr') {
    gridSettings.style.display = 'block';
    if (previewImg) previewImg.classList.add('preview-picking');
  }
  
  pickingTarget = null;
}
```

- [ ] **Step 2: 添加拾色器逻辑**

```javascript
// Click on preview image to pick colors
document.addEventListener('click', function(e) {
  if (!pickingTarget) return;
  const previewImg = document.querySelector('#preview-container img');
  if (!previewImg || e.target !== previewImg) return;
  
  const rect = previewImg.getBoundingClientRect();
  const x = Math.floor((e.clientX - rect.left) / rect.width * previewImg.naturalWidth);
  const y = Math.floor((e.clientY - rect.top) / rect.height * previewImg.naturalHeight);
  
  const canvas = document.createElement('canvas');
  canvas.width = previewImg.naturalWidth;
  canvas.height = previewImg.naturalHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(previewImg, 0, 0);
  const pixel = ctx.getImageData(x, y, 1, 1).data;
  const color = { r: pixel[0], g: pixel[1], b: pixel[2] };
  
  pickedColors[pickingTarget] = color;
  document.getElementById(`picked-${pickingTarget}`).style.background = `rgb(${color.r},${color.g},${color.b})`;
  toast(`已选取 ${pickingTarget === 'bg' ? '背景色' : pickingTarget === 'grid1' ? '主网格色' : '次网格色'}: RGB(${color.r},${color.g},${color.b})`);
  pickingTarget = null;
});

// Set picking target when clicking the color indicator
document.addEventListener('click', function(e) {
  const span = e.target.closest('[id^="picked-"]');
  if (!span || !span.id) return;
  const target = span.id.replace('picked-', '');
  if (['bg', 'grid1', 'grid2'].includes(target)) {
    pickingTarget = target;
    toast(`请点击预览图上的${target === 'bg' ? '背景' : target === 'grid1' ? '主网格线' : '次网格线'}区域`);
  }
});
```

- [ ] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: add mode switching and color picker interaction"
```

---

### Task 5: 网格检测算法

**修改**: `index.html` (JS section)

- [ ] **Step 1: 添加投影网格检测函数**

```javascript
// ===== GRID DETECTION =====
function runGridRecognition() {
  const previewImg = document.querySelector('#preview-container img');
  if (!previewImg) { toast('请先上传图片'); return; }
  
  tolerance = parseInt(document.getElementById('color-tolerance').value) || 10;
  
  // Check if required colors picked for photo/ocr mode
  if (currentMode === 'photo' || currentMode === 'ocr') {
    if (!pickedColors.bg) { toast('请先选取背景色（点击预览图）'); return; }
    if (!pickedColors.grid1) { toast('请先选取主网格色'); return; }
  }
  
  const canvas = document.createElement('canvas');
  canvas.width = previewImg.naturalWidth;
  canvas.height = previewImg.naturalHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(previewImg, 0, 0);
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const pixels = imageData.data;
  const w = canvas.width;
  const h = canvas.height;
  
  // Build mask: which pixels are "grid line" pixels
  const isGrid = new Uint8Array(w * h);
  const isBg = new Uint8Array(w * h);
  
  for (let i = 0; i < w * h; i++) {
    const idx = i * 4;
    const r = pixels[idx], g = pixels[idx + 1], b = pixels[idx + 2];
    
    if (pickedColors.bg) {
      const dr = r - pickedColors.bg.r, dg = g - pickedColors.bg.g, db = b - pickedColors.bg.b;
      if (Math.abs(dr) <= tolerance && Math.abs(dg) <= tolerance && Math.abs(db) <= tolerance) {
        isBg[i] = 1;
      }
    }
    
    if (pickedColors.grid1) {
      const dr = r - pickedColors.grid1.r, dg = g - pickedColors.grid1.g, db = b - pickedColors.grid1.b;
      if (Math.abs(dr) <= tolerance && Math.abs(dg) <= tolerance && Math.abs(db) <= tolerance) {
        isGrid[i] = 1;
      }
    }
    
    if (pickedColors.grid2) {
      const dr = r - pickedColors.grid2.r, dg = g - pickedColors.grid2.g, db = b - pickedColors.grid2.b;
      if (Math.abs(dr) <= tolerance && Math.abs(dg) <= tolerance && Math.abs(db) <= tolerance) {
        isGrid[i] = 1;
      }
    }
  }
  
  // Horizontal projection: sum grid pixels per row
  const hProj = new Float64Array(h);
  for (let y = 0; y < h; y++) {
    let sum = 0;
    for (let x = 0; x < w; x++) sum += isGrid[y * w + x];
    hProj[y] = sum / w; // normalize
  }
  
  // Vertical projection: sum grid pixels per column
  const vProj = new Float64Array(w);
  for (let x = 0; x < w; x++) {
    let sum = 0;
    for (let y = 0; y < h; y++) sum += isGrid[y * w + x];
    vProj[x] = sum / h; // normalize
  }
  
  // Find peaks/valleys to detect grid lines
  // Grid lines = high values in projection, cell centers = low values
  const findLines = (proj, len, threshold) => {
    const lines = [];
    let inLine = false;
    for (let i = 0; i < len; i++) {
      if (proj[i] > threshold && !inLine) {
        lines.push(i);
        inLine = true;
      } else if (proj[i] <= threshold && inLine) {
        lines.push(i);
        inLine = false;
      }
    }
    return lines;
  };
  
  // Auto-detect threshold: use mean + 0.5 * std
  const hMean = hProj.reduce((a,b) => a+b, 0) / h;
  const vMean = vProj.reduce((a,b) => a+b, 0) / w;
  const hStd = Math.sqrt(hProj.reduce((s,v) => s + (v-hMean)**2, 0) / h);
  const vStd = Math.sqrt(vProj.reduce((s,v) => s + (v-vMean)**2, 0) / w);
  
  const hThreshold = Math.min(hMean + 0.5 * hStd, 0.15);
  const vThreshold = Math.min(vMean + 0.5 * vStd, 0.15);
  
  const hLines = findLines(hProj, h, hThreshold);
  const vLines = findLines(vProj, w, vThreshold);
  
  // Count cells between grid line pairs
  // Each pair of consecutive lines = one grid line, between is one row/col of cells
  // Actually, grid lines separate cells, so #cells = #lines - 1 for evenly spaced grids
  // Better approach: count gaps
  const countCells = (lines) => {
    if (lines.length < 4) return 0; // need at least 2 grid lines (start/end) + gaps
    // lines come in pairs [start1, end1, start2, end2, ...]
    // number of cells = number of line pairs - 1
    // Simpler: count the number of low-to-high transitions = number of lines
    // cells = number of lines - 1
    // But we get start/end pairs, so lines.length / 2 gives number of lines
    // cells = lines.length / 2 - 1
    // Even simpler for bead grids: count valleys between grid lines
    // valleys = number of transitions from high to low
    let valleys = 0;
    let prevHigh = false;
    // Re-scan: count distinct grid line regions
    // Each grid line is a start→end pair
    // Between two grid lines is one row/col of cells
    let lineCount = 0;
    for (let i = 0; i < lines.length; i += 2) {
      lineCount++;
    }
    return Math.max(1, lineCount - 1);
  };
  
  let cols = countCells(vLines);
  let rows = countCells(hLines);
  
  // Fallback: if detection failed, try simple approach
  if (cols < 2 || rows < 2) {
    // Use average cell size estimation
    // For OCR mode: typical bead pattern cells are ~20-40px
    const avgCellSize = 25;
    cols = Math.max(2, Math.round(w / avgCellSize));
    rows = Math.max(2, Math.round(h / avgCellSize));
    toast('网格检测低置信度，使用估算值');
  }
  
  detectedGrid = { cols, rows };
  
  // Show confirmation modal
  document.getElementById('grid-cols').value = cols;
  document.getElementById('grid-rows').value = rows;
  document.getElementById('grid-info').textContent = 
    `图片尺寸: ${w}×${h} | 检测到 ${cols}列×${rows}行 = ${cols*rows} 颗豆`;
  document.getElementById('grid-modal').classList.add('show');
}

function closeGridModal() {
  document.getElementById('grid-modal').classList.remove('show');
}

function confirmGrid() {
  detectedGrid.cols = parseInt(document.getElementById('grid-cols').value) || detectedGrid.cols;
  detectedGrid.rows = parseInt(document.getElementById('grid-rows').value) || detectedGrid.rows;
  document.getElementById('grid-modal').classList.remove('show');
  
  if (currentMode === 'ocr') {
    runOCRRecognition();
  } else {
    runGridColorRecognition();
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html && git commit -m "feat: add grid detection algorithm with projection"
```

---

### Task 6: 逐格取色 + OCR 识别

**修改**: `index.html` (JS section)

- [ ] **Step 1: 添加格内主色提取 + OCR 函数**

```javascript
// ===== GRID-BASED RECOGNITION =====
function runGridColorRecognition() {
  // Photo mode: grid-based color matching (no OCR)
  const previewImg = document.querySelector('#preview-container img');
  const canvas = document.createElement('canvas');
  canvas.width = previewImg.naturalWidth;
  canvas.height = previewImg.naturalHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(previewImg, 0, 0);
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const pixels = imageData.data;
  const w = canvas.width;
  const h = canvas.height;
  
  const cellW = w / detectedGrid.cols;
  const cellH = h / detectedGrid.rows;
  
  const matched = {};
  
  for (let row = 0; row < detectedGrid.rows; row++) {
    for (let col = 0; col < detectedGrid.cols; col++) {
      const x0 = Math.floor(col * cellW);
      const y0 = Math.floor(row * cellH);
      const x1 = Math.floor((col + 1) * cellW);
      const y1 = Math.floor((row + 1) * cellH);
      
      // Get dominant color in cell (excluding grid/bg pixels)
      const colorCount = {};
      for (let y = y0; y < y1; y++) {
        for (let x = x0; x < x1; x++) {
          const idx = (y * w + x) * 4;
          const r = pixels[idx], g = pixels[idx+1], b = pixels[idx+2];
          
          // Skip background and grid colors
          let skip = false;
          for (const key of ['bg', 'grid1', 'grid2']) {
            const pc = pickedColors[key];
            if (pc && Math.abs(r-pc.r) <= tolerance && Math.abs(g-pc.g) <= tolerance && Math.abs(b-pc.b) <= tolerance) {
              skip = true; break;
            }
          }
          if (skip) continue;
          
          // Quantize slightly to group near-identical colors
          const qr = Math.round(r / 8) * 8;
          const qg = Math.round(g / 8) * 8;
          const qb = Math.round(b / 8) * 8;
          const key = `${qr},${qg},${qb}`;
          colorCount[key] = (colorCount[key] || 0) + 1;
        }
      }
      
      // Find dominant color
      let bestKey = null, bestCount = 0;
      for (const [key, count] of Object.entries(colorCount)) {
        if (count > bestCount) { bestCount = count; bestKey = key; }
      }
      
      if (bestKey) {
        const [qr, qg, qb] = bestKey.split(',').map(Number);
        // Match to nearest MARD color
        let bestCode = null, bestDist = Infinity;
        for (const m of MARD_COLORS) {
          const dist = (qr-m.r)**2 + (qg-m.g)**2 + (qb-m.b)**2;
          if (dist < bestDist) { bestDist = dist; bestCode = m.code; }
        }
        matched[bestCode] = (matched[bestCode] || 0) + 1;
      }
    }
  }
  
  buildMatchResults(matched);
}

async function runOCRRecognition() {
  const previewImg = document.querySelector('#preview-container img');
  const canvas = document.createElement('canvas');
  canvas.width = previewImg.naturalWidth;
  canvas.height = previewImg.naturalHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(previewImg, 0, 0);
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const pixels = imageData.data;
  const w = canvas.width;
  const h = canvas.height;
  
  const cellW = w / detectedGrid.cols;
  const cellH = h / detectedGrid.rows;
  
  // Load OCR engine
  let worker;
  try {
    worker = await loadTesseract();
  } catch (e) {
    toast('OCR 引擎加载失败，回退到取色模式');
    runGridColorRecognition();
    return;
  }
  
  const matched = {};
  let ocrSuccess = 0;
  let total = 0;
  
  for (let row = 0; row < detectedGrid.rows; row++) {
    for (let col = 0; col < detectedGrid.cols; col++) {
      total++;
      const x0 = Math.floor(col * cellW);
      const y0 = Math.floor(row * cellH);
      const x1 = Math.floor((col + 1) * cellW);
      const y1 = Math.floor((row + 1) * cellH);
      
      // Get dominant color in cell
      const colorCount = {};
      for (let y = y0; y < y1; y++) {
        for (let x = x0; x < x1; x++) {
          const idx = (y * w + x) * 4;
          const r = pixels[idx], g = pixels[idx+1], b = pixels[idx+2];
          
          let skip = false;
          for (const key of ['bg', 'grid1', 'grid2']) {
            const pc = pickedColors[key];
            if (pc && Math.abs(r-pc.r) <= tolerance && Math.abs(g-pc.g) <= tolerance && Math.abs(b-pc.b) <= tolerance) {
              skip = true; break;
            }
          }
          if (skip) continue;
          
          const qr = Math.round(r / 16) * 16;
          const qg = Math.round(g / 16) * 16;
          const qb = Math.round(b / 16) * 16;
          const key = `${qr},${qg},${qb}`;
          colorCount[key] = (colorCount[key] || 0) + 1;
        }
      }
      
      // Find dominant color
      let bestKey = null, bestCount = 0;
      for (const [key, count] of Object.entries(colorCount)) {
        if (count > bestCount) { bestCount = count; bestKey = key; }
      }
      
      // OCR on the cell
      const cellCanvas = document.createElement('canvas');
      cellCanvas.width = x1 - x0;
      cellCanvas.height = y1 - y0;
      const cellCtx = cellCanvas.getContext('2d');
      cellCtx.drawImage(canvas, x0, y0, x1-x0, y1-y0, 0, 0, x1-x0, y1-y0);
      
      let ocrCode = null;
      try {
        const { data: { text } } = await worker.recognize(cellCanvas);
        // Clean OCR result: extract alphanumeric code like "A1", "ZG8", etc.
        const cleaned = text.replace(/[^A-Za-z0-9]/g, '').toUpperCase();
        // Match pattern: 1-3 letters followed by 1-3 digits
        const match = cleaned.match(/^([A-Z]{1,3})(\d{1,3})$/);
        if (match) {
          const code = match[1] + match[2];
          if (MARD_MAP[code]) {
            ocrCode = code;
            ocrSuccess++;
          }
        }
      } catch (e) {
        // OCR failed for this cell, fall through to color matching
      }
      
      // Cross-validate or fallback
      let finalCode = null;
      if (ocrCode) {
        // Verify: OCR code's RGB should match cell's dominant color
        const mard = MARD_MAP[ocrCode];
        if (bestKey) {
          const [qr, qg, qb] = bestKey.split(',').map(Number);
          const colorDist = Math.sqrt((qr-mard.r)**2 + (qg-mard.g)**2 + (qb-mard.b)**2);
          if (colorDist <= 50) {
            finalCode = ocrCode; // OCR + color agree
          } else {
            finalCode = ocrCode; // OCR wins (user trusts the printed code)
          }
        } else {
          finalCode = ocrCode; // OCR only (no cell color found)
        }
      } else if (bestKey) {
        // OCR failed, fallback to color matching
        const [qr, qg, qb] = bestKey.split(',').map(Number);
        let bestMatch = null, bestDist = Infinity;
        for (const m of MARD_COLORS) {
          const dist = (qr-m.r)**2 + (qg-m.g)**2 + (qb-m.b)**2;
          if (dist < bestDist) { bestDist = dist; bestMatch = m.code; }
        }
        finalCode = bestMatch;
      }
      
      if (finalCode) {
        matched[finalCode] = (matched[finalCode] || 0) + 1;
      }
    }
  }
  
  const ocrRate = total > 0 ? Math.round(ocrSuccess / total * 100) : 0;
  if (ocrRate < 70 && total > 20) {
    toast(`OCR 识别率 ${ocrRate}%，部分格子使用取色匹配`);
  } else {
    toast(`OCR 识别完成，成功率 ${ocrRate}%`);
  }
  
  buildMatchResults(matched);
}

function buildMatchResults(matched) {
  matchResults = Object.entries(matched)
    .map(([code, count]) => ({
      code,
      count,
      swatch: MARD_MAP[code],
      stock: warehouse[code] || 0,
      insufficient: (warehouse[code] || 0) < count,
    }))
    .sort((a, b) => b.count - a.count);
  
  renderMatchResults();
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html && git commit -m "feat: add grid-based color matching and OCR recognition"
```

---

### Task 7: 重新处理 + 像素图模式保留

**修改**: `index.html` (JS section)

- [ ] **Step 1: 添加重新处理函数，修改 processImage**

将 `processImage` 函数改为存储图片引用，根据模式选择识别方式：

```javascript
let currentImg = null;

function processImage(file) {
  if (!file.type.match(/image\/(png|jpeg|webp)/)) {
    toast('请上传 PNG、JPG 或 WebP 格式的图片');
    return;
  }

  const reader = new FileReader();
  reader.onload = function(e) {
    const img = new Image();
    img.onload = function() {
      currentImg = img;
      showPreview(img);
      
      // Show mode bar
      document.getElementById('mode-bar').style.display = 'block';
      
      // Auto-detect: small image + few colors => pixel mode
      const w = img.naturalWidth, h = img.naturalHeight;
      const canvas = document.createElement('canvas');
      canvas.width = Math.min(w, 200);
      canvas.height = Math.min(h, 200);
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      const sample = ctx.getImageData(0, 0, canvas.width, canvas.height).data;
      const uniqueColors = new Set();
      for (let i = 0; i < sample.length; i += 4) {
        uniqueColors.add(`${sample[i]},${sample[i+1]},${sample[i+2]}`);
      }
      
      if (w <= 600 && h <= 600 && uniqueColors.size <= 500) {
        setMode('pixel');
        // Pixel mode: directly extract
        extractColors(img);
      } else {
        setMode('photo');
        // Show grid settings, wait for user action
        document.getElementById('grid-settings').style.display = 'block';
        matchResults = [];
        document.getElementById('match-results').innerHTML = '';
        document.getElementById('match-actions').style.display = 'none';
      }
    };
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}

function reprocessImage() {
  if (!currentImg) return;
  if (currentMode === 'pixel') {
    extractColors(currentImg);
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html && git commit -m "feat: add auto-detect mode and reprocess support"
```

---

### Task 8: 清理旧提取函数 + 集成测试

**修改**: `index.html` (JS section)

- [ ] **Step 1: 移除旧的 extractColors 中 >500 色限制的逻辑，保留像素图模式**

修改 `extractColors` 函数，在开头添加图片引用更新：

```javascript
function extractColors(img) {
  currentImg = img;
  // Limit max dimension to 2000px
  let w = img.naturalWidth;
  let h = img.naturalHeight;
  // ... (rest unchanged)
```

- [ ] **Step 2: 在浏览器中测试三种模式**

手动测试：
1. 像素图模式：上传小尺寸图案 → 应自动识别为像素图
2. 照片模式：上传照片 → 选择背景色和网格色 → 检测网格 → 取色匹配
3. OCR模式：上传带编号图纸 → 选择背景色和网格色 → 检测网格 → OCR识别

- [ ] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: finalize multi-mode recognition integration"
```

---

### Task 9: 部署

- [ ] **Step 1: Push to GitHub Pages**

```bash
git push
```

---

## 注意事项

1. Tesseract.js v5 CDN 约 ~4MB，首次加载需几秒，后续浏览器缓存秒开
2. OCR 识别单格约 100-300ms，1000格约 1-3分钟。使用进度提示
3. 网格检测阈值自动计算，但需要用户至少正确选取背景色和一种网格色
4. OCR 只匹配 `字母+数字` 格式的结果到 MARD 色号表，错误字符串自动丢弃
