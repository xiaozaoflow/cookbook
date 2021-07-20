# 小程序端 绘本阅读



### 1: 业务流程

小程序端绘本分为活动绘本集，vip绘本集，免费绘本集等几种类型。需要根据用户的身份是不是会员，是不是登陆了来校验用户是不是可以看绘本集。如果是活动绘本集需要去user-picbook-collection这张表里查看用户是否拥有该绘本。**拥有**该绘本的逻辑是用户购买了这个绘本或者用户已经邀请了4位好友解锁了活动绘本。参考活动报名代码ActivityController.enroll中的报名事件。**拥有是最高优先级别的权限**，即便这个活动绘本集后面变成了vip而用户并没有充值变成vip，用户仍然是可以看这个绘本集的。



下面是代码逻辑：



####  page![](/assets/截屏2021-07-20 下午8.53.28.png)

#### get

![](/assets/截屏2021-07-20 下午8.54.33.png)

processOn链接：[https://www.processon.com/view/link/5f8573d55653bb06effb304c](https://www.processon.com/view/link/5f8573d55653bb06effb304c)



### 2: 数据库ER图

![](/assets/截屏2021-07-20 下午8.57.22.png)



