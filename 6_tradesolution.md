# 交易系统解决方案

作者：PKUJohnson, symbol, vanvency, 用python的交易员

交易是将量化投资付诸实施的手段，也是实现量化投资理念的重要步骤。

## 仿真交易TradeSim

严谨的量化投资，应该进行一定时间的仿真交易，作为策略实盘前的验证。因此，quantOS提供了一套完备的仿真交易系统TradeSim，供用户使用。

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

## 实盘交易

实盘交易是用户最重要的交易环节，对用户至关重要。我们推荐两类解决方案：

### 实盘方案1：使用vn.py交易

vn.py是一个用户众多的开源交易软件，已经实现了与众多交易柜台的对接。quantOS将JAQS策略系统与VN.PY进行了集成，实现了统一的交易规范，如下图所示：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/trade_api.png?raw=true)

JAQS用户可以通过统一的TradeApi，访问TradeSim系统进行仿真交易，访问vn.py进行实盘交易。

vn.py通过提供jaqsService，实现了TradeApi的支持。详细信息请参看[vn.py官方网站](https://github.com/vnpy/vnpy)。

### 实盘方案2：使用专业版的交易系统TKPro

TKPro是quantOS的专业交易系统，目前尚未开源。其技术架构如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/tkpro.png?raw=true)

JAQS用户可以通过统一的TradeApi，像访问TradeSim一样，访问TKPro。

TKPro支持的交易功能非常丰富，包括：

* 用户登录，选择策略号
* 查询策略持仓
* 查询账户资金
* 查询委托、成交
* 成交推送，委托状态推送
* 普通下单、篮子下单
* 算法下单（TWAP、VWAP，配对交易，最优价格算法等）
* 撤单
* 相对数量下单
* 组合目标下单

除了下单外，TKPro还提供的功能包括：

* 多策略管理
* 多交易通道支持
* 智能交易路由
* 极速交易风控

TKPro是一款适合企业级交易系统，可以本地化部署，有兴趣的同学可以联系[tkpro@quantos.org](mailto:tkpro@quantos.org)。

TradeSim相当于TKPro的在线版本，提供了模拟撮合功能。

