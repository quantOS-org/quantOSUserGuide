# DataView的formula相关

## `DataView`简介

`DataView`本质是一个**数据存储器**，一个研究可能只使用一个`DataView`，用以保证数据在使用过程中的**一致性**。同时，`DataView`也**封装**了增、删、改、查、I/O等操作，增加**便利性**。

## Formula相关

### 设计`add_formula`接口的原因

由数据库取到的数据构建而成的`DataView`，只含有数据提供商所提供的数据，而对这些数据进行清洗、进而构造各种各样的**衍生**数据、指标、信号是研究和回测中非常常用的。纯粹的存储器只提供数据出入的接口，如果需要添加衍生数据，需要取出原始数据，计算后再放回存储器中；而`DataView`还提供了`add_formula`接口，让用户可以通过**传入数学公式的方式，流程化、自动化地进行数据计算、对齐并添加到`DataView`中**。

`add_formula`的signature为`void add_formula(new_field_name, formula, is_quarterly)`。`new_field_name`为字符串，是给运算结果的命名；`is_quarterly`为bool值，表示公式运算结果是日频率还是季度频率（目前只支持这两种频率）`formula`是用字符串表示的数学公式。抽象来看`formula`具有以下形式：
$$
f( x_1, x_2, \dots, x_n)
$$
其中$f$是下方列表中支持的运算或函数，$x_i$为`DataView`中已经存在的字段名。

### formula运算时的几个默认操作

1. 指数成分：`DataView`中包含一段时间内universe的所有股票，但每天universe的股票是变化的，在进行一些横截面运算时（如排序），会先将当天非指数成分的数据置为NaN，再运算
2. 季度数据的自动展开和对齐：当不同频率的数据一起运算时，会将季度数据按照公布日期(ann_date)展开并对齐到交易日，再和日频数据运算。

### 公式支持的运算、函数说明

| 分类                | 说明                                       | 公式                              | 示例                                       |
| ----------------- | ---------------------------------------- | ------------------------------- | ---------------------------------------- |
| 四则运算              | 加法运算                                     | +                               | close + open                             |
| 四则运算              | 减法运算                                     | -                               | close - open                             |
| 四则运算              | 乘法运算                                     | *                               | vwap * volume                            |
| 四则运算              | 除法运算                                     | /                               | close / open                             |
| 基本数学函数            | 符号函数，返回值为{-1,   0, 1}                    | Sign(x)                         | Sign(close-open)                         |
| 基本数学函数            | 绝对值函数                                    | Abs(x)                          | Abs(close-open)                          |
| 基本数学函数            | 自然对数                                     | Log(x)                          | Log(close/open)                          |
| 基本数学函数            | 对x取负                                     | -x                              | -close                                   |
| 基本数学函数            | 幂函数                                      | ^                               | close ^ 2                                |
| 基本数学函数            | 幂函数x^y                                   | Pow(x,y)                        | Pow(close,2)                             |
| 基本数学函数            | 保持符号的幂函数，等价于Sign(x)   * (Abs(x)^e)       | SignedPower(x,e)                | SignedPower(close-open, 0.5)             |
| 基本数学函数            | 取余函数                                     | %                               | oi % 10                                  |
| 逻辑运算              | 判断是否相等                                   | ==                              | close == open                            |
| 逻辑运算              | 判断是否不等                                   | !=                              | close != open                            |
| 逻辑运算              | 大于                                       | >                               | close > open                             |
| 逻辑运算              | 小于                                       | <                               | close < open                             |
| 逻辑运算              | 大于等于                                     | >=                              | close >= open                            |
| 逻辑运算              | 小于等于                                     | <=                              | close <= open                            |
| 逻辑运算              | 逻辑与                                      | &&                              | (close > open) && (close > vwap)         |
| 逻辑运算              | 逻辑或                                      |                                 |                                          |
| 逻辑运算              | 逻辑非                                      | !                               | !(close>open)                            |
| 逻辑运算              | 判断值是否为NaN                                | IsNan(x)                        | IsNan(net_profit)                        |
| 三角函数              | 正弦函数                                     | Sin(x)                          | Sin(close/open)                          |
| 三角函数              | 余弦函数                                     | Cos(x)                          | Cos(close/open)                          |
| 三角函数              | 正切函数                                     | Tan(x)                          | Tan(close/open)                          |
| 三角函数              | 开平方函数                                    | Sqrt(x)                         | Sqrt(close^2 + open^2)                   |
| 取整函数              | 向上取整                                     | Ceil(x)                         | Ceil(high)                               |
| 取整函数              | 向下取整                                     | Floor(x)                        | Floor(low)                               |
| 取整函数              | 四舍五入                                     | Round(x)                        | Round（close）                             |
| 选择函数              | 取 x 和 y 同位置上的较大值组成新的DataFrame返回          | Max(x,y)                        | Max(close, open)                         |
| 选择函数              | 取   x 和 y 同位置上的较小值组成新的DataFrame返回        | Min(x,y)                        | Min(close,open)                          |
| 选择函数              | cond为True取x的值，反之取y的值                     | If(cond,x,y)                    | If(close > open, close,   open) 表示取open和close的较大值 |
| 时间序列函数 - 基本数学运算   | 指标n个周期前的值                                | Delay(x,n)                      | Delay(close,1) 表示前一天收盘价                  |
| 时间序列函数 -   基本数学运算 | 指标在过去n天的和                                | Ts_Sum(x,n)                     | Sum(volume,5) 表示一周成交量                    |
| 时间序列函数 - 基本数学运算   | 指标在过去   n 天的积                            | Ts_Product(x,n)                 | Product(close/Delay(close,1),5) - 1 表示过去5天累计收益 |
| 时间序列函数 -   基本数学运算 | 指标当前值与n天前的值的差                            | Delta(x,n)                      | Delta(close,5)                           |
| 时间序列函数 - 基本数学运算   | 计算指标相比n天前的变化率，默认计算百分比变化率；当log为1是，计算对数变化率 | Return(x,n,log)                 | Return(close,5,True)计算一周对数收益             |
| 时间序列函数 -   基本数学运算 | 计算指标在过去n天的平均值                            | Ts_Mean(x，n)                    | Ts_Mean(close,5)                         |
| 时间序列函数 - 统计       | 指标在过去n天的标准差                              | StdDev(x,n)                     | StdDev(close/Delay(close,1)-1, 10)       |
| 时间序列函数 - 统计       | 两个指标在过去n天的协方差                            | Covariance(x,y,n)               | Covariance(close, open,   10)            |
| 时间序列函数 - 统计       | 两个指标在过去n天的相关系数                           | Correlation(x,y,n)              | Correlation(close,open, 10)              |
| 时间序列函数 - 统计       | 计算指标在过去n天的最小值                            | Ts_Min(x，n)                     | Ts_Min(close，5)                          |
| 时间序列函数 - 统计       | 计算指标在过去n天的最大值                            | Ts_Max(x，n)                     | Ts_Max(close，5)                          |
| 时间序列函数 - 统计       | 计算指标在过去n天的偏度                             | Ts_Skewness(x，n)                | Ts_Skewness(close，20)                    |
| 时间序列函数 - 统计       | 计算指标在过去n天的峰度                             | Ts_Kurtosis(x，n)                | Ts_Kurtosis(close，20)                    |
| 时间序列函数 - 排名       | 计算指标在过去n天的排名，返回值为名次                      | Ts_Rank(x, n)                   | Ts_Rank(close, 5)                        |
| 时间序列函数 - 排名       | 计算指标在过去n天的百分比，返回值为[0.0,   1.0]           | Ts_Percentile(x,   n)           | Ts_Percentile(close, 5)                  |
| 时间序列函数 - 排名       | 计算指标在过去n天所属的quantile，返回值为表示quantile的整数   | Ts_Quantile(x, n)               | Ts_Quantile(close, 5)                    |
| 时间序列函数 - 排名       | 指数移动平均，以halflife的衰减对x进行指数移动平均            | Ewma(x,   halflife)             | Ewma(x, 3)                               |
| 横截面函数 - 排名        | 将指标值在横截面方向排名，返回值为名次                      | Rank(x)                         | Rank(   close/Delay(close,1)-1 ) 表示按日收益率进行排名 |
| 横截面函数 - 排名        | 按分组数据g在每组内将指标值在横截面方向排名，返回值为名次            | GroupRank(x,g)                  | GroupRank(close/Delay(close,1)-1, g)   表示按分组g根据日收益率进行分组排名 |
| 横截面函数 - 排名        | 将指标值在横截面方向排名，返回值为排名百分比                   | Percentile(x)                   | Percentile(close, sw1)   按申万1级行业         |
| 横截面函数 - 排名        | 按分组数据g在每组内将指标值在横截面方向排名，返回值为排名百分比         | GroupPercentile(x,   g, n)      |                                          |
| 横截面函数 - 排名        | 和Rank函数相同，但只有 cond 中值为True的标的参与排名        | ConditionRank(x, cond)          | GroupRank(close/Delay(close,1)-1,   cond) 表示按条件cond根据日收益率进行分组排名 |
| 横截面函数 - 排名        | 根据指标值在横截面方向将标的分成n个quantile，返回值为所属quantile | Quantile(x,   n)                | Quantile( close/Delay(close,1)-1,5)表示按日收益率分为5档 |
| 横截面函数 - 排名        | 按分组数据g在每组内根据指标值在横截面方向将标的分成n个quantile，返回值为所属quantile | GroupQuantile(x, g, n)          | GroupQuantile(close/Delay(close,1)-1,g,5)   表示按日收益率和分组g进行分档，每组分为5档 |
| 横截面函数 - 数据处理      | 将指标标准化，即在横截面上减去平均值后再除以标准差                | Standardize(x)                  | Standardize(close/Delay(close,1)-1) 表示日收益率的标准化 |
| 横截面函数 - 数据处理      | 将指标横截面上去极值，用MAD (Maximum Absolute   Deviation)方法, z_score为极值判断标准 | Cutoff(x, z_score)              | Cutoff(close,3)   表示去掉z_score大于3的极值      |
| 财报函数              | 将累计财务数据转换为单季财务数据                         | CumToSingle(x)                  | CumToSingle(net_profit)                  |
| 财报函数              | 从累计财务数据计算TTM的财务数据                        | TTM(x)                          | TTM(net_profit)                          |
| 其他                | 过去 n 天的指数衰减函数，其中 f 是平滑因子。这里 f 是平滑因子，可以赋一个小于   1 的值。Decay_exp(x, f, n) = (x[date] + x[date - 1] * f + … +x[date – n - 1] *   (f  (n – 1))) / (1 + f + … + f ^ (n -   1)) | Decay_exp(x,f,n)                | Decay_exp(close,0.9,10)                  |
| 其他                | 过去n天的线性衰减函数。Decay_linear(x, n) =   (x[date] * n + x[date - 1] * (n - 1) + … + x[date – n - 1]) / (n + (n - 1) +   … + 1) | Decay_linear(x,n)               | Decay_linear(close,15)                   |
| 其他                | 如果   x 的值介于 lower 和 upper，则将其设定为 newval  | Tail(x,   lower, upper, newval) | Tail(close/open, 0.99, 1.01, 1.0)        |
| 其他                | Step(n) 为每个标的创建一个向量，向量中 n 代表最新日期，n-1   代表前一天，以此类推。 | Step(n)                         | Step(30)                                 |
| 其他                | 时间序列函数，计算   x 中的值在过去 n 天中为 nan （非数字）的次数  | CountNans(x,n)                  | CountNans((close-open)^0.5, 10)   表示过去10天内有几天close小于open |

***注：***

- 时间序列函数计算结果由该指标过去n个值计算得来
- 横截面函数由同一天所有标的的同一个指标值计算得来

