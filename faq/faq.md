# 常见问题

1. 实盘支持哪些券商、通道？

	答：quantOS系统使用vn.py进行实盘交易。vn.py已经实现了与国内各大主流交易系统的对接，可满足个人用户的单帐户交易要求。JAQS和vn.py之间通过TradeApi进行标准化对接，可做到仿真与实盘交易无缝对接。具体列表请见vnpy的gitbuh页面https://github.com/vnpy/vnpy。

2. quantOS的软件在哪里下载？

	答：可以从github下载，地址为https://github.com/quantOS-org。

3. username怎么填？DataApi和TradeApi中的用户名密码和登录网站的用户名密码是什么关系？

	答：两套用户名和密码相同。原来使用用户名和密码登录DataApi和TradeApi，改成使用token登录。

4. python-snappy的安装，为什么用pip就会报错？

	答：请见官网JAQS安装指南 https://www.quantos.org/jaqs/doc.html

5. 能否支持使用TA-lib？

	答：支持。

6. 若在国外没有中国手机如何注册？
	答：现在只支持中国大陆手机号注册。
7. 哪里可以找到策略模型的示例？
	答：策略模型样例请见jaqs-example文件夹，包括alpha策略，事件驱动策略。
8. 如何处理编码问题？
	答：在程序头部加入一行 # encoding: utf-8。
9. 如何修改data_config和trade_config的密码配置？
	答：请见quantOS用户登录指南，https://www.quantos.org/courses/index.html
10. 实时的tick数据如何获取？是否提供？
	答：quantOS系统不提供tick数据。
11. 有没有将几大组件拼在一起的案例（包括tradesim仿真交易）
	答：请见quantOSUserGuide中“两个量化交易策略样例”一节，地址为https://github.com/quantOS-org/quantOSUserGuide。
12. 目前提供的数据接口有那些？支持那些指数？
	答：请见数据API的介绍文档，地址为https://www.quantos.org/jaqs/doc.html
13. 有么有专门针对商品期货的交易策略样例（包括Universe）？
	答：请见jaqs-eventdriven中的CalendarSpread.py，DoubleMA.py，DualThrust.py策略。
14. tusharepro是否收费？
	答：不收费
15. 回测结果的统计指标哪里看？都有那些？
	答：回测的统计指标包括年化收益率，年化波动率，beta值和Sharpe ratio，在策略相应的report.html中可以看到。
16. TradeSim是否支持期权模拟？如何使用？
	答：支持。
17. 外盘交易支持吗？
	答：具体列表请见vnpy的gitbuh页面https://github.com/vnpy/vnpy。