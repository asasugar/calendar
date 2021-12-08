# calendar
日历组件（Vue版本、微信小程序版本）


## UI效果
![20211207175255](https://s2.loli.net/2021/12/07/xo9aW68FwNACq7c.png)

## 重点

- `spot` 可以传入时间戳或者带有`/`格式 、`-`格式的字符串，默认ui是绿点，可根据需求传入slot自定义样式

- 目前像素单位是小程序版本的`rpx`，可根据自己需求调整
## Vue版本用法

```html
<calendar :spot="['2021/12/8']" @onSelect="dayChange" @onToggleOpen="openChange"/>
```
## 微信小程序版本用法

```html
<calendar spot="{{['2021/12/8']}}" bind:onSelect="dayChange" bind:onToggleOpen="openChange"/>
```
## Props type

| _Prop_    | _Type_   | _Defaults_ | _Required_ | _Description_                                                      |
| :-------- | :------- | :--------- | :--------- | ------------------------------------------------------------------ |
|spot|Array|[]|×|设置绿点的时间，样式可自定义|
|isShowSlotSpot|Boolean|false|×|是否显示slot的spot|
|defaultTime|String|''|×|当前默认选中时间，不传则为当前日期|
|onSelect|Function|()=>{year,month,day,dateString}|×|选中日期的回调函数|
|onToggleOpen |Function|()=>true or false|×|展开/收起日历的回调函数|
