# 主页改造计划
最近都在做酷Q插件，主页都完全扔一边了，现在基本上都完成了就再回来弄一弄主页吧。
1. 去除`element-UI`库或者更换更轻量的UI库（待定）
2. 更换文本编辑器为`markdown`
3. 整理路由关系
4. 重新构建数据库
5. 后端改用`golang`
6. 完善后台

其它的什么时候想到再慢慢加。

## 主页部分
现在打算加一个入口页面，因为实在不知道怎么做比较好，就选用简洁风吧。
简洁到什么程度你，就是一个纯色banner加上两行字，最多再加上一个箭头。
![主页-第一屏](https://www.feizhouxianyu.cn/img/01.jpg "主页-第一屏")
但是这样有点太单调了，所以加点动态效果吧。所以给小箭头加个动画吧，别人的小箭头都会一跳一跳的，我也加上这个吧。
实现方法比较简单，用`keyframes`动画就好了。
```css
/* 因为使用了stylus所以省略了大括号和分号 */
@keyframes shake
  0%
    transform translate(0, 0)
  50%
    transform translate(0, 25%)
  100%
    transform translate(0, 0)
```
然后绑定到箭头的`class`上，`shake`是`keyframes`动画名，`2s`是动画要用2秒完成，`ease-out`是以慢速结束的过渡效果，`infinite`是循环播放。
```css
animation: shake 2s ease-out infinite
```
好像还是有点单调，那就让副标题也一起都起来吧，不过抖的频率要稍微再快点，而且抖两次。
```css
@keyframes shake2
  0%
    transform translate(0, 0)
  20%
    transform translate(0, 0)
  25%
    transform translate(0, -15%)
  30%
    transform translate(0, 0)
  35%
    transform translate(0, -15%)
  40%
    transform translate(0, 0)
  100%
    transform translate(0, 0)
```
再加个选中时改变文字的效果吧。
```css
&:hover
  transition all 0.3s linear
  color transparent
  animation none
  &:after
    font-size 17px
    position absolute
    top 50%
    left 50%
    transform translate(-50%, -50%)
    width 100%
    content '我都说了没什么用咯...'
    color #fff
```
好了，主页完成了，就这样吧（并不）
毕竟有个箭头了，起码页面要能滚动一下吧。
第二页的内容是之前做好的一个菜单组件，这个以后再说，现在先不管。
![主页-第二屏](https://www.feizhouxianyu.cn/img/02.jpg "主页-第二屏")
但是一般的滚动也太普通了，那就改成滚动的时候内容渐变吧。
首先要把第一屏和第二屏的内容用`position: fixed`固定住，然后加上ref引用，同时在`mounted`钩子加入监听。
```javascript
mounted () {
  window.addEventListener('scroll', (e) => {
    // 计算滚动的比例，0为滚动到顶部，1为滚动到底部
    let scrolled = document.documentElement.scrollTop / (document.documentElement.scrollHeight - document.documentElement.clientHeight)
    if (scrolled >= 0.1) {
      // 动态改变第一屏的透明度和z-index
      this.$refs.banner.style.opacity = 1 - scrolled
      this.$refs.banner.style.zIndex = 1 - scrolled
      this.$refs.content.style.opacity = scrolled
    } else {
      this.$refs.banner.style.opacity = 1
      this.$refs.banner.style.zIndex = 1
      this.$refs.content.style.opacity = 0
    }
  })
}
```

### 完整代码
```javascript
<template>
  <div class="background">
    <div class="banner" ref="banner">
      <div class="text">
        <p class="title">feizhouxianyu.cn</p>
        <p class="tip">一个并没什么用的网站</p>
      </div>
      <div class="icon">↓</div>
    </div>
    <div class="content" ref="content">
      <index-menu class="menu"/>
      <div class="icon">↑</div>
    </div>
  </div>
</template>

<script>
import IndexMenu from '../components/IndexMenu'

export default {
  components: {
    IndexMenu
  },
  mounted () {
    window.addEventListener('scroll', (e) => {
      let scrolled = document.documentElement.scrollTop / (document.documentElement.scrollHeight - document.documentElement.clientHeight)
      if (scrolled >= 0.1) {
        this.$refs.banner.style.opacity = 1 - scrolled
        this.$refs.banner.style.zIndex = 1 - scrolled
        this.$refs.content.style.opacity = scrolled
      } else {
        this.$refs.banner.style.opacity = 1
        this.$refs.banner.style.zIndex = 1
        this.$refs.content.style.opacity = 0
      }
    })
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
.background
  max-width 100vw !important
  height calc(100vh + 100px)
  user-select none
  .banner
    z-index 1
    position fixed
    top 0
    width 100%
    height 100vh
    background #666
    .text
      position absolute
      top 50%
      left 50%
      transform translate(-50%, -50%)
      text-align center
      white-space nowrap
      color #fff
    .title
      font-size 25px
      text-shadow 2px 2px 6px rgba(0, 0, 0, 0.5)
    .tip
      font-size 20px
      transition all 0.3s ease-in
      position relative
      animation shake2 3s linear infinite
      text-shadow 2px 2px 6px rgba(0, 0, 0, 0.5)
      &:hover
        transition all 0.3s linear
        color transparent
        animation none
        text-shadow none
        &:after
          font-size 17px
          position absolute
          top 50%
          left 50%
          transform translate(-50%, -50%)
          width 100%
          content '我都说了没什么用咯...'
          color #fff
          text-shadow 2px 2px 6px rgba(0, 0, 0, 0.5)
  .content
    position fixed
    top 0
    width 100%
    height 100vh
    background #eee
    .icon
      color #000

.icon
  position fixed
  bottom 25px
  left calc(50% - 10px)
  width 20px
  color #fff
  animation shake 2s ease-out infinite

@keyframes shake
  0%
    transform translate(0, 0)
  50%
    transform translate(0, 25%)
  100%
    transform translate(0, 0)

@keyframes shake2
  0%
    transform translate(0, 0)
  20%
    transform translate(0, 0)
  25%
    transform translate(0, -15%)
  30%
    transform translate(0, 0)
  35%
    transform translate(0, -15%)
  40%
    transform translate(0, 0)
  100%
    transform translate(0, 0)
</style>
```