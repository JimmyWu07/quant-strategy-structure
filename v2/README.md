# v2 — 第二代量化策略

基于 v1 的改进版本，目标：多策略并行、模块化回测框架、因子研究。

## 目录结构

```
v2/
├── strategies/                  # 策略
│   └── __init__.py
├── config/                      # 配置
│   └── __init__.py
├── core/                        # 回测引擎 & 工具
│   └── __init__.py
├── research/                    # 因子分析、参数优化
├── tests/                       # 测试
├── requirements.txt
└── README.md
```

## 环境安装

```bash
pip install -r requirements.txt
```

## 开发计划

- [ ] 回测引擎搭建
- [ ] 因子库建设
- [ ] 多策略框架
- [ ] 风控模块
