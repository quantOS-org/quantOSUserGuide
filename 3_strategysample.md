# 两个量化交易策略样例

作者：brz, lli, PKUJohnson

quantOS的策略系统JAQS系统有两大神器——阿尔法选股框架和事件驱动框架。下面我们分别以"**板块内股票动量轮动策略**"和"**期货双均线穿越策略**"为例详细介绍。

_**特别说明**：_

* _这只是一个演示策略，不代表能真正赚钱，使用此策略投资，风险自负。_
* _历史研究不代表未来，仅供参考。_

## _安装JAQS_

本例需要要用到JAQS系统，请根据这里的[安装指南](http://www.quantos.org/jaqs/doc.html)，安装好JAQS系统。

## _阿尔法选股框架_

在这里，我们展示一个“**板块内股票动量轮动策略**”是如何通过JAQS阿尔法选股框架实现的。可在[这里](https://github.com/quantOS-org/JAQS/blob/master/example/alpha/wine_industry_momentum.py)直接下载策略源代码。

### 一、策略描述

策略的主要思想如下：

1、选定某一个板块（[中证白酒399997.SZ](http://xn--399997-9v7i456y48wbgwl.sz/)）

2、每天将板块内所有股票按过去20天的收益率从大到小排序，挑选出排名靠前的10%的股票。

3、使用等市值权重构建投资组合，之后与策略持仓进行对比，进行调仓。

4、给定初始资金金额1亿，期间不增减资金。

这是一个典型的动量轮动策略，选个表现更好的股票，剔除表现一般的股票。

### 二、策略实现和回测

在jaqs平台中，这是一个典型的Alpha选股策略，即通过一定的条件，选出理想的股票，然后按照某种方式进行资产配置。具体做法如下：

#### 步骤1、选出中证白酒指数成份股，计算股票20天的收益率，并进行排序。

在Alpha选股策略中，数据的操作是通过DataView对象实现的，具体代码如下：

```py
from jaqs.data import RemoteDataService
from jaqs.data import DataView

data_config= {
  "remote.data.address": "tcp://data.tushare.org:8910",
  "remote.data.username": "Your Username",
  "remote.data.password": "Your Password"
}

BENCHMARK = '399997.SZ'

ds = RemoteDataService()
ds.init_from_config(data_config)
dv = DataView()

props = {'start_date': 20170101, 'end_date': 20171001, 'universe': BENCHMARK,
    'fields': 'close,volume,sw1',
    'freq': 1}

dv.init_from_config(props, ds)
dv.prepare_data()

dv.add_formula('ret', 'Return(close_adj, 20)', is_quarterly=False)
dv.add_formula('rank_ret', 'Rank(ret)', is_quarterly=False)

dv.save_dataview(folder_path=dataview_dir_path)
```

其中`RemoteDataService`是一个从远程服务器通过`DataApi`读取数据的对象，`DataView`根据`props`里面的配置信息，通过`RemoteDataService`读取相应的基础数据，如本例中的`close`，`volume`，`sw1`。

**这里我们使用了quantOS官方数据源data.tushare.org.**

在`DataView`对象中通过`add_formula`方法，可以通过公式定义衍生数据，比如本例中的`Return(close_adj, 20)`表示通过复权的收盘价计算的20天收益率，并被命名为`ret`。

`add_formula`还可以基于衍生数据定义新的衍生数据，比如本例中的`Rank(ret)`表示基于`ret`数据的排序，并被命名为`rank_ret`（在中证白酒指数成份股中的排序，从小到大，返回值范围是\(0，1\]）。

最后通过`save_dataview`把准备的数据保存到本地文件，供后续回测使用。

#### 步骤2、选出收益率最高的前10%的股票。

在Alpha选股策略中，是通过定义股票筛选函数来实现的，代码如下：

```py
def my_selector(context, user_options=None):
    rank_ret = context.snapshot['rank_ret']

    return rank_ret >= 0.9
```

这里的`context`是策略的上下文环境，从中可以提取到之前定义的数据。

#### 步骤3、策略回测

代码如下：

```py
dv = DataView()
dv.load_dataview(folder_path=dataview_dir_path)

data_config = {
  "remote.data.address": "tcp://data.tushare.org:8910",
  "remote.data.username": "Your Username",
  "remote.data.password": "Your Password"
}
props = {
    "benchmark": BENCHMARK,
    "universe": ','.join(dv.symbol),

    "start_date": dv.start_date,
    "end_date": dv.end_date,

    "period": "day",
    "days_delay": 0,

    "init_balance": 1e8,
    "position_ratio": 1.0,
}
props.update(data_config)
#props.update(trade_config)

trade_api = AlphaTradeApi()

stock_selector = model.StockSelector()
stock_selector.add_filter(name='rank_ret_top10', func=my_selector)

strategy = AlphaStrategy(stock_selector=stock_selector, pc_method='equal_weight')
pm = PortfolioManager()

bt = AlphaBacktestInstance()

context = model.Context(dataview=dv, instance=bt, strategy=strategy, trade_api=trade_api, pm=pm)
stock_selector.register_context(context)

bt.init_from_config(props)
bt.run_alpha()

bt.save_results(folder_path=backtest_result_dir_path)
```

回测过程包括：

1. 从本地数据中加载`DataView`对象，设置回测参数。
2. 指定一个交易接口对象`AlphaTradeApi`，用于回测过程发单。
3. 将之前定义的规则加入股票筛选器`stock_selector`，并命名为rank\_ret\_top10。
4. 构造一个`AlphaStrategy`策略对象，其中需要指定stock\_selector和组合构建方法，这里使用的是equal\_weight（等市值权重）
5. 构造一个仓位管理对象`PortfolioManager`。
6. 构造一个回测实例对象`AlphaBacktestInstance`。
7. 构造一个上下文对象`Context`，需要给定`dataview`，回测实例对象，策略对象，交易接口，仓位管理对象
8. 回测实例对象`AlphaBacktestInstance`初始化参数，并执行`run_alpha`进行回测，保存回测结果。

#### 步骤4、回测结果分析

代码如下：

```py
import jaqs.trade.analyze as ana

ta = ana.AlphaAnalyzer()
dv = DataView()
dv.load_dataview(folder_path=dataview_dir_path)

ta.initialize(dataview=dv, file_folder=backtest_result_dir_path)

ta.do_analyze(result_dir=backtest_result_dir_path, selected_sec=list(ta.universe)[:3], brinson_group='sw1')
```

通过AlphaAnalyzer对象，可对回测结果进行分析，查看策略回测情况。![](/assets/pnl_img.png)

### 三、仿真交易

我们使用TradeSim平台进行仿真交易,其代码如下:


### 四、策略实盘交易
我们使用vn.py进行实盘交易.
## _事件驱动框架_

这里我将以股指期货的双均线穿越策略为例,展示如何使用JAQS的事件驱动策略框架. 完整的策略代码,可以在[这里](https://github.com/quantOS-org/JAQS/blob/master/example/eventdriven/DoubleMA.py)下载.

### 一、策略描述

双均线穿越策略总共分几步？答：两步。

1. 首先，确定交易标的。根据历史价格数据计算快速均线和慢速均线。在实盘交易中，数据周期为1 tick（500毫秒），在回测中，周期为1分钟。
2. 当快线上穿慢线时，进行做多操作，或开多或平空；反之进行做空操作，或开空或平多。
   我们可将上穿和下穿视为两个“事件”，利用我们JAQS的事件驱动策略框架实现起来易如反掌。

### 二、策略实现

古人云：工欲善其事，必先利其器。首先让我带大家熟悉一下事件驱动策略框架的组成部分。该框架由类DoubleMaStrategy\(\)实现，完成了“数据采集-信号生成-发单”的整个交易逻辑。

```python
class DoubleMaStrategy(EventDrivenStrategy):
    """"""
    def __init__(self):
        super(DoubleMaStrategy, self).__init__()
        pass

    def init_from_config(self, props):
        super(DoubleMaStrategy, self).init_from_config(props)
        pass

    def buy(self, quote, size=1):
        pass

    def sell(self, quote, size=1):
        pass

    def on_tick(self, quote):
        pass

    def on_quote(self, quote_dic):
        pass

    def on_trade(self, ind):
        pass
```

其中，

* init\_from\_config负责通过props接收策略所需参数，进行策略初始化；
* buy/sell负责发单；
* on\_tick和on\_quote是框架核心，策略的逻辑都在其中实现，区别在于前者接收tick输入，后者接收bar数据；
* on\_trade负责监督发单成交情况。

下面请听我一一介绍。

#### 1. 参数初始化

首先在函数**init**中初始化策略所需变量

```python
def __init__(self):
    super(DoubleMaStrategy, self).__init__()
    self.symbol = ''

    self.fast_ma_len = 0   # 快线周期
    self.slow_ma_len = 0   # 慢线周期

    self.window_count = 0  # 价格序列长度
    self.window = self.slow_ma_len + 1

    self.price_arr = np.zeros(self.window)   # 价格序列保存最近n个价格
    self.fast_ma = 0       # 快线均值
    self.slow_ma = 0       # 慢线均值
    self.pos = 0           # 仓位

    self.buy_size_unit = 1
    self.output = True
```

该策略的关键参数有三个，交易标的（symbol），快线周期（fast\_length）和慢线周期（slow\_length），通过props字典传入。

```python
def init_from_config(self, props):
    super(DoubleMaStrategy, self).init_from_config(props)
    self.symbol = props.get('symbol')
    self.init_balance = props.get('init_balance')
    self.fast_ma_len = props.get('fast_length')
    self.slow_ma_len = props.get('slow_length')
```

#### 2. 逻辑实现

策略逻辑的实现在on\_tick函数中完成。在实盘中，每隔500毫秒就会有一个quote传入，主要包括当前的bid price, ask price，在回测中，每隔1分钟会有一个quote传入，主要包括open, close, high和low。下面我们以实盘为例。  
第一步：根据历史价格数据计算快速均线和慢速均线。  
每当一个quote传入，我们计算当前的midprice并更新价格序列。

```py
if hasattr(quote, 'bidprice1'):
    mid = (quote.bidprice1 + quote.askprice1) / 2.0
else:
    mid = quote.close
self.price_arr[0: self.window - 1] = self.price_arr[1: self.window]
self.price_arr[-1] = mid
```

接着计算快速均线和慢速均线。

```py
self.fast_ma = np.mean(self.price_arr[-self.fast_ma_len - 1:])
self.slow_ma = np.mean(self.price_arr[-self.slow_ma_len - 1:])
```

第二步：当快线上穿慢线时，进行做多操作，或开多或平空；反之进行做空操作，或开空或平多。

```py
if self.fast_ma > self.slow_ma:
    if self.pos == 0:
        self.buy(quote, 1)
    elif self.pos < 0:
        self.buy(quote, 2)

elif self.fast_ma < self.slow_ma:
    if self.pos == 0:
        self.sell(quote, 1)
    elif self.pos > 0:
        self.sell(quote, 2)
```

至此，策略实现，就是这么简单！

### 三、回测与仿真交易

现在我们需要做的就是启动策略，这一步在run\_strategy函数中完成。

```py
trade_config = {
  "remote.trade.address": "tcp://gw.quantos.org:8901",
  "remote.trade.username": "Your Username",
  "remote.trade.password": "Your Password"
}

def run_strategy():
    if is_backtest:
        props = {"symbol": "IH1712.CFE",
                 "start_date": 20170510,
                 "end_date": 20170930,
                 "fast_length": 5,
                 "slow_length": 20,
                 "bar_type": "1M",  # '1d'
                 "init_balance": 2e4}

        tapi = BacktestTradeApi()
        ins = EventBacktestInstance()

    else:
        props = {'symbol': 'IH1712.CFE',
                 "fast_length": 5,
                 "slow_length": 20,
                 'strategy.no': 64}
        tapi = RealTimeTradeApi(trade_config)
        ins = EventRealTimeInstance()
```

你所需要做的只是输入props中的参数，并通过设置全局变量is\_backtest确定是仿真交易还是回测。

### 四、回测结果及分析

![](/assets/doubleMA.png)

回测结果还让你满意么？Nani，居然亏钱了？别慌，让我做个分析。策略的分析模块在analyze函数中完成。

```py
def analyze():
    ta = ana.EventAnalyzer()

    ds = RemoteDataService()
    ds.init_from_config(data_config)

    ta.initialize(data_server_=ds, file_folder=result_dir_path)

    ta.do_analyze(result_dir=result_dir_path, selected_sec=[])
```

### 五、仿真交易展示

待补充

# 总结

至此，我带大家完整地体验一次JAQS的两大策略开发模型。虽然有所体验，但相信您还有很多疑问，比如：研究方法是否科学？数据怎么来的？策略回测的调度逻辑是什么？在实盘交易前，我还需要做那些验证？我怎么能利用quantOS提供的工具，构建自己的交易事业？

在随后的章节里，这些答案将会一一呈现在您面前。
