### 问题
- 精度缺失的原因是什么？
- 为什么最大安全整数是 2^53 - 1？无穷大是 Infinity?
- 为什么 0.1 + 0.2 = 0.30000000000000004 小数点 17 位？而不是 18 位，19 位？而 var x = 0.1 得到的就是 0.1 ? 精度是如何确定的？
- 如何避免精度问题？

### 概念
在 js 中，用的 64 byte 来存储数值，即可存储整数，也可以存储浮点数。 

64 比特位的分解图：
- 第一位 S 表示符号位：正负数
- 中间 11 位表示指数位 E，其中整数的小数部分都是用 负指数 来表示， 11 位的范围是 2^11 - 1 = 2047，对半分，以 1023 位中间值，即 E - 1023 表示当前的指数值。
- 为什么会有指数呢？是因为二进制也用科学计数法表示就是 1 * 2^(E-1023) + M ，其中约定 1 可以省略，因为始终不会改变。省略之后，整个二进制就多出 1 位来了，所以整个二进制位数就变成 53 位。
- 尾数 M 有 52 位，超过的部分根据 IFFF 754 规范是就近“偶数”舍入。

### 解析
```js
// 问题 1
// 例如：0.1，用二进制表示：
0.1.toString(2)
// 0.000 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 11
// 最后的 11 是多余的需要进一舍掉，得到
0.0001100110011001100110011001100110011001100110011001101
// 这个数其实是比 0.1 大的

// 问题 2
// 这个问题其实可以转化为，在安全整数范围呢，每一个二进制位都对应着一个整数。
// 超过这个二进制位，进一舍零后，得到的数值会与前面的重复，即不是唯一表示了。
// 因为尾数是 52 位，用科学记数法表示时，省略的 1 补上一位 就是 53 位。
// 那我们再验证下 2^53 用比特位保存是
Math.pow(2, 53) = 9007199254740992
9007199254740992.toString(2)
// 1.0000..000 即 1 + 53 个零，最后一个 0 会被舍弃
// 之所以可以用 53个0 来表示，是因为我们在用科学记数法表示的时候省略掉了最头上的一位。
// 也就是说 1 * 2^53 ，在实际的二进制存储中，只能存52个0.
// 其实能够得到 2^53 能够精确表示一个数，那为什么不用呢？因为 2^53 + 1 === 2^53
Math.pow(2, 53) === Math.pow(2, 53) + 1 // true
// 2^53 + 1 用二进制位来表示是
100000...0001 // 第53位是一个 1，由于只能存储 52 位
// 所以最后的1被舍掉，舍掉模式是就近舍入，当有两个最接近的可表示的值时首选“偶数”值
// 所以也不算是精确的数值，在 2^53 个指数位上都对应着唯一一个能表示的数
// 所以最大安全整数是 2^53 - 1

// 问题 3
// 最大安全整数是 2^53 - 1 = 9007199254740991 是 16 位
// 据说 v8 引擎对于整数部分的精度最大为 16 位，小数部分默认是 17 位
// 关于 0.30000000000000004 是 17 位的原因猜测：
// 在 17 位精度范围内存在非 0 的数，所以以 17 位显示，否则精度到非 0 的那位数
// 假如 0.1 + 0.2 = 0.30040000000000000 004999
// 得到的数应该是 0.3004

// 问题 4
// 方法 1: 将一个数的精度准确到足够大的位数后再去做 parseFloat。
// 例如 1.005 * 100 = 100.49999999999999
parseFloat((1.005 * 100).toPrecision(16)) = 100.5
// 方法2：将小数转化为整数后再计算
```

### 参考
- [这篇文章得到一个系统认识，但是里面有些逻辑转折太快，理解不了](https://github.com/camsong/blog/issues/9)
- [这篇条评论验证了最大安全整数](https://github.com/camsong/blog/issues/9#issuecomment-385921123)
- http://www.binaryconvert.com/result_double.html 能可视化 64 比特位存储
- https://tool.oschina.net/hexconvert/ 在线的进制转换
- https://github.com/nefe/number-precision 处理 js 精度库
