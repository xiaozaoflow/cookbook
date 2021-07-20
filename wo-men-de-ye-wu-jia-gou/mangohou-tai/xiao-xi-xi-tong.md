## 消息系统

### 1： 业务流程

消息系统为了适应运营多变的需求，考虑了非常多灵活的特性。基本上每个环节都可以拆，但是这也带来了一些复杂度。代码本身并不复杂，但是链条比较长。定义了subscribeMsg公众号关注后消息,marketingMsg营销消息\(绘本并没有使用到\),satifyMsg满足一定条件后的主动投递消息。并且由于和拼音一起使用系统，代码还有很多拼音的部分可以考虑消除。

绘本的mq的逻辑在ActivityKFMessageMq中。

processOn链接：[https://www.processon.com/view/link/5f3b4f1e079129531b6148ff](https://www.processon.com/view/link/5f3b4f1e079129531b6148ff)

### 具体代码流程如下：![](/assets/100008_ 消息系统.png)

### 2: 数据库ER图

![](/assets/截屏2021-07-20 下午7.20.46.png)



