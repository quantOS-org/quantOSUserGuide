# 如何进行仿真交易

作者：vanvency, PKUJohnson

我们提供仿真交易系统TradeSim。

+ TradeSim是一个在线仿真交易平台（未开源），提供账户管理、在线交易、模拟成交等服务，支持股票、期货等品种的交易。
+ TradeSim中的交易系统模块支持多账户管理、多通道交易、实时风控，提供包括VWAP、TWAP等算法交易，是一款企业级应用。

使用TradeSim非常简单，只需要如下几个步骤：

## 注册

+ 注册成为quantOS的用户，即成为TradeSim系统的用户。注册地址是：[http://www.quantos.org/cas/register.html](http://www.quantos.org/cas/register.html)
+ 每个用户初始提供三个交易策略，分别交易沪深300，中证500和股指期货。
+ 安装JAQS，请参考[https://github.com/quantOS-org/JAQS/blob/master/doc/install.md](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

## API程序化交易

+ 使用TradeApi进行程序化交易。样例代码如下：

```py
from jaqs.trade.tradeapi import TradeApi

# 登录仿真系统
tapi = TradeApi(addr="tcp://gw.quantos.org:8901") 
user_info, msg = tapi.login("phone", "token")     

# 选择策略号
tapi.use_strategy(123) # 123为给你分配的策略号，可以从user_info中获得

# 下单接口
task_id, msg = tapi.place_order("000025.SZ", "Buy", 57, 100)
print "msg:", msg
print "task_id:", task_id
```

+ TradeApi的详细使用方法,请参看[这里](https://github.com/quantOS-org/JAQS/blob/master/doc/trade_api.md).

## Web综合客户端

+ 登录TradeSim网页，查看交易情况。访问地址是：[http://www.quantos.org/tradesim/trade.html](http://www.quantos.org/tradesim/trade.html)
+ 查看到自己的策略，当期的交易，持仓，绩效等情况

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_entrust.PNG?raw=true)

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_trade.PNG?raw=true)

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tradesim_pnl.PNG?raw=true)

+ 目前只有查询功能，未来会有更多丰富的分析功能、交易功能等

## 专业交易客户端

+ 使用vnTrader进行交易。详细使用方法请参见[https://github.com/quantOS-org/TradeSim/blob/master/doc/vnTrader.md](https://github.com/quantOS-org/TradeSim/blob/master/doc/vnTrader.md)

+ 交易界面展示

![](https://github.com/quantOS-org/TradeSim/blob/master/doc/img/vnTrader_main.png?raw=true)
