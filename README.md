# 拼豆色号助手 🧩

管理 MARD 拼豆色号库存，支持图片自动匹配色号并扣减库存。部署在 GitHub Pages，手机电脑同步使用。

## 功能

### 📦 色号仓库
- 展示 291 个 MARD 色号，真实颜色标注
- 搜索色号、点击色块修改库存数量
- 保存到云端 / 从云端加载（GitHub API 同步）
- CSV 导入 / 导出（离线备份）

### 🎨 图转色号
- 上传拼豆截图（支持拖拽），自动识别颜色
- RGB 欧几里得距离匹配最近 MARD 色号
- 统计每种色号的像素数量
- 库存不足标黄警告，允许负数扣除
- ✅ 确认扣除 — 一键更新库存
- ↩ 撤销上次扣除 — 误操作可恢复

## 快速开始

### 1. 部署到 GitHub Pages

1. 将此仓库上传到你的 GitHub
2. Settings → Pages → Source: `main` 分支 → Save
3. 等待几分钟，获得网址：`https://<你的用户名>.github.io/<仓库名>/`

### 2. 创建 GitHub Token

1. 打开 [GitHub Token 设置](https://github.com/settings/tokens)
2. 点击 "Generate new token (classic)"
3. 勾选 **repo** 权限
4. 生成并复制 Token

### 3. 配置并开始使用

1. 打开 GitHub Pages 网址
2. 首次访问弹出配置框：粘贴 Token + 输入仓库路径（如 `用户名/仓库名`）
3. 点击「云端加载」获取库存数据（首次为空仓库）
4. 开始使用！

> **提示：** 跳过配置也可使用本地模式（数据存浏览器），但无法多设备同步。

## 仓库文件

```
├── index.html          # 应用主文件（单文件，无依赖）
├── warehouse.csv       # 库存数据（云端同步自动维护）
├── mard.csv            # MARD 色号对照表（参考）
└── README.md
```

## 技术说明

- 纯前端 HTML + CSS + JavaScript，无框架无构建工具
- GitHub API 读写 `warehouse.csv` 实现多设备同步
- Canvas API 逐像素取色 + 欧几里得距离匹配
- localStorage 本地缓冲 + 撤销快照

## 浏览器支持

Chrome / Edge / Safari / Firefox 最新版本均支持。
Firefox 不支持 File System Access API（但该功能未被使用，GitHub API 在所有浏览器中工作）。
