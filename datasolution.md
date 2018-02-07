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

api = DataApi(addr="tcp://data.quantos.org:8910") # 在线数据源

api.login("手机号", "token")  #认证模块，需要修改成www.quantos.org的注册用户

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
                view="jz.instrumentInfo",
                fields="status,list_date, fullname_en, market",
                filter="inst_type=1",
                data_format='pandas')
```

**在线数据服务有一个限制，不提供行情订阅推送功能，只能查询。**

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

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/datacore_architect.png?raw=true)

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
from data_api import DataApi # data_api为存放DataApi的文件夹名

api = DataApi(addr="tcp://127.0.0.1:8910") # 根据本地实际情况修改

# api.login("手机号", "token")  

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

DataCore 1.2 版本开始支持本地历史市场数据查询，包括历史Tick查询和历史分钟线查询。mdlink/qms可以保存收到的Tick数据，利用这些数据可以增量构建本地历史市场数据。如果需要存量历史市场数据，则需要自己准备相应数据，并转为指定格式的h5文件。DataCore的本地历史市场数据架构如下所示：
![](https://github.com/PKUJohnson/LearnJaqsByExample/blob/master/image/case5-5.png)

#### 数据文件准备

一种类型的数据每个市场每个交易日各存为一个h5文件，目前支持tk(Tick)、1M(1分钟线)、5M(5分钟线)、15M(15分钟线)四种类型数据，例如SHF市场20171225交易日的数据可存为如下文件：

```
/data/SHF/2017/SHF20171214-tk.H5
/data/SHF/2017/SHF20171214-1M.H5
/data/SHF/2017/SHF20171214-5M.H5
/data/SHF/2017/SHF20171214-15M.H5
```

使用pandas的HDFStore进行h5文件存储，h5文件的数据存储格式为：每个标的存为一个group，该group中存放DataFrame格式的数据。例如：
```
SHF20171214-1m
	| cu1801.SHF (DataFrame格式1分钟线)
	| cu1802.SHF (DataFrame格式1分钟线)
	| au1801.SHF (DataFrame格式1分钟线)
	| au1802.SHF (DataFrame格式1分钟线)
```

HDFStore使用方法如下所示：
```python
import pandas as pd
# 获取数据代码省略
store = pd.HDFStore(filename, "a")
store['600030.SH'] = df1
store['000001.SH'] = df2
store.close()
```

##### Tick文件数据格式

Tick文件的文件名格式为"@MKT@YYYYMMDD-tk.h5",其中@MKT表示市场名，@YYYYMMDD表示交易日。

Tick数据每个标的的DataFrame包含的字段及其类型如下所示：

|field        |type      |value   |   
|-------------|----------|--------|   
|last         |int64     |* 10000 |
|vwap         |int64     |* 10000 |
|open         |int64     |* 10000 |
|high         |int64     |* 10000 |
|low          |int64     |* 10000 |
|close        |int64     |* 10000 |
|settle       |int64     |* 10000 |
|iopv         |int64     |* 10000 |
|limit_up     |int64     |* 10000 |
|limit_down   |int64     |* 10000 |
|preclose     |int64     |* 10000 |
|presettle    |int64     |* 10000 |
|oi           |int64     |        |
|volume       |int64     |        |
|turnover     |int64     |        |
|date         |int64     |        |
|trade_date   |int64     |        |
|time         |int64     |        |
|preoi        |int64     |        |
|index        |datetime  |        |
|askprice1    |int64     |* 10000 |
|askprice2    |int64     |* 10000 |
|askprice3    |int64     |* 10000 |
|askprice4    |int64     |* 10000 |
|askprice5    |int64     |* 10000 |
|bidprice1    |int64     |* 10000 |
|bidprice2    |int64     |* 10000 |
|bidprice3    |int64     |* 10000 |
|bidprice4    |int64     |* 10000 |
|bidprice5    |int64     |* 10000 |
|askvolume1   |int64     |        |
|askvolume2   |int64     |        |
|askvolume3   |int64     |        |
|askvolume4   |int64     |        |
|askvolume5   |int64     |        |
|bidvolume1   |int64     |        |
|bidvolume2   |int64     |        |
|bidvolume3   |int64     |        |
|bidvolume4   |int64     |        |
|bidvolume5   |int64     |        |

其中value栏"*10000"表示需要将原值乘10000以后存转为int64类型，简单来讲，所有价格相关的字段都需要乘10000然后转int64。

##### 分钟线文件数据格式

1分钟线文件的文件名格式为"@MKT@YYYYMMDD-1M.h5"，5分钟线文件的文件名格式为"@MKT@YYYYMMDD-5M.h5"，15分钟线文件的文件名格式为"@MKT@YYYYMMDD-15M.h5"，@MKT表示市场名，@YYYYMMDD表示交易日。

分钟线数据每个标的的DataFrame包含的字段及其类型如下所示：

|field          |type      |value   | 
|---------------|----------|--------|
|open           |int64     |* 10000 |
|high           |int64     |* 10000 |
|low            |int64     |* 10000 |
|close          |int64     |* 10000 |
|settle         |int64     |* 10000 |
|oi             |int64     |        |
|volume         |int64     |        |
|turnover       |int64     |        |
|total_volume   |int64     |        |
|total_turnover |int64     |        |
|date           |int64     |        |
|trade_date     |int64     |        |
|time           |int64     |        |
|index          |datetime  |        |
|askprice1      |int64     |* 10000 |
|askprice2      |int64     |* 10000 |
|askprice3      |int64     |* 10000 |
|askprice4      |int64     |* 10000 |
|askprice5      |int64     |* 10000 |
|bidprice1      |int64     |* 10000 |
|bidprice2      |int64     |* 10000 |
|bidprice3      |int64     |* 10000 |
|bidprice4      |int64     |* 10000 |
|bidprice5      |int64     |* 10000 |
|askvolume1     |int64     |        |
|askvolume2     |int64     |        |
|askvolume3     |int64     |        |
|askvolume4     |int64     |        |
|askvolume5     |int64     |        |
|bidvolume1     |int64     |        |
|bidvolume2     |int64     |        |
|bidvolume3     |int64     |        |
|bidvolume4     |int64     |        |
|bidvolume5     |int64     |        |

##### 利用QMS保存的tk文件生成数据文件

目前运行mdlink和qms系统后，会在data/tk文件夹下按市场保存收到的tick数据，如下所示：
```
  617134798 Dec 14 15:20 SHF20171214.tk
   62738353 Dec 14 16:34 CFE20171214.tk
  391967678 Dec 14 15:00 CZC20171214.tk
  424926986 Dec 14 15:03 DCE20171214.tk
 1608054336 Dec 14 16:39 SH20171214.tk
 2395156236 Dec 14 16:39 SZ20171214.tk
```

我们提供了python版的tkreader工具来读取这些二进制tk文件，并将其转换为h5格式的Tick文件和分钟线文件。

使用tkreader读取示例如下：
```python
from tkreader import TkReader
user   = "phone number"
passwd = "your token"
tkreader = TkReader() 
file_name = "SHF20171218.tk"
start_time = "21:00:00"
end_time   = "15:00:00"

if( not tkreader.login(user, passwd)):
    print("login failed")
    return False

if (not tkreader.open(file_name,"", start_time, end_time)):
    print("can't open file %s " %file_name)
    return False
tk = tkreader.get_next()
while tk is not None:
    print(tk['symbol'])
    print(tk['date'])
    print(tk['time'])
    print(tk['last'])
```
读出的Tick数据字段参考DataApi使用文档的quote字段。

将tk文件转为h5格式Tick数据可以使用tkreader.py文件中的tk_to_h5函数来完成，使用示例在tkreader.py文件的main函数下。


将tk文件转为h5格式分钟线数据可以使用tk2bar.py中的tk2bar类来完成，使用示例在tk2bar.py的main函数下。

#### DataServer配置数据文件路径
修改etc/dataserver-dev.conf文件下的his_bar项，如下所示：
```
"his_bar" : {
    "bar1m_path"  : "D:/store/data/raw_data/BarONE/@MKT/@YYYY/@MKT@YYYYMMDD-1M.h5",
    "bar5m_path"  : "D:/store/data/raw_data/BarFIVE/@MKT/@YYYY/@MKT@YYYYMMDD-5M.h5",
    "bar15m_path" : "D:/store/data/raw_data/BarQUARTER/@MKT/@YYYY/@MKT@YYYYMMDD-15M.h5",
    "tick_path"   : "D:/store/data/raw_data/Tick/@MKT/@YYYY/@MKT@YYYYMMDD-tk.h5"
}
```
将数据文件的存放路径配置到his_bar里面：1分钟线数据文件路径配置到bar1m_path，
5分钟线数据文件路径配置到bar5m_path，15分钟线数据文件路径配置到bar15m_path，Tick数据文件路径配置到tick_path。

注意：@MKT、@YYYY和@YYYYMMDD为程序可识别的通配符，@MKT表示市场名，@YYYY表示年份，@YYYYMMDD表示日期。文件名中的通配符必须保留，文件名格式固定为@MKT@YYYYMMDD-XX.h5，不可修改。文件路径中的通配符可去掉，如：
```
"his_bar" : {
    "bar1m_path"  : "D:/store/data/raw_data/@MKT@YYYYMMDD-1M.h5",
    "bar5m_path"  : "D:/store/data/raw_data/@MKT@YYYYMMDD-5M.h5",
    "bar15m_path" : "D:/store/data/raw_data/@MKT@YYYYMMDD-15M.h5",
    "tick_path"   : "D:/store/data/raw_data/@MKT@YYYYMMDD-tk.h5"
}
```
配置好后本地运行DataServer，使用DataApi连接本地DataServer就可以获取分钟线数据和Tick数据了。


### 三、构建本地参考数据系统

comming soon......

