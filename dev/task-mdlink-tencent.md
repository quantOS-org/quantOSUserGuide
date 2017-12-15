
# quantOS开源社区开放式开发任务书 mdlink-tencent

## 需求概览

DataCore是一款企业级开源量化数据系统，通过标准化接口提供高速实时行情、历史行情和参考数据等核心服务，覆盖股票、商品期货、股指期货、国债期货等品种，适配CTP、万得、聚源、Tushare等各类数据。

DataCore的内部逻辑架构如下：

![](/assets/datacore_architect.png)

这个架构可支持用户根据自己的实际情况，对接自己的行情源。

考虑到互联网广大用户获取行情的复杂度，我们拟提供免费的解决方案，即通过对互联网免费行情源进行封装，为用户提供实时行情服务。

目前互联网上两个免费的股票行情源是：

- 新浪的股票数据接口：http://hq.sinajs.cn/list=股票代码，如：sh600000，sz000913 (这里sh是上海，sz为深圳)，参看http://finalbone.iteye.com/blog/424608
- 腾讯的股票数据接口：http://qt.gtimg.cn/q=sz000858，参看http://blog.csdn.net/robertsong2004/article/details/46340621

新浪行情源我们已经对接好了，还需要开发一个腾讯行情源 **mdlink-tencent** ，可以参考mdlink-sina的代码，进行适当的修改。

mdlink-sina的地址如下：

[https://github.com/quantOS-org/DataCore/tree/master/mdlink/src/mdlink/mdlink\_sina](https://github.com/quantOS-org/DataCore/tree/master/mdlink/src/mdlink/mdlink_sina)

## 接口规范

这里我们讲解一下mdlink-sina的接口规范，在开发mdlink-tencent时，需要遵循同样的规范。

![](/assets/mdlink-sina.png)

规范1：由于新浪提供的方式是行情查询，mdlink-sina需要提供全市场的股票行情，因此需要不断查询新浪的行情数据，并且保存在自己的内存中。查询频率需要参数化配置。

规范2：mdlink-sina对外提供服务的方式是zmq+protobuf，是一种广播模式，任何客户端连上mdlink-sina后，都会接受到其publish的所有行情。publish的行情报文是protobuf格式，具体请参看代码和protobuf报文定义，地址如下：

[https://github.com/quantOS-org/DataCore/blob/master/mdlink/src/protocol/md.proto](https://github.com/quantOS-org/DataCore/blob/master/mdlink/src/protocol/md.proto)

规范3：项目采用C++开发，cmake负责编译，编译文件配置是：

[https://github.com/quantOS-org/DataCore/blob/master/mdlink/src/mdlink/mdlink\_sina/CMakeLists.txt](https://github.com/quantOS-org/DataCore/blob/master/mdlink/src/mdlink/mdlink_sina/CMakeLists.txt)

新项目需要按照相同的方式处理，同时在linux和windows平台上编译成功才行。

## 如何开发 **mdlink-tencent**

整个开发建议分成三步：

1、分析一下 [新浪接口](http://qt.gtimg.cn/q=股票代码) 和 [腾讯接口](http://hq.sinajs.cn/list=股票代码) 之间的报文区别和对应关系，确定好报文结果的解析方法。

2、拷贝一份mdlink-sina，对文件名进行适当修改，在此基础上加入腾讯行情源的处理逻辑。

3、编译通过后，测试行情发布的正确性。

