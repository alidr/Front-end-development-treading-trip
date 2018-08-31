### 移动开发之安卓与ios兼容性问题整理

#### 时间（Time）

1. ios中对时间处理时时间格式必须为‘’2018/8/28 13:18",**日期中间必须为’/ ‘**，使用’-‘时ios中会为null

#### 图片（Image）
1. 利用swiper可以做图片双指缩放功能，但是缩放出来的图片在ios上会模糊失真，但是安卓上正常。

   原因：这个问题是由于IOS对transform的兼容性存在一定的问题。当translate3d和scale一起使用的时候，就会出现图片放大后模糊的问题。

   <span style="color:#bd3333">解决</span>

   ``` 
   transform:translate3d(0,0,0)==>transform:translate(0,0)//将3d换成2d
   ```

   <span style="color:#fecbcb">参考链接：https://segmentfault.com/q/1010000015552531</span>
