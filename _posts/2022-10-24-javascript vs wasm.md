---
layout: page
title: webassmbly benchmark pointInPolygon
date:   2022-10-24 10:30:43 +0800
tags:
  - webassmbly
comments: true
jekyll-theme-WuK:
  default:
    sidebar:
      open: false
  archive:
    group_by: "%b %Y" # 见<https://liquid.bootcss.com/filters/date/>
    vega_lite: # 显示一个统计图，需要引入 vega-lite
      enable: true
---
有这么一个算法：判断圆圈是否在用户画的不规则polygon内，如图:
这是一个比较占用CPU的算法(尤其当数据量大了之后)

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/218124/1587121326019-27235c5d-02df-4d7c-8f83-ba90025c3546.png#align=left&display=inline&height=663&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1004&originWidth=1006&size=1883889&status=done&style=none&width=664)
其中碰撞算法采用 Jordan曲线定理的射线法，js 实现：
```javascript
function isPointInPoly (poly, pt) {
  for(var c = false, i = -1, l = poly.length, j = l - 1; ++i < l; j = i)
    ((poly[i].y <= pt.y && pt.y < poly[j].y) || (poly[j].y <= pt.y && pt.y < poly[i].y))
  && (pt.x < (poly[j].x - poly[i].x) * (pt.y - poly[i].y) / (poly[j].y - poly[i].y) + poly[i].x)
  && (c = !c);
  return c;
}
```
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/218124/1587118482646-c6d67047-46df-4989-9730-cc465e40b9b7.png#align=left&display=inline&height=243&margin=%5Bobject%20Object%5D&name=image.png&originHeight=196&originWidth=317&size=4619&status=done&style=none&width=393)
原理很简单：**从测试点水平运行半无限光线（增加x，固定y），并计算它穿过的边数。 在每个交叉点，射线在内部和外部之间切换。 这被称为Jordan曲线定理。**
每当水平光线穿过任何边缘时，变量c从0切换到1并且从1切换到0。 所以基本上它是跟踪交叉的边数是偶数还是奇数。 0表示偶数，1表示奇数。

考虑到当准备被碰撞的座位很大 这一实际场景时，顾虑到这个算法可能会占据太多scripting time，减少CPU的计算，或者提高运算效率，首先考虑到的就是 **webassembly**~

综合社区内对 webassembly 的适用场景的总结：**单次做复杂运算，和JS线程通讯次数很低的场景**，webassembly的效率完胜javascript ，所以尝试采用相同算法用C语言实现，编译成wasm对比下效率：

测试Case：
_传入一个10个节点的polygon，和一个固定测试Point: {x: 100, y: 100}，循环调用function N次_
_翻译为实际场景就是，用户在一个N个座位的看台内画了一个有10个节点的 polygon，判断多少个座位在polygon内_

javascript: 循环调用isPointInPoly方法2000次。
C: pnpoly function内自循环2000次，减少和JS线程的通信次数。

如下为测试结果： **wasm完胜**
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/218124/1587347476160-b5609264-ca10-47c7-97d6-a427093f4f28.png#align=left&display=inline&height=486&margin=%5Bobject%20Object%5D&name=image.png&originHeight=673&originWidth=952&size=169838&status=done&style=none&width=687)
把N改成2w，再对比一下较大数据量的差异：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/218124/1587382780311-517bb8bd-58de-4878-ac4d-c856028e7158.png#align=left&display=inline&height=466&margin=%5Bobject%20Object%5D&name=image.png&originHeight=686&originWidth=977&size=158874&status=done&style=none&width=663)
可以看到，wasm 的计算性能在这个场景下远超于javascript。
回归到业务场景，虽然webassembly 的性能高于纯JS几十倍，甚至上百倍，但是这个算法在纯JS引擎下毕竟还是毫秒级别的，并且上面的Demo仅是一个测试，还不是实际业务场景，还没有处理wasm的内存分配、js传参，这个复杂度并不低。
