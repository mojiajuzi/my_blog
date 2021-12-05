---
title: "使用Laravel配合GatewayWorker与微信小程序通过websocket进行语音播报"
date: 2021-12-05T16:21:57+08:00
draft: false
tags: ["服务器","socket","Laravel","workerman","微信小程序"]
categories: ["服务器","websocket","小程序"]
---

最近接受了一个类似于外卖的小程序，需要使用websocket在骑手端进行语音的实时播报
在由于微信小程序切换到后台后无法实时进行通话，这里采用一个比较投机的方法来进行处理

## 小程序端

### 建立websocket连接

首先在`app.js`文件中创建一个方法，建立websocket的连接
```js
  initSocket(){

    let that = this

    let localsocket = wx.connectSocket({
      url: 'wss://exmple.com/websocket',
    })

    //当连接打开的时候需要进行的操作
    localsocket.opOpen(function(){
        ...
        //播放空白音频
        that.iniAudio()
    })

    //当socket关闭时需要进行的操作
    localsocket.onClose(function(){

    })

    //接收来自服务器的信息
    localsocket.onMessage(function(res){
        ....
        //播放提示音
        that.playNoticeVoice()

        ...
        //后台心跳检测，播放空白音频
        that.playNoVoice()
    })

    //错误时间处理
    localsocket.onError(function(err){
        
    })

    //主动向服务器发送信息,发送的信息string/ArrayBuffer格式的数据，
    //因此需要使用JSON。stringify进行格式化数据
    localsocket.send({})

    //将socket绑定到全局上面以便于随时可以调用websocket
    that.globalData.localSocket = localsocket
  }
```
### 语音播报

1.  使用小程序`同声传译`插件,在`app.json`中引入插件
    ```json
      "plugins":{
        "WechatSI":{
          "version": "0.3.5",
          "provider": "wx069ba97219f66d99"
        }
      },
    ```

1.  在程序中声明使用后台服务,在`app.json`中添加如下字段

    ```json
      "requiredBackgroundModes":["audio"],
    ```

   
1. 初始化`getBackgroundAudioManager`

   ```js
   //初始化音频连接
   iniAudio(){
       let that = this
       let voice  = wx.getBackgroundAudioManager()
       voice.title="" //标题
       voice.epname=""  //专辑名称
       voice.singer="" //歌手
       voice.src="" //音频连接
       voice.coverImgUrl="" //封面

       that.globalData.localVoice = voice
   }
   ```

1. 这里设置有一定时间长度的无声音频

   ```json
   playNoVoice(){
       let that = this
       if(that.globalData.localVoice){
         that.globalData.localVoice.src=""
       } 
   }
   ```


1. 播放提示音

```json
playNoticeVoice(){
    //使用语音合成
    let that =this
    var plugin = requirePlugin("WechatSI")
    plugin.textToSpeech({
      lang: "zh_CN",
      tts: true,
      content: "你有新的订单，请及时接单",
      success: function(res) {
          that.globalData.localVoice.src=res.filename
          that.globalData.localVoice.play()
          that.globalData.localVoice.onEnded(function(){
              that.playNoVoice()
          })
      },
      fail: function(res) {
          
      }
  })  
}
```

## 后端
后端使用`Laravel`框架并配合`GatewayWorker`以及`GatewayClient`开启websocket服务

1. 安装对应的compoer包
   ```
   composer require workerman/gateway-worker
   composer require workerman/gatewayclient
   ```

1. 在Laravel中新建一个命令
   ```
   php artisan make:command  WorkmanCommand
   ```
1. 完善命令行,可以参考文章[在 Laravel 中使用 Workerman 进行 socket 通讯](https://learnku.com/articles/13151/using-laravel-to-carry-out-socket-communication-in-workerman)

## nginx

在项目的Nginx配置文件中，增加如下的配置

```conf
 location /websocket {
     //端口为WorkmanCommand中new Gateway所填写的端口号
     proxy_pass http://127.0.0.1:23350; 
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection "upgrade";
     proxy_redirect off;
     proxy_set_header Host $host;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header X-Forwarded-Host $server_name;
     proxy_read_timeout 36000s;
     proxy_send_timeout 36000s;
 }
```


## 使用`supervisor`进行workerman进程的守护

```
[program:xxxx]
process_name=%(program_name)s_%(process_num)02d
command=php /your_path/artisan workerman --d
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=root
numprocs=1
redirect_stderr=true
stdout_logfile=/your_path/websocket.log
stopwaitsecs=4600
```




参考文档
- [微信小程序websocket](https://developers.weixin.qq.com/miniprogram/dev/api/network/websocket/wx.connectSocket.html)
- [微信小程序-音频](https://developers.weixin.qq.com/miniprogram/dev/api/media/background-audio/wx.getBackgroundAudioManager.html)
- [微信小程序-同声传译](https://developers.weixin.qq.com/miniprogram/dev/platform-capabilities/extended/translator.html)
- [workerman gateway-worker doc](https://www.workerman.net/doc/gateway-worker/README.html)
- [walkor/GatewayWorker](https://github.com/walkor/GatewayWorker)
- [walkor/GatewayClient](https://github.com/walkor/GatewayClient)