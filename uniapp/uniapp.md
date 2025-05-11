## uniapp

# 常用组件库

uv-ui(前端交互组件):https://www.uvui.cn/components/intro.html
wot-design-uni(前端交互组件):https://wot-design-uni.netlify.app/
秋云(图表组件):https://www.ucharts.cn/v2/#/

# 分包

pages.json 中配置

```js
 "subPackages": [
    {
      "root": "subpkg/order",
      "pages": [
        {
          "path": "list",
          "style": {
            "navigationBarTitleText": "订单列表"
          }
        }
      ]
    }
  ]
```

# 常用钩子

onLoad(data)：页面加载完毕，可以加载一些不需要实时更新的数据

onShow：每次进入页面时加载

onReady：dom 挂载后执行一次，和 onLoad 执行时间不同

onHide：跳转至其他页面时执行一次，页面没销毁

onUnload：页面卸载/销毁了，每个页面执行一次

# css 原子化

Tailwind css:https://www.tailwindcss.cn/
