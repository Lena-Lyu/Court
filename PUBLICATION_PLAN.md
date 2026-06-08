# Court 公开发表方案
> 2026-06-08 | 目的：建立优先权 + 吸引同类人 + 找工作杠杆，而非开源全部代码

---

## 一、仓库结构

```
court/
├── README.md              # 中英双语，5分钟读完
├── README.zh.md           # 中文完整版
├── DESIGN.md              # court-agent-a2a 完整设计哲学
├── COMPETITIVE_LANDSCAPE.md  # 竞争分析（证明做过功课）
├── STATUS.md              # 诚实状态：什么跑通了，什么还没
├── docs/
│   ├── court-memory-design.md    # L0/L1/L3 架构设计
│   ├── five-invariants.md        # 五条脊柱详解
│   └── a2a-six-problems.md       # A2A 六子问题
├── LICENSE.code           # MIT — 代码
├── LICENSE.docs           # CC-BY-SA-4.0 — 文档
└── .gitignore
```

## 二、每份文件的内容要点

### README.md（中英双语）

**一句话定义：** Court — provenance-first memory for AI agents. Not a warehouse, a courtroom.

**结构：**
1. The problem (3段) — AI forgets → provenance loss → hallucinations → silent degradation
2. The insight (2段) — 808/斯巴鲁的 analogy → "raw evidence is the only truth"
3. The solution (压缩的五条脊柱 + 法庭流程)
4. What exists (诚实状态表)
5. Comparison (vs Eywa / TierMem / Mem0 / Anthropic Dreaming)
6. Who this is for / invitation

### DESIGN.md

court-agent-a2a-guide.md 的清理版——去掉内部对话痕迹，保留逻辑骨架。

### COMPETITIVE_LANDSCAPE.md

把 audit/competitive-landscape-update-2026-06.md 清理后放进来——证明：我知道每个人在做什么，而且我的框架比他们的都完整。

### STATUS.md（诚实状态）

| 组件 | 状态 | 详情 |
|------|------|------|
| Court design | ✅ 完成 | 五条脊柱 + 三层架构 + 七步开庭 |
| L0 raw storage | ⚠️ 部分 | observations 表存在，数据量小 |
| L1 10-channel RRF | ✅ 运行中 | ChannelRegistry 可插拔 |
| L3 adjudication | ⚠️ 部分 | action_score/recall_counter/cites_layer0 已实现，闸门机制修复中 |
| Agent layer | ⚠️ 部分 | AgentLoop 存在，Court 闸门接入中 |
| Weaver supervision | ✅ 运行中 | 10 条规则，硬边界 |
| A2A interface contracts | 🔜 设计中 | schema 强制校验开发中 |
| End-to-end benchmark | 🔜 待完成 | 目标 6 月底 |

## 三、不放入仓库的内容

完整代码仓库（Loom ~58K Rust + 织知 ~25K Python）保持在本地和 NAS 备份中。README 中指向它们作为 "full implementation reference (private during active development)"。

以下内容不放入公开 repo：
- Loom 全部源码（loomd/weaver/semcache/cli）
- 织知全部源码
- DS4 fork 修改
- 内部审计文档（audit/ 中的详细报告）
- 未来路线图（ROADMAP.md）
- 未验证的新想法

## 四、License 策略

- **代码片段（如有）：MIT** — 允许商业使用，要求保留版权声明。不限制别人用你的代码做产品。
- **文档：CC-BY-SA-4.0** — 必须署名，衍生作品必须同样共享。防止别人把你的设计文档改头换面后闭源。

**为什么不是更严格的协议？**
你的目标不是靠 license 收费——是让这套思想传播，而你被认可为源头。MIT + CC-BY-SA 最大化传播，同时保留署名权。未来做产品时，产品代码可以用商业 license——不影响。

## 五、发表时间线

**第 1 步（现在）：** 建 repo，写 README + DESIGN + COMPETITIVE_LANDSCAPE + STATUS
**第 2 步（本周）：** 知乎长文发表 → 引流到 GitHub
**第 3 步（下周）：** 英文版（dev.to / Medium）→ HN 试探
**第 4 步（持续）：** 一边修代码跑 benchmark，一边更新 STATUS.md

## 六、关于"论文"

你不需要发表学术论文来建立可信度。你的目标是：
- 工程师和团队 lead 读到你的 README → 想联系你
- 面试官看到你的 repo → 知道你不是在追风口
- 同类人找到你 → 一起做事

GitHub + 一篇好文章比一篇 arXiv 预印本对你的目标更有用。

## 七、关于"优先权"

一旦 GitHub repo 公开、带 timestamp 的 commit，你就有了优先权证据。不需要 arXiv。
如果以后有人做了类似的事——你说："我的 repo 比他早 X 个月/年"，链接贴过去。
