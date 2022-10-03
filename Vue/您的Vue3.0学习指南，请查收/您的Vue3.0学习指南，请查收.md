# 您的Vue3学习指南，请查收！！！

## 前言

前端真的是太卷了，`vue2.0`学透，`vue3.0`就又来了，感觉前端更新迭代是真的快，但是又不能不学，加油卷王，卷死他们，宁死也不愿意淹没的人流之中。![20200830183701_3ZzSR](https://raw.githubusercontent.com/No1white/picGo/main/img/20220806090240.jpeg)

## 思维导图

![image-20220806090042036](https://raw.githubusercontent.com/No1white/picGo/main/img/20220806090042.png)

## 响应原理

想必大家对`Vue2`的`源码`了解颇深，不了解也没关系

Vue2的响应式`主要是基于Object.defineProperty`方法实现

Vue3的响应式`主要是基于Es6的Proxy`实现

有什么区别呢

首先是`Vue2`，大家在编写代码中一定有发现几个问题

- Vue2对于`对象`新增`属性`是通过`Vue.$set`方法才能被监听到
- 数组直接`下标赋值`是无法实现刷新,需要通过`splice`或者用`this.$set`方法

造成这些现象就是因为`Vue2.0`响应式是基于`Object.definePropety`方法

举个例子:

```
	// 响应式函数
    function reactive(obj, key, value) {
        Object.defineProperty(obj, key, {
            get() {
                console.log(`访问了${key}属性`)
                return value
            },
            set(val) {
                console.log(`将${key}由->${value}->设置成->${val}`)
                if (value !== val) {
                    value = val
                }
            }
        })
    }
    const obj = {
        name: '空白',
        age: 18
    }
    for (let key in obj) {
        reactive(obj, key, obj[key]);
    }
    obj.name = '张三' // 触发set打印将name由->空白->设置成->张三
    obj.birth = '1999.10' // 没有触发set这里birth属性没有通过defineProperty方法定义
```

从上述例子大家就应该了解，为什么Vue2的时候如果想要改变对象`属性`或者`数组`为什么用要`$set`去实现了吧

接下来看一下`Vue3`的Proxy

```
    function reactive(target) {
        const handler = {
            get(target, key, receiver) {
                console.log(`访问了${key}属性`)
                return Reflect.get(target, key, receiver)
            },
            set(target, key, value, receiver) {
                console.log(`将${key}由->${target[key]}->设置成->${value}`)
                Reflect.set(target, key, value, receiver)
                return true;
            }
        }

        return new Proxy(target, handler)
    }
    const obj = {
        name: '空白',
        age: 18
    }
    const newObj = reactive(obj)
    newObj.name = '张三' // 触发set打印将name由->空白->设置成->张三
    newObj.birth = '1999.10' // 触发set将birth由->undefined->设置成->1999.10
```

可以看到如果是通过`Proxy`则可以直接添加新的`属性`同时也能被监听到。

## Proxy知识补充(了解可以跳过)

`Proxy`对象是用于创建对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

`简单说`：通过`Proxy`我们可以对`属性查找(get)、赋值(set)、枚举、函数调用`的时候，自定义一些操作。

```
//在以下简单的例子中，当对象中不存在属性名时，默认返回值为 37。下面的代码以此展示了 get handler 的使用场景。
//这里就是把对象的属性查找做了处理，当获取不到属性名的时候就返回37 
const handler = {
    get: function(obj, prop) {
        return prop in obj ? obj[prop] : 37;
    }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);      // 1, undefined
console.log('c' in p, p.c); // false, 37
```

## 组合式Api

这啥东西呀，又整活了

![image-20220730190415013](https://raw.githubusercontent.com/No1white/picGo/main/img/20220730190422.png)

`setup`：替代了以前的`beforeCreate和create`，同时还不能访问`this`,那么肯定也获取不到`data`、`computed`、`methods`，然而，`setup`返回一个对象，给计算属性、方法、生命周期钩子等使用。`。

### Reacttive

这是什么东西，字面是意思是`反应or响应`，其实这个函数就是如字面一样，创建一个`响应式`的数据

官方解释: `返回对象的响应式副本`

这里也许有人就不太清楚，那么我们这里来一个例子你就懂了

```
import { reactive } from 'vue'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  setup() {
    const obj = reactive({
      name: '张三',
      age: '18'
    });
    // 其实reactive的方法就是帮你把数据转化成响应式的。什么是响应式？ 例如：目前name为张三
    //那么调用setPerson方法，网页上看见会立即更新，注意赋值的方式，因为vue3是使用Proxy方法所以直接赋值属性也是会更新的
    const setPerson = () => {
      obj.name = '李四'; // 注意 这里是直接设置值obj.name 如果是vue而应该用vue.$set
      obj.age = '24'
    }
    return {
      obj,
      setPerson
    }
  }
}
```

### Ref

`ref`与`reactive`函数类似，只是说ref一般是传一个值作为参数，返回一个`响应式`的`ref`对象，另外需要注意的是`ref`若果传的值是`对象`则会被`reactive`函数处理，但是还是有些许不同

例如：

```
const count = ref(0);
const addCount = () => {
	count.value++
}
```

返回Ref对象，值为`0`同时是响应式的

![image-20220731073118976](https://raw.githubusercontent.com/No1white/picGo/main/img/20220731073126.png)

`reactive`和`ref`对象

```
	// ref创建对象
    const person = ref({
      name: '张三',
      age: '18'
    })
    // reacttive创建对象
    const obj = reactive({
      name: '张三',
      age: '18'
    });
    console.log(person);
    console.log(obj);
```

![image-20220731081742687](https://raw.githubusercontent.com/No1white/picGo/main/img/20220731081742.png)

可以看到如果`ref`传的值是一个对象，其实内部`value`本质与`reactive`生成的对象是一致的

### toRefs

`toRefs`：用于解构响应式对象里的属性，同时让解构的`属性`还保留响应式

举个例子：现在我们创建一个`person`对象，里面有`name`、`age`,此时我如果想把`person`里的属性解构出来，那么我直接把它拿出来使用会有什么效果？

```
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  setup() {
    // ref创建对象
    const person = reactive({
      name: '张三',
      age: '18'
    })
    const setPerson = () => {
      person.name = '李四';
      person.age = '24'
    }
    return {
      ...person,
      setPerson,
    }
  }
}
```

![123123](C:%5CUsers%5Clin%5CDesktop%5C20220731090203.gif)

可以看到如果是通过直接解构的方式，`响应式`的效果已经消失了

我们就需要通过`toRefs`把`reactive`的属性解构出来

```
import { reactive, toRefs } from 'vue'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  setup() {
    // ref创建对象
    const person = reactive({
      name: '张三',
      age: '18'
    })
    const { name, age } = toRefs(person) // 解构
    const setPerson = () => {
      person.name = '李四';
      person.age = '24'
    }
    return {
      name,
      age,
      setPerson,
    }
  }
}
</script>
```

![1](https://raw.githubusercontent.com/No1white/picGo/main/img/20220731092359.gif)

可以看到此时我们再去修改`person的属性`已经实时响应到页面上了

### watch

`watch`与`vue2.0`的时候类似，但是可以同时监听多个，这里不过多赘述，大家知道怎么用就可以了

监听单一源

```// 侦听一个 getter
// 侦听一个 getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听一个 ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

监听多个源

```
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

### watchEffect

`watchEffect`:也是用来监听值变化后执行一系列操作，但是最主要的区别是`watchEffect`不用指定监听源，会自动收集，意味着`watchEffect`里的任意值改变都会`触发`

来做个实验吧

```
import { reactive, watchEffect, toRefs } from 'vue'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  setup() {
    // ref创建对象
    const person = reactive({
      name: '张三',
      age: '18'
    })
    watchEffect(() => {
      console.log('person对象改变了', person.name, person.age);
    })
    const setName = () => {
      person.name = '李四'
    }
    const setAge = () => {
      person.age = '23'
    }
    return {
      ...toRefs(person),
      setName,
      setAge
    }
  }
}
```

![2](C:%5CUsers%5Clin%5CDesktop%5C20220731101653.gif)

可以看到我们此时并没有指定对象，但是当里面的属性改变了，`watchEffect`的回调函数就自动执行了

### computed

`计算属性`：与vue2.0基本没什么区别。返回值变成了返回不可变的`ref`对象

```
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // 错误 不能去修改computed返回的对象
```

```
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: val => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

### 生命周期

生命周期其他也没有很大的改变主要是一些`生命周期钩子`的命名变了，在加上`setup`

![img](https://raw.githubusercontent.com/No1white/picGo/main/img/20220731103533.webp)

在下图中可以看到官方推荐用`setup`替代`beforeCreate和create`，同时`setup`的执行是在这两个钩子函数之前的，另外我们在`vue3`中照样可以使用`beforeCreate和create`这两个方法



![img](https://raw.githubusercontent.com/No1white/picGo/main/img/20220731103555.webp)

## Fragment

不得不说在`Vue3`中有一点还是挺舒服的，那就是`Fragment`这个概念其实很简单，就是在`Vue2.0`中`templete`其实是只能有一个`根元素`的而在`vue3`中我们可以有`多个根元素`

举个例子

```
<template>
  <div>Fragment尝试</div>
  <div>Fragment尝试</div>
  <div>Fragment尝试</div>
  <div>Fragment尝试</div>
</template>
```

结果：正常输出

![image-20220803195152056](https://raw.githubusercontent.com/No1white/picGo/main/img/20220803195159.png)

## teleport（传送门）

`teleport`:也是`vue3`中的创新，主要功能是：将某个`元素`渲染到其他指定`位置`，那不是多此一举？有什么用？

不知道大家有没有遇到这样的场景

你要显示某个结果层级不够被`弹窗挡住`？

那么一般这种调用在`组件内`被一层层覆盖，如果我们能将元素挪去与Vue的根元素`同级`那不是美滋滋。

于是就诞生了`teleport`现在我们来实践一下

> 首先在index.html声明元素，并且赋予对应的id

![image-20220803211259858](https://raw.githubusercontent.com/No1white/picGo/main/img/20220803211259.png)

> 接着使用teleport把需要传送的元素包裹，然后就要开始施法了

![image-20220803211505522](https://raw.githubusercontent.com/No1white/picGo/main/img/20220803211505.png)

可以从图片看到我们此时的代码是声明在`HelloWorld`组件中，那么在浏览器中会渲染到哪里？

![image-20220803211617857](https://raw.githubusercontent.com/No1white/picGo/main/img/20220803211617.png)

与`app`根元素同级，并不是在`组件中`，那么到这里我还有个疑问，此时的组件拿的`数据`是哪里的数据？

`HelloWorld`这个层级的吗？ 我们来实验一下，通过点击按钮改变`HelloWorld`组件中的`data`

![23](C:%5CUsers%5Clin%5CDesktop%5C20220803212416.gif)

从动图中我们可以看到，点击按钮后`组件`中的值确实改变了，则可以断定`vue其实只是把元素挪到指定地方`，获取的数据还有逻辑还是从`原有的地方获取`

## Tree-shaking

`Tree-shaking`：俗称`摇树`，功能简单说就是让`打包的体积更小`，去掉不需要的东西，这个其实是属于`webpack`的东西，有兴趣的同学可以去看看`webpack的优化tree-shaking`，这里建议深究，毕竟要把时间花在刀刃上

## Suspense（异步组件）

`Suspense`：目前这个组件还是实验性而已，在以后的版本中有可能被删除也有可能保留，目前只需要做了解即可

> 异步组件，主要是为了处理异步加载时的显示，平常我们在请求还未拿到数据时经常是显示loading

![23](https://raw.githubusercontent.com/No1white/picGo/main/img/20220806083520.gif)

为了处理现实loading在`vue2`的时候我们一般是使用`v-if`判断

在`Vue3`中提供异步组件`Suspense`可以帮助我们解决这个问题

```
<template>
  <div id="center">
    <span>我是主体内容部分</span>
    <Suspense>
      <template #default>
        <C></C>
      </template>
      <template #fallback>
        <div>正在加载中，请稍等。。。。</div>
      </template>
    </Suspense>
  </div>
</template>
<script setup lang="ts">
import { ref, defineAsyncComponent } from 'vue';
const C = defineAsyncComponent(() => import('./C.vue'));
</script>
<style scoped>
#center {
  width: 100%;
  height: 100%;
  background: seagreen;
  text-align: center;
  font-size: 20px;
}
</style>
```

