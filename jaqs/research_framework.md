# JAQS研究框架介绍

完整的量化交易策略包含信号计算、交易指令、仓位管理、资金管理等多个方面，其中**信号研究**是一个重要组成部分。JAQS的`SignalDigger`（信号挖据）模块提供了alpha信号研究、事件驱动信号研究、CTA信号研究等功能，本文主要介绍`SignalDigger`模块。

## 什么是信号

信号是可以指导何时交易、交易多少的量化的指标，以下都可以算作信号：

- 等金额购买A、B两只股票（信号是购买金额为1：1）
- 购买沪深300中PE低于15的股票（若PE<15信号为1，否则为0）
- 按市值权重购买A、B两只股票（信号是购买金额之比等于市值之比）
- 9:37:45开仓天然橡胶期货3手（信号是购买3手）
- 按等波动性贡献的标准，配置股票指数和债券指数（信号是根据风险中性标准优化得到的权重）

## 为什么需要信号

信号代表了模型对标的的预期，并给出了定量的买卖目标。好的模型能给出有效的信号，进而转化为模型在相关标的上的主动收益。

## 常用的信号评估方法

我们问“信号是否有效”，就是想问“在信号满足某些条件下的条件分布，和无条件分布是否有区别”。一般有：

- 收益分析：收益的分布、均值、标准差，累计收益等；
- 相关性分析：信号值与收益的相关系数、秩相关系数等

## JAQS的信号分析模块

### 股票alpha信号分析（多标的）

对于股票alpha信号，我们主要关注**横截面数据**，即每个交易日投资范围内所有股票的数据。对任意一个交易日，各个股票的信号和各个股票的收益（可以是n天的收益）构成一组数据点，可以分多组比较平均收益，也可以计算相关系数。下面为示例代码（完整版本参见[这里](https://github.com/quantOS-org/JAQS/blob/master/example/research/signal_return_ic_analysis.py)）：

```python
from jaqs.research import SignalDigger

# 实例化SignalDigger对象，用于分析、存储结果
# output_format为输出格式，也可选择直接绘图/输出base64编码等
# output_folder为输出文件夹路径
digger = SignalDigger(output_format='pdf',
                   output_folder='../../output/test_signal')

# 在此处传入待分析的数据：
# signal是index为交易日、columns为股票、value为信号值的DataFrame
# price是index为交易日，columns为股票，value为价格的DataFrame
# benchmark_price是业绩基准的价格序列（单column的DataFrame）
# n_quantiles是对信号分几组，period是考虑多久后的收益
digger.process_signal_before_analysis(signal, price=price,
                                      mask=mask_all,
                                      n_quantiles=5, period=my_period,
                                      benchmark_price=price_bench,
                                     )
# 利用处理好的数据直接生成分析报告
res = digger.create_full_report()
```



### CTA信号分析（单标的）

对于单标的，信号组成的时间序列数据和标的价格组成的时间序列数据共同构成一组数据点，用于分析。我们同样根据信号值大小进行分组，比较每一组在信号出现后n天的收益均值、标准差；也可以计算信号与收益的相关系数。

```python
from jaqs.research import SignalDigger

# 实例化SignalDigger对象，用于分析、存储结果
# output_format为输出格式，也可选择直接绘图/输出base64编码等
# output_folder为输出文件夹路径
digger = SignalDigger(output_format='pdf',
                      output_folder='../../output')

# 在此处传入待分析的数据：
# signal是index为交易日、columns为股票、value为信号值的DataFrame
# price是index为交易日，columns为股票，value为价格的DataFrame
# n_quantiles是对信号分几组，periods是考虑多久后的收益
# trade_condition用于进行简单的回测：
# column列的值满足filter条件后按direction方向多/空hold天数，自动绘制累计收益
digger.create_single_signal_report(signal, price, 
                                   periods=[1, 5, 9, 21], n_quantiles=6,
                                   mask=None,
                                   trade_condition={'cond1': {'column': 'quantile',
                                                              'filter': lambda x: x > 3,
                                                              'hold': 5,
                                                              'direction': 1},
                                                    'cond2': {'column': 'quantile',
                                                              'filter': lambda x: x > 5,
                                                              'hold': 5,
                                                              'direction': 1},
                                                    'cond3': {'column': 'quantile',
                                                              'filter': lambda x: x > 5,
                                                              'hold': 9,
                                                              'direction': -1},
                                                    }
                                  )
```



### 事件驱动信号分析（多标的）

```python
obj = SignalDigger(output_format='pdf',
                   output_folder='../../output')

obj.create_binary_event_report(signal, price, 
                               mask_all, 
                               price_bench, 
                               periods=[20, 60, 121, 242], group_by=None)


```

