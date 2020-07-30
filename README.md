# appleTools
### 简介

苹果相关工具

以后按需会添加支付相关



### 登陆

开发APP苹果登陆功能时参考各种渠道的代码然后顺手封装一下。

登陆主要参考：https://github.com/tptpp/sign-in-with-apple

在此基础上增加了解密苹果最终返回数据的id_token，用于验证客户端post过来的userIdentifier

其中id_token解密后有个sub字段，该字段一般和userIdentifier一致

通过对比这两个字段是否相等来处理后续登陆业务

比如如下使用方式

```go
// @Summary 苹果登陆
// @Produce  application/json
// @Param data body request.AppleLoginCode true "苹果登陆"
// @Success 200 {string} string "{"success":true,"data":{},"msg":"登陆成功"}"
// @Router /appLogin/AppleLoginCode [post]
func AppleLogin(c *gin.Context) {
  //一个请求信息的结构体，接收authorizationCode和userIdentifier
   var A request.AppleLoginCodeAndId
   _ = c.ShouldBindJSON(&A)
  
  //生成client_secret，参数：苹果账户的KeyId,TeamId, ClientID, KeySecret
   clientSecret := appleTools.GetAppleSecret(KeyId,TeamId, ClientID, KeySecret)
  
  //获取用户信息，参数：ClientID, 上面的clientSecret, authorizationCode
   data, err := appleTools.GetAppleLoginData(ClientID, clientSecret, A.Code)
  
  //检查用户信息，参数：上面的data，客户端传过来的userIdentifier，上面的err
   check := appleTools.CheckAppleID(data, A.ID, err)
   if check == false {
      response.FailWithMessage("APPLE登陆失败[error 1]，请重试或使用其他方式登陆", c)
      return
   }
  
  fmt.Println("苹果校验结果OK")
  
  //你的登陆业务
  
}
```