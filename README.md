# taro-mini-picker-date-time
### 初衷
由于小程序原生组件并不能满足日期和时间同时选择，所以在实际产品的需求开发中，自己通过pick-view来进行组件封装，以此分享

### 使用方法
* 拷贝 picker-date-time 到自己的公共组件目录下，并引用
```javascript
components: {
  ...,
  PickerDateTime: () => import('xxx/picker-date-time'),
},
```

* 使用的地方
```javascript
<PickerDateTime
  :text="xxx"
  :timeSlot="xxx"
  :minDateTime="xxx"
  :maxDateTime="xxx"
  @handleConfirm="
    (v) => {
      // TODO 选完以后的操作
    }
  "
/>
```
### 效果

![image](https://user-images.githubusercontent.com/49790909/149069071-2cecc23d-65c2-4917-9842-79e5ab661567.png)
