
# quantOS开源社区开放式开发任务书 tick2bar

## 需求概览

在目前的DataCore工程中，已经完美支持了实时行情的接收和处理，并由qms将实时行情转换成分钟线，供用户使用。

qms同时记录和保存了所有的tick数据，位于data/tk目录下，按照市场分开存储，如下：

```
  617134798 Dec 14 15:20 SHF20171214.tk
   62738353 Dec 14 16:34 CFE20171214.tk
  391967678 Dec 14 15:00 CZC20171214.tk
  424926986 Dec 14 15:03 DCE20171214.tk
 1608054336 Dec 14 16:39 SH20171214.tk
 2395156236 Dec 14 16:39 SZ20171214.tk
```

目前遗留的问题是，用户无法通过DataCore获取历史分钟数据，历史tick。

我们提供的解决方案是:

+ 从每天存储的tick文件读取出所有的tick，转存成HDF5. 转换工具从这里[下载](https://www.quantos.org/datacore/download.html)
+ 从tick.H5文件合并分钟线，并保存成Bar.H5文件。
+ DataCore会发布一个模块，支持从HDF5中读取数据，提供历史tick，分钟线给用户使用。

**_划重点_**：

**本次的开发任务是，从tick.H5文件合并分钟线，并保存成Bar.H5文件**

## 开发要求

1. 本任务要求使用python开发。

2. 首先使用我们提供的TkReader，将SHF20171214.tk转换成SHF20171214.H5（HDF5格式，按标的存储，每个标的是一个DataFrame）

3. 通过python将SHF20171214.H5转换成分钟线（1m，5m，15m），存储成SHF20171214-1m.H5，SHF20171214-5m.H5，SHF20171214-15m.H5.

4. 分钟线计算的几个注意事项：
+ (1) 分钟线的open, high, low, close, volume, turnover指这个时间段的统计数据。
+ (2) 如果某个时间段内没有任何tick，需要用前一分钟的close补齐, volume和turnover为0。
+ (3) 1分钟bar的时间点标注是93100，93200，93300，。。。，113000，130100，130200，130300，。。。，150000.
+ (4) 其他分钟的时间点，规则参考1分钟。
+ (5) 商品期货国内有夜盘，夜盘分钟线也需要合成，商品期货夜盘时间统一为21:00-2:30，不到2：30就收盘的品种统一补齐到2：30。
+ (6) 分钟线里面的ask/bid信息，记录本时段内最后一笔tick的ask/bid信息。如果本时段没有任何tick，则全部为0.
+ (7) total_turnover和total_volume，记录截止该时段结束前该标的日内成交总量和成交总额，即该时段结束前收到的最后一笔tick的turnover和volume值。
+ (8) settle和oi同total_turnover和total_volume类似，取时段结束前收到的最后一笔tick的settle和oi值
+ (9) 中金所股指期货交易时间段为9:30-11:30, 13:00-15:00; 国债期货交易时间为9:15-11:30, 13:00-15:15
+ (10) 其他市场交易时间段参考mdlink/etc下的Market.csv文件
+ (11) 生成h5文件后可以用HDFView打开文件查看数据结构等

5. 增加偏移量选项(offset)，单位为秒。生成分钟线的时间段做对应偏移，如offset=3, 则9:00:03-9:01:03之间的数据生成9:31:00的分钟线。

6. 转出的HDF5数据，保存目录结构如下：
```
/data/SHF/2017/SHF20171214-tk.H5
/data/SHF/2017/SHF20171214-1M.H5
/data/SHF/2017/SHF20171214-5M.H5
/data/SHF/2017/SHF20171214-15M.H5
```
这里的SHF是上期所，其他还包括SH，SZ，CFE，DCE，CZC

7. HDF5文件结构
```
SHF20171214-1m
	| cu1801.SHF (DataFrame保存分钟线)
	| cu1802.SHF (DataFrame保存分钟线)
	| au1801.SHF (DataFrame保存分钟线)
	| au1802.SHF (DataFrame保存分钟线)
```

tick数据每个标的的DataFrame包含的字段及其类型包括：

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


根据tick数据合成bar数据，字段如下：


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


## 招募开发达人

要求：

+ 有热情，志愿从事开源开发
+ 精通python，熟悉Pandas
+ 如果有一点关于K线的知识就更好了

联系方式：

在微信、QQ群@群主，要求加入开发即可。

参与开发的志愿者，将获得更多更深入的理论指导，并获得广泛的署名权。
