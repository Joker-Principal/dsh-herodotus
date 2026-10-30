# dsh-herodotus

> Herodotus 系统的 **DSH 客户端插件**（TypeScript / Cordis）。
>
> 这是一个**瘦客户端**：只做「捕获转发」与「召回注入」两件事，不含任何记忆逻辑。

## 职责

| 接缝 | 作用 |
| --- | --- |
| `session/event` 事件 | 捕获会话消息 → `POST /ingest` |
| `agent/pre-step` hook | 取召回结果 → **在用户消息之前 prepend 一条合成消息** |
| `ctx.tools.register` | 注册记忆工具（或经 MCP 提供） |
| `agent.ctx.systemPrompt.context()` | 注入**稳定内容**（人格、L3 画像快照） |
| `sidebar.panellist` / `main` / `settings.section` | 记忆面板与设置页（客户端半边） |

## 规划中的目录布局

```
dsh-herodotus\
├─ package.json  tsconfig.json          （V0）
├─ src\           host 侧：捕获、注入、工具、稳定区注入
├─ client\        浏览器半边：面板、设置页、召回可视化
└─ tests\
```

## 契约落点

契约正文在 [`../docs/PROTOCOL.md`](../docs/PROTOCOL.md) §5（C1–C7）。本文件**不复述**，只记在这个框架里怎么落。

| 契约 | 本框架的落点 |
| --- | --- |
| C1 降级不阻塞 | 捕获与召回全域 no-op 包裹；首次失败只告警一次 |
| C2 动态内容只进消息侧 | `agent/pre-step` 里 prepend 一条合成消息，带 `source: { kind: 'plugin', plugin: 'memory', form: 'recall' }` |
| C3 稳定内容进系统提示词 | `agent.ctx.systemPrompt.context()`，只放人格定义与 L3 画像快照 |
| C4 / C5 数据非指令 + 防回捕 | 🟡 待定：捕获侧只收 `source.kind === 'user'`，注入内容包裹并声明 |
| C6 显式 `personaId` | 由 persona 行 / preset 解析，逐请求携带 |
| C7 版本校验 | 启动时 `GET /health` 比对协议版本，不匹配则整体禁用并报错 |

### 框架自己的约束（与契约无关）

- `pre-step` 必须**先 `await next()` 再改写**消息列表，且以 prepend 方式注册。
- 面板 / 设置页属客户端半边，不参与记忆逻辑。

## 许可

**GPL-3.0**。归属声明见主仓库 [`NOTICE`](../NOTICE)。

DSH 侧的注入接缝设计参考 `dsh-layered-memory`（MIT）。
