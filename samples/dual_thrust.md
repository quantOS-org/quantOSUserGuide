
### 商品期货的Dual Thrust日内交易策略

本帖主要介绍了基于事件驱动回测框架实现Dual Thrust日内交易策略。可在[这里](https://github.com/quantOS-org/JAQS/blob/master/example/eventdriven/DualThrust.py)直接下载策略源代码。

#### 一. 策略介绍
Dual Thrust是一个趋势跟踪策略，具有简单易用、适用度广的特点，其思路简单、参数较少，配合不同的参数、止盈止损和仓位管理，可以为投资者带来长期稳定的收益，被投资者广泛应用于股票、货币、贵金属、债券、能源及股指期货市场等。
在本文中，我们将Dual Thrust应用于商品期货市场中。
简而言之，该策略的逻辑原型是较为常见的开盘区间突破策略，以今日开盘价加减一定比例确定上下轨。日内突破上轨时平空做多，突破下轨时平多做空。
在Dual Thrust交易系统中，对于震荡区间的定义非常关键，这也是该交易系统的核心和精髓。Dual Thrust系统使用
$$Range = Max(HH-LC,HC-LL)$$
来描述震荡区间的大小。其中HH是过去N日High的最大值，LC是N日Close的最小值，HC是N日Close的最大值，LL是N日Low的最小值。

#### 二. 参数准备
我们在test_spread_commodity.py文件中的test_spread_trading()函数中设置策略所需参数，例如交易标的，策略开始日期，终止日期，换仓频率等，其中$k1，k2$为确定突破区间上下限的参数。
```python
props = {
         "symbol"                : "rb1710.SHF",
         "start_date"            : 20170510,
         "end_date"              : 20170930,
         "buffersize"            : 2,
         "k1"                    : 0.7,
         "k2"                    : 0.7,
         "bar_type"              : "MIN",
         "init_balance"          : 1e5,
         "future_commission_rate": 0.00002,
         "stock_commission_rate" : 0.0001,
         "stock_tax_rate"        : 0.0000
         }
```

#### 三. 策略实现
策略实现全部在DualThrust.py中完成，创建名为DualThrustStrategy()的class继承EventDrivenStrategy，具体分为以下几个步骤：

##### 1. 策略初始化
这里将后续步骤所需要的变量都创建好并初始化。其中self.bufferSize为窗口期长度，self.pos记录了实时仓位，self.Upper和self.Lower记录了突破区间上下限。
```python
def __init__(self):
    EventDrivenStrategy.__init__(self)
    self.symbol      = ''
    self.quote       = None
    self.bufferCount = 0
    self.bufferSize  = ''
    self.high_list   = ''
    self.close_list  = ''
    self.low_list    = ''
    self.open_list   = ''
    self.k1          = ''
    self.k2          = ''
    self.pos         = 0
    self.Upper       = 0.0
    self.Lower       = 0.0
```

##### 2. 从props中得到变量值
这里将props中设置的参数传入。其中，self.high_list为固定长度的list，保存了最近$N$天的日最高价，其他变量类似。
```python
def init_from_config(self, props):
    super(DualThrustStrategy, self).init_from_config(props)

    self.symbol       = props.get('symbol')
    self.init_balance = props.get('init_balance')
    self.bufferSize   = props.get('buffersize')
    self.k1           = props.get('k1')
    self.k2           = props.get('k2')
    self.high_list    = np.zeros(self.bufferSize)
    self.close_list   = np.zeros(self.bufferSize)
    self.low_list     = np.zeros(self.bufferSize)
    self.open_list    = np.zeros(self.bufferSize)
```

##### 3. 策略实现
在每天开始时，首先调用initialize()函数，得到当天的open，close，high和low的值，并对应放入list中。
```python
def initialize(self):
    self.bufferCount += 1

    # get the trading date
    td = self.ctx.trade_date
    ds = self.ctx.data_api

    # get the daily data
    df, msg = ds.daily(symbol=self.symbol, start_date=td, end_date=td)

    # put the daily value into the corresponding list
    self.open_list[0:self.bufferSize - 1] =
                   self.open_list[1:self.bufferSize]
    self.open_list[-1] = df.high
    self.high_list[0:self.bufferSize - 1] =
                   self.high_list[1:self.bufferSize]
    self.high_list[-1] = df.high
    self.close_list[0:self.bufferSize - 1] =
                   self.close_list[1:self.bufferSize]
    self.close_list[-1] = df.close
    self.low_list[0:self.bufferSize - 1] =
                   self.low_list[1:self.bufferSize]
    self.low_list[-1] = df.low
```

策略的主体部分在on_bar()函数中实现。因为我们选择分钟级回测，所以会在每分钟调用on_bar()函数。

首先取到当日的quote，并计算过去$N$天的HH，HC，LC和LL，并据此计算Range和上下限Upper，Lower
```python
HH = max(self.high_list[:-1])
HC = max(self.close_list[:-1])
LC = min(self.close_list[:-1])
LL = min(self.low_list[:-1])

Range = max(HH - LC, HC - LL)
Upper = self.open_list[-1] + self.k1 * Range
Lower = self.open_list[-1] - self.k2 * Range
```
几个关键变量的意义如下图所示：
![illustrationdual](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/event_drivent_illustration_dual.png?raw=true)

我们的交易时间段为早上9:01:00到下午14:28:00,交易的逻辑为：
1. 当分钟Bar的open向上突破上轨时，如果当时持有空单，则先平仓，再开多单；如果没有仓位，则直接开多单；
2. 当分钟Bar的open向下突破下轨时，如果当时持有多单，则先平仓，再开空单；如果没有仓位，则直接开空单；
```python
if self.pos == 0:
    if self.quote.open > Upper:
        self.short(self.quote, self.quote.close, 1)
    elif self.quote.open < Lower:
        self.buy(self.quote, self.quote.close, 1)
elif self.pos < 0:
    if self.quote.open < Lower:
        self.cover(self.quote, self.quote.close, 1)
        self.long(self.quote, self.quote.close, 1)
else:
    if self.quote.open > Upper:
        self.sell(self.quote, self.quote.close, 1)
        self.short(self.quote, self.quote.close, 1)
```
由于我们限制该策略为日内策略，故当交易时间超过14:28:00时，进行强行平仓。
```python
elif self.quote.time > 142800:
    if self.pos > 0:
        self.sell(self.quote, self.quote.close, 1)
    elif self.pos < 0:
        self.cover(self.quote, self.quote.close, 1)
```
我们在下单后，可能由于市场剧烈变动导致未成交，因此在on_trade_ind()函数中记录具体成交情况，当空单成交时，self.pos减一，当多单成交时，self.pos加一。
```python
def on_trade_ind(self, ind):
    if ind.entrust_action == 'sell' or ind.entrust_action == 'short':
        self.pos -= 1
    elif ind.entrust_action == 'buy' or ind.entrust_action == 'cover':
        self.pos += 1
    print(ind)
```

#### 四. 回测结果
回测结果如下图所示：
![dualthrustresult](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/event_drivent_dual_thrust_result.png?raw=true)


#### 五、参考文献
> [1]: https://www.ricequant.com/community/topic/392/
> [2]: https://xueqiu.com/5256769224/32429363
