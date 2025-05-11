# 搭建Docsify网站

官方文档地址：https://docsify.js.org/#/

## 一. 全局安装Docsify

```bash
npm i docsify-cli -g
```

## 二. 初始化Docsify文档

文档初始化后会在当前目录下新建文档架构

```bash
docsify init ./docs
```

## 三. 配置Docsify插件

1.搜索插件配置：
```js
window.$docsify = {
 search: {
        maxAge: 86400000, // 本地缓存时间，单位毫秒
        paths: 'auto', // 自动索引所有页面
        placeholder: '搜索文档...',
        noData: '找不到结果',
        depth: 2 // 搜索标题的深度
      }
}
```

引入cdn:
```js
 <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
```

2.字数以及文档阅读时间插件配置：

插件github及文档：https://github.com/827652549/docsify-count?tab=readme-ov-file

```js
      // 字数同级插件
      count: {
        countable: true,
        fontsize: '0.9em',
        color: 'rgb(90,90,90)',
        language: 'chinese'
      }
```
引入cdn：

```js
 <!-- 字数统计 -->
  <script src="//unpkg.com/docsify-count/dist/countable.js"></script>
```

3.配置黑暗模式：

插件github及文档：https://github.com/Plugin-contrib/docsify-plugin/tree/master/packages/docsify-dark-mode

```js
 // 暗模式
 // 按钮颜色
      themeColor: "#42b983",
      darkMode: {
        dark: {
          background: "#1c2022",
          toggleBtnBg: "#34495e",
          textColor: "#b4b4b4"
        },
        light: {
          background: "white",
          toggleBtnBg: "var(--theme-color)",
          textColor: "var(--theme-color)"
        }
      }
```
引入cdn
```js
// 样式
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-dark-mode@latest/dist/style.min.css" />
// js
<script src="//cdn.jsdelivr.net/npm/docsify-dark-mode@latest/dist/index.min.js"></script>
```
此处如果同时引入字数与暗黑模式插件时会导致插件位置重叠，可以找到对应样式属性进行修改
```css
#dark_mode{
    position: absolute;
    right: 0px;
    top: -30px;
}
```