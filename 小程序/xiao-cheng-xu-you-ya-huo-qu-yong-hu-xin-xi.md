## 小程序如何优雅的获取用户信息（包括unionid）

#### 背景：

> 很多小程序开发过程中需要获取用户信息（包括unionid），而对于没有关注过小程序主题公众号的应用来说，前端是无法直接获取到unionid的，但是可以获取到openid。

#### 方案：

根据我们的业务需求及现有环境，我们主要是为了获取用户unionid，而用户几乎都是来自微信公众号，所以我们选择了优先从直接通过[wx.login](https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html)+[code2Session](https://developers.weixin.qq.com/miniprogram/dev/api/code2Session.html)获取到该用户 UnionID，如果拿不到unionid，再启用[wx.getUserInfo](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getUserInfo.html) 方法。

##### wx.login + code2session :

> fe:
>
> ```js
> getUnionid(cb) {
>         let _this = this;
>         wx.login({
>             success(res) {
>                 //   用户登录凭证（有效期五分钟）,只有code
>                 let { code } = res;
>                 wx.cloud.callFunction({
>                     name: 'login',
>                     data: {
>                         code: code
>                     },
>                     complete: res => {
>                         wx.hideLoading()
>                         console.log(res)
>                         let result = JSON.parse(res.result)
>
>                         let unionid = result.unionid
>
>                         //获取到unionid后存储备用
>                         wx.setStorageSync('unionid', unionid)
>                         cb && cb({
>                             unionid:unionid
>                         })
>                         console.log('callFunction login result: ', result)
>                     }
>                 })
>             }
>         })
>     }
> ```
>
> be:
>
> ```js
> const cloud = require('wx-server-sdk')
> const request = require('request');
> cloud.init()
>
> // 云函数入口函数
> exports.main = async (event, context) => {
>     const res = await cloud.callFunction({
>         name:'config',
>         data:{}
>     })
>     var APPID =  res.result.appid
>     var SECRET = res.result.secret
>     let {code} = event
>     let url = `https://api.weixin.qq.com/sns/jscode2session?appid=${APPID}&secret=${SECRET}&js_code=${code}&grant_type=authorization_code`
>
>
>     return new Promise((resolve, reject) => {
>         request.get(url,(err, res, body)=>{
>             return resolve(body)
>         })
>     })
>
> }
> ```

如果还没获取到unionid，则通过

##### wx.getUserInfo：

> fe:
>
> ```js
> getUserInfo(e){
>         let _this = this;
>         wx.login({
>             success(res) {
>                 //   用户登录凭证（有效期五分钟）,只有code
>                 console.log('login:',res)
>                 let { code } = res;
>                 wx.cloud.callFunction({
>                     name: 'login',
>                     data: {
>                         code: code
>                     },
>                     complete: res => {
>                         console.log(res)
>                         let result = JSON.parse(res.result)
>                         // let result = res.result
>                         let unionid = result.unionid
>                         let session_key = result.session_key
>                         
>                         if(util.checkUnionid(unionid)){
>                             wx.setStorageSync('unionid', unionid)
>
>                             _this.toAuthentication()
>
>                         }else {
>                             _this.getEncrypt(session_key,(result)=>{
>                                 let unionid = result.result.data.unionId
>                                 wx.setStorageSync('unionid', unionid)
>                                 
>                                 //拿到unionid后开始调用方法
>                                 _this.toAuthentication()
>                                 // console.log('解密后的数据为：',result)
>                             })
>                         }
>                         
>                         
>                     }
>                 })
>             }
>         })
>     },
>     getEncrypt(session_key,cb){
>         wx.getSetting({
>             success (res){
>                 if (res.authSetting['scope.userInfo']) {
>                     // 已经授权，可以直接调用 getUserInfo 获取头像昵称
>                     wx.getUserInfo({
>                         success: function(user) {
>                             console.log('userInfo:',user)
>                             let encryptedData = user.encryptedData
>                             let iv = user.iv
>
>                             wx.cloud.callFunction({
>                                 name: 'encrypt',
>                                 data: {
>                                     encryptedData:encryptedData,
>                                     iv:iv,
>                                     sessionKey:session_key
>                                 },
>                                 complete: info => {
>
>                                     cb && cb(info)
>
>                                     console.log('云函数encrypt方法: ', info)
>                                 }
>                             })
>                         }
>                     })
>                 }
>             }
>         })
>     },
> ```
>
> be:
>
> ```
> var crypto = require('crypto')
>
> function WXBizDataCrypt(appId, sessionKey) {
>   this.appId = appId
>   this.sessionKey = sessionKey
> }
>
> WXBizDataCrypt.prototype.decryptData = function (encryptedData, iv) {
>   // base64 decode
>   var sessionKey = new Buffer(this.sessionKey, 'base64')
>   encryptedData = new Buffer(encryptedData, 'base64')
>   iv = new Buffer(iv, 'base64')
>
>   try {
>      // 解密
>     var decipher = crypto.createDecipheriv('aes-128-cbc', sessionKey, iv)
>     // 设置自动 padding 为 true，删除填充补位
>     decipher.setAutoPadding(true)
>     var decoded = decipher.update(encryptedData, 'binary', 'utf8')
>     decoded += decipher.final('utf8')
>     
>     decoded = JSON.parse(decoded)
>
>   } catch (err) {
>     throw new Error('Illegal Buffer')
>   }
>
>   if (decoded.watermark.appid !== this.appId) {
>     throw new Error('Illegal Buffer')
>   }
>
>   return decoded
> }
>
> module.exports = WXBizDataCrypt
>
>
> var WXBizDataCrypt = require('./WXBizDataCrypt')
> const cloud = require('wx-server-sdk')
> cloud.init()
>
> exports.main = async (event,context) => {
>     const res = await cloud.callFunction({
>         name:'config',
>         data:{}
>     })
>     var appId =  res.result.appid
>     
>     let {
>         iv,
>         sessionKey,
>         encryptedData,
>     } = event
>
>     var pc = new WXBizDataCrypt(appId, sessionKey)
>
>     var data = pc.decryptData(encryptedData , iv)
>     
>     return {
>         code:0,
>         data:data
>     }
> }
> ```



