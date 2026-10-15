# TelyClaw Growth Engine

> **Real-time Social Trend → Content Growth Engine**
> TelyClaw Growth Lead · Vibe Coding Challenge 交付物

监控 X 上的财经账号 → 自动识别正在加速的热点 → 用可解释的机会分排出最值得跟进的话题 → 生成三个角度的内容草稿 → 人工审核后发布，全程沉淀增长指标。

工作流：`Monitor → Detect → Rank → Ideate → Craft → Review → Publish`
ai负责产能，人负责判断。

| 交付物 | 位置 |
|---|---|
| 可运行的 Demo（在线） | <https://telyclaw-growth-engine-28237.app.workbuddy.host/> |
| 源代码仓库 | <https://github.com/jaxie-growth/telyclaw-2.0> |
| 详细说明文档（17 节） | [`DETAILED.md`](./DETAILED.md) |

运行方式：`node server.js`（零第三方依赖，Node 18+），点击在线demo，点概览页「运行完整流水线」即可端到端跑通。

---

## Product Logic（产品逻辑）

**问题**：热点窗口太短。一个话题从起量到被写完只有几小时，人工"看盘 → 判断 → 写稿 → 审核"追不上。

**解法**：把热点捕捉变成一条有节奏的流水线。机器做机器擅长的（监控、聚类、打分、起稿、自评），判断权完整留给人（编辑、批准、发布）。核心指标对齐受众质量而非曝光量：北极星指标 = 由热点内容带来的互动 × ICP 匹配度。

## System Architecture（系统架构）

零第三方依赖的 Node.js 单体服务（`node:http` + ESM），`node server.js` 即可运行。

```
public/（前端）──► server.js（HTTP + API 路由 + 角色权限闸门）──► src/
                                                    ├─ ingest.js    数据源适配（simulated | X API v2）
                                                    ├─ pipeline.js  流水线编排
                                                    ├─ topics.js    聚类与话题识别
                                                    ├─ ranking.js   7 因子机会打分
                                                    ├─ content.js   内容生成 + 编辑 Agent + 原创度校验
                                                    ├─ scheduler.js 排期监控（定时 + 去重 + 静默）
                                                    ├─ metrics.js   指标口径
                                                    └─ store.js     JSON 文件持久化（可换 Postgres，接口不变）
```

## Trend Detection Logic（热点识别）

帖子做加权聚类（cashtag 权重更高），同一话题的碎片簇按主题词典合并，IDF 加权自动命名并生成摘要。趋势判断用**两套互补口径**：环比（窗口对半切，看趋势形状）+ 短窗动能（近段速度 ÷ 全窗平均，看是否正在爆）。单用前者会漏掉平台期话题，单用后者会被小样本抖动误导；两者组合能区分"正在起爆 / 退潮 / 平台期 / 噪声"。

## Opportunity Ranking Logic（机会排序）

评分是机会值不是热度，7 因子加权：爆发动能 18%、热度增速 22%、互动规模 16%、独立账号数 12%、时效性 8%、与 TelyClaw 相关性 14%、内容可生产性 10%。爆发动能单列（不并入热度增速），因为它衡量"相对自身基线放大了多少倍"——增长能抢到先手的是绝对热度不高但短时间翻倍的话题。每个因子归一化、可拆解，输出"为什么排第一"的理由 + 时效建议（立即动手 / 正常排期）。权重在设置页可调，实时重算。

## Content Generation Logic（内容生成）

针对 Top 话题生成三个角度（Hot Take / Educational / Product Angle），至少一条标记可直接发布（≤280 字符校验）。生成后有两道检查：**编辑 Agent 评审**（独立于生成环节，从钩子、具体性、原创度、受众匹配、清晰度五维打分，必须给出可执行的修改建议）和**原创度校验**（硬闸门：与源帖语义相似度 ≥45% 判涉嫌改写，连续 5 词重叠 ≥20% 判涉嫌复制，服务端拒绝批准与发布）。前者是提示，后者是闸门。

## 使用的 AI / Vibe Coding Tools

WorkBuddy、deepseek、gemini

## 当前 MVP 有哪些局限

- **数据是仿真**：数据集具备真实趋势形态（升温 / 衰减 / 爆发），用于验证口径与闭环，但不代表真实 X 表现；真实噪声（黑话、梗、跨语言）未覆盖。
- **发布是模拟**：真实 X 写入需要 OAuth2 用户上下文权限，未在本版实现，没有假装接了。
- **存储是 JSON 文件**：单进程写，多实例并发会冲突；生产换 Postgres，`store.js` 接口不变。
- **权限是演示级**：角色分离的校验逻辑在服务端真实生效，但身份切换是下拉选择，无真实登录体系。
- **调度是单实例**：`setInterval` 轮询，多实例部署需换分布式调度，避免重复提醒。
- 生产化路径（数据获取三阶段、真实发布接入）写在 [`DETAILED.md`](./DETAILED.md) §4 与 §11。

---

> 补充能力：排期监控（定时轮询 + 提醒去重 + 静默时段穿透）、角色分离与权限（作者 / 审核人 / 发布人，服务端强制校验）、编辑 Agent 复评。详细设计见 [`DETAILED.md`](./DETAILED.md)。
