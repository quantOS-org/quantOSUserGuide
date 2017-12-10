
### Calendar Spread交易策略


本帖主要介绍了基于事件驱动回测框架实现calendar spread交易策略。可在[这里](https://github.com/quantOS-org/JAQS/blob/master/example/eventdriven/CalendarSpread.py)直接下载策略源代码。

#### 一. 策略介绍
在商品期货市场中，同一期货品种不同到期月份合约间的价格在短期内的相关性较稳定。该策略就利用这一特性，在跨期基差稳定上升时进场做多基差，反之做空基差。
在本文中我们选择了天然橡胶作为交易品种，时间范围从2017年7月到2017年11月，选择的合约为RU1801.SHF和RU1805.SHF，将基差定义为近期合约价格减去远期合约价格。

#### 二. 参数准备
我们在test_spread_commodity.py文件中的test_spread_trading()函数中设置策略所需参数，例如交易标的，策略开始日期，终止日期，换仓频率等。
```python
props = {
         "symbol"                : "ru1801.SHF,ru1805.SHF",
         "start_date"            : 20170701,
         "end_date"              : 20171109,
         "bar_type"              : "DAILY",
         "init_balance"          : 2e4,
         "bufferSize"            : 20,
         "future_commission_rate": 0.00002,
         "stock_commission_rate" : 0.0001,
         "stock_tax_rate"        : 0.0000
         }
```

#### 三. 策略实现
策略实现全部在spread_commodity.py中完成，创建名为SpreadCommodity()的class继承EventDrivenStrategy，具体分为以下几个步骤：

##### 1. 策略初始化
这里将后续步骤所需要的变量都创建好并初始化。
```python
def __init__(self):
    EventDrivenStrategy.__init__(self)

    self.symbol      = ''
    self.s1          = ''
    self.s2          = ''
    self.quote1      = None
    self.quote2      = None

    self.bufferSize  = 0
    self.bufferCount = 0
    self.spreadList  = ''
```

##### 2. 从props中得到变量值
这里将props中设置的参数传入。其中，self.spreadList记录了最近$n$天的spread值，$n$是由self.bufferSize确定的。
```python
def init_from_config(self, props):
    super(SpreadCommodity, self).init_from_config(props)
    self.symbol       = props.get('symbol')
    self.init_balance = props.get('init_balance')
    self.bufferSize   = props.get('bufferSize')
    self.s1, self.s2  = self.symbol.split(',')
    self.spreadList = np.zeros(self.bufferSize)
```

##### 3. 策略实现
策略的主体部分在on_bar()函数中实现。因为我们选择每日调仓，所以会在每天调用on_bar()函数。
首先将两个合约的quote放入self.quote1和self.quote2中，并计算当天的spread
```python
q1 = quote_dic.get(self.s1)
q2 = quote_dic.get(self.s2)
self.quote1 = q1
self.quote2 = q2
spread = q1.close - q2.close
```
接着更新self.spreadList。因为self.spreadList为固定长度，更新方法为将第2个到最后1个元素向左平移1位，并将当前的spread放在队列末尾。
```python
self.spreadList[0:self.bufferSize - 1] = self.spreadList[1:self.bufferSize]
self.spreadList[-1] = spread
self.bufferCount += 1
```
接着将self.spreadList中的数据对其对应的编号（例如从1到20）做regression，观察回归系数的pvalue是否显著，比如小于0.05。如果结果不显著，则不对仓位进行操作；如果结果显著，再判断系数符号，如果系数大于0则做多spread，反之做空spread。
```python
X, y = np.array(range(self.bufferSize)), np.array(self.spreadList)
X = X.reshape(-1, 1)
y = y.reshape(-1, 1)
X = sm.add_constant(X)

est = sm.OLS(y, X)
est = est.fit()

if est.pvalues[1] < 0.05:
    if est.params[1] < 0:
        self.short_spread(q1, q2)
    else:
        self.long_spread(q1, q2)
```

#### 四. 回测结果
![calendarspreadresult](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/event_driven_calendar_spread_result.png?raw=true)
