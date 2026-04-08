---
title: "Crypto Trading Bot：基于 LLM 的 Go 量化交易 Agent"
date: 2026-04-08
description: "用 Go + 多智能体工作流，把行情分析、交易决策与风控串起来，并可在币安期货自动执行的交易系统。"
tags: ["Go", "LLM", "TradingBot", "Binance", "Quant"]
---

项目仓库：`Oakshen/crypto-trading-bot`

这是一个 **Go 语言实现** 的加密货币量化交易 Agent：用 **LLM 分析市场数据 → 生成交易信号 →（可选）自动下单**，并配套一个 Web Dashboard 做监控与数据查询。整体偏向 **趋势交易**：更强调「极致选择性 + 高盈亏比 + 耐心持有」，风控上以 **ATR 追踪止损** 为主（不做止盈，让利润奔跑）。

## 核心能力（从 README 提炼）

- **LLM 驱动的分析与决策**：把技术指标与市场状态整理成报告，交给模型输出结构化决策。
- **多交易对并行分析**：同时监控多个交易对，选择更优机会（README 建议不超过 3 个，避免过度分散）。
- **多时间周期（可选）**：短周期用于计算指标，长周期提供趋势背景（`ENABLE_MULTI_TIMEFRAME=true` + `CRYPTO_LONGER_TIMEFRAME`）。
- **风控优先**：保持 `ENABLE_STOPLOSS=true`，并建议用「单向持仓模式」运行（`BINANCE_POSITION_MODE=oneway`）。
- **Web 监控与查询**：默认 `http://localhost:8080`；支持 `make query` 查看 stats / 最近会话 / 指定交易对历史。
- **本地持久化**：使用 SQLite 保存运行与统计数据，便于回看与复盘。

## 架构思路：多智能体工作流（Eino Graph）

README 给出的编排大致是：

```text
START → [市场分析师, 情绪分析师]（并行）
           ↓
市场分析师 → 加密货币分析师 → 持仓信息
           ↓                 ↓
       情绪分析师 ──────→ 交易员（综合决策）
                              ↓
                            END
```

你可以把它理解成把「看盘 / 情绪 / 持仓 / 下单」拆成多段并行流水线：并行提高吞吐，也让 Prompt 更聚焦（每个 Agent 做一件事）。

## 技术栈（README）

- Go 1.21+
- 工作流编排：Cloudwego Eino
- Web：Hertz
- 交易所：go-binance
- 配置：Viper
- 日志：zerolog
- 数据库：SQLite3

## 快速开始（最小可跑通）

前置要求：Go 1.21+、币安期货账户、OpenAI 兼容 API Key（OpenAI / DeepSeek / Qwen 等）。

```bash
git clone https://github.com/Oakshen/crypto-trading-bot.git
cd crypto-trading-bot
make deps
make build-all
cp .env.example .env
```

然后在 `.env` 里至少确认这些关键项（示例值来自 README）：

```bash
# LLM
LLM_PROVIDER=openai
DEEP_THINK_LLM=deepseek-reasoner
QUICK_THINK_LLM=deepseek-chat
LLM_BACKEND_URL=https://api.deepseek.com
OPENAI_API_KEY=你的-api-key

# 交易对与时间周期
CRYPTO_SYMBOLS=BTC/USDT,ETH/USDT,SOL/USDT
CRYPTO_TIMEFRAME=15m
TRADING_INTERVAL=15m

# 多时间周期（可选）
ENABLE_MULTI_TIMEFRAME=true
CRYPTO_LONGER_TIMEFRAME=1h

# 风险管理与执行模式（重要）
ENABLE_STOPLOSS=true
AUTO_EXECUTE=false
BINANCE_POSITION_MODE=oneway
```

运行方式：

```bash
make run      # 单次执行（分析一次后退出）
make run-web  # 持续运行 + Web 界面
```

## 安全与风险提示（强烈建议照做）

- **先观察再自动**：先用 `AUTO_EXECUTE=false` 跑 `make run-web` 观察 1-2 天，再考虑开启自动执行。
- **从小仓位开始**：小仓位 + 保守杠杆，逐步调参。
- **只用单向持仓**：README 提示双向持仓模式可能存在 bug，建议 `BINANCE_POSITION_MODE=oneway`。
- **保护 API Key**：IP 白名单、最小权限、永不泄露密钥。
- **风险声明**：加密货币交易高风险，可能导致资金损失；本项目更适合学习与研究用途。

> 备注：GitHub 仓库侧边栏显示 License 为 Apache-2.0，但 README 末尾写了 MIT；使用时请以仓库 `LICENSE` 文件为准。

