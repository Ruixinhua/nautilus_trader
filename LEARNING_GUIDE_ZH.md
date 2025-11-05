# NautilusTrader 量化交易平台学习指南

## 📋 目录

- [一、平台概述](#一平台概述)
- [二、代码库结构](#二代码库结构)
- [三、数据获取和处理](#三数据获取和处理)
- [四、量化交易策略开发](#四量化交易策略开发)
- [五、可视化和性能分析](#五可视化和性能分析)
- [六、快速上手工作流](#六快速上手工作流)
- [七、学习路径建议](#七学习路径建议)
- [八、重要资源](#八重要资源)
- [九、实用技巧](#九实用技巧)

---

## 一、平台概述

### 1.1 什么是 NautilusTrader？

**NautilusTrader** 是一个开源的、生产级的高性能算法交易平台，具有以下核心特性：

- **AI-first**：在 Python 原生环境中开发和部署
- **混合架构**：核心用 Rust 编写（异步、高性能），Python 绑定通过 Cython 和 PyO3
- **事件驱动**：与实时交易环境兼容的事件驱动引擎
- **通用性**：支持多资产类（FX、股票、期货、期权、加密、DeFi、体育博彩）
- **多交易所**：14+ 个集成交易所/数据源

### 1.2 核心优势

| 特性 | 说明 |
|------|------|
| **统一代码库** | 回测和实盘使用相同的策略代码，无需重写 |
| **高性能** | Rust 核心 + 异步 I/O，支持高频交易 |
| **可扩展** | 灵活的适配器系统，易于集成新交易所 |
| **完整工具链** | 数据管理、回测、实盘、分析、可视化一站式解决方案 |
| **生产就绪** | 经过实战验证，支持真实资金交易 |

### 1.3 技术栈

```
编程语言：
- Python 3.12-3.14
- Rust 1.91.0+
- Cython 3.1.6

核心依赖：
- 数据处理：numpy, pandas, pyarrow
- 异步网络：uvloop (Unix), tokio (Rust)
- 序列化：msgspec
- 可视化：plotly (可选)
```

### 1.4 支持的交易所

- **加密货币**：Binance, Bybit, OKX, dYdX, Coinbase, Hyperliquid, Polymarket
- **传统金融**：Interactive Brokers
- **数据源**：Databento, Tardis
- **其他**：Betfair (体育博彩), BitMEX

---

## 二、代码库结构

### 2.1 主目录结构

```
nautilus_trader/
├── nautilus_trader/          # Python 主包 (~26k 行代码)
│   ├── accounting/           # 会计和资金管理
│   ├── adapters/             # 交易所适配器 (14+)
│   ├── analysis/             # 分析工具和报告生成
│   ├── backtest/             # 回测引擎
│   ├── cache/                # 数据缓存和状态管理
│   ├── common/               # 通用工具和常量
│   ├── config/               # 配置管理
│   ├── core/                 # 核心 Rust/Cython 绑定
│   ├── data/                 # 数据管理和处理
│   ├── execution/            # 执行引擎
│   ├── indicators/           # 技术指标库 (50+)
│   ├── live/                 # 实盘交易模块
│   ├── model/                # 领域模型（订单、持仓、数据等）
│   ├── persistence/          # 持久化存储
│   ├── portfolio/            # 投资组合管理
│   ├── risk/                 # 风险管理
│   ├── serialization/        # 序列化
│   ├── system/               # 系统内核
│   ├── test_kit/             # 测试工具
│   └── trading/              # 交易逻辑（策略、控制器）
│
├── crates/                   # Rust 源码 (24 个 crates)
│   ├── core/                 # 核心时间和计算
│   ├── model/                # Rust 领域模型
│   ├── backtest/             # Rust 回测引擎
│   ├── indicators/           # 技术指标实现
│   ├── adapters/             # 交易所适配器 (8 个)
│   └── ...                   # 其他模块
│
├── docs/                     # 完整文档 (Sphinx/Markdown)
│   ├── getting_started/      # 入门指南
│   ├── concepts/             # 核心概念
│   ├── api_reference/        # API 文档
│   ├── integrations/         # 集成指南
│   └── tutorials/            # 教程
│
├── examples/                 # 可运行示例
│   ├── backtest/             # 回测示例 (11 个)
│   ├── live/                 # 实盘示例 (14 个交易所)
│   ├── strategies/           # 策略示例 (10+)
│   └── notebooks/            # Jupyter 教程
│
└── tests/                    # 完整测试套件
    ├── unit_tests/
    ├── integration_tests/
    └── performance_tests/
```

### 2.2 关键模块职责

| 模块 | 职责 | 关键类/功能 |
|------|------|------------|
| **model** | 领域模型 | Instrument, Order, Position, Bar, Tick |
| **backtest** | 回测引擎 | BacktestEngine, BacktestNode |
| **live** | 实盘交易 | TradingNode, LiveEngine |
| **execution** | 执行管理 | ExecutionEngine, OrderBook |
| **trading** | 交易逻辑 | Strategy, Trader, Controller |
| **portfolio** | 投资组合 | Portfolio, PortfolioManager |
| **data** | 数据管理 | DataEngine, DataCatalog |
| **indicators** | 技术指标 | EMA, SMA, RSI, MACD (50+) |
| **analysis** | 性能分析 | PortfolioAnalyzer, Tearsheet |
| **adapters** | 交易所连接 | Binance, Bybit, IB 等适配器 |

---

## 三、数据获取和处理

### 3.1 数据流架构

```
数据源 (CSV/数据库/交易所API)
    ↓
DataClient (统一接口)
    ↓
DataEngine (数据引擎)
    ├─ BarAggregator (K线聚合器)
    ├─ OrderBookEngine (订单簿管理)
    └─ DataCatalog (数据持久化)
    ↓
Strategy (策略接收数据)
```

**核心组件**：

1. **DataClient**：统一的数据客户端接口
   - `LiveDataClient`：实盘数据（WebSocket/REST API）
   - `BacktestDataClient`：回测数据（文件/数据库）

2. **DataEngine**：数据处理引擎
   - 管理数据订阅
   - 聚合原始数据（Tick → Bar）
   - 分发数据到策略

3. **DataCatalog**：数据持久化
   - Parquet 格式存储
   - 支持时间范围查询
   - 批量加载和增量更新

### 3.2 支持的数据类型

#### 市场数据

| 类型 | 说明 | 用途 |
|------|------|------|
| **Bar (K线)** | OHLCV 数据 | 趋势分析、技术指标 |
| **QuoteTick** | 买卖报价 | 高频策略、做市 |
| **TradeTick** | 成交记录 | 成交量分析 |
| **OrderBook** | 订单簿深度 | 订单簿策略 |
| **OrderBookDeltas** | 订单簿增量 | 高频订单簿分析 |

#### K线聚合方式

NautilusTrader 支持 **6 种** K线聚合方式：

1. **时间聚合**：固定时间间隔（1分钟、5分钟、1小时等）
2. **Tick 聚合**：固定 Tick 数量（如每 100 个 Tick）
3. **成交量聚合**：固定成交量（如每 1000 单位）
4. **金额聚合**：固定交易金额（如每 10000 USD）
5. **Renko 聚合**：固定价格变化（如每 0.0050 点）
6. **Range 聚合**：固定价格范围

### 3.3 数据管理实战

#### 3.3.1 使用 DataCatalog

```python
from nautilus_trader.persistence.catalog.parquet import ParquetDataCatalog

# 1. 创建数据目录
catalog = ParquetDataCatalog("./data_catalog")

# 2. 写入数据
bars: list[Bar] = [...]
catalog.write_data(bars)
catalog.write_data([instrument])  # 写入合约定义

# 3. 查询数据
filtered_bars = catalog.bars(
    bar_types=["EUR/USD-1-MINUTE-LAST-EXTERNAL"],
    start="2024-01-01",
    end="2024-01-31",
)

# 4. 在回测中使用
engine.add_data(filtered_bars)
```

#### 3.3.2 从 CSV 加载数据

```python
import pandas as pd
from nautilus_trader.persistence.wranglers import BarDataWrangler

# 1. 读取 CSV
df = pd.read_csv("eurusd_1min.csv")
df = df.reindex(columns=["timestamp", "open", "high", "low", "close", "volume"])
df["timestamp"] = pd.to_datetime(df["timestamp"])
df = df.set_index("timestamp")

# 2. 转换为 Bar 对象
bar_type = BarType.from_str("EUR/USD-1-MINUTE-LAST-EXTERNAL")
wrangler = BarDataWrangler(bar_type, instrument)
bars: list[Bar] = wrangler.process(df)

# 3. 添加到回测引擎
engine.add_data(bars)
```

#### 3.3.3 自定义聚合器

```python
from nautilus_trader.data.aggregation import (
    TimeBarAggregator,
    TickBarAggregator,
    VolumeBarAggregator,
)

# 时间聚合（5分钟）
time_aggregator = TimeBarAggregator(
    instrument=instrument,
    bar_type=BarType.from_str("EUR/USD-5-MINUTE-LAST-INTERNAL"),
    handler=strategy.on_bar,
)

# Tick 聚合（每100个tick）
tick_aggregator = TickBarAggregator(
    instrument=instrument,
    bar_type=BarType.from_str("EUR/USD-100-TICK-LAST-INTERNAL"),
    handler=strategy.on_bar,
)

# 成交量聚合（每1000单位）
volume_aggregator = VolumeBarAggregator(
    instrument=instrument,
    bar_type=BarType.from_str("EUR/USD-1000-VOLUME-LAST-INTERNAL"),
    handler=strategy.on_bar,
)
```

### 3.4 数据配置选项

```python
from nautilus_trader.data.config import DataEngineConfig

config = DataEngineConfig(
    time_bars_interval_type="left-open",      # 间隔类型
    time_bars_timestamp_on_close=True,        # 在K线闭合时打时间戳
    time_bars_build_with_no_updates=True,     # 即使无更新也构建K线
    validate_data_sequence=True,              # 验证时间序列
    buffer_deltas=True,                       # 缓冲订单簿变化
    debug=False,                              # 调试模式
)
```

### 3.5 回测 vs 实盘数据流差异

| 特性 | 回测 | 实盘 |
|------|------|------|
| **数据来源** | CSV/数据库文件 | 交易所 API/WebSocket |
| **时间控制** | TestClock（可加速） | 实时时钟 |
| **I/O 模式** | 同步读取 | 异步网络请求 |
| **数据流量** | 预加载 | 流式推送 |
| **延迟** | 无（模拟） | 真实网络延迟 |
| **错误处理** | 数据验证 | 异常恢复、重试 |

---

## 四、量化交易策略开发

### 4.1 策略基类架构

```
Component (基础组件)
    └─ Actor (通用执行者)
        └─ Strategy (交易策略)
```

### 4.2 策略生命周期

```python
from nautilus_trader.trading.strategy import Strategy
from nautilus_trader.indicators.average.ema import ExponentialMovingAverage

class MyStrategy(Strategy):
    """完整的策略生命周期示例"""

    def __init__(self, config):
        """1. 初始化：创建指标对象"""
        super().__init__(config)

        # 创建技术指标
        self.fast_ema = ExponentialMovingAverage(10)
        self.slow_ema = ExponentialMovingAverage(20)

        # 初始化状态变量
        self.position_id = None

    def on_start(self):
        """2. 启动：订阅数据，注册指标"""
        # 注册指标（自动更新）
        self.register_indicator_for_bars(self.bar_type, self.fast_ema)
        self.register_indicator_for_bars(self.bar_type, self.slow_ema)

        # 订阅数据
        self.subscribe_bars(self.bar_type)

        self.log.info("策略已启动")

    def on_bar(self, bar: Bar):
        """3. 运行：接收数据，执行交易逻辑"""
        # 检查指标是否初始化完成
        if not self.indicators_initialized():
            self.log.info("等待指标预热...")
            return

        # 择时逻辑
        if self.fast_ema.value > self.slow_ema.value:
            # 多头信号
            if self.portfolio.is_flat(self.instrument_id):
                self.buy()
            elif self.portfolio.is_net_short(self.instrument_id):
                self.close_all_positions(self.instrument_id)
                self.buy()
        else:
            # 空头信号
            if self.portfolio.is_flat(self.instrument_id):
                self.sell()
            elif self.portfolio.is_net_long(self.instrument_id):
                self.close_all_positions(self.instrument_id)
                self.sell()

    def buy(self):
        """提交买入订单"""
        order = self.order_factory.market(
            instrument_id=self.instrument_id,
            order_side=OrderSide.BUY,
            quantity=self.instrument.make_qty(0.1),
        )
        self.submit_order(order)

    def sell(self):
        """提交卖出订单"""
        order = self.order_factory.market(
            instrument_id=self.instrument_id,
            order_side=OrderSide.SELL,
            quantity=self.instrument.make_qty(0.1),
        )
        self.submit_order(order)

    def on_stop(self):
        """4. 停止：清理资源，平仓"""
        self.cancel_all_orders(self.instrument_id)
        self.close_all_positions(self.instrument_id)
        self.log.info("策略已停止")

    def on_reset(self):
        """5. 重置：重置指标状态"""
        self.fast_ema.reset()
        self.slow_ema.reset()
        self.position_id = None
```

### 4.3 策略核心回调方法

#### 4.3.1 生命周期回调

- `on_start()` - 策略启动时调用
- `on_stop()` - 策略停止时调用
- `on_resume()` - 策略恢复时调用
- `on_reset()` - 策略重置时调用

#### 4.3.2 数据回调

- `on_bar(bar)` - K线数据回调
- `on_quote_tick(tick)` - 报价数据回调
- `on_trade_tick(tick)` - 成交数据回调
- `on_order_book(order_book)` - 订单簿更新
- `on_order_book_deltas(deltas)` - 订单簿增量更新

#### 4.3.3 订单状态回调

- `on_order_initialized(event)` - 订单创建
- `on_order_submitted(event)` - 订单已提交
- `on_order_accepted(event)` - 订单被接受
- `on_order_rejected(event)` - 订单被拒绝
- `on_order_filled(event)` - 订单成交
- `on_order_canceled(event)` - 订单取消

#### 4.3.4 持仓回调

- `on_position_opened(event)` - 新仓位开仓
- `on_position_changed(event)` - 仓位变化
- `on_position_closed(event)` - 仓位平仓

### 4.4 技术指标使用

#### 4.4.1 内置指标（50+ 个）

**趋势指标**：
- SMA (Simple Moving Average)
- EMA (Exponential Moving Average)
- DEMA (Double Exponential MA)
- HMA (Hull Moving Average)
- WMA (Weighted Moving Average)

**动量指标**：
- RSI (Relative Strength Index)
- MACD (Moving Average Convergence Divergence)
- CCI (Commodity Channel Index)
- CMO (Chande Momentum Oscillator)

**波动率指标**：
- ATR (Average True Range)
- Bollinger Bands
- Keltner Channel

**成交量指标**：
- OBV (On Balance Volume)
- KVO (Klinger Volume Oscillator)

#### 4.4.2 指标使用最佳实践

```python
class IndicatorStrategy(Strategy):
    def __init__(self, config):
        super().__init__(config)

        # 创建多个指标
        self.ema_fast = ExponentialMovingAverage(10)
        self.ema_slow = ExponentialMovingAverage(20)
        self.rsi = RelativeStrengthIndex(14)
        self.atr = AverageTrueRange(20)

    def on_start(self):
        # 方式1：为K线数据注册指标（推荐）
        self.register_indicator_for_bars(self.bar_type, self.ema_fast)
        self.register_indicator_for_bars(self.bar_type, self.ema_slow)
        self.register_indicator_for_bars(self.bar_type, self.rsi)
        self.register_indicator_for_bars(self.bar_type, self.atr)

        # 订阅数据
        self.subscribe_bars(self.bar_type)

    def on_bar(self, bar: Bar):
        # 关键：检查指标初始化状态
        if not self.indicators_initialized():
            return

        # 现在可以安全使用指标值
        if (self.ema_fast.value > self.ema_slow.value and
            self.rsi.value < 70):
            self.buy()
```

### 4.5 选股策略（如何选择交易品种）

#### 4.5.1 选股维度

```python
# 1. 技术面选股
def screen_by_technical(self):
    """基于技术指标选股"""
    selected = []
    for instrument_id in self.all_instruments:
        rsi = self.get_indicator(instrument_id, 'rsi')
        if rsi.value < 30:  # 超卖
            selected.append(instrument_id)
    return selected

# 2. 成交量选股
def screen_by_volume(self):
    """基于成交量选股"""
    for instrument_id in self.all_instruments:
        volume = self.cache.trade_tick_count(instrument_id)
        if volume > MIN_VOLUME_THRESHOLD:
            yield instrument_id

# 3. 波动率选股
def screen_by_volatility(self):
    """基于波动率选股"""
    for instrument_id in self.all_instruments:
        atr = self.get_indicator(instrument_id, 'atr')
        if atr.value > VOLATILITY_THRESHOLD:
            yield instrument_id
```

### 4.6 择时策略（何时买入/卖出）

#### 4.6.1 择时方法

```python
# 方法1：趋势跟踪（EMA交叉）
if self.fast_ema.value > self.slow_ema.value:
    signal = "BUY"
elif self.fast_ema.value < self.slow_ema.value:
    signal = "SELL"

# 方法2：均值回归（RSI）
if self.rsi.value < 30:
    signal = "BUY"  # 超卖
elif self.rsi.value > 70:
    signal = "SELL"  # 超买

# 方法3：订单簿不平衡
book = self.cache.order_book(self.instrument_id)
bid_qty = book.best_bid_quantity()
ask_qty = book.best_ask_quantity()
if bid_qty / ask_qty > 1.5:
    signal = "BUY"

# 方法4：多因子综合
score = 0
if self.fast_ema.value > self.slow_ema.value:
    score += 1
if self.rsi.value > 50:
    score += 1
if self.obv.value > self.obv_sma.value:
    score += 1
if score >= 2:
    signal = "BUY"
```

### 4.7 风险管理

#### 4.7.1 动态仓位计算

```python
def calculate_position_size(self, account_balance, risk_per_trade,
                           entry_price, stop_loss_price):
    """Kelly公式或固定风险计算"""
    # 风险金额
    risk_amount = account_balance * risk_per_trade  # 如 2%

    # 价格差异
    price_difference = abs(entry_price - stop_loss_price)

    if price_difference == 0:
        return 0

    # 仓位大小
    position_size = risk_amount / price_difference

    # 限制最大仓位
    max_position = account_balance / (entry_price * MAX_LEVERAGE)

    return min(position_size, max_position)
```

#### 4.7.2 括号单（自动止损止盈）

```python
from nautilus_trader.model.orders import OrderType

# 创建括号单
bracket_order = self.order_factory.bracket(
    instrument_id=self.instrument_id,
    order_side=OrderSide.BUY,
    quantity=self.instrument.make_qty(0.1),
    entry_trigger_price=self.instrument.make_price(1900.00),
    entry_order_type=OrderType.LIMIT_IF_TOUCHED,
    sl_trigger_price=self.instrument.make_price(1800.00),  # 止损
    tp_price=self.instrument.make_price(2000.00),          # 止盈
)

self.submit_order_list(bracket_order)
```

#### 4.7.3 尾随止损（保护利润）

```python
def on_position_opened(self, event: PositionOpened):
    """仓位开仓时，立即提交尾随止损单"""
    atr_stop_distance = 2.0 * self.atr.value

    trailing_stop = self.order_factory.trailing_stop_market(
        instrument_id=self.instrument_id,
        order_side=Order.closing_side(event.side),
        quantity=event.quantity,
        trailing_offset=atr_stop_distance,
        trailing_offset_type=TrailingOffsetType.POINTS,
    )

    self.submit_order(trailing_stop)
```

#### 4.7.4 组合风险控制

```python
def check_portfolio_risk(self):
    """组合级别的风险控制"""
    # 1. 最多同时持有N个仓位
    open_positions = self.cache.positions_open(strategy_id=self.id)
    if len(open_positions) >= MAX_POSITIONS:
        return False

    # 2. 总风险敞口不超过账户的X%
    total_notional = sum(
        pos.quantity * pos.entry_price
        for pos in open_positions
    )
    account_balance = self.portfolio.account_balance()
    if total_notional > account_balance * 0.1:
        return False

    # 3. 日度最大亏损
    daily_pnl = self.calculate_daily_pnl()
    if daily_pnl < -MAX_DAILY_LOSS:
        self.close_all_positions()
        self.stop()
        return False

    return True
```

### 4.8 订单管理

#### 4.8.1 订单类型

```python
# 1. 市价单
market_order = self.order_factory.market(
    instrument_id=instrument_id,
    order_side=OrderSide.BUY,
    quantity=Quantity(1.0),
)

# 2. 限价单
limit_order = self.order_factory.limit(
    instrument_id=instrument_id,
    order_side=OrderSide.BUY,
    price=Price(1900.00),
    quantity=Quantity(1.0),
)

# 3. 止损单
stop_order = self.order_factory.stop_market(
    instrument_id=instrument_id,
    order_side=OrderSide.SELL,
    quantity=Quantity(1.0),
    trigger_price=Price(1800.00),
)

# 4. 限价止损单
stop_limit_order = self.order_factory.stop_limit(
    instrument_id=instrument_id,
    order_side=OrderSide.SELL,
    price=Price(1799.00),
    quantity=Quantity(1.0),
    trigger_price=Price(1800.00),
)
```

#### 4.8.2 订单操作

```python
# 提交订单
self.submit_order(order)

# 修改订单
self.modify_order(order, price=new_price)

# 取消订单
self.cancel_order(order)

# 批量取消所有订单
self.cancel_all_orders(instrument_id)

# 平仓
self.close_position(position)
self.close_all_positions(instrument_id)
```

### 4.9 实际策略示例

#### 示例1：EMA交叉策略

**位置**：`examples/strategies/ema_cross.py`

**策略逻辑**：
- 快EMA(10) > 慢EMA(20)：做多
- 快EMA(10) < 慢EMA(20)：做空

#### 示例2：EMA交叉+括号单

**位置**：`examples/strategies/ema_cross_bracket.py`

**创新点**：使用ATR动态计算止损止盈距离

#### 示例3：订单簿不平衡策略

**位置**：`examples/strategies/orderbook_imbalance.py`

**策略逻辑**：
- bid盘深度 > ask盘深度 × 1.5 → 买入
- ask盘深度 > bid盘深度 × 1.5 → 卖出

#### 示例4：做市商策略

**位置**：`examples/strategies/market_maker.py`

**策略逻辑**：在买卖两侧维护报价，赚取价差

---

## 五、可视化和性能分析

### 5.1 Tearsheet 系统

**Tearsheet** 是 NautilusTrader 的一键式回测报告生成系统，能够创建专业的交互式 HTML 报告。

#### 5.1.1 快速开始

```python
from nautilus_trader.analysis.tearsheet import create_tearsheet

# 运行回测
engine.run()

# 生成默认 tearsheet
create_tearsheet(
    engine=engine,
    output_path="backtest_results.html",
)
```

#### 5.1.2 自定义配置

```python
from nautilus_trader.analysis import TearsheetConfig

config = TearsheetConfig(
    charts=["run_info", "stats_table", "equity", "drawdown", "monthly_returns"],
    theme="nautilus_dark",  # 主题：plotly_white, plotly_dark, nautilus, nautilus_dark
    title="Q4 2024 策略性能",
    height=1800,
    include_benchmark=True,
    benchmark_name="SPY",
)

create_tearsheet(
    engine=engine,
    output_path="custom_tearsheet.html",
    config=config,
    benchmark_returns=benchmark_series,  # pd.Series
)
```

### 5.2 内置图表类型（8种）

| 图表名称 | 类型 | 描述 |
|---------|------|------|
| `run_info` | Table | 运行元数据和账户余额 |
| `stats_table` | Table | 性能统计（PnL、收益、常规） |
| `equity` | Line | 累计权益曲线（支持基准对比） |
| `drawdown` | Area | 回撤百分比 |
| `monthly_returns` | Heatmap | 按年月的收益百分比 |
| `distribution` | Histogram | 单日收益分布 |
| `rolling_sharpe` | Line | 60日滚动夏普比率 |
| `yearly_returns` | Bar | 年度收益百分比 |
| `bars_with_fills` | Candlestick | K线图 + 订单成交标记 |

### 5.3 性能指标

#### 5.3.1 PnL 统计

- `PnL (total)` - 总损益
- `PnL% (total)` - 损益百分比
- `Profit Factor` - 利润因子（总盈利/总亏损）
- `Win Rate` - 胜率
- `Avg Winner` - 平均盈利
- `Avg Loser` - 平均亏损

#### 5.3.2 收益统计

- `Sharpe Ratio (252 days)` - 夏普比率
- `Sortino Ratio` - 索提诺比率
- `CAGR` - 年化复合增长率
- `Max Drawdown` - 最大回撤
- `Volatility` - 波动率
- `Calmar Ratio` - 卡玛比率

#### 5.3.3 常规统计

- `Total Trades` - 总交易数
- `Avg Trade Duration` - 平均交易时间
- `Long Ratio` - 做多比例
- `Expectancy` - 期望值

### 5.4 报告生成

```python
# 从 Trader 对象生成报告
orders_report = engine.trader.generate_orders_report()
fills_report = engine.trader.generate_fills_report()
positions_report = engine.trader.generate_positions_report()
account_report = engine.trader.generate_account_report(venue)

# 保存为 CSV
positions_report.to_csv("positions.csv")
fills_report.to_csv("fills.csv")

# 显示报告
import pandas as pd
with pd.option_context("display.max_rows", 100):
    print(positions_report)
```

### 5.5 完整分析流程

```python
# 1. 运行回测
engine.run()

# 2. 获取统计数据
analyzer = engine.portfolio.analyzer
stats_pnls = analyzer.get_performance_stats_pnls()
stats_returns = analyzer.get_performance_stats_returns()
stats_general = analyzer.get_performance_stats_general()

# 3. 生成可视化
create_tearsheet(
    engine=engine,
    output_path="full_analysis.html",
)

# 4. 打印关键指标
print(f"总 PnL: {stats_pnls.get('PnL (total)'):.2f}")
print(f"夏普比率: {stats_returns.get('Sharpe Ratio (252 days)'):.2f}")
print(f"最大回撤: {stats_returns.get('Max Drawdown'):.2f}%")
```

---

## 六、快速上手工作流

### 6.1 安装

```bash
# 1. 安装核心包
pip install nautilus_trader

# 2. 安装可视化支持
pip install "nautilus_trader[visualization]"
# 或
pip install plotly>=6.3.1

# 3. 安装特定交易所适配器（可选）
pip install "nautilus_trader[ib]"      # Interactive Brokers
pip install "nautilus_trader[dydx]"    # dYdX
```

### 6.2 第一个回测（5步）

```python
#!/usr/bin/env python3
from decimal import Decimal
from nautilus_trader.backtest.engine import BacktestEngine
from nautilus_trader.backtest.config import BacktestEngineConfig
from nautilus_trader.config import LoggingConfig
from nautilus_trader.model.currencies import USD
from nautilus_trader.model.enums import AccountType, OmsType
from nautilus_trader.model.identifiers import TraderId, Venue
from nautilus_trader.model.objects import Money
from nautilus_trader.examples.strategies.ema_cross import EMACross
from nautilus_trader.test_kit.providers import TestInstrumentProvider

# 1️⃣ 创建回测引擎
config = BacktestEngineConfig(
    trader_id=TraderId("BACKTESTER-001"),
    logging=LoggingConfig(log_level="INFO"),
)
engine = BacktestEngine(config=config)

# 2️⃣ 配置交易所和初始资金
engine.add_venue(
    venue=Venue("BINANCE"),
    oms_type=OmsType.NETTING,
    account_type=AccountType.CASH,
    starting_balances=[Money(10_000, USD)],
)

# 3️⃣ 添加交易品种和数据
ETHUSDT = TestInstrumentProvider.ethusdt_binance()
engine.add_instrument(ETHUSDT)
# bars = load_data(...)  # 加载历史数据
# engine.add_data(bars)

# 4️⃣ 添加策略
strategy_config = {
    "instrument_id": ETHUSDT.id,
    "bar_type": "ETHUSDT.BINANCE-5-MINUTE-LAST-INTERNAL",
    "trade_size": Decimal("0.1"),
    "fast_ema_period": 10,
    "slow_ema_period": 20,
}
strategy = EMACross(config=strategy_config)
engine.add_strategy(strategy)

# 5️⃣ 运行回测和生成报告
engine.run()

from nautilus_trader.analysis.tearsheet import create_tearsheet
create_tearsheet(engine, output_path="results.html")

engine.dispose()
```

### 6.3 从回测到实盘

**关键差异**：只需要改变引擎类型，策略代码完全相同！

```python
# 回测
from nautilus_trader.backtest.engine import BacktestEngine
engine = BacktestEngine(config=backtest_config)

# 实盘（只需改这一行）
from nautilus_trader.live.node import TradingNode
node = TradingNode(config=live_config)
```

### 6.4 完整回测示例

参考文件：`examples/backtest/crypto_ema_cross_ethusdt_trade_ticks.py`

---

## 七、学习路径建议

### 第一阶段：基础入门（1-2周）

**目标**：了解平台架构和核心概念

**学习内容**：
1. 阅读 `docs/getting_started/` 目录
   - `installation.md` - 安装指南
   - `quickstart.md` - 快速开始
   - `backtest_high_level.md` - 回测入门

2. 运行基础示例
   - `examples/backtest/example_01_load_bars_from_custom_csv/` - 数据加载
   - `examples/backtest/example_04_using_data_catalog/` - 数据管理
   - `examples/backtest/example_07_using_indicators/` - 指标使用

3. 理解核心概念
   - `docs/concepts/architecture.md` - 架构设计
   - `docs/concepts/backtesting.md` - 回测原理
   - `docs/concepts/strategies.md` - 策略开发

**练习任务**：
- [ ] 成功运行一个回测示例
- [ ] 从CSV加载自己的数据
- [ ] 使用 DataCatalog 管理数据
- [ ] 生成第一个 Tearsheet 报告

### 第二阶段：策略开发（2-4周）

**目标**：开发和回测自己的交易策略

**学习内容**：
1. 学习内置策略
   - `examples/strategies/ema_cross.py` - 趋势跟踪
   - `examples/strategies/ema_cross_bracket.py` - 风险管理
   - `examples/strategies/orderbook_imbalance.py` - 高频策略

2. 技术指标深入
   - 阅读 `docs/api_reference/indicators.md`
   - 学习常用指标：EMA, RSI, MACD, ATR
   - 理解指标组合使用

3. 开发第一个策略
   - 选择1-2个技术指标
   - 实现简单的择时逻辑
   - 添加基础风险管理

**练习任务**：
- [ ] 实现一个双均线交叉策略
- [ ] 添加 RSI 过滤条件
- [ ] 实现动态止损（基于ATR）
- [ ] 回测并分析性能指标
- [ ] 优化策略参数

### 第三阶段：高级功能（4-8周）

**目标**：掌握高级功能和最佳实践

**学习内容**：
1. 多品种策略
   - 同时监控多个交易品种
   - 资金分配和风险管理
   - 相关性分析

2. 高级订单类型
   - 括号单（Bracket Orders）
   - 尾随止损（Trailing Stop）
   - 条件单（Contingent Orders）

3. 订单簿策略
   - L2 订单簿数据处理
   - 订单簿不平衡检测
   - 做市商策略

4. 性能优化
   - 数据预处理和缓存
   - 指标计算优化
   - 回测加速技巧

**练习任务**：
- [ ] 开发多品种轮动策略
- [ ] 实现完整的风险管理系统
- [ ] 尝试订单簿策略
- [ ] 进行参数优化和前向验证
- [ ] 创建策略组合（多策略）

### 第四阶段：实盘准备（持续）

**目标**：为实盘交易做准备

**学习内容**：
1. 交易所集成
   - 阅读 `docs/integrations/` 相关文档
   - 配置 API 密钥和权限
   - 理解交易所限制和费用

2. 纸面交易（Paper Trading）
   - 使用测试网络
   - 验证策略在实时环境的表现
   - 监控和调试

3. 风险管理升级
   - 实盘风控规则
   - 异常处理和恢复
   - 监控和告警

4. 实盘部署
   - 小资金测试
   - 持续监控
   - 定期回顾和调整

**练习任务**：
- [ ] 配置交易所连接（如 Binance）
- [ ] 在测试网进行纸面交易
- [ ] 建立监控和告警系统
- [ ] 小资金实盘测试
- [ ] 建立交易日志和复盘流程

---

## 八、重要资源

### 8.1 核心文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 快速开始 | `docs/getting_started/quickstart.md` | 15分钟入门 |
| 架构设计 | `docs/concepts/architecture.md` | 系统架构 |
| 策略开发 | `docs/concepts/strategies.md` | 策略完整指南 |
| 数据管理 | `docs/concepts/data.md` | 数据处理详解 |
| 回测指南 | `docs/concepts/backtesting.md` | 回测系统 |
| 可视化 | `docs/concepts/visualization.md` | Tearsheet 文档 |
| API 参考 | `docs/api_reference/` | 完整 API 文档 |

### 8.2 示例代码

| 类型 | 位置 | 数量 |
|------|------|------|
| 回测示例 | `examples/backtest/` | 11 个 |
| 策略模板 | `examples/strategies/` | 10+ 个 |
| 实盘示例 | `examples/live/` | 14 个交易所 |
| Jupyter 教程 | `examples/backtest/notebooks/` | 多个 |

### 8.3 关键文件路径

```
# Python 核心模块
nautilus_trader/trading/strategy.pyx          # 策略基类
nautilus_trader/data/engine.pyx               # 数据引擎
nautilus_trader/backtest/engine.pyx           # 回测引擎
nautilus_trader/indicators/                   # 指标库
nautilus_trader/analysis/tearsheet.py         # 可视化系统

# 配置
nautilus_trader/config/                       # 配置类

# 示例策略
examples/strategies/ema_cross.py              # EMA交叉
examples/strategies/orderbook_imbalance.py    # 订单簿策略
examples/strategies/market_maker.py           # 做市商

# 文档
docs/getting_started/                         # 入门指南
docs/concepts/                                # 核心概念
docs/integrations/                            # 交易所集成
```

### 8.4 在线资源

- **官方网站**：https://nautilustrader.io
- **GitHub**：https://github.com/nautechsystems/nautilus_trader
- **文档**：https://nautilustrader.io/docs/
- **Discord 社区**：https://discord.gg/nautilustrader
- **Twitter**：@nautilus_trader

---

## 九、实用技巧

### 9.1 开发技巧

#### 1. 从简单开始
```python
# ✅ 好的做法：从单品种、单指标开始
class SimpleEMAStrategy(Strategy):
    def __init__(self, config):
        self.ema = ExponentialMovingAverage(20)

    def on_bar(self, bar):
        if self.ema.value > bar.close:
            self.sell()
        else:
            self.buy()

# ❌ 避免：一开始就写复杂策略
class ComplexMultiFactorStrategy(Strategy):
    # 10+ 个指标，多个品种，复杂逻辑...
```

#### 2. 充分回测
- 至少使用 **1 年** 的历史数据
- 包含不同市场环境（牛市、熊市、震荡市）
- 进行 **样本外验证**（out-of-sample testing）

#### 3. 参数优化
```python
# 网格搜索示例
for fast_period in range(5, 20, 5):
    for slow_period in range(20, 50, 10):
        config = {
            'fast_ema_period': fast_period,
            'slow_ema_period': slow_period,
        }
        # 运行回测...
        # 记录结果...
```

#### 4. 使用日志
```python
class MyStrategy(Strategy):
    def on_bar(self, bar):
        self.log.info(f"收到K线: {bar}")
        self.log.debug(f"EMA值: {self.ema.value}")

        if condition:
            self.log.warning("风险警告：...")
```

### 9.2 性能优化

#### 1. 数据管理
```python
# ✅ 使用 Parquet 数据目录
catalog = ParquetDataCatalog("./data_catalog")
bars = catalog.bars(start="2024-01-01", end="2024-12-31")

# ✅ 批量添加数据
engine.add_data(all_bars)

# ❌ 避免：逐条添加
for bar in bars:
    engine.add_data([bar])  # 慢！
```

#### 2. 选择合适的K线粒度
```python
# 日内策略：5分钟或15分钟
bar_type = "ETHUSDT.BINANCE-5-MINUTE-LAST-INTERNAL"

# 日线策略：1天
bar_type = "ETHUSDT.BINANCE-1-DAY-LAST-INTERNAL"

# 高频策略：考虑使用 Tick 或 OrderBook
```

#### 3. 指标预热
```python
def on_bar(self, bar):
    # 总是检查指标是否初始化完成
    if not self.indicators_initialized():
        return  # 跳过，等待更多数据

    # 现在可以安全使用指标
    ...
```

### 9.3 风险管理最佳实践

#### 1. 单笔风险控制
```python
# 每笔交易风险不超过账户的 1-2%
RISK_PER_TRADE = 0.02  # 2%

position_size = (account_balance * RISK_PER_TRADE) / (entry_price - stop_loss)
```

#### 2. 组合风险控制
```python
# 最多同时持有 3-5 个仓位
MAX_POSITIONS = 3

if len(self.portfolio.positions_open()) >= MAX_POSITIONS:
    return  # 不开新仓
```

#### 3. 日度最大亏损
```python
# 触发日度止损，停止交易
MAX_DAILY_LOSS = 0.05  # 5%

daily_pnl = self.calculate_daily_pnl()
if daily_pnl < -MAX_DAILY_LOSS:
    self.close_all_positions()
    self.stop()
```

#### 4. 使用止损单
```python
# 总是使用止损保护资金
bracket_order = self.order_factory.bracket(
    entry_trigger_price=entry_price,
    sl_trigger_price=stop_loss_price,  # 必须设置
    tp_price=take_profit_price,
)
```

### 9.4 调试技巧

#### 1. 使用回测日志
```python
config = BacktestEngineConfig(
    logging=LoggingConfig(
        log_level="DEBUG",  # DEBUG, INFO, WARNING, ERROR
        log_colors=True,
    ),
)
```

#### 2. 打印关键变量
```python
def on_bar(self, bar):
    print(f"当前价格: {bar.close}")
    print(f"快线EMA: {self.fast_ema.value}")
    print(f"慢线EMA: {self.slow_ema.value}")
    print(f"持仓数量: {len(self.portfolio.positions_open())}")
```

#### 3. 可视化订单和持仓
```python
# 使用 bars_with_fills 图表查看订单成交
from nautilus_trader.analysis.tearsheet import create_bars_with_fills

fig = create_bars_with_fills(
    engine=engine,
    bar_type="ETHUSDT.BINANCE-5-MINUTE-LAST-INTERNAL",
)
fig.show()
```

### 9.5 常见陷阱

#### 1. 过拟合
```python
# ❌ 避免：在相同数据上反复优化参数
# 解决：使用样本外数据验证

# 训练期：2023-01-01 到 2023-06-30
# 验证期：2023-07-01 到 2023-12-31
```

#### 2. 前视偏差（Look-ahead Bias）
```python
# ❌ 错误：使用未来数据
if bar.close > tomorrow_high:  # 不能用明天的数据！
    self.buy()

# ✅ 正确：只使用当前和历史数据
if bar.close > self.yesterday_high:
    self.buy()
```

#### 3. 忽略交易成本
```python
# 在回测配置中设置真实的手续费
venue_config = {
    'commission_maker': 0.0002,  # 0.02%
    'commission_taker': 0.0004,  # 0.04%
}
```

#### 4. 未处理滑点
```python
# 使用限价单而非市价单可以控制滑点
limit_order = self.order_factory.limit(
    price=bar.close * 0.999,  # 限制最高买入价
    ...
)
```

---

## 十、总结

### 10.1 核心要点

✅ **NautilusTrader 是什么**：
- 生产级量化交易平台
- Rust 核心 + Python 接口
- 回测和实盘统一代码库

✅ **主要功能**：
- 完整的数据管理系统
- 事件驱动的回测引擎
- 50+ 内置技术指标
- 14+ 交易所集成
- 专业的性能分析和可视化

✅ **学习路径**：
1. 基础入门（1-2周）
2. 策略开发（2-4周）
3. 高级功能（4-8周）
4. 实盘准备（持续）

✅ **最佳实践**：
- 从简单策略开始
- 充分回测和验证
- 严格的风险管理
- 持续监控和优化

### 10.2 下一步行动

**立即开始**：
1. [ ] 安装 NautilusTrader：`pip install nautilus_trader`
2. [ ] 运行第一个回测示例：`examples/backtest/`
3. [ ] 阅读核心文档：`docs/getting_started/quickstart.md`
4. [ ] 修改示例策略参数，观察结果变化

**本周目标**：
1. [ ] 理解平台架构和核心概念
2. [ ] 成功运行 3 个不同的回测示例
3. [ ] 从 CSV 加载自己的数据
4. [ ] 生成第一个 Tearsheet 报告

**本月目标**：
1. [ ] 开发第一个简单策略（如双均线交叉）
2. [ ] 添加风险管理（止损、仓位控制）
3. [ ] 进行参数优化
4. [ ] 分析性能指标，找出改进方向

### 10.3 获取帮助

如遇到问题，可以：
- 查阅官方文档：https://nautilustrader.io/docs/
- 搜索 GitHub Issues：https://github.com/nautechsystems/nautilus_trader/issues
- 加入 Discord 社区：https://discord.gg/nautilustrader
- 查看示例代码：`examples/` 目录

---

## 附录

### A. 快速参考

#### 常用命令
```bash
# 安装
pip install nautilus_trader
pip install "nautilus_trader[visualization]"

# 运行示例
python examples/backtest/crypto_ema_cross_ethusdt_trade_ticks.py

# 运行测试
pytest tests/unit_tests/
```

#### 常用代码片段

**创建策略**：
```python
class MyStrategy(Strategy):
    def __init__(self, config):
        super().__init__(config)
        self.ema = ExponentialMovingAverage(20)

    def on_start(self):
        self.register_indicator_for_bars(self.bar_type, self.ema)
        self.subscribe_bars(self.bar_type)

    def on_bar(self, bar):
        if not self.indicators_initialized():
            return
        # 交易逻辑...
```

**运行回测**：
```python
engine = BacktestEngine(config=config)
engine.add_venue(venue, starting_balances=[...])
engine.add_instrument(instrument)
engine.add_data(bars)
engine.add_strategy(strategy)
engine.run()
```

**生成报告**：
```python
from nautilus_trader.analysis.tearsheet import create_tearsheet
create_tearsheet(engine, output_path="results.html")
```

### B. 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 回测 | Backtesting | 使用历史数据测试策略 |
| 实盘 | Live Trading | 真实资金交易 |
| K线 | Bar/Candlestick | OHLCV 数据 |
| 择时 | Timing | 何时买入/卖出 |
| 选股 | Selection | 选择交易品种 |
| 止损 | Stop Loss | 限制亏损的订单 |
| 止盈 | Take Profit | 锁定利润的订单 |
| 仓位 | Position | 持有的资产数量 |
| 夏普比率 | Sharpe Ratio | 风险调整后的收益 |
| 最大回撤 | Max Drawdown | 最大资金回撤百分比 |

---

**文档版本**：v1.0
**最后更新**：2025-11-05
**适用于**：NautilusTrader v1.222.0+

祝您在量化交易学习之路上取得成功！🚀
