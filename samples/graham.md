### 格雷厄姆选股策略

主要介绍基于回测框架实现格雷厄姆模型。格雷厄姆模型分为两步，首先是条件选股，其次按照市值从小到大排序，选出排名前五的股票。
可在[这里](https://github.com/quantOS-org/JAQS/blob/master/example/alpha/Graham.py)直接下载策略源代码。
#### 一. 数据准备

我们选择如下指标，对全市场的股票进行筛选，实现过程如下：
a. 首先在数据准备模块save_dataview()中通过props设置数据起止日期，股票版块，以及所需变量
```python
props = {
'start_date': 20150101,
'end_date': 20170930,
'universe':'000905.SH',
'fields': ('tot_cur_assets,tot_cur_liab,inventories,pre_pay,deferred_exp, eps_basic,ebit,pe,pb,float_mv,sw1'),
'freq': 1
}
```
b. 接着创建0-1变量表示某只股票是否被选中，并通过add_formula将变量添加到dataview中
> * 市盈率（pe ratio）低于 20
> * 市净率（pb ratio）低于 2
> * 同比每股收益增长率（inc_earning_per_share）大于 0
> * 税前同比利润增长率（inc_profit_before_tax）大于 0
> * 流动比率（current_ratio）大于 2
> * 速动比率（quick_ratio）大于 1

```python
factor_formula = 'pe < 20'
dv.add_formula('pe_condition', factor_formula, is_quarterly=False)
factor_formula = 'pb < 2'
dv.add_formula('pb_condition', factor_formula, is_quarterly=False)
factor_formula = 'Return(eps_basic, 4) > 0'
dv.add_formula('eps_condition', factor_formula, is_quarterly=True)
factor_formula = 'Return(ebit, 4) > 0'
dv.add_formula('ebit_condition', factor_formula, is_quarterly=True)
factor_formula = 'tot_cur_assets/tot_cur_liab > 2'
dv.add_formula('current_condition', factor_formula, is_quarterly=True)
factor_formula = '(tot_cur_assets - inventories - pre_pay - deferred_exp)/tot_cur_liab > 1'
dv.add_formula('quick_condition', factor_formula, is_quarterly=True)
```
需要注意的是，涉及到的财务数据若不在secDailyIndicator表中，需将is_quarterly设置为True，表示该变量为季度数据。
c. 由于第二步中需要按流通市值排序，我们将这一变量也放入dataview中
```python
dv.add_formula('mv_rank', 'Rank(float_mv)', is_quarterly=False)
```

#### 二. 条件选股

条件选股在my_selector函数中完成：
> * 首先我们将上一步计算出的0/1变量提取出来，格式为Series
> * 接着我们对所有变量取交集，选中的股票设为1，未选中的设为0，并将结果通过DataFrame形式返回
```python
def my_selector(context, user_options=None):
    #
    pb_selector      = context.snapshot['pb_condition']
    pe_selector      = context.snapshot['pe_condition']
    eps_selector     = context.snapshot['eps_condition']
    ebit_selector    = context.snapshot['ebit_condition']
    current_selector = context.snapshot['current_condition']
    quick_selector   = context.snapshot['quick_condition']
    #
    merge = pd.concat([pb_selector, pe_selector, eps_selector,     ebit_selector, current_selector, quick_selector], axis=1)

    result = np.all(merge, axis=1)
    mask = np.all(merge.isnull().values, axis=1)
    result[mask] = False
    return pd.DataFrame(result, index=merge.index, columns=['selector'])
```

#### 三、按市值排序

按市值排序功能在signal_size函数中完成。我们根据流通市值排序变量'mv_rank'对所有股票进行排序，并选出市值最小的5只股票。
```python
def signal_size(context, user_options = None):
    mv_rank = context.snapshot_sub['mv_rank']
    s = np.sort(mv_rank.values)[::-1]
    if len(s) > 0:
        critical = s[-5] if len(s) > 5 else np.min(s)
        mask = mv_rank < critical
        mv_rank[mask] = 0.0
        mv_rank[~mask] = 1.0
    return mv_rank
```

#### 四、回测

我们在test_alpha_strategy_dataview()模块中实现回测功能

##### 1. 载入dataview，设置回测参数
该模块首先载入dataview并允许用户设置回测参数，比如基准指数，起止日期，换仓周期等。
```python
dv = DataView()

fullpath = fileio.join_relative_path('../output/prepared', dv_subfolder_name)
dv.load_dataview(folder=fullpath)

props = {
    "benchmark": "000905.SH",
    "universe": ','.join(dv.symbol),

    "start_date": dv.start_date,
    "end_date": dv.end_date,

    "period": "week",
    "days_delay": 0,

    "init_balance": 1e8,
    "position_ratio": 1.0,
}
```

##### 2. StockSelector选股模块
接着我们使用StockSelector选股模块，将之前定义的my_selector载入
```python
stock_selector = model.StockSelector(context)
stock_selector.add_filter(name='myselector', func=my_selector)
```

##### 3. FactorRevenueModel模块
在进行条件选股后，使用FactorRevenueModel模块对所选股票进行排序
```python
signal_model = model.FactorRevenueModel(context)
signal_model.add_signal(name='signalsize', func = signal_size)
```

##### 4. 策略回测模块
将上面定义的stockSelector和FactorRevenueModel载入AlphaStrategy函数进行回测
```python
    strategy = AlphaStrategy(
                stock_selector=stock_selector,
                revenue_model=signal_model，
                pc_method='factor_value_weight')
```

##### 5. 启动数据准备及回测模块
```python
t_start = time.time()

test_save_dataview()
test_alpha_strategy_dataview()
test_backtest_analyze()

t3 = time.time() - t_start
print "\n\n\nTime lapsed in total: {:.1f}".format(t3)
```

#### 五、回测结果

回测的参数如下：

| 指标             | 值   |
| --------         | --:  |
| Beta             | 0.87 |
| Annual Return    | 0.08 |
| Annual Volatility| 0.29 |
| Sharpe Ratio     | 0.28 |

回测的净值曲线图如下：

![backtestgraham](https://raw.githubusercontent.com/quantOS-org/jaqs/master/doc/img/backtest_Graham_result.png?raw=true)
