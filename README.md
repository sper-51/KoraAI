# KoraAI

一个我用来研究和实践「多 Agent 协作 + Deep Research」的个人项目：输入一个竞品名称，系统会自动组队、联网查资料、交叉验证，最后产出一份带证据来源的竞品分析报告。

做这个项目主要是想搞明白几个问题：

- 多个 Agent 怎么编排才能真正完成一件事，而不是各说各话？
- LLM 生成的结论怎么保证"有据可查"，而不是一本正经地胡说？
- 全流程的思考过程（Prompt、输入输出、决策）怎么做到可视化、可回放？

## 它能做什么

1. **自动组队**：内置一套分层专家体系（决策 / 策略 / 执行三层），接到任务后按需挑选 Agent、下发指令、终审把关。
2. **真实采集**：接入真实搜索 API 和网页抓取，多轮多角度搜索，对抓回来的正文做乱码和相关性过滤。查不到的就如实写"未采集到"，不编数据。
3. **结构化产出**：报告之外还会输出功能树、定价对比、用户画像三类结构化数据，前端直接渲染成矩阵图和卡片。
4. **可信度打分**：每条证据按来源权威性、时效性、抓取质量真实计算可信度（0–100），不是写死的假分数。
5. **过程全透明**：每个 Agent 的 Prompt、输出、Token 消耗、决策理由全部记录成 Trace，工作台实时滚动，报告页可以"回放"决策过程。
6. **批注追问**：看报告时对某段划线批注，可以触发针对这一段的补充调研，自动更新对应章节。

## 技术栈

| 层 | 选型 |
|---|---|
| 前端 | React 19 + TypeScript + Vite，状态用 Zustand，流程可视化用 ReactFlow，图表 ECharts/D3，动效 Framer Motion |
| 后端 | FastAPI + SQLite，Agent 编排参考 LangGraph 的思路自己实现，进度推送用 SSE |
| 模型 | 智谱 GLM 系列（OpenAI 兼容网关），核心章节和辅助章节分配不同档位的模型，吃满并发额度 |
| 搜索 | 博查 Bocha Web Search |

## 跑起来

环境要求：Node.js ≥ 22.12，Python ≥ 3.9。

```bash
# 1. 后端配置密钥
cp backend/.env.example backend/.env
# 编辑 backend/.env，至少填 ZHIPU_API_KEY；需要联网采集再填 BOCHA_API_KEY

# 2. 启动后端
cd backend
python -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m uvicorn app.main:app --reload --port 8011

# 3. 启动前端
cd frontend
npm install
npm run dev
```

前端跑在 `http://localhost:3401`，后端 `http://localhost:8011`（健康检查 `/health`，LLM 连通性自检 `/api/llm/ping`）。

### 密钥说明

所有密钥只走环境变量，不硬编码。主要变量：

| 变量 | 用途 |
|---|---|
| `ZHIPU_API_KEY` | GLM 模型调用（[open.bigmodel.cn](https://open.bigmodel.cn) 申请） |
| `BOCHA_API_KEY` | 网络搜索（[open.bocha.cn](https://open.bocha.cn) 申请，`sk-` 开头） |
| `DOUYIN_COOKIE` / `BILIBILI_COOKIE` / `XHS_COOKIE` | 对应平台舆情采集，按需 |
| `ENABLE_DEMO_FALLBACK` | 没 key 时是否用缓存兜底，避免演示时崩 |

`.env`、数据库文件、运行日志都已写进 `.gitignore`，不会被提交。

## 目录结构

```
├── frontend/          # React 前端（pages 下是工作台、报告、图谱、Trace 等页面）
├── backend/           # FastAPI 后端（本地开发主目录）
│   └── app/
│       ├── core/      # 编排、LLM、搜索、抓取、可信度、Trace
│       └── data/      # 专家定义 experts.json
├── api/               # Vercel Serverless 部署入口（backend 的部署镜像）
├── docs/              # 架构、Agent 协议、部署等文档
└── restart.sh / stop.sh
```

## 一点设计笔记

- **返工闭环**：流水线里 audit 环节发现报告不达标可以打回去重写，而不是硬着头皮往下走。
- **多模型分档**：不是所有章节都值得用最好的模型，把模型按任务难度分配，速度和成本都能接受。
- **backend 与 api 目录重复**：是有意为之——Vercel 的 Serverless 函数约定放在 `api/`，本地开发用 `backend/`，两边逻辑保持一致。

## 来源与致谢

本仓库是我**学习研究多 Agent 系统的实践仓库**：基于优秀的开源项目 [VerdaAI-Investigator](https://github.com/kangjiayao14/VerdaAI-Investigator) 二次学习整理而来，核心架构与实现出自原作者，感谢开源。我在此基础上做了本地环境适配、文档重组与个人学习笔记，目标是吃透这套 Deep Research + 多 Agent 编排的设计思路。

## 许可证

AGPL-3.0。个人学习随意用；如果要对外提供服务或再分发，需要遵守 AGPL-3.0 的开源义务（详见 [LICENSE](./LICENSE)）。
