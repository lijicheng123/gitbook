# 小程序登录

#### 背景：

> 为能通过微信识别用户,通过微信官方提供的登录能力方便地获取微信提供的用户身份标识，快速建立小程序内的用户体系。所以需要获取openid或者unionid

#### 目的：

> **获取openid或者unionid**
>
> > openid和unionid的区别：
> >
> > openid是指定应用的，unionid是一个开放平台的。即每一个公众号都有自己的openid，每一个小程序都有自己的openid，而同一个开放平台共享同一个unionid。unionid需要加入某个开放平台才有。

#### 实现方式：

绑定了开发者帐号的小程序，可以通过下面 4 种途径获取 UnionID。

1. 调用接口[wx.getUserInfo](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getUserInfo.html)，从解密数据中获取 UnionID。注意本接口需要用户授权，请开发者妥善处理用户拒绝授权后的情况。

2. 如果开发者帐号下存在**同主体的**公众号，并且该用户已经关注了该公众号。开发者可以直接通过[wx.login](https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html)+[code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/code2Session.html)获取到该用户 UnionID，无须用户再次授权。

3. 如果开发者帐号下存在**同主体的**公众号或移动应用，并且该用户已经授权登录过该公众号或移动应用。开发者也可以直接通过[wx.login](https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html)+[code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/code2Session.html)获取到该用户 UnionID ，无须用户再次授权。

4. 小程序端调用[云函数](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/capabilities.html?t=18121111#云函数)时，如果开发者帐号下存在**同主体的**公众号，并且该用户已经关注了该公众号，可在云函数中通过[cloud.getWXContext](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-server-api/utils/getWXContext.html?t=18121111)获取 UnionID

5. 小程序端调用[云函数](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/capabilities.html?t=18121111#云函数)时，如果开发者帐号下存在**同主体的**公众号或移动应用，并且该用户已经授权登录过该公众号或移动应用，也可在云函数中通过[cloud.getWXContext](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-server-api/utils/getWXContext.html?t=18121111)获取 UnionID

#### 登录流程时序

![](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/image/api-login.jpg)

#### 说明： {#说明：}

1. 调用[wx.login\(\)](https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html)获取**临时登录凭证code**，并回传到开发者服务器。
2. 调用[code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/code2Session.html)接口，换取**用户唯一标识 OpenID**和**会话密钥 session\_key**。

之后开发者服务器可以根据用户标识来生成自定义登录态，用于后续业务逻辑中前后端交互时识别用户身份。

**注意：**

1. 会话密钥`session_key`是对用户数据进行[加密签名](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/signature.html)的密钥。为了应用自身的数据安全，开发者服务器**不应该把会话密钥下发到小程序，也不应该对外提供这个密钥**。
2. 临时登录凭证 code 只能使用一次

#### 方案选择：



根据我们的业务需求及现有环境，我们主要是为了获取用户unionid，而用户几乎都是来自微信公众号，所以我们选择了优先从直接通过[wx.login](https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html)+[code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/code2Session.html)获取到该用户 UnionID，如果拿不到unionid，再启用[wx.getUserInfo](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getUserInfo.html) 方法。

具体解决方案：[小程序优雅获取用户信息](/小程序/xiao-cheng-xu-you-ya-huo-qu-yong-hu-xin-xi.md)

