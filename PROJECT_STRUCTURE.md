# 聚宽量化策略 - 项目结构

```
quant-platform/
├── strategies/                  # 策略（每个文件独立可运行）
│   ├── double_ma_adx.py         # 双均线+ADX趋势
│   ├── mean_reversion.py        # 均值回归
│   └── momentum_breakout.py     # 动量突破
│
├── config/                      # 参数配置
│   ├── strategy_params.py       # 策略参数
│   ├── universe.py              # 股票池
│   └── risk_control.py          # 风控参数
│
├── core/                        # 本地回测工具（聚宽平台不可import）
│   ├── indicators.py            # 技术指标
│   ├── risk_manager.py          # 风控逻辑
│   └── data_utils.py            # 数据处理
│
├── research/                    # 因子分析、参数寻优
│   ├── factor_analysis.ipynb
│   └── parameter_optimization.py
│
├── tests/                       # 本地测试
│   ├── test_indicators.py
│   └── test_risk_manager.py
│
├── README.md
├── .gitignore
└── requirements.txt
```

---

## 聚宽注意事项

| 事项 | 说明 |
|------|------|
| import限制 | 聚宽不支持自定义模块import，所有函数必须写在策略文件里 |
| 未来函数 | `attribute_history()` 不含当日，`iloc[-1]` = 昨日数据 |
| g全局变量 | 用聚宽自带 `g` 对象存策略状态，非Python全局变量 |
| 手续费 | 回测默认万2.5+滑点，实盘可用 `set_order_cost()` 覆盖 |

---

## 多策略管理

- **单仓库多分支**：所有策略放一个仓库，简单直观
- **一策略一仓库**：每个策略独立版本管理，适合多策略迭代
