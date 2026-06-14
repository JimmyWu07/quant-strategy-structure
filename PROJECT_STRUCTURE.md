# 聚宽(JoinQuant)量化策略 - 项目结构管理指南

> 一套系统化的聚宽策略项目管理方案，帮助你从单文件脚本走向工程化管理。

---

## 📁 推荐项目结构

```
quant-platform/
├── README.md                          # 项目说明
├── .gitignore                         # Git忽略规则
├── requirements.txt                   # 本地回测依赖(非聚宽平台)
│
├── strategies/                        # 📌 策略目录(每个策略一个文件)
│   ├── __init__.py
│   ├── double_ma_adx.py               # 双均线+ADX趋势
│   ├── mean_reversion.py              # 均值回归
│   └── momentum_breakout.py           # 动量突破
│
├── config/                            # ⚙️ 配置目录
│   ├── __init__.py
│   ├── strategy_params.py             # 策略参数配置
│   ├── universe.py                    # 股票池配置
│   └── risk_control.py                # 风控参数
│
├── core/                              # 🧠 核心模块
│   ├── __init__.py
│   ├── indicators.py                  # 自定义技术指标(TA库封装)
│   ├── risk_manager.py                # 风控逻辑(仓位/止损/回撤)
│   ├── order_manager.py               # 订单管理(分批/滑点控制)
│   └── data_utils.py                  # 数据处理工具
│
├── research/                          # 🔬 研究分析
│   ├── factor_analysis.ipynb          # 因子分析notebook
│   ├── backtest_report.ipynb          # 回测报告分析
│   └── parameter_optimization.py      # 参数寻优
│
├── tests/                             # 🧪 测试目录
│   ├── __init__.py
│   ├── test_indicators.py             # 指标计算测试
│   └── test_risk_manager.py           # 风控模块测试
│
└── docs/                              # 📖 文档
    ├── strategy_notes.md              # 策略思路笔记
    ├── risk_management.md             # 风控体系说明
    └── changelog.md                   # 变更日志
```

---

## 🗂️ 各目录职责详解

### 1. `strategies/` — 策略核心

每个策略文件是**聚宽可直接运行的完整代码单元**，遵循标准化模板：

```python
"""
策略名称: 双均线+ADX趋势策略
创建日期: 2026-06-15
版本: v1.2
描述: 5/10日双均线金叉+ADX趋势过滤
"""

# ============ 初始化 ============
def initialize(context):
    """聚宽入口: 设置参数、基准、定时任务"""
    g.stock = '300502.XSHE'
    g.short_window = 5
    g.long_window = 10
    g.adx_period = 14
    g.adx_threshold = 10
    g.in_position = False

    set_benchmark('000300.XSHG')
    run_daily(daily_check, time='14:50')

# ============ 主逻辑 ============
def daily_check(context):
    """每日14:50执行: 计算信号 → 判断 → 下单"""
    # 1. 拉取历史K线(不含当日,避免未来函数)
    # 2. 计算技术指标
    # 3. 判断信号
    # 4. 执行下单
    pass
```

**关键原则:**
- 每个策略文件必须**独立可运行**，不依赖本地模块导入
- 使用注释分隔区块：初始化、数据、信号、下单
- 避免未来函数：`attribute_history()`获取的历史K线**不含当日**

### 2. `config/` — 参数配置

```python
# config/strategy_params.py

STRATEGY_PARAMS = {
    'double_ma_adx': {
        'short_window': 5,
        'long_window': 10,
        'adx_period': 14,
        'adx_threshold': 10,
    },
    'mean_reversion': {
        'lookback': 20,
        'entry_std': 2.0,
        'exit_std': 0.5,
    }
}

# config/universe.py
STOCK_UNIVERSE = {
    'hs300': '000300.XSHG',          # 沪深300
    'zz500': '000905.XSHG',          # 中证500
    'custom': ['300502.XSHE', '000858.XSHE'],  # 自选池
}
```

**为什么需要config?**
- 参数集中管理，修改一行全局生效
- 便于参数寻优（遍历config而非改代码）
- 团队协作时减少冲突

### 3. `core/` — 核心工具库

封装可复用的技术指标和风控逻辑，**本地回测**时调用，**聚宽平台**使用时复制所需函数到策略文件头部。

```python
# core/indicators.py

def calc_adx(high, low, close, period=14):
    """计算ADX趋势强度指标
    本地可用,聚宽平台需复制此函数到策略文件
    """
    up_move = high.diff()
    down_move = -low.diff()
    # ... 完整ADX计算逻辑
    return adx
```

### 4. `research/` — 研究分析

- **Jupyter Notebook** 做探索性分析（因子IC、参数敏感性）
- **参数寻优脚本** 跑网格搜索/贝叶斯优化
- 研究代码**不进入策略文件**，保持策略代码干净

### 5. `tests/` — 测试

```python
# tests/test_indicators.py

def test_adx_trending_market():
    """验证ADX在趋势行情中返回高值"""
    # 构造模拟K线
    # 断言adx > threshold

def test_adx_ranging_market():
    """验证ADX在震荡行情中返回低值"""
```

---

## 🔄 工作流：本地开发 → 聚宽部署

```
┌──────────────────────────┐
│  本地 VSCode / Cursor    │
│  ├─ 编写策略代码          │
│  ├─ 单元测试验证指标      │
│  ├─ 本地回测(可选)        │
│  └─ Git版本管理          │
└──────────┬───────────────┘
           │  完成开发
           ▼
┌──────────────────────────┐
│  聚宽研究环境             │
│  ├─ 粘贴策略代码          │
│  ├─ 运行回测              │
│  ├─ 模拟交易验证          │
│  └─ 参数微调              │
└──────────┬───────────────┘
           │  回测通过
           ▼
┌──────────────────────────┐
│  聚宽模拟交易 / 实盘     │
│  ├─ 创建策略              │
│  ├─ 上传最终代码          │
│  └─ 启动交易              │
└──────────────────────────┘
```

---

## 🎯 多策略管理方案

### 方案A：单仓库多策略（推荐个人开发者）

```
quant-strategies/
├── strategies/
│   ├── s01_double_ma_adx.py      # 编号管理
│   ├── s02_mean_reversion.py
│   └── s03_momentum.py
└── shared/                        # 共享代码(本地使用)
    ├── indicators.py
    └── backtest_utils.py
```

**优点:** 简单直观，一个仓库管理所有策略  
**缺点:** 聚宽平台无法import shared模块，需手动复制

### 方案B：一策略一仓库（推荐团队协作）

```
double-ma-adx-strategy/
├── strategy.py                    # 策略主文件
├── config.py                      # 参数配置
├── research/                      # 该策略的研究资料
└── README.md                      # 该策略的独立文档
```

**优点:** 策略完全独立，可独立版本管理  
**缺点:** 仓库数量多，共享代码需复制多份

---

## 📋 `.gitignore` 模板

```gitignore
# Python
__pycache__/
*.py[cod]
*.so
.Python
*.egg-info/
dist/
build/

# Jupyter
.ipynb_checkpoints/
*.ipynb.meta/

# IDE
.vscode/
.idea/
*.swp
*.swo

# 环境
.env
.venv/
venv/

# 聚宽
joinquant/
*.pkl                # 缓存数据
research/data/       # 本地数据文件

# macOS / Windows
.DS_Store
Thumbs.db
```

---

## 📝 策略文档模板（每个策略的README）

```markdown
# 策略名称

## 基本信息
- **策略类型**: 趋势跟踪 / 均值回归 / 动量
- **标的**: 个股 / ETF / 指数
- **持仓周期**: 1-20天
- **初始资金**: 10万

## 核心逻辑

简述策略的买卖条件

## 参数表

| 参数 | 默认值 | 说明 |
|------|--------|------|
| short_window | 5 | 短期均线周期 |
| long_window | 10 | 长期均线周期 |
| adx_threshold | 10 | ADX趋势阈值 |

## 回测表现

| 指标 | 数值 |
|------|------|
| 年化收益 | XX% |
| 最大回撤 | XX% |
| 夏普比率 | X.X |
| 胜率 | XX% |

## 注意事项

- 可能存在的风险
- 适用的市场环境
```

---

## 🛠️ 常用命令速查

```bash
# 初始化项目结构
mkdir -p strategies config core research tests docs

# 初始化Git仓库
git init
git add .
git commit -m "init: 聚宽量化策略项目初始化"

# 创建新策略
cp strategies/_template.py strategies/s04_new_strategy.py

# 运行本地测试
python -m pytest tests/ -v

# 同步到GitHub
git remote add origin <repo-url>
git push -u origin main
```

---

## ⚠️ 聚宽平台注意事项

| 事项 | 说明 |
|------|------|
| **未来函数** | `attribute_history()` 拉取的历史K线**不含当日**，用`iloc[-1]`取的是昨日数据，天然避免未来函数 |
| **无法import** | 聚宽不支持自定义模块导入，所有依赖函数必须直接写在策略文件里 |
| **g全局变量** | 聚宽用`g`对象存储策略状态（聚宽特供），不是Python全局变量 |
| **定时任务** | `run_daily(func, time='14:50')` 在指定时刻执行，不是按K线频率 |
| **回测频率** | 默认按天回测，如需分钟级需在`initialize()`中设置`run_daily`+分钟数据 |
| **手续费/滑点** | 回测默认包含万2.5手续费+滑点，实盘可能不同，需在`set_order_cost()`中设置 |

---

## 🔗 参考资源

- [聚宽官方文档](https://www.joinquant.com/help/api/help)
- [聚宽API手册](https://www.joinquant.com/help/api/help#name:api)
- [JoinQuant GitHub](https://github.com/JoinQuant)

---

> **维护者:** KKKL  
> **更新日期:** 2026-06-15  
> **版本:** v1.0
