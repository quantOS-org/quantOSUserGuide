# quantOS用户登录指南

作者：PKUJohnson


如果需要使用quantOS线上资源（数据、仿真交易、论坛等），您需要注册成为quantOS用户。

## 用户注册

+ quantOS用户注册入口是：[http://www.quantos.org/cas/register.html](http://www.quantos.org/cas/register.html)
+ 注册成功后，请妥善保存用户名和密码。

## 令牌签发和使用

用户注册成功后，系统会为每个用户签发一个令牌，用于DataApi和TradeApi的登录。

![](https://github.com/quantOS-org/quantOSUserGuide/blob/master/assets/token.png?raw=true)


**用户可使用令牌登录访问在线数据系统**
```python
from jaqs.data import DataApi
api = DataApi(addr="tcp://data.tushare.org:8910") 
api.login("手机号", "token")  
```

**用户可使用令牌登录访问仿真交易系统**
```python
from jaqs.trade import TradeApi
tapi = TradeApi(addr="tcp://gw.quantos.org:8901") 
user_info, msg = tapi.login("手机号", "token")     
```

**用户可登录网站后，主动更新令牌**

## quantOS安全机制

+ 用户访问网站www.quantOS.org使用https加密机制
+ 用户通过签发令牌，访问线上资源，不传输密码
+ 用户可主动定期更新令牌
+ 论坛不显示用户手机信息。
+ 用户需要妥善保存自己的密码信息。
