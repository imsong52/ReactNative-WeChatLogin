# ReactNative-WeChatLogin
基于react-native-wechat 实现的微信登录.只需 CV 大法就可以.(自发现教程,没有发现现成 demo 所以我就简单搞了一下按照网上的教程写的 demo)

**欢迎大家加群讨论**

点击链接加入群[ReactNative-解决问题交流群](https://jq.qq.com/?_wv=1027&k=4EZwdSd) :644124441

点击链接加入群[ReactNative技术交流群2](https://jq.qq.com/?_wv=1027&k=55Dujm4)  :687663534

**需要真机 run**
> 具体怎么配置我就不说了网上一堆.我这里只提供实现☺
## 第一步：请求CODE
 ### 移动应用微信授权登录
 关键代码:
 ```
  //发送授权请求
     WeChat.sendAuthRequest(scope, state)
    .then(responseCode => {
    //返回code码，通过code获取access_token
     console.log(responseCode.code);
     this.getAccessToken(responseCode.code);
     })
     .catch(err => {
       Alert.alert('登录授权发生错误：', err.message, [
        {text: '确定'}
     ]);
  })
 ```
## 第二步：通过code获取access_token
  ### 获取第一步的code后，请求以下链接获取access_token：
  ```
  // 获取 access_token
  getAccessToken(responseCode){
    // ToastUtil.showShort(responseCode, true);
    var AccessTokenUrl = 'https://api.weixin.qq.com/sns/oauth2/access_token?appid='+appid+'&secret='+secretID+'&code='+responseCode+'&grant_type=authorization_code';
    // console.log('AccessTokenUrl=',AccessTokenUrl);
    fetch(AccessTokenUrl,{
        method:'GET',
        timeout: 2000,
        headers:{
            'Content-Type':'application/json; charset=utf-8',
        },
        })
        .then((response)=>response.json())
        .then((responseData)=>{
            console.log('responseData.refresh_token=',responseData);
            this.getRefreshToken(responseData.refresh_token);
        })
        .catch((error)=>{
            if(error){
                console.log('error=',error);
            }
        })
     }
  ```
  ### 刷新access_token有效期
      access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种：
      1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；
      2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。
      refresh_token拥有较长的有效期（30天），当refresh_token失效的后，需要用户重新授权。
      请求方法
      获取第一步的code后，请求以下链接进行refresh_token：
      ```
      //  获取 refresh_token
     getRefreshToken(refreshtoken){
      var getRefreshTokenUrl = 'https://api.weixin.qq.com/sns/oauth2/refresh_token?appid='+appid+'&grant_type=refresh_token&refresh_token='+refreshtoken;
      // console.log('getRefreshTokenUrl=',getRefreshTokenUrl);
      fetch(getRefreshTokenUrl,{
          method:'GET',
          timeout: 2000,
          headers:{
              'Content-Type':'application/json; charset=utf-8',
          },
          })
          .then((response)=>response.json())
          .then((responseData)=>{
              console.log('responseData.accesstoken=',responseData);
              this.getUserInfo(responseData);
          })
          .catch((error)=>{
              if(error){
                  console.log('error=',error);
              }
          })
     }
      ```
## 第三步：通过access_token调用接口
获取access_token后，进行接口调用，有以下前提：
1. access_token有效且未超时；
2. 微信用户已授权给第三方应用帐号相应接口作用域（scope）。
```
 //获取用户信息
     getUserInfo(responseData){
       console.log(responseData);
      var getUserInfoUrl = 'https://api.weixin.qq.com/sns/userinfo?access_token='+responseData.access_token+'&openid='+responseData.openid;
      console.log('getUserInfoUrl=',getUserInfoUrl);
      fetch(getUserInfoUrl,{
          method:'GET',
          timeout: 2000,
          headers:{
              'Content-Type':'application/json; charset=utf-8',
          },
          })
          .then((response)=>response.json())
          .then((responseData)=>{
              console.log('getUserInfo=',responseData);
              ToastUtil.showLong([responseData.nickname,responseData.province,responseData.city,responseData.openid],true) 

          })
          .catch((error)=>{
              if(error){
                  console.log('error=',error);
              }
          })
     }
```
## 第四步:已经判断是否安装微信.如果没有安装将会跳转到 App Store 微信安装页面进行安装.
实现的关键代码:
```
//安装微信
      installWechat(){
        var getWeChatUrl = 'itms-apps://itunes.apple.com/cn/app/%E5%BE%AE%E4%BF%A1/id414478124?mt=8';
        Linking.canOpenURL(getWeChatUrl)
        .then((supported)=>{  
          if (!supported){  
            console.log('Can\'t handle url: ' + url);  
            Alert.alert(  
              '提示',   
              'Can\'t handle url: ' + url,  
              [  
                {text: 'OK', onPress:()=>{}}  
              ]  
            );  
          }else{  
            return Linking.openURL(getWeChatUrl);  
          }  
        }) 
        .catch((err)=>{  
          console.log('An error occurred', err);  
          Alert.alert(  
            '提示',   
            'An error occurred: ' + err,  
            [  
              {text: 'OK', onPress:()=>{}}  
            ]  
          );  
        });   
       
      }

```

> 注意：

> 1、Appsecret 是应用接口使用密钥，泄漏后将可能导致应用数据泄漏、应用的用户数据泄漏等高风险后果；存储在客户端，极有可能被恶意窃取（如反编译获取Appsecret）；

> 2、access_token 为用户授权第三方应用发起接口调用的凭证（相当于用户登录态），存储在客户端，可能出现恶意获取access_token 后导致的用户数据泄漏、用户微信相关接口功能被恶意发起等行为；

> 3、refresh_token 为用户授权第三方应用的长效凭证，仅用于刷新access_token，但泄漏后相当于access_token 泄漏，风险同上。

> 建议将Appsecret、用户数据（如access_token）放在App云端服务器，由云端中转接口调用请求。

