## uniapp

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
