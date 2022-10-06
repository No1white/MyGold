# Foreach与for循环哪个性能更好？

先说`结论`：肯定是`for`循环性能更好，为什么呢？ 如有有去研究过`forEach`底层的小伙伴应该知道`forEach`就是对`for`进行了一次二次封装，那么`forEach`还需要考虑创建上下文等等操作，性能肯定不如纯粹的`for`循环

无凭无据数据为证：

```
    let arrs = new Array(100000);
    console.time('for');
    for (let i = 0; i < arrs.length; i++) {
    };
    console.timeEnd('for');
    console.time('forEach');
    arrs.forEach((arr) => {
    });
    console.timeEnd('forEach');
```

10万级别的数据

```
    for: 0.97216796875 ms
	forEach: 0.22509765625 ms
```

可以看出for性能低于forEach好几倍

100万级别数据两者迅速拉近

```
for: 2.841796875 ms
forEach: 2.39697265625 ms
```

1000万数据,for比forEach快乐好几倍

```
for: 7.574951171875 ms
forEach: 24.1201171875 ms
```

那我们平常场景使用可能会比较少遇到这么多的数据，但是我们从数据证明也依然得成为for性能比forEach好，最主要的原因也是因为forEach是针对for进行封装的

## forEach

先看一下forEach的实现原理

```
		Array.prototype.myForEach = function(f) {
            // this指向的是函数的调用者
            for (let index = 0; index < this.length; index++) {
                f(this[index], index, this);
            }
        }
```

forEach循环的内部实现也是通过for去遍历数组，在通过回调返回参数，看到这里应该就可以理解为什么forEach会比for性能要差了吧