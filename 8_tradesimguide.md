# 如何进行仿真交易

作者：vanvency, PKUJohnson

我们提供仿真交易系统TradeSim。

+ TradeSim是一个在线仿真交易平台（未开源），提供账户管理、在线交易、模拟成交等服务，支持股票、期货等品种的交易。
+ TradeSim中的交易系统模块支持多账户管理、多通道交易、实时风控，提供包括VWAP、TWAP、配对交易、篮子下单在内的算法交易，是一款企业级应用。

使用TradeSim非常简单，只需要如下几个步骤：

1、注册成为quantOS的用户，即成为TradeSim系统的合法用户。每个用户初始提供三个交易策略，分别交易沪深300，中证500和股指期货。注册地址是：[http://www.quantos.org/cas/register.html](http://www.quantos.org/cas/register.html)

2、下载TradeApi。下载地址是：[https://github.com/quantOS-org/TradeApi](https://github.com/quantOS-org/TradeApi)

3、使用TradeApi进行程序化交易。样例代码如下：

```py
from tradeapi import TradeApi

# 登录仿真系统
tapi = TradeApi(addr="tcp://gw.quantos.org:8901") 
user_info, msg = tapi.login("demo", "token")     

# 选择策略号
tapi.use_strategy(123) # 123为给你分配的策略号，可以从user_info中获得

# 下单接口
task_id, msg = tapi.place_order("000025.SZ", "Buy", 57, 100)
print "msg:", msg
print "task_id:", task_id
```

TradeApi的详细使用方法,请参看[这里](http://www.quantos.org/tradesim/doc.html).

4、登录TradeSim网页，查看交易情况。访问地址是：[http://www.quantos.org/tradesim/trade.html](http://www.quantos.org/tradesim/trade.html)

在TradeSim网页上，我们能看到自己的策略，当期的交易，持仓，绩效等情况。未来会有更多丰富的分析功能。

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_entrust.PNG?raw=true)

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_trade.PNG?raw=true)

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_pnl.PNG?raw=true)
