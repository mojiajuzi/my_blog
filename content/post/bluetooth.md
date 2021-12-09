---
title: "微信小程序使用CPCL指令打印电子面单（一）：连接蓝牙打印机"
date: 2021-12-05T23:06:53+08:00
draft: false
tags: ["微信小程序","电子面单","蓝牙","连接"]
categories: ["小程序","蓝牙"]
---


在项目的开发过程中需要打印一联快递电子面单，这里使用微信小程序中的蓝牙硬件能力使用CPCL指令驱动蓝牙打印机，
该文章总共有三篇，分别是

- Part 1 搜索蓝牙设备
- Part 2 连接蓝牙设备
- Part 3 构建CPCL指令并打印电子面单

# 搜索蓝牙设备


我们搜索蓝牙打印机的一般步骤就是：
1. 打开手机蓝牙
2. 打开打印机蓝牙
3. 搜索能够写入数据的蓝牙设备并展示

首先我们需要提醒用户打开手机以及打印机的蓝牙

```js
let that = this
wx.showModal({
  title:"连接打印机",
  content:"请确保蓝牙和打印机已打开",
  success:(res)=>{
    if(res.confirm){
      that.openBLTAdapter()
    }
  }
})

```


接下来我们就需要初始化我们的手机蓝牙模块

```js
  /**
   * 初始化蓝牙模块
   */
  openBLTAdapter(){
    let that = this
    wx.openBluetoothAdapter({
      mode:"central",
      success:(res)=>{
        //初始化成功，开始搜索蓝牙设备
        that.searchBLTDevice(res.data)
      },
      fail:(res)=>{
        if(res.errCode && res.errCode ===10001){
           if(res.available){
            that.searchBLTDevice()
           }else{
            //提示用户
          }
        }else{
          //依据情况提示用户
        }
      }
    })
  },
```

搜索周围的蓝牙设备

```js
  /**
   * 搜索蓝牙设备
   */
  searchBLTDevice(){
    let that = this
    wx.startBluetoothDevicesDiscovery({
      allowDuplicatesKey:true,
      interval:0,
      powerLevel:"high",
      success:(res)=>{
        that.recordBLTDevice()
      },
      fail:(err)=>{
          //依据情况提示用户
      },
      complete:(res)=>{
          //依据情况提示用户
      }
    })
  },
```

将搜索到的蓝牙添加到记录中，以便用户选择需要连接的打印机

```js
  /**
   * 将搜索到的蓝牙添加到列表
   */
 recordBLTDevice(){
   let that = this
    wx.onBluetoothDeviceFound((res)=>{
      res.devices.forEach(device =>{
        if((!device.name) && (!device.localName)){
          return
        }
        const foundDevices = that.data.devices
        const idx = that.inArray(foundDevices,'deviceID',device.deviceId)
        const data = {}

        if(idx === -1){
          data[`devices[${foundDevices.length}]`] = device
        }else{
          data[`devices[${idx}]`] = device
        }
        data['showBluetoothList']=true
        that.setData(data)
      })
    })
 },
```

判断搜索到的设备是否在列表中

```js
 /**
  * 判断新搜索到的蓝牙在列表中
  * @param {*} arr 
  * @param {*} key 
  * @param {*} val 
  * @returns 
  */
inArray(arr, key, val) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i][key] === val) {
      return i;
    }
  }
  return -1;
},
```







参考文档

- [微信小程序蓝牙模块文档](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.stopBluetoothDevicesDiscovery.html)
