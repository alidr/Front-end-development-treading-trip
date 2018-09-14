### 那些天，走过的mpvue小程序开发

#### 项目开发参考的主要的文档
<p style="color:#9ccccc">mpvue官方文档：http://mpvue.com/mpvue/#_1</p>
<p style="color:#9ccccc">微信官方文档：https://developers.weixin.qq.com/miniprogram/dev/index.html?t=18080816</p>
<p style="color:#9ccccc">vue官方文档：https://cn.vuejs.org/index.html</p>
<p style="color:#9ccccc">美团小程序框架mpvue蹲坑指南：https://github.com/noahlam/articles/blob/master/%E7%BE%8E%E5%9B%A2%E5%B0%8F%E7%A8%8B%E5%BA%8F%E6%A1%86%E6%9E%B6mpvue%E8%B9%B2%E5%9D%91%E6%8C%87%E5%8D%97.md</p>
<p style="color:#9ccccc">美团小程序入门指南：https://www.cnblogs.com/noahlam/p/8910026.html</p>



#### 一定要搞清楚的生命周期

<p style="color:#9ccccc">微信生命周期：https://developers.weixin.qq.com/miniprogram/dev/image/mina-lifecycle.png?t=18082414</p>
<p style="color:#9ccccc">vue生命周期：https://cn.vuejs.org/images/lifecycle.png</p>
<p style="color:#9ccccc">mpvue生命周期：http://mpvue.com/assets/lifecycle.jpg</p>

mpvue与vue不同的是在小程序 onReady 后，才会再去触发 vue mounted 生命周期；除了 Vue 的生命周期外，mpvue 还兼容了小程序生命周期，但是官网表示除特殊情况外，不建议使用小程序的生命周期钩子。

小程序中获取上一页传过来的参数是在onLoad中，例：options.id

mpvue获取上一页传过来的参数，直接通过 `this.$root.$mp.query` 获取相应的参数数据，其调用需要在 `onLoad` 生命周期触发之后使用，比如 `onShow` 

**mpvue进入页面是钩子函数触发过程**

**进入已销毁的page组件时依次触发onLoad,onShow,onReady,beforeMount,mounted**
**第一次进入已销毁的子组件时依次触发onLoad,onReady,beforeMount,mounted**
**第二次进入已销毁的子组件时依次触发onLoad,onShow,onReady**
**再次进入 未被销毁的page组件、子组件时只触发onShow**



#### 敲下小黑板

* 【**mpvue**】小程序中的bindChange事件在mpvue中写为@change
* 【**mpvue**】mpvue中data不声明变量在v-model使用时不会报错，但是改变的值不会获取到
* 【**mpvue**】不要在选项属性或回调上使用箭头函数，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数是和父级上下文绑定在一起的，`this` 不会是如你做预期的 Vue 实例，且 `this.a` 或 `this.myMethod` 也会是未定义的。
* 【**mpvue**】可以使用vuex
* 【**原生小程序**】原生小程序中可以通过placeholder-class来给placeholder设置样式
* 【**原生小程序**】调用微信小程序的API时候注意this的指向！！！
* 【**原生小程序**】小程序中没有dom对象！！！不要去操作dom！！！用微信wx.setStorage替代localStorage
* 【**原生小程序**】头像和昵称可以使用open-data,不需要调用API，但是获取到的内容只能展示不会存进数据库，需要存库要用getuserInfo


#### 排坑指南

* 【**mpvue**】使用flyio进行全局请求拦截器以及接口token验证

  ``` 
  
  //根据文档步骤，引入
  var Fly = require("flyio/dist/npm/wx.js") //wx.js为flyio的微信小程序入口文件
  var fly = new Fly();
  fly.config.baseURL = BASE_URL
  var token = ''
  //添加拦截器（在拦截器中处理相关token验证等逻辑）
  fly.interceptors.request.use(async function (config, promise) {
  //展示loading
    wx.showLoading({
      title: '加载中...'
    })
    //设置token
    // let token = wx.getStorageSync('token')
    if (!token) {
      fly.lock()
      // getToken().then(res => {
      return getToken().then(res => {
        token = res
        config.headers['token'] = token
        return config
      }).finally(() => {
        fly.unlock(); //解锁后，会继续发起请求队列中的任务，详情见后面文档
      });
    } else {
      config.headers['token'] = token
      return config
    }
  })
  //拦截返回的数据处理
  fly.interceptors.response.use(
    (response, promise) => {
      wx.hideLoading()
      if (!response.data.Success){
        if (response.data.Code === 401000) {
          token = ''
          return fly.request(response.request);
        }
        wx.showToast({
          title: response.data.Message,
          icon: 'none',
          duration: 2000
        })
      }
      return response.data
    },
    (err) => {}
  )
  
  export default fly
  
  ```

  <span style="color:#9ccccc">参考链接： https://github.com/wendux/fly</span>




* 【**mpvue**】同一路由切换时，上一次的页面数据会保留(页面切换大概是这样一个过程 pageA -> pageB?id=1 -> pageB?id=2 --back-> pageB?id=1最后返回至 id=1 这个页面时，页面会显示 id=2 的数据）

  <span style="color:#9c1e22">解决</span>

  ```
    onUnload () {
      //解决同一页面不同参数进入后页面缓存问题
      //mpvue踩坑
      Object.assign(this.$data,this.$options.data())
    }
    //在onUnload钩子中将数据清空
  ```

  *<span style="color:#666;font-size=10px">原生小程序不存在这个问题，因为原生中每个页面的 data 里都有一个隐藏属性 **webviewId**，同一个页面参数不同时 **webviewId** 是不同的，而参数相同时 **webviewId** 也是同一个，但 mpvue 似乎并没有对这个做处理，同一个页面不论参数是什么，数据都是同一份)</span>*

  >  <span style="color:#9ccccc">参考链接https://github.com/Meituan-Dianping/mpvue/issues/140</span>   



* 【**mpvue**】新增文件并且在project.config.json中配置后提示’‘’xxx在app.json中未定义‘

  <span style="color:#9c1e22">解决</span>

  ``` 
  npm run dev
  ```

  *<span style="color:#666;font-size=10px">因为 webpack 编译的文件是由配置的 entry 决定的，新增的页面并没有添加进 entry，所以需要手动 `npm run dev` 一下</span>*



* 【**原生小程序**】富文本中改变图片宽度

  <span style="color:#9c1e22">解决</span>

  ```  
  <rich-text nodes="{{Detail.content}}" type='text'></rich-text>
  js文件中
  //重点是这句话 res.content是从后台获取的数据 进行正则匹配的
  res.content = content.replace(/\<img/gi, '<img style="width:100%;height:auto;margin:0;padding:0;vertical-align:top> ')
  ```

  *<span style="color:#666;font-size=10px">利用正则给图片设置样式，（利用vertical-align:top属性去除图片之间的空白间隙）；其他解决方法：强制要求富文本编辑的时候控制图片宽度，使大图缩放至320px，以此来适应多屏幕，达到图片不溢出的效果</span>*



* 【**原生小程序**】右上角转发功能配置

  <span style="color:#9c1e22">解决</span>
    ``` 
        在 Page 中定义 onShareAppMessage 事件处理函数，自定义该页面的转发内容。
        此事件需要 return 一个 Object，用于自定义转发内容
         onShareAppMessage: function (res) {
            if (res.from === 'button') {
              // 来自页面内转发按钮
              console.log(res.target)
            }
            return {
              title: '自定义转发标题',
              path: '/page/user?id=123'
            }
          }
    ```
  *<span style="color:#666;font-size=10px">全局配置无效，只能在指定页面中	配置函数进行转发</span>*



* 【**原生小程序**】关于有无tabBar页面之间跳转

  <span style="color:#9c1e22">解决</span>

    ``` 
  wx.navigateTo:保留当前页面，跳转到应用内的某个页面，(之前页面不会被销毁，只是切换到后台)使用				  wx.navigateBack可以返回到原页面。
  wx.redirectTo：关闭当前页面，跳转到应用内的某个页面。
  wx.switchTab：跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面
    ```
  *<span style="color:#666;font-size=10px">tabBar可以通过api控制显示隐藏，隐藏掉batBar的页面跳转到tabBar 页面,也要使用wx.switchTab进行跳转</span>*
  <span style="color:#666;font-size=10px">tabBar显示隐藏</span>

  ``` 
  
  onShow(){
      wx.hideTabBar()
  },
  onHide(){
      wx.showTabBar()
  }
  
  ```



* 【**原生小程序**】在设置自定义弹层时遮罩遮不住微信定义的tabBar

  <span style="color:#9c1e22">解决</span>

  	不使用微信配置tabBar而是自己写一个底部导航

#### 补充一条微信小程序分享接口调整公告

7月5日起新提交的版本，用户从**小程序**、**小游戏**中分享消息给好友时，开发者将无法获知用户是否分享完成，也无法在分享后立即获得群ID。该策略在最新版开发者工具上，可以选择基础库 2.0.8版本预先体验。具体调整点为：

（1）分享接口调用后，将不再返回分享结果事件。详情可参考[转发介绍](https://developers.weixin.qq.com/miniprogram/dev/api/share.html#onshareappmessageoptions)

（2）通过调用 wx.showShareMenu 并且设置 withShareTicket 为 true ，当用户将小程序转发到任一群聊之后，不再支持获取到此次转发的 shareTicket。但是当此转发卡片在群聊中被其他用户打开时，依然可以在 App.onLaunch() 或 App.onShow 获取到 shareTicket。详情可参考[获取更多转发信息](https://developers.weixin.qq.com/miniprogram/dev/api/share.html#%E8%8E%B7%E5%8F%96%E6%9B%B4%E5%A4%9A%E8%BD%AC%E5%8F%91%E4%BF%A1%E6%81%AF) 


