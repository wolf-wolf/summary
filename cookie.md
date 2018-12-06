# say say Cookie

## 一、基本概念

### 1. 什么是Cookie

**Cookie**是存储于电脑文本文件中的一些数据，以名/值对形式存储。进行HTTP通讯时存在于报文头中，服务端和前端均可获取和操作（httpOnly标记的前端不可获取），主旨在于协助无状态的HTTP协议进行稳定的数据记录，因为是明文传输，存在信息泄露等安全隐患。一个单一完整的Cookie格式如下所示：

```
name=value; Expires=Wed, 21 Nov 2018 08:06:47 GMT; Path=/; Domain=www.xxx.com; Secure; HttpOnly
```

### 2. 主要用途
Cookie主要用于以下三个方面：

* 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
* 个性化设置（如用户自定义设置、主题等）
* 浏览器行为跟踪（如跟踪分析用户行为等）

### 3. 组成

#### （1）基本
**name**：数据的标识
**value**：数据的值

#### （2）时效
时效定义了浏览器留存Cookie的时间限度。默认情况下，Cookie 在浏览器关闭时删除
**Expires属性**：到期时间，值为UTC 格式（可以使用`Date.prototype.toUTCString()`进行格式转换）
**Max-Age属性**：有效期，值为相对于浏览器接收到Cookie之后的秒数

#### （3）作用域
**Domain属性**：指定哪些主机可以接受Cookie。默认情况下，为当前文档的主机（不包含子域名）
**Path属性**：指定主机下的哪些路径可以接受Cookie（该URL路径必须存在于请求URL中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。

#### （4）安全
**Secure标记**：只通过被HTTPS协议加密过的请求发送给服务端
**httpOnly标记**：客户端 JavaScript 脚本无法获取该Cookie

### 4. 分类

 - 按存储位置：可分为内存Cookie和硬盘Cookie
 - 按存在时间：可分为非持久（会话期）Cookie和持久Cookie

**会话期Cookie**：Cookie仅在会话期内有效，浏览器关闭后自动删除，无需指定过期时间（Expires）或者有效期（Max-Age）。若浏览器提供了*会话恢复功能*，则会保留该Cookie。

**持久性Cookie**：需指定特定的过期时间（Expires）或有效期（Max-Age）。过期时间设定后，日期和时间只与客户端相关，而非服务端。

### 5. 缺点及限制
1. Cookie会被附加在每个HTTP请求中，所以无形中增加了流量。
2. 由于在HTTP请求中的Cookie是明文传递的，所以安全性成问题，除非用HTTPS。
3. Cookie的大小限制在4KB左右，不同浏览器对大小的限制有所不同。
4. 单个域名设置的 Cookie 不应超过30个，不同浏览器对数量的限制有所不同。
5. 有的浏览器会限制所有网站Cookie的总个数。

## 二、前端操作

### 1. GET
```javascript
/**
 * 以对象的形式，获取所有的cookie字段
 * @returns {*}
 */
function getCookies() {
    const ca = document.cookie && document.cookie.replace(/\s+/g, '').split(';');
    let res = null;
    if (ca.length) {
        res = {};
        ca.forEach(item => {
            let kv = item.split('=');
            res[kv[0]] = kv[1];
        });
    }

    return res;
}

/**
 * 获取制定cookie字段
 * @param {String} name - 需要获取的cookie字段名称
 * @returns {RegExpExecArray | string | string}
 */
function getCookieByName(name) {
    let cookie = document.cookie.replace(/\s/g, "");
    let pattern = new RegExp(name + '=([^;]*);?');
    let res = pattern.exec(cookie);

    return res && encodeURIComponent(res[1]) || '';
}
```

### 2. SET&MODIFY
```javascript
/**
 * 设置cookie字段
 * @param {String} name - cookie的name
 * @param {String} value
 * @param {Object} [config] - 其他设置
 * @param {Number | String} [config.exDays] - expires的设置
 * @param {String} [config.path] - path的设置参数
 * @param {String} [config.domain] - domain的设置参数
 */
function setCookie(name, value, config) {
    let expires = '', path = '', domain = '';

    if (config) {
        if (config.exDays) {
            let d = new Date();
            d.setTime(d.getTime() + (config.exDays * 24 * 60 * 60 * 1000));
            expires = `expires=${d.toGMTString()};`;
        }

        config.path && (path = `path=${config.path}`);

        config.domain && (domain = `domain=${config.domain}`);
    }

    document.cookie = `${name}=${encodeURIComponent(value)}; ${expires} ${path} ${domain}`;
}
```
### 3. CHECK
```javascript
/**
 * 检测特定的cookie是否存在
 * @param name
 * @returns {boolean}
 */
function checkCookie(name) {
    return !!getCookieByName(name);
}
```

## 三、安全性
虽然Cookie可以帮助无状态的HTTP协议进行稳定的数据记录，但是其本身的机制导致在使用时存在诸多安全隐患，如下举例并非全部。`因此在使用Cookie进行信息记录时，应该注意尽量减少私密敏感信息的保存（如密码等），并要对安全性做一定防范和考量`。
1. 会话劫持和XSS
2. 跨站请求伪造（CSRF）
3. 发布假的子域：DNS缓存污染
4. 网络窃听

## 四、新特性

### 1. SameSite Cookie
SameSite Cookie允许服务器要求某个cookie在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）。但目前SameSite Cookie还处于实验阶段，并不是所有浏览器都支持

## 五、总结
本文仅是对Cookie进行基本阐述，或者说更像是对基础概念的分类整理。在最后给出了JavaScript的简单处理方法，以便使用。关于服务端对Cookie的设置，以及跨域的处理，在这里并未提及，感觉可以作为单章来写。

业界对Cookie的看法褒贬不一，各种替代方案也是层出不穷，有兴趣的看官可以Google下查阅，Wikipedia中也有基本简述，在此不再赘述。自我观点看来，没有最好的技术，只有最适合当下的技术。

写这篇文章也是颇有感悟，Cookie算是技术的基本知识，但是在实际开发中，若未使用到可能便不会深究，往往看过之后也会逐渐淡忘。所以对技术的认知不在于知道的多么细致，而应该是对其主体有整体层次的认知和析构，在碰触时，能够思考基本思路，知道要问什么，知道要查什么，就够了。以上为个人见解，若看官有不同意见欢迎留言~

祝好~

## 相关参考
[wikipedia](https://zh.wikipedia.org/wiki/Cookie)

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)

[前端必备HTTP技能之cookie技术详解](https://www.jianshu.com/p/2ceeaef92f20)

[Http协议中Cookie详细介绍](https://www.cnblogs.com/bq-med/p/8603664.html)

[细说Cookie](https://www.cnblogs.com/fish-li/archive/2011/07/03/2096903.html)

[关于Cookie的知识的总结](https://blog.csdn.net/p77ll9l53x/article/details/72675645)

[Cookie详解](https://blog.csdn.net/zcl_love_wx/article/details/51992999)

[菜鸟教程](http://www.runoob.com/js/js-cookies.html)

[阮一峰JavaScript标准参考教程](http://javascript.ruanyifeng.com/bom/cookie.html)