设计师傅想要一个毛玻璃效果，我当时听着一脸懵逼什么是毛玻璃效果，这也太高级了吧，后来她跟我找了一个样例我看了下，就是那种滑动的时候背景隐隐约约可见的效果，很有意思。不过 CSS 里真的有属性可以实现这个效果，就是 backdrop-filter 属性。

用这个属性实现毛玻璃效果有个前提是：元素背景至少部分透明

```css
  background: rgba(235, 235, 235, 0.88);
  -webkit-backdrop-filter: blur(50px);
  backdrop-filter: blur(50px);
```

样例参考：https://css-tricks.com/almanac/properties/b/backdrop-filter/

兼容性：https://caniuse.com/css-backdrop-filter

**有坑的地方**：backdrop-filter 会影响position:fixed的布局

与 backdrop-filter 类似的属性，filter属性表示让当前元素自身模糊或者高亮等

好文章：https://www.zhangxinxu.com/wordpress/2019/11/css-backdrop-filter/

