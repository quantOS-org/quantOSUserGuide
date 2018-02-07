# quantOS典型应用场景分析

作者：quantOS.org

不同的用户需求，决定了不同的系统架构，quantOS提供几种典型的组合场景，供用户自行选择。主要包括：

+ 个人投资者
+ 小型投资机构
+ 中大型投资机构
+ 独立数据用户
+ 独立交易用户

## 场景1：适合个人投资者

个人投资者进行量化投资，一般会面临几个具体的困难：

*  获取数据的成本比较高，没有自己的数据源
*  没有完整的策略研究框架，用于开发策略和验证策略的有效性
*  实盘交易需要开发对接各种交易接口。

以上各种问题，在quantOS上都能得到完美的解决。我们建议的解决方案如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/solution_case1.png?raw=true)

1. 使用在线数据源\(data.quantos.org\)，作为自己的数据源，数据质量及时可靠，使用DataApi进行访问，简单易用。
2. 使用JAQS平台进行策略研究。JAQS集成了信号研究、策略回测、回测分析等功能模块，同时支持Alpha、CTA、套利等，可以快速开发策略，进行回测。
3. 使用TradeSim进行仿真交易，TradeSim支持股票、期货等品种的交易，根据实时行情进行模拟撮合，最大程度接近实盘效果，提供绩效分析功能，方便用户跟踪策略在模拟盘中的绩效，做到心中有数。
4. 使用vn.py进行实盘交易，vn.py已经实现了与国内各大主流交易系统的对接，可满足个人用户的单帐户交易要求。JAQS和vn.py之间通过TradeApi进行标准化对接，可做到仿真与实盘交易无缝对接。

**注意**：这个方案只需要安装JAQS即可使用。

JAQS安装文档参见：[https://github.com/quantOS-org/JAQS/blob/master/doc/install.md](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

## 场景2：适合小型投资机构

与个人投资者不同，小型投资机构进行量化投资，遇到的困难包括：

*  有自己的研究数据，业务上要求数据系统部署在本地，但没有一套高效的数据系统，将各类数据整合在一起。
*  机构内部有多个用户需要使用数据，对数据标准化有要求。
*  有交易需求，但没有能力维护一套大的交易系统。
*  没有完整的策略研究框架，用于开发策略和验证策略的有效性

quantOS专门为小型投资机构量身定做的解决方案如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/solution_case2.png?raw=true)

*  使用DataCore项目，搭建自己的数据系统。DataCore是一款企业级开源量化数据系统，通过标准化接口提供高速实时行情、历史行情和参考数据等核心服务，覆盖股票、商品期货、股指期货、国债期货等品种，适配CTP、万得、聚源、Tushare等各类数据。
*  使用JAQS进行策略研究。JAQS集成了信号研究、策略回测、回测分析等功能模块，同时支持Alpha、CTA、套利等，可以快速开发策略，进行回测。
*  使用TradeSim进行仿真交易，TradeSim支持股票、期货等品种的交易，根据实时行情进行模拟撮合，最大程度接近实盘效果，提供绩效分析功能，方便用户跟踪策略在模拟盘中的绩效，做到心中有数。
*  使用vn.py进行实盘交易，vn.py已经实现了与国内各大主流交易系统的对接，可满足小型投资机构少量帐户的交易要求。JAQS和vn.py之间通过TradeApi进行标准化对接，可做到仿真与实盘交易无缝对接。

**注意**：这个方案需要安装DataCore和JAQS。

DataCore安装文档参见：[https://github.com/quantOS-org/DataCore/blob/master/doc/install.md](https://github.com/quantOS-org/DataCore/blob/master/doc/install.md)

JAQS安装文档参见：[https://github.com/quantOS-org/JAQS/blob/master/doc/install.md](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

## 场景3：适合中大型投资机构

中大型机构的业务要求较高，主要包括：

*  用户数量较多，需要多人共享研究数据和交易通道。
*  有自己的独特的研究数据。数据的标准化、权限管理很重要。
*  交易呈现多个用户、多个产品、多个账户的特点。
*  对实时交易风控有要求。
*  需要有通用的策略研究平台，严谨的研究方法。

quantOS量化交易平台诞生于对冲基金，从一开始就是为中大型机构设计的。我们建议的解决方案如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/solution_case3.png?raw=true)

这个解决方案的特点是：

*  使用DataCore项目，搭建自己的数据系统。DataCore是一款企业级开源量化数据系统，通过标准化接口提供高速实时行情、历史行情和参考数据等核心服务，覆盖股票、商品期货、股指期货、国债期货等品种，适配CTP、万得、聚源、Tushare等各类数据。

*  使用JAQS进行策略研究。JAQS集成了信号研究、策略回测、回测分析等功能模块，同时支持Alpha、CTA、套利等，可以快速开发策略，进行回测。
*  使用TradeSim进行仿真交易，TradeSim支持股票、期货等品种的交易，根据实时行情进行模拟撮合，最大程度接近实盘效果，提供绩效分析功能，方便用户跟踪策略在模拟盘中的绩效，做到心中有数。
*  实盘交易建议使用我们的专业版交易软件TKPro。

**注意**：这个方案需要安装DataCore和JAQS.

DataCore安装文档参见：[https://github.com/quantOS-org/DataCore/blob/master/doc/install.md](https://github.com/quantOS-org/DataCore/blob/master/doc/install.md)

JAQS安装文档参见：[https://github.com/quantOS-org/JAQS/blob/master/doc/install.md](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

**TKPro目前尚未开源**

TKPro支持的交易功能非常丰富，包括：

*  用户登录，选择策略号
*  查询策略持仓
*  查询账户资金
*  查询委托、成交
*  成交推送，委托状态推送
*  普通下单、篮子下单
*  算法下单（TWAP、VWAP，配对交易，最优价格算法等）
*  撤单
*  相对数量下单
*  组合目标下单

除了下单外，TKPro还提供的功能包括：

*  多策略管理
*  多交易通道支持
*  智能交易路由
*  极速交易风控

TKPro是一款适合中大型交易机构的企业级交易系统，可以本地化部署，有兴趣的同学可以联系[tkpro@quantos.org](mailto:tkpro@quantos.org)。

## 场景4：独立数据用户

独立数据客户希望通过获取数据后，进行数据分析。我们建议的方案如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/solution_case4.png?raw=true)

1. 使用在线数据源\(data.quantos.org\)，作为自己的数据源，数据质量及时可靠，使用DataApi进行访问，简单易用。
2. 使用JAQS平台进行数据分析系统。JAQS提供的DataView组件，可方便快捷的取到用户需要的数据，支持通过公式定义衍生数据，支持本地化存储。
3. 用户获得数据后，根据自己的业务生成报表和分析结果。

**注意**：这个方案只需要安装JAQS即可使用。

JAQS安装文档参见：[https://github.com/quantOS-org/JAQS/blob/master/doc/install.md](https://github.com/quantOS-org/JAQS/blob/master/doc/install.md)

## 场景5：独立交易用户

独立交易客户希望能直接进行交易，我们建议的方案如下：

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/solution_case5.png?raw=true)

*  安装TradeApi，通过统一的TradeApi可以对接仿真交易和实盘交易
*  使用TradeSim进行仿真交易，TradeSim支持股票、期货等品种的交易，根据实时行情进行模拟撮合，最大程度接近实盘效果，提供绩效分析功能，方便用户跟踪策略在模拟盘中的绩效，做到心中有数。
*  使用vn.py进行实盘交易，vn.py已经实现了与国内各大主流交易系统的对接，可满足小型投资机构少量帐户的交易要求。TradeApi可以无缝对接vnpy。
*  使用我们的专业版交易软件TKPro，这个一般适合中大型投资机构。

