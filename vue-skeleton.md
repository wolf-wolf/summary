# say say Vue添加骨架屏

## 相关版本信息

```JSON
"vue": "^2.2.2",
"webpack": "^2.2.1"
"html-webpack-plugin": "^2.28.0"
```

## 背景
使用`Vue`和`Webpack`进行**MPA(多页面应用)**的开发，一般会在页面进行数据接口等待时增加Loading动画，为用户提供较好的交互体验。但是会发现，Loading展示的时机是在Vue框架解析后，也就是说需要如下几个条件才能显示：

 1. HTML文件加载完成
 2. JavaScript文件加载完成
 3. window对象中完成webpackJsonp方法的添加

因此：`等待的时间=HTML加载时间+JS加载时间+JS全局环境创建的执行时间`。若是JS资源文件较大，或者存在过多的图片资源，导致资源速度下载较慢时，用户所能看到的白屏时间便较长。因此添加骨架屏，其优势在于：

 1. 写于HTML文件中，独立于Vue框架，节省了`JS加载时间+JS全局环境创建的执行时间`的时间
 2. 只在主页面根据页面结构独立编写，预先展示页面结构，进行视觉暂留，提供更好的交互感官
 3. 只在页面结构变化时进行修改，维护成本相对较低

## 步骤及相关代码

### 在`index.html`文件中添加骨架屏的HTML和CSS代码

使用Vue进行MPA开发，独立页面都存在`index.html`，`index.js`，`index.vue`三个基本文件。

`index.html`文件中定义Vue负责渲染的节点元素，`index.js`中进行Vue的挂载，存在两种写法：

第一种：
```HTML
...
<body>
    <div id="app"></div>
</body>
...
```

```JavaScript
...
let app = new Vue({
    el: '#app',
    components: {App},
    template: '<App/>'
});
...
```

第二种：
```HTML
<body>
...
    <div id="app"><app></app></div>
...
</body>
```

```JavaScript
...
let app = new Vue({
    el: '#app',
    components: {
        app
    }
});
...
```

`注意：需选用第一种方式，如采用第二种方式，则会影响骨架屏的展示`

在使用第一种方式时，`<div id="app"></div>`标签内的任何html结构都会在被Vue动态替换，这也便是进行骨架屏不用主动操作DOM元素的原因。

根据最终页面结构，进行骨架屏开发，并将html和css代码进行压缩混淆后，加入`index.html`中，示例代码如下：

```html
<head>
    ...
    <style>骨架屏的css代码</style>
</head>
<body>
    ...
    <div id="app">骨架屏的html代码</div>
    ...
</body>
```

### 修改JS文件的加载方式

在测试目前已有的插件时发现，由于移动端和PC端浏览器内核对页面渲染的机制*可能（不同的原因尚未找到）*存在不同。`在移动端访问页面时，会发现，在JS文件同步加载时会阻塞页面的渲染，使得骨架屏无法在JS文件加载时展示，当JS文件加载完成时便会自动执行，此时Vue会替换掉div节点中的元素，导致骨架屏代码消失，最终结果便是无法展示骨架屏。`针对此种情况，需要对JS文件加载的方式进行调整。

`HtmlWebpackPlugin`插件的配置属性`inject`为`true`时，会自动将JS和CSS文件注入到`index.html`，根据官方文档提供的选项，可以对JS文件进行sync同步或async异步加载。上文提到同步加载无法满足骨架屏的渲染要求。而且一般进行Webpack打包都会进行代码分割，提取公共代码为独立JS文件，各个页面的文件需要依赖于公共代码文件的优先执行。测试后发现可以通过检测`webpackJsonp`全局方法是否存在判断公用代码文件加载并执行完成，基本流程如下：

 1. 异步加载公用代码JS文件
 2. 针对页面独立JS文件使用`link`标签进行预加载
 3. 设置定时器，检测`webpackJsonp`方法存在，加载页面独立JS文件

Webpack配置文件部分代码如下：
```
...
webpackConfig.plugins.push(new HtmlWebpackPlugin({
    ...
    template: 'index.html',
    inject: false,
    ...
}));
...
```

index.html文件注入代码如下：
```
...
<head>
...
    <!-- 注入CSS样式文件-->
    <% for (var i = 0; i <  htmlWebpackPlugin.files.css.length; i++) { %>
    <link rel="stylesheet" href="<%= htmlWebpackPlugin.files.css[i] %>"/>
    <% }%>

    <!-- 注入除公共代码外的JS文件 -->
    <% for (var chunk in htmlWebpackPlugin.files.chunks) {
        if (!/vendor/.test(chunk)&&htmlWebpackPlugin.files.chunks[chunk]){
    %>
    <script>
        (function () {
            const chunkSrc = '<%=htmlWebpackPlugin.files.chunks[chunk].entry %>';
            // 预加载页面独立JS文件
            const preloadLink = document.createElement('link');
            preloadLink.href = chunkSrc;
            preloadLink.rel = 'preload';
            preloadLink.as = 'script';
            document.head.appendChild(preloadLink);
            // 设置定时器，检测公用文件是否执行OK
            const tick = setInterval(() => {
                if (window && window.webpackJsonp) {
                    clearInterval(tick);
                    // 加载页面独立JS文件
                    const preloadedScript = document.createElement('script');
                    preloadedScript.src = chunkSrc;
                    document.body.appendChild(preloadedScript);
                }
            }, 10);
        })();
    </script>
    <%}}%>
...
</head>
<body>
<% if (htmlWebpackPlugin.files.chunks.vendor) {%>
<script src="<%=htmlWebpackPlugin.files.chunks.vendor.entry %>" async></script>
<% } %>
</body>
...
```

这里使用了插件`HtmlWebpackPlugin`的模板写法，通过`htmlWebpackPlugin`对象进行代码`chunk`的注入，使用方法可参见[官方文档](https://www.npmjs.com/package/html-webpack-plugin)。

## 相关材料

- [骨架屏的渲染时机](https://zhuanlan.zhihu.com/p/34550387)
- [为vue项目添加骨架屏](https://xiaoiver.github.io/coding/2017/07/30/%E4%B8%BAvue%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E9%AA%A8%E6%9E%B6%E5%B1%8F.html)
- [Vue页面骨架屏注入实践](https://segmentfault.com/a/1190000014832185)
- [饿了么的 PWA 升级实践](https://huangxuan.me/2017/07/12/upgrading-eleme-to-pwa/)
- [讲讲PWA](https://segmentfault.com/a/1190000012353473?utm_source=tag-newest)
- [PWA 的探索与最佳实践](http://www.beiyv.com/index.php?m=content&c=index&a=show&catid=14&id=14)
- [vue-skeleton-webpack-plugin骨架屏与page-skeleton-webpack-plugin骨架屏生成插件](https://blog.csdn.net/zmkyf1993/article/details/82866649)
- [这个控件叫：Skeleton Screen/加载占位图](https://zhuanlan.zhihu.com/p/26014116)
