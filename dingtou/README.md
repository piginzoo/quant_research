# 项目由来

看了解读《基金定投》基金定投的方法！如何买基金，基金定投亏钱怎么办? ，对他说的定投可以赚钱的说法甚是喜欢。

就觉得 定投+网格交易法，一定是一个很靠谱的投资方法，是一个投资发财的好方法，于是就迫不及待地写了一个回测来证明一下。

然而，开始写了一些测试效果都不好，直到看到了[王纯迅](http://book.piginzoo.com/knowledge/period-invest/period-invest.html)
的分享视频，意识到一些问题，修改后，效果好了很多。

# 思路

2021年以来我家的定投赔了20%，虽然之前看过鼓吹定投的说法，也亲自实践了一把，结果还是这种成绩，很沮丧，目前，定投也已经停掉了。

但是，最近看了[解读《基金定投》基金定投的方法！](https://www.bilibili.com/video/BV1TA411x77e/),
对定投重试信心，于是开始写一个回测程序。

1.开始的时候，就是定投，每周定投一个固定金额，但是，结果是跑不过基金本身的。
` python -m dingtou.period.test_period -c 003095 -s 20180101 -e 20211201 -b sh000905 -p 200 -a 200000`

2.然后我改进了，用年线分割，年线下才买，而且是下跌趋势才买（用4周的周线k线做直线拟合，斜率是负的），而且是bi前一日更才买。
`python -m dingtou.period.test_timing -c 003095 -s 20180101 -e 20211201 -b sh000905 -p 45 -a 200000`
效果还是不行，而且，还得调优参数，因为买的次数不定了，所以得找到`-p 45`才能使得收益率最优，但是，还是不如基金本身。

3、然后，我继续改进，之前都是定投，没有获利了结、止损过，这次加入了10%的止盈和止损，继续跑。
`python -m dingtou.period.test_timing_optimize -c 003095 -s 20180101 -e 20211201 -b sh000905 -p 200 -a 5000`
结果效果更差了。

4、我还是不甘心，继续改进，采用了比3更科学的网格模型，就是越跌越买，越涨越卖，我采用了大家一般用的ATR作为1个网格的高度，
当超过均线几个ATR就卖更多，越涨卖越多；跌破均线几个ATR就买更多，越跌买的越多；
否则，就少买（事实上这里我犯了个错误，不是网格模型，因为，我没有记住上次的网格位置，但当时没意识到）。
结果效果依然是不好：
`python -m dingtou.grid.grid -c 003095 -s 20180101 -e 20211201 -b sh000905`

至此，我对网格、定投，都不是信息很大了。

直到看到了[王纯迅](http://book.piginzoo.com/knowledge/period-invest/period-invest.html)
的分享视频，意识到之前的一些问题，就又开始新的一轮改进：
- 记录了上次买入和卖出的点位
- 没用ATR，采用了80%分位数（其实，我觉得ATR效果应该差不多，将来试试）
- 听取了他的意见，不碰普通的股票基金了，只搞ETF基金
- 多只ETF基金（宽基、行业、题材）同时跑做组合投资
`python -m dingtou.pyramid.test_compose`

当然，也可以单独跑1只ETF基金：
`python -m dingtou.pyramid.pyramid -c 510500 -s 20180101 -e 20211201 -b sh000905`

恩，这次效果好很多了：
- 收益率超过ETF基金和大盘
- 很抗跌，即使是基金、大盘都跌，也不会损失太多
- 回撤小，这点我最看中

当然，还有一些不足：
- 单边行情容易错过，他挣的不是这种前，所以更适合波动的宽基ETF
- 还有诸多细节参数可以改进


# 运行

- 需要预先安装各类包：`pip install -r requirements.txt`
- 按照上述命令挨个运行


# 一些设计

做了一个小的回测框架，再backtest中，

核心类是BackTester，他会读数据中的所有日期，合并，排序后，拿这个当做一个迭代日期，然后开始回测迭代。

当然只有在start_date和end_date中的日期，才会回测，否则就忽略。

每天，都会先去调用一下策略Strategy类的next，这个类里会触发各种买卖信号，并产生交易单。

交易单，都是第二天的价格来成交的，基金也是，这个是参考的之前的股票的规则。

为了统一，把所有的日期都改名成date，价格都改名称close，且，都把日期当做了dataframe的索引。

不愿意用backtrader，主要是因为它太重，我自己写个灵巧的，自己够用就好。

运行完会得到一个console结果：

数据源用的是[AKShare](https://www.akshare.xyz/data/index/index.html)。

# 参考

[【知识精讲】网格交易详解与回测 | 优缺点及应对方法 | 胜率真能达到100%？](https://www.bilibili.com/video/BV1Yf4y177cA)

[据说稳赚的《网格交易法》，怎么用？是真的吗？](https://www.youtube.com/watch?v=Bn6mJjyxTwU&t=740s) 

[解读《基金定投》基金定投的方法！如何买基金，基金定投亏钱怎么办？_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1TA411x77e/) 

[网格交易法——数学模型+传统智慧战胜华尔街-学习视频教程-腾讯课堂](https://ke.qq.com/course/335450) 

[掘进量化的各种策略](https://www.myquant.cn/docs/python_strategyies/104)

[网格交易法：数学+传统智慧战胜华尔街](https://cread.jd.com/read/startRead.action?bookId=30150108&readType=1)

代码：

[网格交易法以及在数字货币中基于Python的量化实现](https://juejin.cn/post/6844903998793728007)