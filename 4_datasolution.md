# 数据系统解决方案

作者：米哥、PKUJohnson，symbol

数据是量化研究的基础原料，我们提供两种模式的数据系统解决方案：

| 解决方案 | 适用对象 | 重要前提条件 |
| :--- | :--- | :--- |
| 使用在线数据平台 | 普通投资者 | 在www.quantos.org注册账户 |
| 使用DataCore搭建本地数据系统 | 机构用户 | 需要有本地数据源。 |

## 方案1：使用在线数据平台

在线数据平台由TusharePro提供，数据包括：

| 数据接口 | 主要内容 | 备注 |
| :--- | :--- | :--- |
| daily | 提供历史日线数据 |  |
| bar | 提供历史分时数据和实时分时数据 | 支持的范围包括：1分钟、5分钟、15分钟 |
| bar\_quote | 提供历史分时数据和实时分时数据 | 支持的范围包括：1分钟、5分钟、15分钟 |
| quote | 提供实时行情快照 |  |
| query | 提供参考数据查询服务 | 范围非常广泛，包括证券信息、财务数据、指数、基金数据等 |

用户只需要通过如下步骤，即可使用：

1、在[http://www.quantos.org](http://www.quantos.org)上注册用户。

2、在Github上下载DataApi，项目地址如下：[https://github.com/quantOS-org/DataApi](https://github.com/quantOS-org/DataApi)。

3、根据需求，获取相应的数据。

参考代码如下：

```py
from data_api import DataApi

api = DataApi(addr="tcp://data.tushare.org:8910") # 在线数据源

api.login("username", "password")  #认证模块，需要修改成www.quantos.org的注册用户

symbol = 'T1712.CFE, TF1712.CFE, rb1712.SHF'
fields = 'open,high,low,last,volume'

# 获取实时行情
df, msg = api.quote(symbol=symbol, fields=fields)
print(df)
print(msg)

# 获取实时分钟线, trade_date = 0 表示获取实时分钟线, freq='1M' 表示1分钟线
df, msg = api.bar(symbol=symbol, trade_date=0, freq='1M', start_time=90000, end_time=150000)
print(df)
print(msg)

# 获取日线数据
df, msg = api.daily(
                symbol="600832.SH, 600030.SH", 
                start_date="2012-10-26",
                end_date="2012-11-30", 
                fields="", 
                adjust_mode="post")

# 获取证券信息
df, msg = api.query(
                view="lb.instrumentInfo",
                fields="status,list_date, fullname_en, market",
                filter="inst_type=&status=1&symbol=",
                data_format='pandas')
```

**在线数据服务有一个限制，不提供行情订阅推送功能，只能查询。**

最全的功能文档，请参考：[http://tushare.org/pro/index.html](http://tushare.org/pro/index.html)

对于一般普通用户，如果上述数据能满足你的要求，我们推荐你使用这种方案。

如果您拥有自己的数据源，且有数据系统本地化部署的要求，可以使用DataCore开源项目，搭建本地数据系统。

## 方案2：使用DataCore搭建本地数据系统

DataCore是一款企业级开源量化数据系统，通过标准化接口提供高速实时行情、历史行情和参考数据等核心服务，覆盖股票、商品期货、股指期货、国债期货等品种，适配CTP、万得、聚源、Tushare等各类数据。

搭建本地数据系统，需要有一些前提条件：

* 用于提供行情的实时行情源
* 历史市场数据（如分钟线）
* 参考数据库（如万得、聚源等厂商提供的参考数据库）

### 一、构建本地行情系统

DataCore本地行情系统的逻辑架构如下：

![](/assets/datacore_architect.png)

整个行情系统由几个部分构成：

* mdlink系列，用于行情源接口转换，包括
  * mdlink\_ctp：CTP期货行情接口
  * mdlink\_tdf：万得宏汇行情接口
  * mdlink\_sina：新浪股票行情接口
  * mdlink\_merge：把多个行情源聚合成统一的行情服务，提供全标的广播接口
* qms（query market data service）：用于生成实时分钟线，提供实时行情查询
* dataserver：综合数据服务器，提供集成的数据服务，包括实时行情按照代码订阅推送，实时行情查询。

构建工作分如下几步：

1、准备工作，包括：

* 下载可执行程序。下载地址如下：[http://www.quantos.org/datacore/download.html](http://www.quantos.org/datacore/download.html)
* 准备相应的行情账号。
  * 期货账号，可以使用SimNow的账号，请从SimNow官网注册。[http://www.simnow.com.cn/static/register1.action](http://www.simnow.com.cn/static/register1.action)，特别注意，SimNow用手机注册后，会分配一个InvestorID，请登录查看。
  * 万得宏汇行情接口，这个你只能找万得买了。如果你搞不到万得宏汇的行情账户，可以使用新浪的股票行情接口，不过质量就要你自己评估了。

2、部署mdlink和qms。部署过程请参考：[http://www.quantos.org/datacore/doc.html](http://www.quantos.org/datacore/doc.html)

这里务必要注意几点：

**（1）需要自行修改程序里面的账号配置，同一种标的，只能提供一个源。**

**（2）注意不同程序之间的访问端口，按照文档的要求进行配置。**

3、部署dataserver。部署过程请参考：[http://www.quantos.org/datacore/doc.html](http://www.quantos.org/datacore/doc.html)

这里务必要注意几点：

**（1）dataserver基于Java开发，请安装Java8环境。**

**（2）dataserver需要连接qms和mdlink\_merge。**

4、使用DataApi，访问本地数据服务器，看看行情服务是否正常。实时行情相关的API是：

| API接口 | 含义 |
| :--- | :--- |
| quote | 获取当前行情切片 |
| bar | 获取分钟线 |
| subscribe | 订阅分钟线 |

```py
from data_api import DataApi

api = DataApi(addr="tcp://127.0.0.1:8910") # 根据本地实际情况修改

# api.login("username", "password")  #认证模块，需要用户修改代码

symbol = 'T1712.CFE, TF1712.CFE, rb1712.SHF'
fields = 'open,high,low,last,volume'

# 获取实时行情
df, msg = api.quote(symbol=symbol, fields=fields)
print(df)
print(msg)

# 获取实时分钟线, trade_date = 0 表示获取实时分钟线, freq='1M' 表示1分钟线
df, msg = api.bar(symbol=symbol, trade_date=0, freq='1M', start_time=90000, end_time=150000)
print(df)
print(msg)

# 订阅实时行情
# 行情回调函数，k可以忽略，v是一个字典,保存quote数据
def on_quote(k,v):
    print v['symbol'] // 标的代码
    print v['last'] // 最新成交价
    print v['time'] // 最新成交时间

subs_list,msg = api.subscribe(symbol=symbol, func=on_quote,fields=fields)
```

以上是代码示例。

### 二、构建本地历史市场数据系统

comming soon......

### 三、构建本地参考数据系统

comming soon......

