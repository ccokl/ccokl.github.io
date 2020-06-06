```javascript
render(createElement) {
  return createElement('h1')
}
```

渲染函数`render`始终返回一个由 `createElement()`创建出来的`Vnode`

### createElement

- @return {Vnode}

- 入参

  {String | Object | Function}- HTML标签，Vue组件对象，一个return HTML标签，Vue组件对象的异步函数

  ```javaScript
  new Vue({
    render: h => h(App)
  }).$mount("#app");
  ```

  {Object}-可选，与模板中属性对应的数据对象

  ```javaScript
  Vue.component('custom-element', {
      render: function (createElement) {
          // 第一个参数是一个简单的HTML标签字符 “必选”
          // 第二个参数是一个包含模板相关属性的数据对象 “可选”
          return createElement('div', {
              'class': {
                  foo: true,
                  bar: false
              },
              style: {
                  color: 'red',
                  fontSize: '14px'
              },
              attrs: {
                  id: 'boo'
              },
              domProps: {
                  innerHTML: 'Hello Vue!'
              }
          })
      }
  })
  ```

  {String | Array}-可选

  // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成， 

   // 也可以使用字符串来生成“文本虚拟节点”

  ```javaScript
  Vue.component('blog-post', {
    render: function (createElement) {
      var header = this.$slots.header
      var body   = this.$slots.default
      var footer = this.$slots.footer
      return createElement('div', [
        createElement('header', header),
        createElement('main', body),
        createElement('footer', footer)
      ])
    }
  })
  ```

  #### v2.5.18 开始vue 支持 vnode 复用

```javascript
var Child = {
  props: {
    text: {},
  },
  render: function(createElement) {
    return createElement('p', this.text);
  },
};
Vue.component('test', {
  data() {
    return {
      text:'test'
    }
  },
  render: function (createElement) {
    var myParagraphVNode = createElement(Child, {
      props: { text: this.text },
      nativeOn: {
        click: () => {
          this.text = 'clicked ' + new Date();
        },
      }
    })
    return createElement('div', [
      myParagraphVNode, myParagraphVNode
    ])
  }
})
```

### [JSX语法糖](https://github.com/vuejs/jsx#installation)

```javaScript
Vue.component('blog-post', {
  render() {
    return (
     <div>
      <header slot="header">header</header>
      <p>default</p>
      <footer solt="footer">footer</footer>
     </div>
    )
  }
})
```

### [函数式组件](https://cn.vuejs.org/v2/guide/render-function.html)

无状态组件，组件比较简单，没有管理任何状态，也没有监听任何传递给它的状态，也没有生命周期方法，只是一个接受一些 prop 的函数。无状态 (没有[响应式数据](https://cn.vuejs.org/v2/api/#选项-数据))，也没有实例 (没有 `this` 上下文)

```javaScript
Vue.component('my-component', {
  functional: true,
  // Props 是可选的
  props: {
    // ...
  },
  // 为了弥补缺少的实例
  // 提供第二个参数作为上下文
  render: function (createElement, context) {
    // ...
  }
})
```

使用函数式组件时，该引用将会是 HTMLElement。

[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)

```javaScript
// template 模板函数式组件
<template functional>
</template>

// render函数式组件
<script>
  export default {
    functional:true,
    render(){
      return (
        <div>
        <header>header</header>
        <p>default</p>
        <footer>footer</footer>
      </div>
      )
    }
  };
</script>
```

```javaScript
// 非单文件函数式组件
Vue.component('anchored-heading', {
  functional: true,
  render: function (createElement,context) {
    // 创建 kebab-case 风格的 ID
    var headingId = getChildrenTextContent(context.children)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^-|-$)/g, '')

    return createElement(
      'h' + context.props.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, context.children)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

context:{

- `props`：提供所有 prop 的对象
- `children`：VNode 子节点的数组
- `slots`：一个函数，返回了包含所有插槽的对象
- `scopedSlots`：(2.6.0+) 一个暴露传入的作用域插槽的对象。也以函数形式暴露普通插槽。
- `data`：传递给组件的整个[数据对象](https://cn.vuejs.org/v2/guide/render-function.html#深入数据对象)，作为 `createElement` 的第二个参数传入组件
- `parent`：对父组件的引用
- `listeners`：(2.3.0+) 一个包含了所有父组件为当前组件注册的事件监听器的对象。这是 `data.on` 的一个别名。
- `injections`：(2.3.0+) 如果使用了 [`inject`](https://cn.vuejs.org/v2/api/#provide-inject) 选项，则该对象包含了应当被注入的属性。

}

[Vue.js functional components: what, why, and when?](https://medium.com/js-dojo/vue-js-functional-components-what-why-and-when-439cfaa08713)

[Vue原理之虚拟DOM和render函数](https://juejin.im/post/5cd97c1cf265da03563252fe)

