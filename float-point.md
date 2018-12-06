# say say 浮点数

在进行浮点数运算或判断时，经常会出现很多“奇异”的结果，其实都是浮点数运算及表示规则造的怪

```javascript
0.1 + 0.2 = 0.30000000000000004
8.88 * 100 = 888.0000000000001
1050.6 * 100 = 105059.99999999999
```

## 基本原因

1. 浮点数的运算是基于 IEEE 754 标准的；
2. 二进制基准；
3. 对循环小数通过相对误差进行近似表示；

## 相关参考

[Stack Overflow](https://stackoverflow.com/questions/588004/is-floating-point-math-broken)

[Floating Point Math](https://0.30000000000000004.com/)

[What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)

[THE FLOATING-POINT GUIDE](https://floating-point-gui.de/errors/rounding/)

[Wikipedia](https://en.wikipedia.org/wiki/Floating-point)

[阮一峰](http://www.ruanyifeng.com/blog/2010/06/ieee_floating-point_representation.html)

## 总结
1. 尽量少或不用浮点数进行等于判断
2. 进行浮点数运算时，取整需使用`parseInt`，`Math.round`等函数进行
