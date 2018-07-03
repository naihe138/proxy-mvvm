### Proxy、Reflect的简单概述

>Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。
出自阮一峰老师的ECMAScript 6 入门，详细点击[http://es6.ruanyifeng.com/#docs/proxy](http://es6.ruanyifeng.com/#docs/proxy)

例如：

````
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
````

上面代码对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为。这里暂时先不解释具体的语法，只看运行结果。对设置了拦截行为的对象obj，去读写它的属性，就会得到下面的结果。

````
obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2
````

````
var proxy = new Proxy(target, handler);
````
这里有两个参数，`target`参数表示所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为。

>注意，要使得`Proxy`起作用，必须针对`Proxy`实例（上例是`proxy`对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。



`Reflect对`象与`Proxy`对象一样，也是 `ES6` 为了操作对象而提供的新` API`。

`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是Proxy对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

同样也放上阮一峰老师的链接[http://es6.ruanyifeng.com/#docs/reflect](http://es6.ruanyifeng.com/#docs/reflect)

### 初始化结构

看到这里，我就当大家有比较明白`Proxy`（代理）是做什么用的，然后下面我们看下要做最终的图骗。
![](https://user-gold-cdn.xitu.io/2018/7/2/1645a981eb020e37?w=564&h=475&f=gif&s=57031)

看到上面的图片，首先我们新建一个`index.html`，然后里面的代码是这样子滴。很简单

````
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>简单版mvvm</title>
</head>
<body>
<div id="app">
    <h1>开发语言：{{language}}</h1>
    <h2>组成部分：</h2>
    <ul>
        <li>{{makeUp.one}}</li>
        <li>{{makeUp.two}}</li>
        <li>{{makeUp.three}}</li>
    </ul>
    <h2>描述：</h2>
    <p>{{describe}}</p>
    <p>计算属性：{{sum}}</p>
    <input placeholder="123" v-module="language" />
</div>
<script>
// 写法和Vue一样
const mvvm = new Mvvm({
    el: '#app',
    data: {
        language: 'Javascript',
        makeUp: {
            one: 'ECMAScript',
            two: '文档对象模型（DOM）',
            three: '浏览器对象模型（BOM）'
        },
        describe: '没什么产品是写不了的',
        a: 1,
        b: 2
    },
    computed: {
        sum() {
        return this.a + this.b
    }
})
</script>
</body>
</html>

````
看到上面的代码，大概跟`vue`长得差不多，下面去实现`Mvvm`这个构造函数

### 实现Mvvm这个构造函数

首先声明一个`Mvvm`函数，`options`当作参数传进来，`options`就是上面代码的配置啦，里面有`el`、`data`、`computed`~~

````
function Mvvm(options = {}) {
    // 把options 赋值给this.$options
    this.$options = options
    // 把options.data赋值给this._data
    let data = this._data = this.$options.data
    let vm = initVm.call(this)
    return this._vm
}
````

上面Mvvm函数很简单，就是把`参数options` 赋值给`this.$options`、把`options.data`赋值给`this._data`、然后调用初始化`initVm`函数,并用`call`改变`this`的指向，方便initVm函操作。然后返回一`个this._vm`，这个是在`initVm`函数生产的。

下面继续写initVm函数，

````

function initVm () {
    this._vm = new Proxy(this, {
        // 拦截get
        get: (target, key, receiver) => {
            return this[key] || this._data[key] || this._computed[key]
        },
        // 拦截set
        set: (target, key, value) => {
            return Reflect.set(this._data, key, value)
        }
    })
    return this._vm
}

````

这个`init函数`用到`Proxy`拦截了，`this`对象，生产`Proxy`实例的然后赋值给`this._vm`，最后返回`this._vm`，上面我们说了，要使得`Proxy`起作用，必须针对Proxy实例。

在代理里面，拦截了`get`和`set`，`get函数`里面，返回`this`对象的对应的`key`的值，没有就去`this._data`对象里面取对应的`key`，再没有去`this._computed`对象里面去对应的`key`值。`set函数`就是直接返回修改`this._data`对应`key`。



做好这些各种拦截工作。我们就可以直接从实力上访问到我们相对应的值了。（mvvm使我们第一块代码生成的实例）

````
mvvm.b // 2
mvvm.a // 1
mvvm.language // "Javascript"
````

![](https://user-gold-cdn.xitu.io/2018/7/2/1645b08c5cd9df96?w=1150&h=445&f=png&s=96660)

如上图看控制台。可以设置值，可以获取值，但是这不是响应式的。

打开控制台看一下
![](https://user-gold-cdn.xitu.io/2018/7/3/1645bbf83c26e6f8?w=2012&h=1032&f=jpeg&s=196604)

可以详细的看到。只有`_vm`这个是`proxy`，我们需要的是，`_data`下面所有数据都是有拦截代理的；下面我们就去实现它。

### 实现所有数据代理拦截

我们首先在`Mvvm`里面加一个`initObserve`，如下

````
function Mvvm(options = {}) {
    this.$options = options
    let data = this._data = this.$options.data
    let vm = initVm.call(this)
+   initObserve.call(this, data) // 初始化data的Observe
    return this._vm
}
````
`initObserve`这个函数主要是把，`this._data`都加上代理。如下

````

function initObserve(data) {
    this._data = observe(data) // 把所有observe都赋值到 this._data
}

// 分开这个主要是为了下面递归调用
function observe(data) {
    if (!data || typeof data !== 'object') return data // 如果不是对象直接返回值
    return new Observe(data) // 对象调用Observe
}
````
下面主要实现Observe类

````
// Observe类
class Observe {
    constructor(data) {
        this.dep = new Dep()
        for (let key in data) {
            data[key] = observe(data[key]) // 递归调用子对象
        }
        return this.proxy(data)
    }
    proxy(data) {
      let dep = this.dep
      return new Proxy(data, {
        get: (target, key, receiver) => {
          return Reflect.get(target, key, receiver)
        },
        set: (target, key, value) => {
          const result = Reflect.set(target, key, observe(value)) // 对于新添加的对象也要进行添加observe
          return result  
        }
      })
    }
  }

````

这样子，通过我们层层递归添加`proxy`，把我们的`_data`对象都添加一遍，在看一下控制台


![](https://user-gold-cdn.xitu.io/2018/7/3/1645bcb17b026063?w=1678&h=818&f=png&s=162675)

很不错，很王祖蓝式的完美。

看到我们的html的界面，都是没有数据的，数据我们都准备好了，下面我们就开始把数据结合到html的界面上。

### 套数据，实现hmtl界面

先把计算属性这个html注释掉，后面进行实现

```
<!-- <p>计算属性：{{sum}}</p> -->
```

然后在Mvvm函数中增加一个编译函数，➕号表示是添加的函数

```
function Mvvm(options = {}) {
    this.$options = options
    let data = this._data = this.$options.data
    let vm = initVm.call(this)
+   new Compile(this.$options.el, vm) // 添加一个编译函数
    return this._vm
}
```

上面我们添加了一个`Compile`的构造函数。把配置的`el`作为参数传机进来，把生成`proxy`的实例`vm`也传进去，这样子我们就可以拿到`vm`下面的数据嘛，下面我们就去实现它。顺序读注释就可以了，很好理解

````
// 编译类
class Compile {
    constructor (el, vm) {
        this.vm = vm // 把传进来的vm 存起来，因为这个vm.a = 1 没毛病
        let element = document.querySelector(el) // 拿到 app 节点
        let fragment = document.createDocumentFragment() // 创建fragment代码片段
        fragment.append(element) // 把app节点 添加到 创建fragment代码片段中
        this.replace(fragment) // 套数据函数
        document.body.appendChild(fragment) // 最后添加到body中
    }
    replace(frag) {
        let vm = this.vm // 拿到之前存起来的vm
        // 循环frag.childNodes
        Array.from(frag.childNodes).forEach(node => {
            let txt = node.textContent // 拿到文本 例如："开发语言：{{language}}"
            let reg = /\{\{(.*?)\}\}/g // 定义匹配正则
            if (node.nodeType === 3 && reg.test(txt)) {
            
                replaceTxt()
                
                function replaceTxt() {
                    // 如果匹配到的话，就替换文本
                    node.textContent = txt.replace(reg, (matched, placeholder) => {
                        return placeholder.split('.').reduce((obj, key) => {
                            return obj[key] // 例如：去vm.makeUp.one对象拿到值
                        }, vm)
                    })
                }
            }
            // 如果还有字节点，并且长度不为0 
            if (node.childNodes && node.childNodes.length) {
                // 直接递归匹配替换
                this.replace(node)
            }
        })
    }
}

````

上面的编译函数，总之就是一句话，千方百计的把{{xxx}}的占位符通过正则替换成真实的数据。

然后刷新浏览器，铛铛档铛铛档，就出现我们要的数据了。

![](https://user-gold-cdn.xitu.io/2018/7/2/1645b3fca7857144?w=594&h=448&f=png&s=67866)

很好很好，但是我们现在的数据并不是改变了 就发生变化了。还需要订阅发布和watcher来配合，才能做好改变数据就发生变化了。下面我们先实现订阅发布。


### 实现订阅发布

订阅发布其实是一种常见的程序设计模式，简单直白来说就是:
>把函数push到一个数组里面，然后循环数据调用函数。

例如：举个很直白的例子

````
let arr = [] 
let a = () => {console.log('a')}

arr.push(a) // 订阅a函数
arr.push(a) // 又订阅a函数
arr.push(a) // 双订阅a函数

arr.forEach(fn => fn()) // 发布所有

// 此时会打印三个a
````
很简单吧。下面我们去实现我们的代码

````
// 订阅类
class Dep {
    constructor() {
        this.subs = [] // 定义数组
    }
    // 订阅函数
    addSub(sub) {
        this.subs.push(sub)
    }
    // 发布函数
    notify() {
        this.subs.filter(item => typeof item !== 'string').forEach(sub => sub.update())
    }
}
````
订阅发布是写好了，但是在什么时候订阅，什么时候发布？？这时候，我们是在数据获取的时候订阅`watcher`，然后在数据设置的时候发布`watcher`，在`Observe`类里面里面,看➕号的代码。 .

````
... //省略代码
...
proxy(data) {
    let dep = this.dep
    return new Proxy(data, {
        // 拦截get
        get: (target, prop, receiver) => {
+           if (Dep.target) {
                // 如果之前是push过的，就不用重复push了
                if (!dep.subs.includes(Dep.exp)) {
                    dep.addSub(Dep.exp) // 把Dep.exp。push到sub数组里面
                    dep.addSub(Dep.target) // 把Dep.target。push到sub数组里面
                }
+           }
            return Reflect.get(target, prop, receiver)
        },
        // 拦截set
        set: (target, prop, value) => {
            const result = Reflect.set(target, prop, observe(value))
+           dep.notify()
            return result  
        }
    })
}
````

上面代码说到，watcher是什么鬼？然后发布里面的sub.update()又是什么鬼？？

带着一堆疑问我们来到了watcher

### 实现watcher

看详细注释

```
// Watcher类
class Watcher {
    constructor (vm, exp, fn) {
        this.fn = fn // 传进来的fn
        this.vm = vm // 传进来的vm
        this.exp = exp // 传进来的匹配到exp 例如："language"，"makeUp.one"
        Dep.exp = exp // 给Dep类挂载一个exp
        Dep.target = this // 给Dep类挂载一个watcher对象，跟新的时候就用到了
        let arr = exp.split('.')
        let val = vm
        arr.forEach(key => {
            val = val[key] // 获取值，这时候会粗发vm.proxy的get()函数，get()里面就添加addSub订阅函数
        })
        Dep.target = null // 添加了订阅之后，把Dep.target清空
    }
    update() {
        // 设置值会触发vm.proxy.set函数，然后调用发布的notify，
        // 最后调用update，update里面继续调用this.fn(val)
        let exp = this.exp
        let arr = exp.split('.')
        let val = this.vm
        arr.forEach(key => {
            val = val[key]
        })
        this.fn(val)
    }
}

```

Watcher类就是我们要订阅的watcher，里面有回调函数fn，有update函数调用fn，

我们都弄好了。但是在哪里添加watcher呢？？如下代码


在Compile里面

````
...
...
function replaceTxt() {
    node.textContent = txt.replace(reg, (matched, placeholder) => {
+       new Watcher(vm, placeholder, replaceTxt);   // 监听变化，进行匹配替换内容
        return placeholder.split('.').reduce((val, key) => {
            return val[key]
        }, vm)
    })
}

````

添加好有所的东西了，我们看一下控制台。修改发现果然起作用了。


![](https://user-gold-cdn.xitu.io/2018/7/3/1645be3fb90b3df6?w=1490&h=962&f=gif&s=250133)


然后我们回顾一下所有的流程，然后看见古老(我也是别的地方弄来的)的一张图。

帮助理解嘛

![](https://user-gold-cdn.xitu.io/2018/4/11/162b38ab2d635662?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

响应式的数据我们都已经完成了，下面我们完成一下双向绑定。

### 实现双向绑定

看到我们html里面有个`<input placeholder="123" v-module="language" />`，`v-module`绑定了一个`language`，然后在`Compile类`里面的`replace函数`，我们加上

````
replace(frag) {
    let vm = this.vm
    Array.from(frag.childNodes).forEach(node => {
        let txt = node.textContent
        let reg = /\{\{(.*?)\}\}/g
        // 判断nodeType
+       if (node.nodeType === 1) {
            const nodeAttr = node.attributes // 属性集合
            Array.from(nodeAttr).forEach(item => {
                let name = item.name // 属性名
                let exp = item.value // 属性值
                // 如果属性有 v-
                if (name.includes('v-')){
                    node.value = vm[exp]
                    node.addEventListener('input', e => {
                        // 相当于给this.language赋了一个新值
                        // 而值的改变会调用set，set中又会调用notify，notify中调用watcher的update方法实现了更新操作
                        vm[exp] = e.target.value
                    })
                }
            });
+       }
        ...
        ...
    }
  }

````

上面的方法就是，让我们的`input`节点绑定一个`input事件`，然后当`input事件`触发的时候，改变我们的值，而值的改变会调用`set`，`set`中又会调用`notify`，`notify`中调用`watcher`的`update`方法实现了更新操作。

然后我们看一下，界面
![](https://user-gold-cdn.xitu.io/2018/7/3/1645d969a74cd1b8?w=1304&h=972&f=gif&s=173896)

双向数据绑定我们基本完成了，别忘了，我们上面还有个注释掉的计算属性。

### 计算属性

先把`<p>计算属性：{{sum}}</p>`注释去掉，以为上面一开始initVm函数里面，我们加了这个代码`return this[key] || this._data[key] || this._computed[key]`，到这里大家都明白了，只需要把this._computed也加一个watcher就好了。


````
function Mvvm(options = {}) {
    this.$options = options
    let data = this._data = this.$options.data
    let vm = initVm.call(this)
    initObserve.call(this, data)
+   initComputed.call(this) // 添加计算函数，改变this指向
    new Compile(this.$options.el, vm)
    return this._vm
}


function initComputed() {
    let vm = this
    let computed = this.$options.computed // 拿到配置的computed
    vm._computed = {}
    if (!computed) return // 没有计算直接返回
    Object.keys(computed).forEach(key => {
        // 相当于把sum里的this指向到this._vm，然后就可以拿到this.a、this、b
        this._computed[key] = computed[key].call(this._vm)
        // 添加新的Watcher
        new Watcher(this._vm, key, val => {
            // 每次设置的时候都会计算
            this._computed[key] = computed[key].call(this._vm)
        })
    })
}

````

上面的initComputed 就是添加一个watcher，大致流程：

this._vm改变 ---> vm.set() ---> notify() -->update()-->更新界面

最后看看图片

![](https://user-gold-cdn.xitu.io/2018/7/3/1645e27f452f7159?w=1238&h=974&f=gif&s=329899)

一切似乎没什么毛病~~~~


### 添加mounted钩子

添加mounted也很简单

````
// 写法和Vue一样
let mvvm = new Mvvm({
    el: '#app',
    data: {
        ...
        ...
    },
    computed: {
        ...
        ...
    },
    mounted() {
        console.log('i am mounted', this.a)
    }
})

````
在new Mvvm里面添加mounted，
然后到function Mvvm里面加上

````
function Mvvm(options = {}) {
    this.$options = options
    let data = this._data = this.$options.data
    let vm = initVm.call(this)
    initObserve.call(this, data)
    initComputed.call(this)
    new Compile(this.$options.el, vm)
+   mounted.call(this._vm) // 加上mounted，改变指向
    return this._vm
}

// 运行mounted
+ function mounted() {
    let mounted = this.$options.mounted
    mounted && mounted.call(this)
+ }

````

执行之后会打印出

````
i am mounted 1
````

完结~~~~撒花
