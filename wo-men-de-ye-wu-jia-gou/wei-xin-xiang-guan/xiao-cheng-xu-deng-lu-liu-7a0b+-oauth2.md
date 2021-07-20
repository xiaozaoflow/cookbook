## 小程序小程序登陆+OAuth2

### 1: 业务流程![](/assets/截屏2021-07-20 下午5.11.21.png)

processOn链接： [https://www.processon.com/view/link/607660887d9c08283dc41398](https://www.processon.com/view/link/607660887d9c08283dc41398)

微信用户登陆分为从来没有使用小程序的全新用户和已经登陆过的用户。后者会因为accessToken失效而重新登陆。对于这部分用户来说，后端需要入库的所有资料都已经入库，比如wxuser表中的小程序openid，flowUser中的信息资料。所以静默授权以后就可以直接登录了。而对于前者，全新用户需要通过小程序登陆授权保存小程序openid到后端数据库，所以要多经历一步入库的过程, 参考wxUserController.saveMaUser。对于前端来说，可以通过调用getUserByUnionid这个接口的判断一个用户是全新\[false\]还是已经登陆过\[true\]。

### 2: 数据库ER图

![](/assets/截屏2021-07-20 下午5.28.27.png)

### 3 代码细节 



> oauth

微信是基于oauth登陆。oauth请求代码如下，登陆需要的三个要素分别是公众平台的unionid，用于打通同一平台的公众号和小程序。还有公众号的openid+appid可以通过公众号静默授权获取。

```
curl --location --request POST 'http://flow.xiaozaoai.com/api/auth/oauth/token?unionId=oKUtLw254UHAui910x77zMGA1uLM&grant_type=flow&scope=flow&appId=wxf6ea2272a8e77b2f&openId=o9fv7jpEvoKClBOB4uAi827ziSS8' \
--header 'Authorization: Basic eGlhb3phb19mbG93OmRKSjBaNWI3Z2ZKSHpDUDRjNThK'
```



> 小程序登陆

小程序登陆文档 [https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html\#%E7%99%BB%E5%BD%95%E6%B5%81%E7%A8%8B%E6%97%B6%E5%BA%8F ](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html#%E7%99%BB%E5%BD%95%E6%B5%81%E7%A8%8B%E6%97%B6%E5%BA%8F )

后端调用顺序: 

*  WxUserController.getSessionResult \[存放codeSession相关信息\]

*  WxUserController.saveMaUser\[后端入库wxuser信息，主要是小程序授权信息\]



> 公众号静默授权

公众号网页授权文档[ https://developers.weixin.qq.com/doc/offiaccount/OA\_Web\_Apps/Wechat\_webpage\_authorization.html](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html "公众号网页授权文档")



下面是一个后端静默授权的具体流程

  **  api文档地址：**

* http://test-xiaozao-flow.genshuixue.com/api/doc.html\#/home

* 业务页面:  http://xiaozao-flow.genshuixue.com/m/flow\_landing

* 经过授权以后的业务页面:  http://xiaozao-flow.genshuixue.com/m/flow\_landing?appId=wxa60738f91ad5470b&code=021RPH6r0qNi2l1Hax7r0lm57r0RPH6n&state=326&appid=wxfa2f28350dc3b32b

---

*   oauth地址： POST /api/auth/oauth/token?unionId=oKUtLw6NdTeCboDCslQYTrNCkcYs&grant\_type=flow&scope=flow&appId=wxa60738f91ad5470b&openId=ojAdOwTfOwXbk0qT2m8zgK-Ljelo HTTP/1.1

* 请求头： Authorization : Basic eGlhb3phb19mbG93OmRKSjBaNWI3Z2ZKSHpDUDRjNThK

---

  **第一步：合成入口号静默授权地址 **

   &gt; 【请求后端接口entryBuilder】

   &gt; 【appid = wx18dd82265bdf8e6a】

   &gt; 【next = 目标页地址 前端要编码】      



```
http://xiaozao-flow.genshuixue.com/api/wechat/component/mp/common/entryBuildUrl?
appId=wx18dd82265bdf8e6a&next=http%253A%252F%252Fxiaozao-flow.genshuixue.com
%252Fm%252Fflow_landing%253FappId%253Dwx18dd82265bdf8e6a
```

  &gt; 返回一个字符串

```
"str": "https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx18dd82265bdf8e6a
&redirect_uri=http%3A%2F%2Fxiaozao-flow.genshuixue.com%2Fapi%2Fwechat%2Fcomponent%2Fmp%2F
common%2FentryCallback%3Fnext%3Dhttp%253A%252F%252Fxiaozao-flow.genshuixue.com%252Fm%252Fflow_landing
%253FappId%253Dwx18dd82265bdf8e6a&response_type=code&scope=snsapi_base&state=&component_appid=wxe99301426d324c53#wechat_redirect"
```

---

**第二步：entryBuilder返回一个地址 前端请求这个地址。之后的后续步骤由后端完成。后端通过重定向返回   【经过授权以后的业务页面】**

 &gt; 【请求返回值】

    

---

** 第三步：入口号静默授权回调  【后端完成】**

 &gt; 合成基准号网页授权路径并跳转

```
http://xiaozao-flow.genshuixue.com/api/wechat/component
/mp/common/entryCallback?next=http%3A%2F%2Fxiaozao-flow.genshuixue.com%2Fm%2F
flow_landing%3FappId%3Dwx18dd82265bdf8e6a&code=0213pYgo0nwpSn1RP1ho0ujNgo03pYgt&state=&
appid=wx18dd82265bdf8e6a
```

---

** 第四步：基准号授权回调  【后端完成】** 

 &gt; 跳回【经过授权以后的业务页面】

```
http://xiaozao-flow.genshuixue.com/api/wechat/component/mp/common/baseCallback?
next=http://xiaozao-flow.genshuixue.com/m/flow_landing?appId=wx18dd82265bdf8e6a&
code=021RPH6r0qNi2l1Hax7r0lm57r0RPH6n&state=326&appid=wx18dd82265bdf8e6a
```

---

**第五步：OAUTH  **

【前端用经过授权以后的业务页面带的参数进行请求】

&gt; 参数【unionId】基准号unionId

&gt; openId 入口号openId

&gt; appid 入口号appid

    POST /api/auth/oauth/token?unionId=oKUtLw6NdTeCboDCslQYTrNCkcYs&
    grant_type=flow&scope=flow&appId=wx18dd82265bdf8e6a&openId=ojAdOwTfOwXbk0qT2m8zgK-Ljelo HTTP/1.1```

&gt; oauth 返回

```
"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0ZW5hbnRfaWQiOiIiLCJ1c2VyX25hbWUiOiJvS1V0THc2TmRUZUNib0RDc2xRWVRyTkNrY1lzIiwidGVhY2hlcl9pZCI6IiIsInN0dWRlbnRfaWQiOiIiLCJyZWFsX25hbWUiOiIiLCJhdmF0YXIiOiIiLCJmbG93X3VzZXJfaWQiOiIxNTAiLCJjbGllbnRfaWQiOiJ4aWFvemFvX2Zsb3ciLCJyb2xlX25hbWUiOiIiLCJsaWNlbnNlIjoicG93ZXJlZCBieSB4aWFvemFvIiwidXNlcl9pZCI6IiIsInJvbGVfaWQiOiIiLCJzY29wZSI6WyJmbG93Il0sIm5pY2tfbmFtZSI6IiIsImV4cCI6MTU4OTgyNTA3MiwiZGVwdF9pZCI6IiIsImp0aSI6ImM2ZmNjYTE3LTA2ZGQtNGQzMi1iMmRlLTk2N2U5ZGQ3OGZjMSIsImFjY291bnQiOiJvS1V0THc2TmRUZUNib0RDc2xRWVRyTkNrY1lzIn0.hbgBtQHDVrYoRJBcBnwJfltZVGyM1tfwF6lvL1ceXbI",

"token_type": "bearer",

"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0ZW5hbnRfaWQiOiIiLCJ1c2VyX25hbWUiOiJvS1V0THc2TmRUZUNib0RDc2xRWVRyTkNrY1lzIiwidGVhY2hlcl9pZCI6IiIsInN0dWRlbnRfaWQiOiIiLCJyZWFsX25hbWUiOiIiLCJhdmF0YXIiOiIiLCJmbG93X3VzZXJfaWQiOiIxNTAiLCJjbGllbnRfaWQiOiJ4aWFvemFvX2Zsb3ciLCJyb2xlX25hbWUiOiIiLCJsaWNlbnNlIjoicG93ZXJlZCBieSB4aWFvemFvIiwidXNlcl9pZCI6IiIsInJvbGVfaWQiOiIiLCJzY29wZSI6WyJmbG93Il0sIm5pY2tfbmFtZSI6IiIsImF0aSI6ImM2ZmNjYTE3LTA2ZGQtNGQzMi1iMmRlLTk2N2U5ZGQ3OGZjMSIsImV4cCI6MTU5MDM5Mzg3MiwiZGVwdF9pZCI6IiIsImp0aSI6ImVhNmYwMmZmLWIwMmUtNDcxYS05NjExLTZiYTg5YjY3NDcwMSIsImFjY291bnQiOiJvS1V0THc2TmRUZUNib0RDc2xRWVRyTkNrY1lzIn0.DD2A2mmlgYPePtZbxcPs57PfWBMrsF8LhNOa9q1kqKQ",

"expires_in": 35999,

"scope": "flow",

"tenant_id": "",

"teacher_id": "",

"user_name": "oKUtLw6NdTeCboDCslQYTrNCkcYs",

"student_id": "",

"real_name": "",

"avatar": "",

"flow_user_id": "150",

"client_id": "xiaozao_flow",

"role_name": "",

"license": "powered by xiaozao",

"user_id": "",

"role_id": "",

"nick_name": "",

"dept_id": "",

"account": "oKUtLw6NdTeCboDCslQYTrNCkcYs",

"jti": "c6fcca17-06dd-4d32-b2de-967e9dd78fc1"
```

---

**服务器日志汇总**

```
2020-05-22 14:18:39.962  INFO 1359 --- [XNIO-1 task-290] .b.x.w.c.c.m.MpComponentCommonController : 入口公众号静默授权接收到来自微信服务器的认证消息：[071PwGmb20wxGL0eytkb2UmHmb2PwGm8, wx854d013c29376649, http://xiaozao-flow.genshuixue.com/m/flow_landing]
2020-05-22 14:18:40.149  INFO 1359 --- [XNIO-1 task-290] c.b.x.w.m.c.m.m.MpComponentCommonService : 公众号网页授权后，wxMpOAuth2AccessToken:{"accessToken":"33_GQAeO6glMKtFpPXVCHrZv7NzWNkTNcz_xx9T6dHZWOaNl65T-5Y5x4KezKqiwW8eiGpQ5uZmtKp6sD-_k26CIH2y5PNIWNGg2ddrrY0_A2Y","expiresIn":7200,"refreshToken":"33_4eiVpd76eYlPSsP7u_za-uG6tfFPKdl-ZZ3ve-Hob0CS7EWk3Y2yKKSMtFGQVXcjlf9OWYow6mvqIGI6zZYnqB5Ad4O9HUeOp0_v5Y4nkRY","openId":"oSZBm0ReYxp4XmF3gsTr2Aod20WQ","scope":"snsapi_base"}
2020-05-22 14:18:40.317  INFO 1359 --- [XNIO-1 task-290] c.b.x.w.m.c.m.m.MpComponentCommonService : 保存的微信用户信息wxUser:WxUser(id=163, appid=wx854d013c29376649, mobile=null, openId=oSZBm0ReYxp4XmF3gsTr2Aod20WQ, subscribeFlag=null, lastLoginTime=null, sex=null, nickName=null, city=null, province=null, country=null, avatarUrl=null, ossAvatarUrl=null, language=null, unionId=null, note=null, subscribeTime=null, groupId=null, tagidList=null, subscribeScene=null, qrScene=null, qrSceneStr=null, createTime=null, updateTime=null, delFlag=null)
2020-05-22 14:18:40.318  INFO 1359 --- [XNIO-1 task-290] .b.x.w.c.c.m.MpComponentCommonController : 基准公众号网页授权，开始跳转地址：https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxe839747d21d16607&redirect_uri=http%3A%2F%2Ftest-xiaozao-flow.genshuixue.com%2Fapi%2Fwechat%2Fcomponent%2Fmp%2Fcommon%2FbaseCallback%3Fnext%3Dhttp%3A%2F%2Fxiaozao-flow.genshuixue.com%2Fm%2Fflow_landing&response_type=code&scope=snsapi_userinfo&state=163&component_appid=wx9a5fbdbef1d86850#wechat_redirect
2020-05-22 14:18:41.000  INFO 1359 --- [XNIO-1 task-291] .b.x.w.c.c.m.MpComponentCommonController : 基准公众号网页授权接收到来自微信服务器的认证消息：[071V7YB80FPWpD16WGE80quRB80V7YB8, 163, wxe839747d21d16607, http://xiaozao-flow.genshuixue.com/m/flow_landing]
2020-05-22 14:18:41.169  INFO 1359 --- [XNIO-1 task-291] c.b.x.w.m.c.m.m.MpComponentCommonService : 公众号网页授权后，wxMpOAuth2AccessToken:{"accessToken":"33_uuDNk5MxojJDi7-tSt2TlQTJtF8Q3Z4tL6YgEoaQAZUqQ9pOcZkAyAwG7PN2J9rkyrfsW6Z6CfcjQ7uEbcZWTpDaO6y1xENXUiXhJLnRh0M","expiresIn":7200,"refreshToken":"33_NppZKjqUOZedEITfvLdxLXKSQ8YXWkmEcJOzm2bHgMOXcFVaCeMq7WI8FtG4OEkU5rF-pSmWtHUWz1qpdt4sUr0DFDU90YDnpRFj-zQQ9_Y","openId":"oWirHvuXzgjJh539k4siG2nD6-J8","scope":"snsapi_userinfo","unionId":"oKUtLw6NdTeCboDCslQYTrNCkcYs"}
2020-05-22 14:18:41.365  INFO 1359 --- [XNIO-1 task-291] c.b.x.w.m.c.m.m.MpComponentCommonService : 获取公众号网页授权微信用户信息，wxMpUser:{"openId":"oWirHvuXzgjJh539k4siG2nD6-J8","nickname":"Galaxy","sexDesc":"男","sex":1,"language":"zh_CN","city":"","province":"","country":"中国","headImgUrl":"http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLHsaE7e2KyAetd4TRtvibZJ3aaOYampUTDxKorz0DnI2VeoazqdAic8cCumY9BX2ib6hYLR0L5NntXg/132","unionId":"oKUtLw6NdTeCboDCslQYTrNCkcYs","privileges":[]}
2020-05-22 14:18:41.588  INFO 1359 --- [XNIO-1 task-291] c.b.x.w.m.c.m.m.MpComponentCommonService : 保存的微信用户信息wxUser:WxUser(id=62, appid=wxe839747d21d16607, mobile=null, openId=oWirHvuXzgjJh539k4siG2nD6-J8, subscribeFlag=1, lastLoginTime=null, sex=1, nickName=Galaxy, city=, province=, country=中国, avatarUrl=http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLHsaE7e2KyAetd4TRtvibZJ3aaOYampUTDxKorz0DnI2VeoazqdAic8cCumY9BX2ib6hYLR0L5NntXg/132, ossAvatarUrl=null, language=zh_CN, unionId=oKUtLw6NdTeCboDCslQYTrNCkcYs, note=null, subscribeTime=null, groupId=null, tagidList=null, subscribeScene=null, qrScene=null, qrSceneStr=null, createTime=null, updateTime=null, delFlag=null)
2020-05-22 14:18:41.594  INFO 1359 --- [XNIO-1 task-291] .b.x.w.c.c.m.MpComponentCommonController : 业务页面，开始跳转地址：http://xiaozao-flow.genshuixue.com/m/flow_landing&openId=oSZBm0ReYxp4XmF3gsTr2Aod20WQ&unionId=oKUtLw6NdTeCboDCslQYTrNCkcYs
2020-05-23 11:07:30.381  INFO 8147 --- [XNIO-1 task-148] .b.x.w.c.c.m.MpComponentCommonController : 业务页面，开始跳转地址：http://test-flow-mango.xiaozao100.com/&openId=o9g3Pv0W6kCqnXDIBMj2N9GqcU9s&unionId=oKUtLw6NdTeCboDCslQYTrNCkcY
```





















