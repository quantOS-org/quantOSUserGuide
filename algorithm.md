# 如何进行算法交易

作者：vanvency, PKUJohnson

算法交易是专业投资者的必备工具，这里为大家介绍如何在TradeSim上进行算法交易。


## 使用TradeApi下单

TradeSim支持使用TradeApi进行算法交易。支持的算法包括VWAP，TWAP等

### 以VWAP为例，样例代码如下：

```py
from jaqs.trade.tradeapi import TradeApi

# 登录仿真系统
tapi = TradeApi(addr="tcp://gw.quantos.org:8901") # tcp://gw.quantos.org:8901是仿真系统地址
user_info, msg = tapi.login("demo", "666666")     # 示例账户，用户需要改为自己注册的账户

# 选择策略号
tapi.use_strategy(123) # 123为给你分配的策略号，可以从user_info中获得

# VWAP算法参数
params = {
    "participate_rate": {"600000.SH": 0.5},
    "urgency":          {"600000.SH": 5},
    "price_range":      {"600000.SH": [16.5,17.1]},
    "lifetime":         5000,
    "min_unit_size":    {"600000.SH": 500}
}

# 下单接口，供支持三种下单风格

# Single Order Style
task_id, msg = tapi.place_order("600000.SH", "Buy", 17, 100, "vwap", params)
print "msg:", msg
print "task_id:", task_id

# Batch Order Style
orders = [
    {"security":"600030.SH", "action" : "Buy", "price": 16.10,  "size":1000},
    {"security":"600868.SH", "action" : "Buy", "price": 5.89,   "size":1000},
]

params = {
    "participate_rate": {"600030.SH": 0.5, "600868.SH": 0.5},
    "urgency":          {"600030.SH": 5, "600868.SH": 5},
    "lifetime":         5000
}

task_id, msg = tapi.place_batch_order(orders, "vwap", params)
print "msg:", msg
print "task_id:", task_id

# Basket Order Style
orders = [
    {"security":"600030.SH", "ref_price": 16.10,  "inc_size":1000},
    {"security":"600868.SH", "ref_price": 5.89,   "inc_size":-1000},
]

params = {
    "participate_rate": {"600030.SH": 0.5, "600868.SH": 0.5},
    "urgency":          {"600030.SH": 5, "600868.SH": 5},
    "lifetime":         5000,
}

task_id, msg = tapi.basket_order(orders, "vwap", params)
print "msg:", msg
print "task_id:", task_id

```

### 以TWAP为例，样例代码如下：

```py
from tradeapi import TradeApi

# 登录仿真系统
tapi = TradeApi(addr="tcp://gw.quantos.org:8901") # tcp://gw.quantos.org:8901是仿真系统地址
user_info, msg = tapi.login("demo", "666666")     # 示例账户，用户需要改为自己注册的账户

# 选择策略号
tapi.use_strategy(123) # 123为给你分配的策略号，可以从user_info中获得

# 下单接口，供支持三种下单风格

# Single Order Style
params = {
    "urgency":          {"600868.SH": 5},
    "cycle":            1000,
    "max_unit_size":    {"600868.SH": 5},
    "lifetime":         5000，
    "price_range":      {"600868.SH": [5.8, 6.2]},
    "smart_speed":      [0.8,1.1,1.3]
}

task_id, msg = tapi.place_order("600868.SH", "Buy", 6.57, 100, "twap", params)

# Batch Order Style
orders = [
    {"security":"600030.SH", "action" : "Buy", "price": 16.10,  "size":1000},
    {"security":"600868.SH", "action" : "Buy", "price": 5.89,   "size":1000},
]

params = {
    "urgency":          {"600030.SH": 5, "600868.SH": 5},
    "cycle":            1000,
    "lifetime":         5000,
    "price_range_factor":0.1
}

task_id, msg = tapi.place_batch_order(orders, "twap", params)

```

查询、成交回报等使用方法，请参考TradeApi的使用说明。

