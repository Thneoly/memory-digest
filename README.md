# memory-digest · 跨应用共享记忆系统的读端 demo

> **OctoSense 黑客松参赛项目 · 商店应用赛道**
> 演示"让助手用你自己的兴趣点组织一段 digest"的交互模型。

> ℹ️ **本仓库是被动 bundle**——`bundle/main.splash` 是纯脚本 + 数据；不安装 hook / 不触发浏览器跳转 / 不发起任何 HTTP 调用。你看到 `vscode.dev/github/...` 这类链接是被你本地 IDE / GitHub 扩展 / 浏览器插件打开的，不是本仓库干的。

---

## 一句话

`Memory Digest` 把你的若干兴趣点（chips）聚合，**点一下 Refresh** → 走 `octos.turn.start` 让助手组织一段话；失败 → 用本地 prefs **拼一句本地 digest**——AI 是渐进增强层。

> 📘 完整方案与三仓联合演示见 [`docs/JOINT-DEMO.md`](../os-memory/docs/JOINT-DEMO.md)

---

## 角色

在三仓架构里，本仓是**读端**——演示"用 App 自己的 Agent 读自己的 peer，把内容聚合展示"：

```
   ┌────────────┐
   │memory-digest│
   │  （读端）   │   ↑ Refresh 按钮 + chips 编辑
   │   ┌──┐    │   ↓ host.request("octos.turn.start", ...)
   │   │读│  → │  失败 → "No assistant on this device — 
   │   └──┘  → │         from your local interests"
   │           │
        ↓
       peer ─────→ os.memory (主作品) 汇聚
```

**与 `notes` 的对比**：

| | notes | digest（本仓） |
|---|---|---|
| 颗粒度 | 逐条（每行一个按钮） | **聚合**（一次按钮 = 一次完整更新） |
| 编辑 | 增删便签 | 增删兴趣 chips |
| 失败语义 | 单条独立降级 | 整段聚合降级（拼本地 prefs） |

---

## 本仓特异

| 字段 | 值 |
|---|---|
| 应用 id | `memory-digest` |
| 版本 | `0.1.0` |
| 命名空间 | 商店应用（自有 id） |
| 提交路径 | `octo check` + `hub check --publisher-key` |
| 资源上限 | 16 MiB storage（16,777,216 bytes）|
| Agent profile | `read-only` |
| Capabilities | `storage` + `octos.turn.start` |
| Platforms | `windows`（其他平台未验证，**不假装**） |

### Gate 状态

代码完整 + manifest stamp 已就绪 + main.splash 行为符合 BRIEF。本地实测：

```bash
python ../OctoScript-App-Design-Flow/tools/octo check bundle
# 实测：memory-digest 0.1.0 — PASSED
```

### 关键源码（`bundle/main.splash` · 100 行 · 摘要）

- 持久状态 `prefs[]`（写入 `prefs.json`）
- `load()`：启动读 `prefs.json`，渲染 chips
- `local_digest()`：拼成本地句子 `"You like: a, b, c."`
- `refresh()`：
  1. `ui.source.set_text("Asking the assistant…")`
  2. `host.request("octos.turn.start", {text: "In one short paragraph, summarize what you remember about my interests and preferences."}, callback)`
  3. 成功：`ui.digest.set_text(r.data.text)` + `ui.source.set_text("From your assistant")`
  4. 失败：`show_local("No assistant on this device — from your local interests")` —— **诚实降级**
- `add_pref()` / `remove_pref(i)`：编辑 + save
- 启动：`start_timeout(0.05, || { load(); refresh() })` —— **进入即加载 + 主动尝试一次 Refresh**
- UI：白底 + 大标题 + 圆角卡片（digest / source / Refresh 按钮）+ 输入条 + Add + ScrollYView chips（点 chip 删）+ 灰色提示行

---

## 关键决策（锚定）

完整决策清单见 [`docs/JOINT-DEMO.md` § 3](../os-memory/docs/JOINT-DEMO.md)。本仓最相关：

- **A3.1 诚实降级是核心 UX**——Refresh 失败时显示本地拼句 `You like: a, b, c.`，绝不能因 AI 不可用而坏 UI
- **A1.2 不发明 API**——`octos.turn.start` 是模板先例；本仓只用它（manifest `capabilities` 与代码一致，未引入未文档化字段）
- **A1.4 可见窗口演示**——启动 0.05s 即调 refresh（演示态可见）

---

## 演示与复现

### 截图

- `build/demo-digest.png` — 本地演示快照
- `bundle/screenshots/01-main.png` — 主屏（首启动空 prefs → "no interests yet"）
- `bundle/screenshots/02-with-prefs.png` — 3 个兴趣 chip + Refresh 按钮
- `bundle/screenshots/03-after-refresh.png` — 点 Refresh 后（card-host 无 `octos.*` → 降级到本地 prefs 拼句）

### 复现命令

```bash
git clone https://github.com/Thneoly/memory-digest.git
cd memory-digest

# 自检
python ../OctoScript-App-Design-Flow/tools/octo check bundle

# 启动 card-host 跑起来（独立 16 MiB）
card-host bundle --port 8142
```

### 现场试：chips 增删 + Refresh + 看诚实降级

1. 输入 "dark mode" → Enter → 添加 chip
3. 点 **Refresh**
4. 状态行：`Asking → From your assistant`（成功）或 `Asking → from your local interests`（降级）
5. 关掉 card-host → 再点 → 确认降级路径生效（"You like: dark mode."）

---

## 已知边界

- **Windows only** — `platforms: ["windows"]`
- **未签名** — `publisher-signature: unsigned`（首次可）
- **未转 public** — 赛前必转 public
- **octo check 已 PASSED** — 2026-10-02 实测；赛后重新 `octo check` stamp 会再次变化
- **首启动 prefs.json 空** — `.local-state/memory-digest/prefs.json = []`，冷启动显示引导态

---

## 项目结构

```
memory-digest/
├── README.md            ← 你在这
├── BRIEF.md             ← 简报
├── AGENTS.md            ← 跨 agent 开发说明
├── CLAUDE.md / GEMINI.md ← @AGENTS.md 转发
├── bundle/
│   ├── manifest.json    ← stamp manifest
│   ├── listing.json     ← store 元数据（publisher 占位待替换）
│   ├── main.splash      ← 主程序（100 行 · Refresh + chips）
│   ├── screenshots/01-main.png
│   └── assets/icon.svg
├── build/
│   └── demo-digest.png
└── .local-state/memory-digest/prefs.json  ← 运行时兴趣数据
```

---

## 提交前 TODO

见 [`docs/JOINT-DEMO.md` § 6](../os-memory/docs/JOINT-DEMO.md)。本仓特异：
- [x] 跑 `octo check` 实测 gate（2026-10-02 PASSED，2026-10-03 重截后重算 stamp）
- [x] 生成 Packet（`build/review.json` + `build/REVIEW-ANSWERS.md`）
- [x] publisher 占位替换（Thneoly）
- [x] 补 screenshots 3 张：空 / with prefs / after-refresh（2026-10-03 重截）
- [ ] 生成 publisher key（ed25519）+ sign-manifest（人类步骤）

---

## 参考

- 联合演示：[`docs/JOINT-DEMO.md`](../os-memory/docs/JOINT-DEMO.md)
- 比赛规则：[App Hub docs/PUBLISHING.md](https://github.com/OctoSense-org/OctoSense-App-Hub/blob/main/docs/PUBLISHING.md)

---

*生成于 2026-10-02 · 锚定大会话 `6d0c2850...`（2026-09-29 → 2026-10-02）· mcp memory #30*