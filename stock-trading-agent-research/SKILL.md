---
name: stock-trading-agent-research
description: Reference notes for building an A股/美股 trading agent — not an active project, pull this out when the user wants to actually start. Use when user says something like "我们之前收藏的炒股那个" or wants to resume the trading-agent idea.
---

# 炒股 Agent —— 方案收藏（未启动，待用户主动提起再展开）

用户对"写一套规则让 agent 参照执行炒股"感兴趣，目前只是收藏调研结果，没有决定要不要做。下次被提起时，先确认范围（A股/美股、模拟盘/实盘）再动手，不要跳过这一步直接开始搭建。

## 通用架构（跨市场通用，可直接参考）

拆成 4 个角色，不要让一个 agent 全干：
1. **数据获取** agent —— 拉行情、新闻、财报
2. **信号/策略评估** agent —— 对着写好的规则判断买卖点
3. **订单执行** agent —— 真正下单（如果接实盘）
4. **风控/仓位管理** agent —— 止损、仓位上限

数据获取和数值计算（回测、指标计算）应该用传统工具（pandas/numpy 或现成框架），不要指望本地 LLM 处理原始行情数据；LLM 只负责"看着算好的指标做最后判断/给解释"。

## A股相关项目（国内市场，这是用户的目标市场）

- **[easytrader](https://github.com/shidenggui/easytrader)**（9.7K+ star，MIT）：不需要券商开放 API，用 `pywinauto` 模拟操作同花顺/华泰/海通/国金/银河等券商**电脑客户端**，实现自动下单。这是国内"怎么连接交易软件"问题的现实答案。**风险**：可能违反券商用户协议里"禁止自动化工具"的条款，用前看清楚条款。
- **[vnpy](https://github.com/vnpy/vnpy)**（VeighNa）：国内最知名的开源量化交易框架，40+ 交易接口，覆盖股票/期货/期权，自带回测和参数优化模块。更专业、更完整，适合做正式的量化底层。
- **[TradingAgents-CN](https://github.com/hsliuping/TradingAgents-CN)**：中文金融市场的多智能体 LLM 交易框架，思路跟用户最初想法最契合（写规则给 agent 执行），支持 A股/港股/美股分析。
- **[24mlight/A_Share_investment_Agent](https://github.com/24mlight/A_Share_investment_Agent)**：LLM 作为独立第三方分析师，给出多空观点，结合传统置信度计算。
- **[dromara/northstar](https://github.com/dromara/northstar)**：Java 实现，国内量化平台，可接 DeepSeek/Kimi 等大模型 API。

## 美股相关项目（仅供架构参考，API 接入方式国内用不上）

- **[HKUDS/AI-Trader](https://github.com/HKUDS/AI-Trader)**：明确支持 Claude Code/OpenClaw/Codex/Cursor 直接用，跨市场（股票/加密/外汇/期货/期权）。
- Alpaca、Interactive Brokers 的官方 API 集成方案——美股券商对散户开放 API，国内券商不开放，这部分不能照搬。

## 国内 vs 美股的关键差异（别忘了这几条）

- 国内券商无官方散户交易 API，只能走 easytrader 那种客户端模拟操作
- A股 T+1（当天买当天不能卖）+ 涨跌停限制，策略逻辑跟美股 T+0 不一样
- 国内数据订阅对应 Tushare/聚宽/掘金量化，跟美股 Polygon/IEX 不是一个量级
- 自动化交易工具在国内有合规/账号风险，务必先看券商条款

## 下次启动时该做的第一件事

跟用户确认：模拟盘验证可行性，还是直接想搭一套能用的系统？范围不同，路径完全不同（模拟盘只需要 vnpy/easytrader 的回测模块，不用碰真实下单逻辑）。
