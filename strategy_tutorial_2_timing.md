# quantOS量化策略样例(2)：择时

本文将向大家展示如何利用JAQS系统的事件驱动框架开发择时策略，并通过TradeSim进行仿真交易。这里我将以贵州茅台（600519.SH）的双均线穿越策略为例。

JAQS安装教程，参见 [JAQS安装指南](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

**完整代码**请点击[这里](https://github.com/quantOS-org/JAQS/tree/master/example/eventdriven)，安装JAQS后即可直接运行（安装教程，参见[JAQS安装指南](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)）。

## 一、策略描述
双均线穿越策略分为两步：
1. 首先，确定交易标的，根据历史价格数据计算快速均线和慢速均线。在本文的回测模式中，数据周期为1天。
2. 当快线均值上穿慢线均值时，若此时没有持仓，则买入股票；反之当快线均值下穿慢线均值时，若此时有股票持仓，则卖出所有股票。

## 二、策略实现
首先向大家介绍一下事件驱动策略框架的组成部分。该框架由类`DoubleMaStrategy()`实现，完成了“数据采集-信号生成-发单”的整个交易逻辑。
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

    def on_bar(self, quote_dic):
        pass

    def on_trade(self, ind):
        pass
```
其中，

 - `init_from_config()`负责通过props接收策略所需参数，进行策略初始化；
 - `buy()`/`sell()`负责发单；
 - `on_tick()`和`on_bar()`是框架核心，策略的逻辑都在其中实现，区别在于`on_tick()`接收单个quote变量，而`on_bar()`接收多个quote组成的dictionary。此外`on_tick()`是在tick级回测和实盘/仿真交易中使用，而`on_bar()`是在bar回测中使用；
 - `on_trade()`负责监督发单成交情况。

### 1. 参数初始化
首先在函数`__init__()`中初始化策略所需变量
```python
def __init__(self):
    super(DoubleMaStrategy, self).__init__()

    # 标的
    self.symbol = ''

    # 快线和慢线周期
    self.fast_ma_len = 0
    self.slow_ma_len = 0
    
    # 记录当前已经过的天数
    self.window_count = 0
    self.window = 0

    # 固定长度的价格序列
    self.price_arr = None

    # 快线和慢线均值
    self.fast_ma = 0
    self.slow_ma = 0

    # 当前仓位
    self.pos = 0

    # 下单量乘数
    self.buy_size_unit = 1
    self.output = True
```
该策略的关键参数有三个，交易标的（symbol），快线周期（fast_ma_len）和慢线周期（slow_ma_len），通过props字典传入。
```python
def init_from_config(self, props):
    """
    将props中的用户设置读入
    """
    super(DoubleMaStrategy, self).init_from_config(props)
    # 标的
    self.symbol = props.get('symbol')

    # 初始资金
    self.init_balance = props.get('init_balance')

    # 快线和慢线均值
    self.fast_ma_len = props.get('fast_ma_length')
    self.slow_ma_len = props.get('slow_ma_length')
    self.window = self.slow_ma_len + 1

    # 固定长度的价格序列
    self.price_arr = np.zeros(self.window)

```
### 2. 逻辑实现
策略的日级回测在`on_bar()`函数中完成。在回测中，每天会有一个quote传入，主要包括open, close, high和low。下面我们以回测为例。
第一步：根据历史价格数据计算快速均线和慢速均线。
每当一个quote传入，我们计算当前的midprice并更新价格序列。
```python
if isinstance(quote, Quote):
    # 如果是Quote类型，mid为bidprice和askprice的均值
    bid, ask = quote.bidprice1, quote.askprice1
    if bid > 0 and ask > 0:
        mid = (quote.bidprice1 + quote.askprice1) / 2.0
    else:
        # 如果当前价格达到涨停板或跌停板，系统不交易
        return
else:
    # 如果是Bar类型，mid为Bar的close
    mid = quote.close

# 将price_arr序列中的第一个值删除，并将当前mid放入序列末尾
self.price_arr[0: self.window - 1] = self.price_arr[1: self.window]
self.price_arr[-1] = mid
```
接着计算快速均线和慢速均线，
```python
self.fast_ma = np.mean(self.price_arr[-self.fast_ma_len:])
self.slow_ma = np.mean(self.price_arr[-self.slow_ma_len:])
```
第二步：当快线向上穿越慢线且当前没有持仓，则买入100股；当快线向下穿越慢线且当前有持仓，则平仓。
```python
if self.fast_ma > self.slow_ma:
    if self.pos == 0:
        self.buy(quote, 100)

elif self.fast_ma < self.slow_ma:
    if self.pos > 0:
        self.sell(quote, self.pos)
```
第三步：交易完成后，会触发`on_trade()`函数，通过`self.ctx.pm.get_pos()`函数的到最新仓位并更新self.pos
```python
def on_trade(self, ind):
    print("\nStrategy on trade: ")
    print(ind)
    self.pos = self.ctx.pm.get_pos(self.symbol)
```
## 三、回测启动
策略启动在`run_strategy()`函数中完成。
```python
def run_strategy():
    if is_backtest:
        """
        回测模式
        """
        props = {"symbol": '600519.SH',
                 "start_date": 20170101,
                 "end_date": 20171104,
                 "fast_ma_length": 5,
                 "slow_ma_length": 15,
                 "bar_type": "1d",  # '1d'
                 "init_balance": 50000}

        tapi = BacktestTradeApi()
        ins = EventBacktestInstance()
        
    else:
        """
        实盘/仿真模式
        """
        props = {'symbol': '600519.SH',
                 "fast_ma_length": 5,
                 "slow_ma_length": 15,
                 'strategy.no': 1062}
        tapi = RealTimeTradeApi(trade_config)
        ins = EventLiveTradeInstance()
```
## 四、回测结果及分析
回测结果如下图所示：
![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/doubleMA.png?raw=true)

回测结果的分析在`analyze()`中完成
```python
def analyze():
    ta = ana.EventAnalyzer()
     
    ds = RemoteDataService()
    ds.init_from_config(data_config)
    
    ta.initialize(data_server_=ds, file_folder=result_dir_path)
    
    ta.do_analyze(result_dir=result_dir_path, selected_sec=[])
```
*__注__：本文只介绍了策略核心函数和逻辑，完整代码请点击[这里](https://github.com/quantOS-org/JAQS/blob/master/example/eventdriven/DoubleMA.py)。*

## 五、仿真交易及结果展示
本例展示的是日线交易策略,可以通过TradeApi进行手动下单。需要输入的信息包括股票代码,交易方向,价格和交易量。
```python
from jaqs.trade.tradeapi import TradeApi
tapi = TradeApi(trade_config['remote.trade.address'])
user_info, msg = tapi.login(trade_config['remote.trade.username'], trade_config['remote.trade.password'])
task_id, msg = tapi.place_order(security = '600519.SH', action = 'Buy', price = 667, size = 100)
```
发单成功后,trade_api为任务编号。具体订单持仓盈亏可在[仿真交易网站](https://www.quantos.org/tradesim/trade.html)查看,如下图所示:
![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/doubleMA_tradeapi.PNG?raw=true)
