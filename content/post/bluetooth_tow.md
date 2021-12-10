---
title: "微信小程序使用CPCL指令打印电子面单（二）：连接蓝牙打印机"
date: 2021-12-10T21:27:08+08:00
draft: false
tags: ["微信小程序","电子面单","蓝牙","连接"]
categories: ["小程序","蓝牙"]
---

在项目的开发过程中需要打印一联快递电子面单，这里使用微信小程序中的蓝牙硬件能力使用CPCL指令驱动蓝牙打印机，
该文章总共有三篇，分别是

- [Part 1 搜索蓝牙设备](https://blog.tsecloud.cn/post/bluetooth/)
- [Part 2 连接蓝牙设备](https://blog.tsecloud.cn/post/bluetooth_tow)
- Part 3 构建CPCL指令并打印电子面单

# 连接蓝牙设备
上一篇文章我们已经搜索到蓝牙并将发现的蓝牙设备添加到数组中记录起来了，接下来我们学习如何连接蓝牙

首先我们将选中的蓝牙信息传入函数中
```js
createBLEconnect(ds){

  //先弹个窗，因为连接的过程需要耗费点时间
  wx.showLoading({
    title:"蓝牙连接中....",
    mask:true
  })

  //获取唯一标识以及名称
  const deviceId=ds.deviceid
  const name=ds.name
  let that = this
  wx.createBLEConnection({
    deviceId:deviceId,
    success:(res)=>{
      //停止搜索新的设备，减少耗电量
      wx.stopBluetoothDevicesDiscovery()
      that.setData({
        connected:true,
        name,
        deviceId
      })
      that.getBLTDeviceAllService(deviceId)
    },
    fail:(err)=>{
      if(err.errCode == -1){
        //蓝牙已经连接成功
        that.setData({
          connected:true
        })
      }else{

      }
    },
    complete:()=>{
      wx.hideLoading()
    }
  })
}
```

获取选定蓝牙的所有服务，看是否具备写入或者读取的权限

```js
getBLTDeviceAllService(id){
  let that = this
  wx.getBLEDeviceServices({
    deviceId:id,
    success:(res)=>{
        for (let i = 0; i < res.services.length; i++) {
          if(res.services[i].isPrimary){
             that.getBLTdeviceAllCharact(id,res.services[i].uuid)
             return
          }
        }
    },
  })
},
```

接着我们获取所连接蓝牙特的服务的特征值
```js
/**
 * 获取蓝牙中的所有特征
 * @param {*} deviceId 
 * @param {*} uuid 
 */
getBLTdeviceAllCharact(deviceId,serviceId){
  wx.getBLEDeviceCharacteristics({
    deviceId,
    serviceId,
    success:(res)=>{
      for (let i = 0; i < res.characteristics.length; i++) {
        let item = res.characteristics[i]

        //是否具备读取数据的能力
        if (item.properties.read) {
          wx.readBLECharacteristicValue({
            deviceId,
            serviceId,
            characteristicId: item.uuid,
          })
        }
        //是否具备写的能力
        if (item.properties.write) {
          this.setData({
            canWrite: true,
            selectedDeviceId:deviceId,
            selectedServiceId:serviceId,
            selectedCharacteristicId:item.uuid
          })
        }

        if (item.properties.notify || item.properties.indicate) {
          wx.notifyBLECharacteristicValueChange({
            deviceId,
            serviceId,
            characteristicId: item.uuid,
          })
        }
      }
    },
    fail:(res)=>{

    }
  })
}
```