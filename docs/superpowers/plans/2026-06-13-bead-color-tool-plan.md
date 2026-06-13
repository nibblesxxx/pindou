# 拼豆色号助手 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML web app for managing MARD bead color inventory, with image-to-color matching and GitHub-synced CSV persistence.

**Architecture:** Single `index.html` with embedded CSS and JS. Two-tab interface (Warehouse + Image Matcher). State managed in plain JS objects, persisted to localStorage (local fallback) and GitHub API (cloud sync). Canvas API for pixel extraction, Euclidean distance for color matching.

**Tech Stack:** HTML5 + CSS3 + Vanilla JS (no frameworks, no build tools, no dependencies)

---

### Task 1: HTML Document Shell and CSS Foundation

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the full HTML document structure**

Write the complete file with CSS variables, tab navigation, and placeholder sections:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>拼豆色号助手</title>
  <style>
    :root {
      --bg: #ffffff;
      --surface: #f8f9fa;
      --border: #e9ecef;
      --text: #212529;
      --text-secondary: #6c757d;
      --primary: #4a90d9;
      --danger: #dc3545;
      --warning: #ffc107;
      --success: #28a745;
      --radius: 8px;
      --shadow: 0 1px 3px rgba(0,0,0,0.08);
      --transition: 0.15s ease;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    /* Header */
    header {
      padding: 12px 16px;
      border-bottom: 1px solid var(--border);
      background: var(--bg);
      position: sticky;
      top: 0;
      z-index: 100;
    }
    .header-title {
      font-size: 18px;
      font-weight: 700;
      margin-bottom: 10px;
      color: var(--text);
    }

    /* Tabs */
    .tabs {
      display: flex;
      gap: 4px;
      background: var(--surface);
      border-radius: var(--radius);
      padding: 3px;
    }
    .tab {
      flex: 1;
      padding: 8px 16px;
      border: none;
      background: transparent;
      border-radius: 6px;
      cursor: pointer;
      font-size: 14px;
      font-weight: 500;
      color: var(--text-secondary);
      transition: all var(--transition);
      white-space: nowrap;
      text-align: center;
    }
    .tab.active {
      background: var(--bg);
      color: var(--text);
      box-shadow: var(--shadow);
    }

    /* Main */
    main {
      flex: 1;
      padding: 16px;
      max-width: 800px;
      margin: 0 auto;
      width: 100%;
    }

    /* Tab panels */
    .tab-panel {
      display: none;
    }
    .tab-panel.active {
      display: block;
    }

    /* Footer status bar */
    footer {
      padding: 8px 16px;
      border-top: 1px solid var(--border);
      font-size: 12px;
      color: var(--text-secondary);
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: var(--bg);
      position: sticky;
      bottom: 0;
    }
    .status-dot {
      display: inline-block;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      margin-right: 6px;
    }
    .status-dot.cloud { background: var(--success); }
    .status-dot.local { background: var(--warning); }

    /* Modal overlay */
    .modal-overlay {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.4);
      z-index: 1000;
      align-items: center;
      justify-content: center;
    }
    .modal-overlay.show {
      display: flex;
    }
    .modal {
      background: var(--bg);
      border-radius: 12px;
      padding: 24px;
      max-width: 420px;
      width: 90%;
      box-shadow: 0 8px 32px rgba(0,0,0,0.15);
    }
    .modal h2 {
      font-size: 18px;
      margin-bottom: 8px;
    }
    .modal p {
      font-size: 13px;
      color: var(--text-secondary);
      margin-bottom: 16px;
      line-height: 1.5;
    }
    .modal label {
      display: block;
      font-size: 13px;
      font-weight: 600;
      margin-bottom: 4px;
      margin-top: 12px;
    }
    .modal input {
      width: 100%;
      padding: 10px 12px;
      border: 1px solid var(--border);
      border-radius: var(--radius);
      font-size: 14px;
      font-family: inherit;
    }
    .modal input:focus {
      outline: none;
      border-color: var(--primary);
      box-shadow: 0 0 0 3px rgba(74,144,217,0.15);
    }
    .modal .btn-row {
      display: flex;
      gap: 8px;
      margin-top: 20px;
    }

    /* Buttons */
    .btn {
      padding: 10px 20px;
      border: none;
      border-radius: var(--radius);
      cursor: pointer;
      font-size: 14px;
      font-weight: 500;
      font-family: inherit;
      transition: all var(--transition);
    }
    .btn-primary {
      background: var(--primary);
      color: #fff;
    }
    .btn-primary:hover { opacity: 0.9; }
    .btn-danger {
      background: var(--danger);
      color: #fff;
    }
    .btn-outline {
      background: var(--bg);
      color: var(--text);
      border: 1px solid var(--border);
    }
    .btn-outline:hover { background: var(--surface); }
    .btn-sm {
      padding: 6px 12px;
      font-size: 12px;
    }
    .btn:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    /* Controls bar */
    .controls {
      display: flex;
      gap: 8px;
      margin-bottom: 12px;
      flex-wrap: wrap;
    }
    .search-input {
      flex: 1;
      min-width: 150px;
      padding: 10px 12px;
      border: 1px solid var(--border);
      border-radius: var(--radius);
      font-size: 14px;
      font-family: inherit;
    }
    .search-input:focus {
      outline: none;
      border-color: var(--primary);
      box-shadow: 0 0 0 3px rgba(74,144,217,0.15);
    }

    /* Color grid */
    .color-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(72px, 1fr));
      gap: 8px;
    }
    .color-card {
      background: var(--surface);
      border-radius: var(--radius);
      padding: 8px;
      text-align: center;
      cursor: pointer;
      border: 2px solid transparent;
      transition: all var(--transition);
    }
    .color-card:hover {
      border-color: var(--primary);
      box-shadow: var(--shadow);
    }
    .color-card.dirty {
      border-color: var(--warning);
    }
    .color-card .swatch {
      width: 100%;
      aspect-ratio: 1;
      border-radius: 4px;
      margin-bottom: 6px;
      border: 1px solid rgba(0,0,0,0.08);
    }
    .color-card .code {
      font-size: 11px;
      font-weight: 700;
    }
    .color-card .qty {
      font-size: 10px;
      color: var(--text-secondary);
      margin-top: 2px;
    }
    .color-card .qty.negative {
      color: var(--danger);
      font-weight: 700;
    }
    .color-card .qty.low {
      color: #e67e22;
    }

    /* Edit modal */
    .edit-modal-body {
      text-align: center;
    }
    .edit-swatch {
      width: 80px;
      height: 80px;
      border-radius: 8px;
      margin: 0 auto 16px;
      border: 1px solid rgba(0,0,0,0.1);
    }
    .edit-code {
      font-size: 20px;
      font-weight: 700;
      margin-bottom: 4px;
    }
    .edit-rgb {
      font-size: 12px;
      color: var(--text-secondary);
      margin-bottom: 16px;
    }
    .edit-input {
      width: 120px;
      padding: 10px;
      border: 1px solid var(--border);
      border-radius: var(--radius);
      font-size: 16px;
      text-align: center;
      font-family: inherit;
    }

    /* Image upload zone */
    .upload-zone {
      border: 2px dashed #d0d5dd;
      border-radius: 12px;
      padding: 32px 24px;
      text-align: center;
      background: var(--surface);
      cursor: pointer;
      transition: all var(--transition);
      margin-bottom: 16px;
    }
    .upload-zone:hover, .upload-zone.dragover {
      border-color: var(--primary);
      background: rgba(74,144,217,0.04);
    }
    .upload-zone .icon {
      font-size: 40px;
      margin-bottom: 8px;
    }
    .upload-zone p {
      font-size: 14px;
      color: var(--text-secondary);
      margin: 4px 0;
    }
    .upload-zone .hint {
      font-size: 12px;
      color: #adb5bd;
    }

    /* Preview image */
    .preview-container {
      margin-bottom: 16px;
      text-align: center;
    }
    .preview-container img {
      max-width: 100%;
      max-height: 300px;
      border-radius: var(--radius);
      border: 1px solid var(--border);
    }

    /* Match results */
    .match-list {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
      gap: 6px;
      margin-bottom: 16px;
      max-height: 300px;
      overflow-y: auto;
    }
    .match-item {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 8px 10px;
      background: var(--surface);
      border-radius: var(--radius);
      font-size: 12px;
    }
    .match-item .m-swatch {
      width: 20px;
      height: 20px;
      border-radius: 4px;
      flex-shrink: 0;
      border: 1px solid rgba(0,0,0,0.08);
    }
    .match-item .m-code {
      font-weight: 600;
      min-width: 30px;
    }
    .match-item .m-count {
      margin-left: auto;
      color: var(--text-secondary);
    }
    .match-item.insufficient {
      background: #fff9db;
      border: 1px solid var(--warning);
    }
    .match-item.insufficient .m-count {
      color: var(--danger);
      font-weight: 700;
    }

    /* Action bar */
    .action-bar {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
    }

    /* Undo toast */
    .toast {
      position: fixed;
      bottom: 60px;
      left: 50%;
      transform: translateX(-50%);
      background: #333;
      color: #fff;
      padding: 10px 20px;
      border-radius: 20px;
      font-size: 13px;
      z-index: 500;
      opacity: 0;
      transition: opacity 0.3s;
      pointer-events: none;
    }
    .toast.show {
      opacity: 1;
    }

    /* Responsive */
    @media (max-width: 480px) {
      .color-grid {
        grid-template-columns: repeat(auto-fill, minmax(56px, 1fr));
        gap: 6px;
      }
      .match-list {
        grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
      }
      .btn { padding: 8px 14px; font-size: 13px; }
    }
  </style>
</head>
<body>

<header>
  <div class="header-title">🧩 拼豆色号助手</div>
  <nav class="tabs">
    <button class="tab active" data-tab="warehouse" onclick="switchTab('warehouse')">📦 色号仓库</button>
    <button class="tab" data-tab="matcher" onclick="switchTab('matcher')">🎨 图转色号</button>
  </nav>
</header>

<main>
  <!-- Tab 1: Warehouse -->
  <section id="panel-warehouse" class="tab-panel active">
    <div class="controls">
      <input type="text" class="search-input" id="warehouse-search" placeholder="🔍 搜索色号..." oninput="renderWarehouse()">
      <button class="btn btn-outline btn-sm" onclick="showSetupModal()">⚙️ 设置</button>
      <button class="btn btn-outline btn-sm" id="btn-load-cloud" onclick="loadFromGitHub()">🔄 云端加载</button>
      <button class="btn btn-primary btn-sm" id="btn-save-cloud" onclick="saveToGitHub()">💾 保存到云端</button>
    </div>
    <div class="color-grid" id="warehouse-grid"></div>
  </section>

  <!-- Tab 2: Image Matcher -->
  <section id="panel-matcher" class="tab-panel">
    <div class="upload-zone" id="upload-zone" onclick="document.getElementById('file-input').click()">
      <div class="icon">🖼️</div>
      <p>拖拽拼豆截图到此处，或点击上传</p>
      <p class="hint">支持 PNG / JPG / WebP</p>
    </div>
    <input type="file" id="file-input" accept="image/*" style="display:none" onchange="handleFileSelect(event)">
    <div class="preview-container" id="preview-container" style="display:none;"></div>
    <div id="match-results"></div>
    <div class="action-bar" id="match-actions" style="display:none;">
      <button class="btn btn-danger btn-sm" id="btn-undo" onclick="undoLastDeduction()" disabled>↩ 撤销上次扣除</button>
      <button class="btn btn-outline btn-sm" onclick="clearMatches()">🔄 重新识别</button>
    </div>
  </section>
</main>

<footer>
  <span id="status-text">
    <span class="status-dot local"></span> 本地模式
  </span>
  <span id="dirty-indicator"></span>
</footer>

<!-- Edit quantity modal -->
<div class="modal-overlay" id="edit-modal">
  <div class="modal">
    <div class="edit-modal-body">
      <div class="edit-swatch" id="edit-swatch"></div>
      <div class="edit-code" id="edit-code"></div>
      <div class="edit-rgb" id="edit-rgb"></div>
      <label>库存数量</label>
      <input type="number" class="edit-input" id="edit-qty" min="-9999" max="99999" onkeydown="if(event.key==='Enter')saveEdit()">
      <div class="btn-row" style="margin-top:16px;justify-content:center;">
        <button class="btn btn-outline btn-sm" onclick="closeEditModal()">取消</button>
        <button class="btn btn-primary btn-sm" onclick="saveEdit()">保存</button>
      </div>
    </div>
  </div>
</div>

<!-- Setup modal -->
<div class="modal-overlay" id="setup-modal">
  <div class="modal">
    <h2>⚙️ 配置 GitHub 同步</h2>
    <p>需要 GitHub Personal Access Token 来读写云端仓库中的库存数据。</p>
    <ol style="font-size:12px;color:var(--text-secondary);padding-left:16px;line-height:1.8;">
      <li>打开 <a href="https://github.com/settings/tokens" target="_blank">GitHub Token 设置</a></li>
      <li>点击 "Generate new token (classic)"</li>
      <li>勾选 <b>repo</b> 权限，生成 Token</li>
      <li>复制 Token 粘贴到下方</li>
    </ol>
    <label>GitHub Token</label>
    <input type="password" id="setup-token" placeholder="ghp_xxxxxxxxxxxx">
    <label>仓库路径</label>
    <input type="text" id="setup-repo" placeholder="用户名/仓库名">
    <div class="btn-row">
      <button class="btn btn-outline" onclick="skipSetup()">跳过（本地模式）</button>
      <button class="btn btn-primary" onclick="saveSetup()" style="margin-left:auto;">保存并连接</button>
    </div>
  </div>
</div>

<!-- Toast -->
<div class="toast" id="toast"></div>

<script>
// ===== MARD COLOR DATA =====
// (will be filled in Task 2)

// ===== STATE =====
// (will be filled in subsequent tasks)
</script>

</body>
</html>
```

- [ ] **Step 2: Verify the shell renders correctly**

Open `index.html` in a browser. Verify:
- Header with two tabs visible
- Clicking tabs switches panels (no content yet but structure works)
- Footer with status appears
- Both modals are hidden by default

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: create HTML shell with CSS foundation and tab navigation"
```

---

### Task 2: MARD Color Data and State Initialization

**Files:**
- Modify: `index.html` (add to existing `<script>` block)

- [ ] **Step 1: Add MARD color array and state management**

Replace the placeholder comment `// ===== MARD COLOR DATA =====` and `// ===== STATE =====` with:

```javascript
// ===== MARD COLOR DATA (291 colors) =====
const MARD_COLORS = [
  {code:'A1',r:250,g:244,b:200},
  {code:'A10',r:247,g:124,b:49},
  {code:'A11',r:255,g:221,b:153},
  {code:'A12',r:254,g:159,b:114},
  {code:'A13',r:255,g:195,b:101},
  {code:'A14',r:253,g:84,b:61},
  {code:'A15',r:255,g:243,b:101},
  {code:'A16',r:255,g:255,b:159},
  {code:'A17',r:255,g:227,b:110},
  {code:'A18',r:254,g:190,b:125},
  {code:'A19',r:253,g:124,b:114},
  {code:'A2',r:255,g:255,b:213},
  {code:'A20',r:255,g:213,b:104},
  {code:'A21',r:255,g:227,b:149},
  {code:'A22',r:244,g:245,b:125},
  {code:'A23',r:230,g:201,b:183},
  {code:'A24',r:247,g:248,b:162},
  {code:'A25',r:255,g:214,b:125},
  {code:'A26',r:255,g:200,b:48},
  {code:'A3',r:254,g:255,b:139},
  {code:'A4',r:251,g:237,b:86},
  {code:'A5',r:244,g:215,b:56},
  {code:'A6',r:254,g:172,b:76},
  {code:'A7',r:254,g:139,b:76},
  {code:'A8',r:255,g:218,b:69},
  {code:'A9',r:255,g:153,b:91},
  {code:'B1',r:230,g:238,b:49},
  {code:'B10',r:149,g:211,b:194},
  {code:'B11',r:93,g:114,b:42},
  {code:'B12',r:22,g:111,b:65},
  {code:'B13',r:202,g:235,b:123},
  {code:'B14',r:173,g:233,b:70},
  {code:'B15',r:46,g:81,b:50},
  {code:'B16',r:197,g:237,b:156},
  {code:'B17',r:155,g:177,b:58},
  {code:'B18',r:230,g:238,b:73},
  {code:'B19',r:36,g:184,b:140},
  {code:'B2',r:99,g:243,b:71},
  {code:'B20',r:194,g:240,b:204},
  {code:'B21',r:21,g:106,b:107},
  {code:'B22',r:11,g:60,b:67},
  {code:'B23',r:48,g:58,b:33},
  {code:'B24',r:238,g:252,b:165},
  {code:'B25',r:78,g:132,b:109},
  {code:'B26',r:141,g:122,b:53},
  {code:'B27',r:204,g:225,b:175},
  {code:'B28',r:158,g:229,b:185},
  {code:'B29',r:197,g:226,b:84},
  {code:'B3',r:158,g:247,b:128},
  {code:'B30',r:226,g:252,b:177},
  {code:'B31',r:176,g:231,b:146},
  {code:'B32',r:156,g:171,b:90},
  {code:'B4',r:93,g:224,b:53},
  {code:'B5',r:53,g:227,b:82},
  {code:'B6',r:101,g:226,b:166},
  {code:'B7',r:61,g:175,b:128},
  {code:'B8',r:28,g:156,b:79},
  {code:'B9',r:39,g:82,b:58},
  {code:'C1',r:232,g:255,b:231},
  {code:'C10',r:62,g:188,b:226},
  {code:'C11',r:40,g:221,b:222},
  {code:'C12',r:28,g:51,b:77},
  {code:'C13',r:205,g:232,b:255},
  {code:'C14',r:213,g:253,b:255},
  {code:'C15',r:34,g:196,b:198},
  {code:'C16',r:21,g:87,b:168},
  {code:'C17',r:4,g:209,b:246},
  {code:'C18',r:29,g:51,b:68},
  {code:'C19',r:24,g:135,b:162},
  {code:'C2',r:169,g:249,b:252},
  {code:'C20',r:23,g:109,b:175},
  {code:'C21',r:190,g:221,b:255},
  {code:'C22',r:103,g:180,b:190},
  {code:'C23',r:200,g:226,b:255},
  {code:'C24',r:124,g:196,b:255},
  {code:'C25',r:169,g:229,b:229},
  {code:'C26',r:60,g:174,b:216},
  {code:'C27',r:211,g:223,b:250},
  {code:'C28',r:187,g:207,b:237},
  {code:'C29',r:52,g:72,b:142},
  {code:'C3',r:160,g:226,b:251},
  {code:'C4',r:65,g:204,b:255},
  {code:'C5',r:1,g:172,b:235},
  {code:'C6',r:80,g:170,b:240},
  {code:'C7',r:54,g:119,b:210},
  {code:'C8',r:15,g:84,b:192},
  {code:'C9',r:50,g:75,b:202},
  {code:'D1',r:174,g:180,b:242},
  {code:'D10',r:54,g:24,b:81},
  {code:'D11',r:185,g:186,b:225},
  {code:'D12',r:222,g:154,b:212},
  {code:'D13',r:185,g:0,b:149},
  {code:'D14',r:139,g:39,b:155},
  {code:'D15',r:47,g:31,b:144},
  {code:'D16',r:227,g:225,b:238},
  {code:'D17',r:196,g:212,b:246},
  {code:'D18',r:164,g:94,b:199},
  {code:'D19',r:216,g:195,b:215},
  {code:'D2',r:133,g:142,b:221},
  {code:'D20',r:156,g:50,b:178},
  {code:'D21',r:154,g:0,b:155},
  {code:'D22',r:51,g:58,b:149},
  {code:'D23',r:235,g:218,b:252},
  {code:'D24',r:119,g:134,b:229},
  {code:'D25',r:73,g:79,b:199},
  {code:'D26',r:223,g:194,b:248},
  {code:'D3',r:47,g:84,b:175},
  {code:'D4',r:24,g:42,b:132},
  {code:'D5',r:184,g:67,b:197},
  {code:'D6',r:172,g:123,b:222},
  {code:'D7',r:136,g:84,b:179},
  {code:'D8',r:226,g:211,b:255},
  {code:'D9',r:213,g:185,b:248},
  {code:'E1',r:253,g:211,b:204},
  {code:'E10',r:211,g:55,b:147},
  {code:'E11',r:252,g:221,b:210},
  {code:'E12',r:247,g:143,b:195},
  {code:'E13',r:181,g:0,b:109},
  {code:'E14',r:255,g:209,b:186},
  {code:'E15',r:248,g:199,b:201},
  {code:'E16',r:255,g:243,b:235},
  {code:'E17',r:255,g:226,b:234},
  {code:'E18',r:255,g:199,b:219},
  {code:'E19',r:254,g:186,b:213},
  {code:'E2',r:254,g:192,b:223},
  {code:'E20',r:216,g:199,b:209},
  {code:'E21',r:189,g:157,b:161},
  {code:'E22',r:183,g:133,b:161},
  {code:'E23',r:147,g:122,b:141},
  {code:'E24',r:225,g:188,b:232},
  {code:'E3',r:255,g:183,b:231},
  {code:'E4',r:232,g:100,b:158},
  {code:'E5',r:245,g:81,b:162},
  {code:'E6',r:241,g:61,b:116},
  {code:'E7',r:198,g:52,b:120},
  {code:'E8',r:255,g:219,b:233},
  {code:'E9',r:233,g:112,b:204},
  {code:'F1',r:253,g:149,b:123},
  {code:'F10',r:138,g:69,b:38},
  {code:'F11',r:90,g:33,b:33},
  {code:'F12',r:253,g:78,b:106},
  {code:'F13',r:243,g:87,b:68},
  {code:'F14',r:255,g:169,b:173},
  {code:'F15',r:211,g:0,b:34},
  {code:'F16',r:254,g:194,b:166},
  {code:'F17',r:230,g:156,b:121},
  {code:'F18',r:211,g:124,b:70},
  {code:'F19',r:193,g:68,b:74},
  {code:'F2',r:252,g:61,b:70},
  {code:'F20',r:205,g:147,b:145},
  {code:'F21',r:247,g:180,b:198},
  {code:'F22',r:253,g:192,b:208},
  {code:'F23',r:246,g:126,b:102},
  {code:'F24',r:230,g:152,b:170},
  {code:'F25',r:229,g:75,b:79},
  {code:'F3',r:247,g:73,b:65},
  {code:'F4',r:252,g:40,b:60},
  {code:'F5',r:231,g:0,b:47},
  {code:'F6',r:148,g:54,b:48},
  {code:'F7',r:151,g:25,b:55},
  {code:'F8',r:188,g:0,b:40},
  {code:'F9',r:226,g:103,b:122},
  {code:'G1',r:255,g:226,b:206},
  {code:'G10',r:217,g:140,b:57},
  {code:'G11',r:224,g:197,b:147},
  {code:'G12',r:255,g:200,b:144},
  {code:'G13',r:183,g:113,b:74},
  {code:'G14',r:141,g:97,b:76},
  {code:'G15',r:252,g:249,b:224},
  {code:'G16',r:242,g:217,b:186},
  {code:'G17',r:120,g:82,b:75},
  {code:'G18',r:255,g:228,b:204},
  {code:'G19',r:224,g:121,b:53},
  {code:'G2',r:255,g:196,b:170},
  {code:'G20',r:169,g:64,b:35},
  {code:'G21',r:184,g:133,b:88},
  {code:'G3',r:244,g:195,b:165},
  {code:'G4',r:225,g:179,b:131},
  {code:'G5',r:237,g:176,b:69},
  {code:'G6',r:233,g:156,b:23},
  {code:'G7',r:157,g:91,b:62},
  {code:'G8',r:117,g:56,b:50},
  {code:'G9',r:230,g:180,b:131},
  {code:'H1',r:253,g:251,b:255},
  {code:'H10',r:238,g:233,b:234},
  {code:'H11',r:206,g:205,b:213},
  {code:'H12',r:255,g:245,b:237},
  {code:'H13',r:245,g:236,b:210},
  {code:'H14',r:207,g:215,b:211},
  {code:'H15',r:152,g:166,b:168},
  {code:'H16',r:29,g:20,b:20},
  {code:'H17',r:241,g:237,b:237},
  {code:'H18',r:255,g:253,b:240},
  {code:'H19',r:246,g:239,b:226},
  {code:'H2',r:254,g:255,b:255},
  {code:'H20',r:148,g:159,b:163},
  {code:'H21',r:255,g:251,b:225},
  {code:'H22',r:202,g:202,b:212},
  {code:'H23',r:154,g:157,b:148},
  {code:'H3',r:182,g:177,b:186},
  {code:'H4',r:137,g:133,b:140},
  {code:'H5',r:72,g:70,b:78},
  {code:'H6',r:47,g:43,b:47},
  {code:'H7',r:0,g:0,b:0},
  {code:'H8',r:231,g:214,b:219},
  {code:'H9',r:237,g:237,b:237},
  {code:'M1',r:188,g:198,b:184},
  {code:'M10',r:197,g:178,b:188},
  {code:'M11',r:159,g:117,b:148},
  {code:'M12',r:100,g:71,b:73},
  {code:'M13',r:209,g:144,b:102},
  {code:'M14',r:199,g:115,b:98},
  {code:'M15',r:117,g:125,b:120},
  {code:'M2',r:138,g:163,b:134},
  {code:'M3',r:105,g:125,b:128},
  {code:'M4',r:227,g:210,b:188},
  {code:'M5',r:208,g:204,b:170},
  {code:'M6',r:176,g:167,b:130},
  {code:'M7',r:180,g:164,b:151},
  {code:'M8',r:179,g:130,b:129},
  {code:'M9',r:165,g:135,b:103},
  {code:'P1',r:252,g:247,b:248},
  {code:'P10',r:217,g:199,b:234},
  {code:'P11',r:243,g:236,b:201},
  {code:'P12',r:230,g:238,b:242},
  {code:'P13',r:170,g:203,b:239},
  {code:'P14',r:51,g:118,b:128},
  {code:'P15',r:102,g:133,b:117},
  {code:'P16',r:254,g:191,b:69},
  {code:'P17',r:254,g:163,b:36},
  {code:'P18',r:254,g:184,b:159},
  {code:'P19',r:255,g:254,b:236},
  {code:'P2',r:176,g:169,b:172},
  {code:'P20',r:254,g:190,b:207},
  {code:'P21',r:236,g:190,b:191},
  {code:'P22',r:228,g:168,b:159},
  {code:'P23',r:165,g:98,b:104},
  {code:'P3',r:175,g:220,b:171},
  {code:'P4',r:254,g:164,b:159},
  {code:'P5',r:238,g:140,b:62},
  {code:'P6',r:95,g:208,b:167},
  {code:'P7',r:235,g:146,b:112},
  {code:'P8',r:240,g:217,b:88},
  {code:'P9',r:217,g:217,b:217},
  {code:'Q1',r:242,g:165,b:232},
  {code:'Q2',r:233,g:236,b:145},
  {code:'Q3',r:255,g:255,b:0},
  {code:'Q4',r:255,g:235,b:250},
  {code:'Q5',r:118,g:206,b:222},
  {code:'R1',r:213,g:13,b:33},
  {code:'R10',r:255,g:219,b:76},
  {code:'R11',r:255,g:235,b:250},
  {code:'R12',r:216,g:213,b:206},
  {code:'R13',r:85,g:81,b:76},
  {code:'R14',r:159,g:228,b:223},
  {code:'R15',r:119,g:206,b:233},
  {code:'R16',r:62,g:207,b:202},
  {code:'R17',r:74,g:134,b:122},
  {code:'R18',r:127,g:205,b:157},
  {code:'R19',r:205,g:229,b:93},
  {code:'R2',r:249,g:47,b:131},
  {code:'R20',r:232,g:199,b:180},
  {code:'R21',r:173,g:111,b:60},
  {code:'R22',r:108,g:55,b:47},
  {code:'R23',r:254,g:184,b:114},
  {code:'R24',r:243,g:193,b:192},
  {code:'R25',r:201,g:103,b:94},
  {code:'R26',r:210,g:147,b:190},
  {code:'R27',r:234,g:140,b:177},
  {code:'R28',r:156,g:135,b:214},
  {code:'R3',r:253,g:131,b:36},
  {code:'R4',r:248,g:236,b:49},
  {code:'R5',r:53,g:199,b:91},
  {code:'R6',r:35,g:136,b:145},
  {code:'R7',r:25,g:119,b:157},
  {code:'R8',r:26,g:96,b:195},
  {code:'R9',r:154,g:86,b:180},
  {code:'T1',r:255,g:255,b:255},
  {code:'Y1',r:253,g:111,b:180},
  {code:'Y2',r:254,g:180,b:129},
  {code:'Y3',r:215,g:250,b:160},
  {code:'Y4',r:139,g:219,b:250},
  {code:'Y5',r:233,g:135,b:234},
  {code:'ZG1',r:218,g:171,b:179},
  {code:'ZG2',r:214,g:170,b:135},
  {code:'ZG3',r:193,g:189,b:141},
  {code:'ZG4',r:150,g:134,b:159},
  {code:'ZG5',r:132,g:144,b:166},
  {code:'ZG6',r:148,g:191,b:226},
  {code:'ZG7',r:226,g:169,b:210},
  {code:'ZG8',r:171,g:145,b:192},
];

// Create lookup map for fast matching
const MARD_MAP = {};
MARD_COLORS.forEach(c => { MARD_MAP[c.code] = c; });

// ===== STATE =====
let warehouse = {};        // {A1: 500, A10: 200, ...}
let lastSnapshot = null;   // snapshot before last deduction for undo
let isDirty = false;       // unsaved changes in warehouse
let matchResults = [];     // current image match: [{code, count, ...}]
let editingCode = null;    // which color card is being edited

// ===== STORAGE KEYS =====
const STORAGE_KEY_WAREHOUSE = 'bead_warehouse';
const STORAGE_KEY_TOKEN = 'bead_github_token';
const STORAGE_KEY_REPO = 'bead_github_repo';
```

- [ ] **Step 2: Add initialization function**

Add after the state definitions:

```javascript
// ===== INITIALIZATION =====
function init() {
  // Load warehouse from localStorage
  const saved = localStorage.getItem(STORAGE_KEY_WAREHOUSE);
  if (saved) {
    try { warehouse = JSON.parse(saved); } catch(e) { initEmptyWarehouse(); }
  } else {
    initEmptyWarehouse();
  }
  
  // Check GitHub config
  updateStatusBar();
  checkSetup();
  
  // Render warehouse
  renderWarehouse();
}

function initEmptyWarehouse() {
  warehouse = {};
  MARD_COLORS.forEach(c => { warehouse[c.code] = 0; });
}

// Call init on page load
init();
```

- [ ] **Step 3: Verify data loads correctly**

Open `index.html` in browser, open console (F12), verify:
- `MARD_COLORS.length` is 291
- `warehouse` has all 291 keys with value 0

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add 291 MARD colors and state initialization"
```

---

### Task 3: Warehouse Grid Rendering, Search, and Edit

**Files:**
- Modify: `index.html` (add JS functions)

- [ ] **Step 1: Add renderWarehouse, switchTab, search functionality**

Add after the `init()` call:

```javascript
// ===== TAB SWITCHING =====
function switchTab(tab) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
  document.querySelector(`.tab[data-tab="${tab}"]`).classList.add('active');
  document.getElementById(`panel-${tab}`).classList.add('active');
}

// ===== WAREHOUSE RENDERING =====
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
    
    const dirtyClass = (editingCode === c.code) ? ' dirty' : '';
    
    return `
      <div class="color-card${dirtyClass}" onclick="openEditModal('${c.code}')" title="${c.code} - 库存: ${qty}">
        <div class="swatch" style="background:rgb(${c.r},${c.g},${c.b})"></div>
        <div class="code">${c.code}</div>
        <div class="qty ${qtyClass}">${qty}</div>
      </div>
    `;
  }).join('');
}

// ===== EDIT MODAL =====
function openEditModal(code) {
  const c = MARD_MAP[code];
  editingCode = code;
  document.getElementById('edit-swatch').style.background = `rgb(${c.r},${c.g},${c.b})`;
  document.getElementById('edit-code').textContent = c.code;
  document.getElementById('edit-rgb').textContent = `RGB(${c.r}, ${c.g}, ${c.b})`;
  document.getElementById('edit-qty').value = warehouse[code] || 0;
  document.getElementById('edit-modal').classList.add('show');
  setTimeout(() => document.getElementById('edit-qty').select(), 100);
}

function closeEditModal() {
  document.getElementById('edit-modal').classList.remove('show');
  editingCode = null;
  renderWarehouse();
}

function saveEdit() {
  const newQty = parseInt(document.getElementById('edit-qty').value) || 0;
  warehouse[editingCode] = newQty;
  isDirty = true;
  saveToLocal();
  closeEditModal();
  updateStatusBar();
  toast('已修改 ' + editingCode);
}
```

- [ ] **Step 2: Add toast and saveToLocal helper**

```javascript
// ===== TOAST =====
let toastTimer;
function toast(msg) {
  const el = document.getElementById('toast');
  el.textContent = msg;
  el.classList.add('show');
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => el.classList.remove('show'), 2000);
}

// ===== LOCAL PERSISTENCE =====
function saveToLocal() {
  localStorage.setItem(STORAGE_KEY_WAREHOUSE, JSON.stringify(warehouse));
}

function updateStatusBar() {
  const hasToken = !!(localStorage.getItem(STORAGE_KEY_TOKEN) && localStorage.getItem(STORAGE_KEY_REPO));
  const statusEl = document.getElementById('status-text');
  if (hasToken) {
    statusEl.innerHTML = '<span class="status-dot cloud"></span> 云端模式';
  } else {
    statusEl.innerHTML = '<span class="status-dot local"></span> 本地模式';
  }
  
  const dirtyEl = document.getElementById('dirty-indicator');
  dirtyEl.textContent = isDirty ? '⚠ 有未保存的修改' : '';
}

// Close modals on overlay click
document.addEventListener('click', function(e) {
  if (e.target.classList.contains('modal-overlay')) {
    e.target.classList.remove('show');
    editingCode = null;
    renderWarehouse();
  }
});
```

- [ ] **Step 3: Verify warehouse works**

Open `index.html`, test:
- All 291 colors display in grid with correct RGB swatches
- Type in search box filters colors (e.g., "A1" shows A1, A10-A19)
- Click a color card → modal opens with correct color and quantity
- Change quantity → saves, card updates, toast shows
- Refresh page → quantity persists (localStorage)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add warehouse grid, search, inline edit with modal"
```

---

### Task 4: GitHub Token Setup Guide Modal

**Files:**
- Modify: `index.html` (add JS for setup modal)

- [ ] **Step 1: Add setup modal functions**

Add after existing JS:

```javascript
// ===== SETUP MODAL =====
function checkSetup() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN);
  const repo = localStorage.getItem(STORAGE_KEY_REPO);
  if (!token || !repo) {
    showSetupModal();
  }
}

function showSetupModal() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN) || '';
  const repo = localStorage.getItem(STORAGE_KEY_REPO) || '';
  document.getElementById('setup-token').value = token;
  document.getElementById('setup-repo').value = repo;
  document.getElementById('setup-modal').classList.add('show');
}

function skipSetup() {
  document.getElementById('setup-modal').classList.remove('show');
  updateStatusBar();
}

function saveSetup() {
  const token = document.getElementById('setup-token').value.trim();
  const repo = document.getElementById('setup-repo').value.trim();
  
  if (!token || !repo) {
    alert('请填写 Token 和仓库路径，或点击"跳过"使用本地模式');
    return;
  }
  
  localStorage.setItem(STORAGE_KEY_TOKEN, token);
  localStorage.setItem(STORAGE_KEY_REPO, repo);
  document.getElementById('setup-modal').classList.remove('show');
  updateStatusBar();
  toast('已连接 GitHub！点击"云端加载"获取数据');
}
```

- [ ] **Step 2: Verify setup flow**

Open `index.html` with clean localStorage:
- Setup modal appears immediately
- Click "Skip" → modal closes, status shows local mode
- Open setup again via ⚙️ button → enter dummy data → saves
- Status shows cloud mode

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add GitHub token setup guide modal"
```

---

### Task 5: GitHub API Integration (Save/Load)

**Files:**
- Modify: `index.html` (add GitHub API functions)

- [ ] **Step 1: Add GitHub API helper functions**

Add after setup functions:

```javascript
// ===== GITHUB API =====
function getApiHeaders() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN);
  return {
    'Authorization': `Bearer ${token}`,
    'Accept': 'application/vnd.github.v3+json',
  };
}

function getRepo() {
  return localStorage.getItem(STORAGE_KEY_REPO);
}

function getApiUrl() {
  return `https://api.github.com/repos/${getRepo()}/contents/warehouse.csv`;
}

async function loadFromGitHub() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN);
  if (!token || !getRepo()) {
    toast('请先配置 GitHub 连接');
    showSetupModal();
    return;
  }
  
  const btn = document.getElementById('btn-load-cloud');
  btn.disabled = true;
  btn.textContent = '⏳ 加载中...';
  
  try {
    const resp = await fetch(getApiUrl(), { headers: getApiHeaders() });
    if (resp.status === 404) {
      // No warehouse.csv yet — initialize empty
      initEmptyWarehouse();
      saveToLocal();
      isDirty = true;
      renderWarehouse();
      toast('云端无库存文件，已初始化为全 0');
      updateStatusBar();
      return;
    }
    if (!resp.ok) {
      const err = await resp.json();
      throw new Error(err.message || `HTTP ${resp.status}`);
    }
    
    const data = await resp.json();
    const content = atob(data.content.replace(/\n/g, ''));
    const parsed = parseCSV(content);
    
    // Merge: update warehouse with cloud values
    MARD_COLORS.forEach(c => {
      if (parsed[c.code] !== undefined) {
        warehouse[c.code] = parseInt(parsed[c.code]) || 0;
      }
    });
    
    isDirty = false;
    saveToLocal();
    renderWarehouse();
    updateStatusBar();
    toast('已从云端加载库存');
  } catch (e) {
    toast('加载失败: ' + e.message);
    console.error(e);
  } finally {
    btn.disabled = false;
    btn.textContent = '🔄 云端加载';
  }
}

async function saveToGitHub() {
  const token = localStorage.getItem(STORAGE_KEY_TOKEN);
  if (!token || !getRepo()) {
    toast('请先配置 GitHub 连接');
    showSetupModal();
    return;
  }
  
  const btn = document.getElementById('btn-save-cloud');
  btn.disabled = true;
  btn.textContent = '⏳ 保存中...';
  
  try {
    const csv = warehouseToCSV();
    const content = btoa(unescape(encodeURIComponent(csv)));
    
    // Get current file SHA (if exists)
    let sha = null;
    try {
      const existing = await fetch(getApiUrl(), { headers: getApiHeaders() });
      if (existing.ok) {
        const data = await existing.json();
        sha = data.sha;
      }
    } catch(e) { /* file doesn't exist yet */ }
    
    const body = {
      message: 'Update warehouse.csv',
      content: content,
    };
    if (sha) body.sha = sha;
    
    const resp = await fetch(getApiUrl(), {
      method: 'PUT',
      headers: { ...getApiHeaders(), 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
    });
    
    if (!resp.ok) {
      const err = await resp.json();
      throw new Error(err.message || `HTTP ${resp.status}`);
    }
    
    isDirty = false;
    updateStatusBar();
    toast('已保存到云端 ✅');
  } catch (e) {
    toast('保存失败: ' + e.message);
    console.error(e);
  } finally {
    btn.disabled = false;
    btn.textContent = '💾 保存到云端';
  }
}

// ===== CSV HELPERS =====
function warehouseToCSV() {
  let csv = '色号,库存\n';
  MARD_COLORS.forEach(c => {
    csv += `${c.code},${warehouse[c.code] || 0}\n`;
  });
  return csv;
}

function parseCSV(text) {
  const result = {};
  const lines = text.trim().split('\n');
  // Skip header if present
  const start = lines[0].toLowerCase().includes('色号') || lines[0].toLowerCase().includes('code') ? 1 : 0;
  for (let i = start; i < lines.length; i++) {
    const parts = lines[i].split(',');
    if (parts.length >= 2) {
      result[parts[0].trim()] = parseInt(parts[1].trim()) || 0;
    }
  }
  return result;
}
```

- [ ] **Step 2: Verify API flow**

Test with a real GitHub token and repo:
- Click "云端加载" → fetches warehouse.csv or initializes empty
- Edit some quantities in warehouse
- Click "保存到云端" → saves CSV to GitHub
- Check GitHub repo → warehouse.csv exists with correct data
- Test error case: bad token → shows error toast

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add GitHub API save/load for warehouse.csv"
```

---

### Task 6: Image Upload and Canvas Color Extraction

**Files:**
- Modify: `index.html` (add image processing functions)

- [ ] **Step 1: Add file upload and drag-drop handlers**

Add JS:

```javascript
// ===== IMAGE UPLOAD & PROCESSING =====
const uploadZone = document.getElementById('upload-zone');

uploadZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  uploadZone.classList.add('dragover');
});

uploadZone.addEventListener('dragleave', () => {
  uploadZone.classList.remove('dragover');
});

uploadZone.addEventListener('drop', (e) => {
  e.preventDefault();
  uploadZone.classList.remove('dragover');
  const file = e.dataTransfer.files[0];
  if (file) processImage(file);
});

function handleFileSelect(event) {
  const file = event.target.files[0];
  if (file) processImage(file);
}

function processImage(file) {
  if (!file.type.match(/image\/(png|jpeg|webp)/)) {
    toast('请上传 PNG、JPG 或 WebP 格式的图片');
    return;
  }
  
  const reader = new FileReader();
  reader.onload = function(e) {
    const img = new Image();
    img.onload = function() {
      extractColors(img);
      showPreview(img);
    };
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}

function showPreview(img) {
  const container = document.getElementById('preview-container');
  container.style.display = 'block';
  container.innerHTML = `<img src="${img.src}" alt="预览">`;
}

function extractColors(img) {
  // Limit max dimension to 2000px
  let w = img.naturalWidth;
  let h = img.naturalHeight;
  const maxDim = 2000;
  if (w > maxDim || h > maxDim) {
    const scale = maxDim / Math.max(w, h);
    w = Math.floor(w * scale);
    h = Math.floor(h * scale);
  }
  
  const canvas = document.createElement('canvas');
  canvas.width = w;
  canvas.height = h;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(img, 0, 0, w, h);
  
  const imageData = ctx.getImageData(0, 0, w, h);
  const pixels = imageData.data;
  
  // Count unique colors (quantized for performance)
  const colorCount = {};
  for (let i = 0; i < pixels.length; i += 4) {
    const r = pixels[i];
    const g = pixels[i + 1];
    const b = pixels[i + 2];
    const a = pixels[i + 3];
    
    if (a < 128) continue; // skip transparent pixels
    
    const key = `${r},${g},${b}`;
    colorCount[key] = (colorCount[key] || 0) + 1;
  }
  
  const uniqueColors = Object.entries(colorCount).map(([key, count]) => {
    const [r, g, b] = key.split(',').map(Number);
    return { r, g, b, count };
  });
  
  if (uniqueColors.length > 500) {
    toast('检测到超过 500 种颜色，可能不是拼豆图，请确认');
  }
  
  matchColorsToMARD(uniqueColors);
}
```

- [ ] **Step 2: Add color matching algorithm**

```javascript
function matchColorsToMARD(colors) {
  // Match each unique color to nearest MARD color
  const matched = {};
  
  colors.forEach(({r, g, b, count}) => {
    let bestCode = null;
    let bestDist = Infinity;
    
    for (const m of MARD_COLORS) {
      const dr = r - m.r;
      const dg = g - m.g;
      const db = b - m.b;
      const dist = dr * dr + dg * dg + db * db; // squared Euclidean (faster, same result)
      
      if (dist < bestDist) {
        bestDist = dist;
        bestCode = m.code;
      }
    }
    
    if (!matched[bestCode]) {
      matched[bestCode] = 0;
    }
    matched[bestCode] += count;
  });
  
  // Convert to sorted array
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

- [ ] **Step 3: Add match results rendering**

```javascript
function renderMatchResults() {
  const container = document.getElementById('match-results');
  const actions = document.getElementById('match-actions');
  
  if (matchResults.length === 0) {
    container.innerHTML = '';
    actions.style.display = 'none';
    return;
  }
  
  const total = matchResults.reduce((s, m) => s + m.count, 0);
  
  container.innerHTML = `
    <div style="margin-bottom:8px;font-size:13px;font-weight:600;">
      匹配结果（共 <b>${total}</b> 颗豆子，<b>${matchResults.length}</b> 种色号）
    </div>
    <div class="match-list">
      ${matchResults.map(m => `
        <div class="match-item${m.insufficient ? ' insufficient' : ''}">
          <div class="m-swatch" style="background:rgb(${m.swatch.r},${m.swatch.g},${m.swatch.b})"></div>
          <span class="m-code">${m.code}</span>
          <span class="m-count">×${m.count}</span>
        </div>
      `).join('')}
    </div>
    <button class="btn btn-primary" onclick="confirmDeduction()" style="width:100%;margin-bottom:8px;">
      ✅ 确认扣除（共 <b>${total}</b> 颗）
    </button>
  `;
  
  actions.style.display = 'flex';
  
  // Disable confirm if any insufficient? No — allow per spec
}
```

- [ ] **Step 4: Verify image processing**

Upload a test bead pattern image:
- Image preview shows
- Match results appear with color swatches and counts
- Colors correctly map to nearest MARD equivalents
- Test with solid color blocks to verify matching accuracy

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add image upload, Canvas color extraction, and MARD matching"
```

---

### Task 7: Deduction Confirmation and Undo

**Files:**
- Modify: `index.html` (add deduction and undo functions)

- [ ] **Step 1: Add confirm and undo functions**

```javascript
// ===== DEDUCTION & UNDO =====
function confirmDeduction() {
  if (matchResults.length === 0) return;
  
  // Save snapshot for undo
  lastSnapshot = JSON.parse(JSON.stringify(warehouse));
  
  // Deduct
  let totalDeducted = 0;
  matchResults.forEach(m => {
    warehouse[m.code] = (warehouse[m.code] || 0) - m.count;
    totalDeducted += m.count;
  });
  
  isDirty = true;
  saveToLocal();
  renderWarehouse();
  updateStatusBar();
  
  // Enable undo button
  document.getElementById('btn-undo').disabled = false;
  
  // Switch to warehouse tab to show results
  switchTab('warehouse');
  
  toast(`已扣除 ${totalDeducted} 颗豆子，记得保存到云端！`);
}

function undoLastDeduction() {
  if (!lastSnapshot) {
    toast('没有可撤销的操作');
    return;
  }
  
  warehouse = JSON.parse(JSON.stringify(lastSnapshot));
  lastSnapshot = null;
  isDirty = true;
  saveToLocal();
  renderWarehouse();
  updateStatusBar();
  
  document.getElementById('btn-undo').disabled = true;
  toast('已撤销上次扣除 ✅');
}

function clearMatches() {
  matchResults = [];
  document.getElementById('match-results').innerHTML = '';
  document.getElementById('match-actions').style.display = 'none';
  document.getElementById('preview-container').style.display = 'none';
  document.getElementById('preview-container').innerHTML = '';
  document.getElementById('file-input').value = '';
}
```

- [ ] **Step 2: Verify deduction and undo flow**

- Upload image → match results appear
- Click confirm deduction → warehouse quantities decrease
- Check insufficient stock items → quantities go negative, shown in red
- Click undo → quantities restored
- Click undo again → "没有可撤销的操作"
- New deduction after undo → old snapshot replaced

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add deduction confirmation and undo mechanism"
```

---

### Task 8: Edge Cases, Error Handling, and Polish

**Files:**
- Modify: `index.html` (add edge case handling)

- [ ] **Step 1: Add image scaling info and edge case guards**

Add at the end of `extractColors`, right after the maxDim scaling:

```javascript
  // After scaling, show processing info
  if (img.naturalWidth > maxDim || img.naturalHeight > maxDim) {
    toast(`图片已缩放到 ${w}×${h} 进行处理`);
  }
```

Update the color count warning in `extractColors`:

```javascript
  if (uniqueColors.length > 500) {
    if (!confirm(`检测到 ${uniqueColors.length} 种不同颜色（超过500种），可能不是拼豆图。仍要继续识别吗？`)) {
      clearMatches();
      return;
    }
  }
```

- [ ] **Step 2: Add beforeunload warning for unsaved changes**

```javascript
// Warn before leaving with unsaved changes
window.addEventListener('beforeunload', (e) => {
  if (isDirty) {
    e.preventDefault();
    e.returnValue = '你有未保存的修改，确定要离开吗？';
    return e.returnValue;
  }
});
```

- [ ] **Step 3: Add "Clear all" warehouse reset**

Add button to warehouse controls (in the `.controls` div):

```html
<button class="btn btn-outline btn-sm" onclick="resetWarehouse()" style="color:var(--danger)">🗑 清空库存</button>
```

And the function:

```javascript
function resetWarehouse() {
  if (!confirm('确定要清空所有库存吗？此操作不可撤销！')) return;
  initEmptyWarehouse();
  isDirty = true;
  saveToLocal();
  renderWarehouse();
  updateStatusBar();
  toast('所有库存已清零');
}
```

- [ ] **Step 4: Add CSV export fallback for non-GitHub users**

Add to warehouse controls:

```html
<button class="btn btn-outline btn-sm" onclick="exportCSV()">📥 导出 CSV</button>
<button class="btn btn-outline btn-sm" onclick="importCSV()">📤 导入 CSV</button>
```

And functions:

```javascript
function exportCSV() {
  const csv = warehouseToCSV();
  const blob = new Blob(['﻿' + csv], { type: 'text/csv;charset=utf-8;' }); // BOM for Excel
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'warehouse.csv';
  a.click();
  URL.revokeObjectURL(url);
  toast('CSV 已下载');
}

function importCSV() {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = '.csv';
  input.onchange = function(e) {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = function(re) {
      const parsed = parseCSV(re.target.result);
      MARD_COLORS.forEach(c => {
        if (parsed[c.code] !== undefined) {
          warehouse[c.code] = parseInt(parsed[c.code]) || 0;
        }
      });
      isDirty = true;
      saveToLocal();
      renderWarehouse();
      updateStatusBar();
      toast('CSV 已导入');
    };
    reader.readAsText(file);
  };
  input.click();
}
```

- [ ] **Step 5: Verify edge cases**

- Upload very large image (>2000px) → automatically scaled
- Upload photo (not bead pattern) with 500+ colors → warning shown
- Close tab with unsaved changes → browser warning
- Export CSV → downloads correctly, re-imports with correct values
- Reset warehouse → all zeros

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add edge case handling, CSV import/export, and polish"
```

---

### Task 9: Final Verification

**Files:**
- Verify: `index.html`

- [ ] **Step 1: Full workflow test**

Test the complete user flow:

1. **First-time setup:**
   - Open `index.html` in fresh browser/profile
   - Setup modal appears → configure GitHub token + repo
   - Click "云端加载" → warehouse initialized to all 0

2. **Warehouse management:**
   - Search "A1" → only A1 shows
   - Click A1 card → edit quantity to 500
   - Search all → verify A1 shows 500
   - Click "保存到云端" → check GitHub for warehouse.csv

3. **Image matching:**
   - Switch to "图转色号" tab
   - Upload a bead pattern screenshot
   - Preview shows, match results appear
   - Verify colors map to expected MARD codes
   - Click "确认扣除" → warehouse quantities update

4. **Undo:**
   - Click "撤销上次扣除" → quantities restore
   - Verify undo button disables after use

5. **Cross-device:**
   - Open the same URL on phone browser
   - Click "云端加载" → same data appears
   - Edit on phone, save to cloud
   - Refresh desktop → load from cloud → changes appear

6. **Local fallback:**
   - Clear localStorage
   - Skip setup → local mode
   - Edit warehouse, refresh → persists in localStorage
   - Import/export CSV works

- [ ] **Step 2: Visual and responsive check**

- Desktop: All 291 colors in grid, cards sized correctly
- Mobile (Chrome DevTools device mode, 375px width): Grid reflows, modals fit screen
- White theme consistent across all elements
- Tab switching smooth, modal animations work

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: final verification and responsive adjustments"
```

---

### Task 10: README and Deployment Guide

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README**

```markdown
# 拼豆色号助手 🧩

管理 MARD 拼豆色号库存，支持图片自动匹配色号并扣减库存。部署在 GitHub Pages，手机电脑同步使用。

## 快速开始

1. **部署**: 将此仓库开启 GitHub Pages（Settings → Pages → Source: main branch → Save）
2. **获取 Token**: 打开 [GitHub Token 设置](https://github.com/settings/tokens)，生成 classic token，勾选 `repo` 权限
3. **打开网页**: 访问 `https://<你的用户名>.github.io/<仓库名>/`
4. **配置**: 首次进入输入 Token + 仓库路径（如 `用户名/仓库名`）
5. **开始使用**: 点击"云端加载"获取库存数据

## 功能

### 📦 色号仓库
- 展示 291 个 MARD 色号，真实颜色标注
- 搜索色号、点击修改库存
- 保存到云端 / 从云端加载
- CSV 导入 / 导出

### 🎨 图转色号
- 上传拼豆截图自动识别颜色
- 自动匹配最近 MARD 色号
- 确认扣除库存（支持撤销）

## 仓库文件

- `index.html` — 应用主文件
- `warehouse.csv` — 库存数据（自动生成和维护）
```

- [ ] **Step 2: Verify deployment**

Push to GitHub, enable Pages, open the URL. Everything works.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README with setup and usage guide"
```

---

## Summary

**Total tasks:** 10
**Files created:** `index.html`, `README.md`
**No external dependencies.** Everything runs in the browser with no build step, no npm install, no framework.

After all tasks complete, the user has a fully functional single-file web app with:
- MARD color warehouse with search and inline editing
- Image-to-color matching with automatic MARD code detection
- GitHub cloud sync for cross-device use
- Undo support
- CSV import/export fallback
- Responsive design for mobile and desktop
