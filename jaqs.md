# JAQS策略系统

## JAQS能做什么

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/procedure.png?raw=true)

策略开发的完整流程一般是以下四步的循环往复，JAQS在这四步都提供了支持：

1. **数据**收集处理：我们提供了标准接口`DataApi`, 便利接口`DataService`, 高效工具`DataView`

2. 对数据进行**研究**：我们提供进行信号/事件研究的`SignalDigger`

3. 根据研究结果开发策略并**回测**：我们提供两种策略回测框架，Alpha选股和事件驱动择时（如CTA、套利）

4. 对回测结果进行**分析**：我们提供直观简洁的报告`Report`，以及分析内容丰富、可进一步开发的分析器`Analyzer`

## 数据收集处理的三重境界

### 一重境界：使用标准的DataApi

`DataApi`是用Python实现的的api调用接口，从在线数据平台查询并返回`DataFrame`格式的数据和消息，采用预定义的标准接口格式和数据格式，是最基础的数据查询方式。

以指数成分查询为例：

```py
res, msg = data_api.query(view="lb.indexCons",
                          fields="",
                          filter="index_code=000300.SH&start_date=20150101&end_date=20170901",
                          orderby="symbol")
```

详细文档，参见[这里](http://www.quantos.org/jaqs/doc.html)

### 二重境界：使用较方便的DataService

`DataService`专为JAQS设计，它仍利用`DataApi`的接口进行查询，但对一些常用功能进行了封装，调用起来更加方便。

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/data_service_api.png?raw=true)

例如，指数成分查询命令为：

```py
res = data_service.get_index_comp(index='000300.SH', start_date=20150101, end_date=20170901)
```

比使用`DataApi`简化了很多。

详细文档，参见代码注释。

### 三重境界：使用神器DataView

`DataView`可以理解为一个针对具体研究对象的小型数据库。可以方便地构建、增删、查询、存储读取等。

#### 亮点

* 极简数据获取：不需要记忆API调用方法，只需要给出字段名称，即可自动通过对应API查询市场数据与基础数据

* 极简数据调用：时间序列数据、截面数据、面板数据

* 自动数据对齐：市场数据根据交易日对齐，财务数据根据财报发布时间对齐

* 衍生数据计算：通过输入数学公式，可自动调用内置函数计算Rank, Quantile, MA, Max等

* 易用数据存取：存储、读取只需提供路径

有了这些特色，使用`DataView`就像拥有了一个数据助理，告诉他需求，就能全部收集、整理好交给你。例如，下面短短几行代码，就完成了自动获取沪深300成份股从2015年1月1日到2017年9月1日的数据，包括是否属于指数成分、成分权重、申万二级行业分组、OHLC价格、成交量成交额、每股收益、ROE，并对这些数据做了对齐操作，存储在`'save_folder'`文件夹内。

```py
props = {'start_date': 20150101, 'end_date': 20170901, 'universe': '000300.SH',
         'fields': ('open,high,low,close,vwap,volume,turnover,eps_basic,roe,sw2'
                    ),
         'freq': 1}
         
dv = DataView()
dv.init_from_config(props, ds)
dv.prepare_data()

dv.save_dataview('save_folder')
```

详细文档，参见[这里](http://www.quantos.org/jaqs/doc.html)。

## 信号研究与事件研究

对投资机会的识别和对未来的预测是投资的基础，这就涉及到各种各样的**信号**。成交量和成交价的相关系数可以作为信号，股价是否创出半年新高也可以作为信号。

_注：此处我们所讲的信号指Alpha策略研究中的信号_

很多时候我们希望对信号进行快速的测试，JAQS提供`SignalDigger`作为测试、研究信号的工具。它根据每天横截面上所有股票的信号及价格，对信号的收益预测能力、信息比率（IC）等进行分析，包括并不仅限于：

* 分组每日收益

* 分组累计收益

* 纯多头组合收益

* 纯空头组合收益

* 多空每日收益差别

* 多空组合累计收益

* 每日IC

* IC分布

* 月度IC

* 事件发生后n天平均收益、分布等

### 收益分析报告

![](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/returns_report.png?raw=true)

_注：目前的收益分析方式是根据因子值对股票池内股票分组，后续我们会提供基于截面回归的方式。_

### 信息系数分析报告

![](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/ic_report.png?raw=true)

### 事件分析报告

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/event_report.png?raw=true)

## 策略回测

### Alpha策略回测

#### 我们对Alpha策略的定义

* 在一个股票**池**内进行投资

* 衡量投资业绩的基准是股票池的指数或其他某个指数，投资收益与基准收益之差称作**超额收益**

* 通过对股票池内投资标的及投资多少进行**选择**，以此追求超额收益

#### 股票Alpha回测中的坑

* 未来函数

  * 选股时用到了当天收盘价，却以开盘价进行买卖。

  * 三季报10月底才发布，却在10月初用这个数据进行选股。

* 停牌没有检测股票是否停牌，在停牌期间进行买卖操作。

* 涨跌停买入开盘涨停的股票或卖出开盘跌停的股票。

* 除权除息由于分红、配送股等原因，股价或持仓量发生变化，却未做出相应调整。

* 指数成分指数（如沪深300）成分是每天动态变化的，2014年1月沪深300中的300支股票，到了2017年可能只有不到200支留在沪深300中。如果只使用静态股票池，无论是用2014年的还是2017年的，都会导致问题。

* 退市在退市前有持仓，且没有卖出，一直持有下去或在退市后以错误的价格卖出。

坑之所以成为坑，是因为总有人前赴后继地往里掉。相比浪费大量时间一个个掉下去、爬上来，不如站在巨人肩头，从一开始就避免这常见问题。JAQS的Alpha策略回测系统已实现对以上问题的**自动处理**，帮大家避坑：

* 未来函数：任何一天，只提供截止当天的数据

* 停牌：自动过滤停牌

* 涨跌停：自动过滤涨跌停

* 指数成分：使用日频动态指数成分，自动过滤非成份股

* 除权除息：以真实价格成交，发生除权除息时默认进行再投资对仓位做相应调整，与真实情况一致

* 退市：在股票退市前，将其仓位清空

#### JAQS想解决的问题

1. 想到/听到一个不错的想法，花1小时将其转化为策略，却**浪费1天甚至好几天实现策略**，错过了机会、消散了激情：JAQS把重复的数据处理、回测逻辑替大家做好，回测时可专注于策略本身

2. 使用在线网站或其他研究性编程语言（MATLAB、R等）初步测试了策略，却难以实现对**细节的控制**及进一步的**自定义开发**，也难以方便地对接行情数据和交易系统：JAQS作为可完全本地化运行的开源软件，使用者具有完全的控制力；同时JAQS作为quantOS的一员，可直接接入DataCore数据系统、TradeSim模拟交易及实盘交易系统。

#### 用JAQS实现Alpha策略

无论是用什么方法来做策略，最终都归结于**在每只股票上的权重是多少**这个问题，我们的回测框架正是基于此，使用者只需在每次调仓时求得**一系列权重**，其中每个值代表在对应股票上的权重。这一步是策略的核心，至于具体的回测逻辑和发单成交，则已由回测框架代劳，一般不需考虑（\#TODO 这一句怎么写比较好）。

更具体来说，我们把求得权重这一过程，分为**两个子过程**：

1. 对股票池进行筛选

2. 给筛选后的股票池中的股票赋予权重

在实现上，我们遵循**模块化设计**的原则，通过定义函数来实现以上子过程，格式为：

```py
# 函数头固定为context和user_options，前者存储了一些全局变量，后者是用户自定义的参数
def selection_function(context, user_options=None):
    # context.snapshot为DataFrame类型，内存储了当前交易日股票池内所有标的、所有字段的数据
    # 我们取出'total_profit_growth'这一字段，并赋值给growth_rate变量（数据类型为Seires），以备使用
    growth_rate = context.snapshot['total_profit_growth']
    # 我们要求增长率growth_rate大于5%，condition是一个值为bool类型的Series
    condition = growth_rate > 0.05
    # 返回值固定为一个Series，index为股票代码，value为True or False，True表示选中该股票，False表示不选
    return condition
```

```py
# 函数头固定为context和user_options，前者存储了一些全局变量，后者是用户自定义的参数
def weight_function(context, user_options=None):
    # context.snapshot_sub为DataFrame类型，内存储了当前交易日【筛选后】股票池内所有标的、所有字段的数据
    # 我们取出'total_mv'这一字段，并赋值给market_value变量（数据类型为Seires），以备使用
    market_value = context.snapshot_sub['total_mv']
    # 我们令权重为总市值（market_value）的相反数，即市值越小，买的越多
    weights = 1.0 / market_value
    # 返回值固定为一个Series，index为股票代码，value为float，数值大小代表权重大小
    return weights
```

此外，使用者也可以将自己研究得到的目标仓位它存入`context`中，直接在权重函数中返回，达到按自定义仓位回测的目的。



### 择时策略回测

#### 我们对择时策略的定义

择时即“选择时机“，依据是数据。也即不断根据最新的市场数据，做出交易决策。这里可以与Alpha策略做一对比：

* Alpha策略总是有持仓，而择时策略不一定随时都有持仓。

* Alpha策略以权重为目标，构建投资组合，而择时策略既可能以持仓为目标，也可能以买/卖量为目标。

#### 择时策略如何运行

* 实盘时，策略通过`on_data`接收数据系统发来的最新行情信息，并进行决策。

* 回测时，遍历历史数据（日线/分钟线/tick），并调用策略的`on_data`，以模拟实盘情况。

_注：为了处理不同频率的数据，`on_data`实际上分为`on_tick`和`on_quote`，前者负责处理Bar类型数据，后者负责处理Tick类型数据。因为Bar数据可以进行对齐，而Tick数据一般难以对齐。_

#### JAQS解决的问题

* 事件驱动引擎：满足大部分非高频策略的需求。

* 简洁的策略框架：集成父类`EventDrivenStrategy`，修改几个关键函数，即可写出自己的策略。

* 回测与实盘一致性：

  择时策略不像Alpha策略逻辑简单清晰（算权重、发单、成交），每一个决策都前后关联，因而此类策略回测的一个重点就是回测与实盘的一致性。由于数据精度、市场冲击等因素，完全一致是不可能的，只能做到尽量一致，包括

  * 代码一致性：回测所用的策略代码，可以少修改甚至不修改上实盘

    JAQS的运行主程序以及交易API都实现了回测和实盘两个版本，回测过的策略，无需修改策略逻辑，可**直接切换**到实盘运行。

  * 结果一致性：同一段时间，回测的模拟交易结果，应和实盘运行相同策略的交易结果接近甚至相同

    JAQS的择时策略回测实现了撮合器，可根据订单类型模拟撮合成交。

#### 用JAQS实现择时策略

前文已经讲到，择时策略回测是对历史数据的遍历，因而要实现一个策略，主要是实现`on_data`函数，例如：

```py
# 我们使用Bar数据回测，因而用`on_bar`而不是`on_tick`
# on_bar作为策略类的一个成员函数，因而有`self`参数
# `bar`是接收到的行情，含有高开低收、成交量等信息
def on_bar(self, bar):
  price = bar.close  #  Bar的close价格
  symbol = bar.symbol  # 标的代码
  if price < 11:
    buy_size = 1  # 购买1手/股
    # 调用trade_api进行发单
    self.ctx.trade_api.place_order(symbol, 'Buy', price + 2, buy_size)
```

这个函数实现了当价格低于11时，以（价格+2）买1手/股标的策略。

除了`on_data`外，还需要实现初始化函数`initialize`, 成交回报函数`on_trade`等，具体参见[本手册第一章]()。

### 回测结果保存

回测结果指成交信息和配置信息。Alpha策略和择时策略均使用统一的方法将这些信息本地保存，以备分析。

```py
# backtest_instance是运行回测的主程序, folder_path是要保存的目录
backtest_instance.save_results(folder_path)
```

调用`save_results`后，成交信息会保存为`trades.csv`文件，配置信息保存为`props.json`文件。由于策略盈亏完全由成交决定，所以基于这些标准化的成交信息，我们在分析时即可复原所有的成交、持仓、盈亏信息。

## 回测/交易结果分析

### 回测结果

上一部分说到，回测后，会把成交信息会保存为`trades.csv`文件，配置信息保存为`props.json`文件。由于策略盈亏完全由成交决定，所以基于这些标准化的成交信息，我们即可复原所有的成交、持仓、盈亏信息。

成交信息格式如下：

| index | commission | entrust\_action | entrust\_no | fill\_date | fill\_no | fill\_price | fill\_size | fill\_time | symbol | task\_id |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 0 | 0.0 | Buy | 56 | 20170103 | 201701030001 | 7.22 | 201000.0 | 143000 | 601111.SH | 201701030001 |
| 1 | 0.0 | Buy | 54 | 20170103 | 201701030002 | 29.38 | 49300.0 | 143000 | 600754.SH | 201701030001 |
| 2 | 0.0 | Buy | 42 | 20170103 | 201701030003 | 26.65 | 54400.0 | 143000 | 600009.SH | 201701030001 |

可以看到其中只有成交价、成交量、成交方向等基本信息。

### 结果分析

我们使用`Analyzer`模块读取基本成交信息，并结合价格信息分析。

对于每一笔交易，`Analyzer`计算：

* 买量BuyVolume, 卖量SellVolume, 总成交量CumVolume, 总成交额CumTurnOver

* 仓位position

* 平均持仓成本AvgPosPrice

* 手续费commission

对于每天的交易情况，除了以上信息的每日加总，每个标的的收盘价，`Analyzer`还会求出

* 交易盈亏trading\_pnl

* 持仓盈亏holding\_pnl

* 总盈亏total\_pnl

此外，我们还提供了预置的画图函数，画出策略pnl，及在每个标的上的具体买卖情况.

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/pnl_img2.png?raw=true)

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/plot_trades.png?raw=true)

### 报告生成

`Analyzer`分析完毕后，会把主要信息（如PnL）保存，生成HTML格式的回测报告。

报告由`Report`对象调用`Jinja2`包通过模板生成，使用者可以自定义模板，添加关心的指标，生成属于个性化的回测报告。

