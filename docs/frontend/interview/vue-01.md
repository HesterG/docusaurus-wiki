# vue-01

## vue2 vue3 响应式

[Vue2 和 Vue3 的响应式原理比对](https://juejin.cn/post/7124351370521477128)

[vue2原理探索——响应式系统](https://github.com/LuckyWinty/blog/blob/master/markdown/vue/vue2%E5%8E%9F%E7%90%86%E6%8E%A2%E7%B4%A2--%E5%93%8D%E5%BA%94%E5%BC%8F%E7%B3%BB%E7%BB%9F.md)

### vue2响应式原理

响应式基本原理就是，在 Vue 的构造函数中，对 options 的 data 进行处理。即在初始化vue实例的时候，对data、props等对象的每一个属性都通过Object.defineProperty定义一次，在数据被set的时候，做一些操作，改变相应的视图。

粗略大局代码

```javascript
class Vue {
    /* Vue构造类 */
    constructor(options) {
        this._data = options.data;
        observer(this._data);
    }
}
function observer (value) {
    if (!value || (typeof value !== 'object')) {
        return;
    }
    
    Object.keys(value).forEach((key) => {
        defineReactive(value, key, value[key]);
    });
}
function defineReactive (obj, key, val) {
    Object.defineProperty(obj, key, {
        enumerable: true,       /* 属性可枚举 */
        configurable: true,     /* 属性可被修改或删除 */
        get: function reactiveGetter () {
            return val;         
        },
        set: function reactiveSetter (newVal) {
            if (newVal === val) return;
            cb(newVal);
        }
    });
}
```

Vue2 的对象数据是通过 `Object.defineProperty `对每个属性进行监听，当对属性进行读取的时候，就会触发 getter，对属性进行设置的时候，就会触发 setter。

```javascript
/**
* 这里的函数 defineReactive 用来对 Object.defineProperty 进行封装。
**/
function defineReactive(data, key, val) {
   // 依赖存储的地方
   const dep = new Dep()
   Object.defineProperty(data, key, {
       enumerable: true,     /* 属性可枚举 */
       configurable: true,   /* 属性可被修改或删除 */
       get: function () {
           // 在 getter 中收集依赖
           dep.depend()
           return val
       },
       set: function(newVal) {
           val = newVal
           // 在 setter 中触发依赖
           dep.notify()
       }
   }) 
}
```

那么是什么地方进行属性读取呢？就是在 `Watcher` 里面，`Watcher` 也就是所谓的依赖。在 `Watcher` 里面读取数据的时候，会把自己设置到一个**全局的变量**中。

[Watcher何时产生](https://segmentfault.com/a/1190000019311529)

```javascript
class Watcher{
    constructor(vm,expOrFn,cb,options){
        //传进来的对象 例如Vue
        this.vm = vm
        //在Vue中cb是更新视图的核心，调用diff并更新视图的过程
        this.cb = cb
        //收集Deps，用于移除监听
        this.newDeps = []
        this.getter = expOrFn
        //设置Dep.target的值，依赖收集时的watcher对象
        this.value =this.get()
    }

    get(){
        //设置Dep.target值，用以依赖收集
        pushTarget(this)
        const vm = this.vm
        let value = this.getter.call(vm, vm)
        return value
    }

    //添加依赖
      addDep (dep) {
          // 这里简单处理，在Vue中做了重复筛选，即依赖只收集一次，不重复收集依赖
        this.newDeps.push(dep)
        dep.addSub(this)
      }

      //更新
      update () {
        this.run()
    	}

    //更新视图
    run(){
        //这里只做简单的console.log 处理，在Vue中会调用diff过程从而更新视图
        console.log(`这里会去执行Vue的diff相关方法，进而更新数据`)
    }
}
```

在 Watcher 读取数据的时候也就触发了这个属性的监听 getter，在 getter 里面就需要进行依赖收集，这些依赖存储的地方就叫 Dep，在 Dep 里面就可以把**全局变量中的依赖**进行收集，收集完毕就会把**全局依赖变量**设置为空。将来数据发生变化的时候，就去 Dep 中把相关的 Watcher 拿出来执行一遍。

```javascript
/**
* 我们把依赖收集的代码封装成一个 Dep 类，它专门帮助我们管理依赖。
* 使用这个类，我们可以收集依赖、删除依赖或者向依赖发送通知等。
**/
class Dep {
    constructor() {
        this.subs = []
    }
    
    addSub(sub) {
        this.subs.push(sub)
    }
    
    removeSub(sub) {
        remove(this.subs, sub)
    }

    depend() {
        if(Dep.target){
            this.addSub(Dep.target)
        }
    }

    notify() {
        const subs = this.subs.slice()
        for(let i = 0, l = subs.length; i < l; i++) {
            subs[i].update()
        }
    }
}

// 删除依赖
function remove(arr, item) {
    if(arr.length) {
        const index = arr.indexOf(item)
        if(index > -1){
            return arr.splice(index, 1)
        } 
    }
}
```

总的来说就是通过 `Object.defineProperty` 监听对象的每一个属性，当读取数据时会触发 getter，修改数据时会触发 setter。

然后我们在 getter 中进行依赖收集，当 setter 被触发的时候，就去把在 getter 中收集到的依赖拿出来进行相关操作，通常是执行一个回调函数。

我们收集依赖需要进行存储，对此 Vue2 中设置了一个 Dep 类，相当于一个管家，负责添加或删除相关的依赖和通知相关的依赖进行相关操作。

在 Vue2 中所谓的依赖就是 Watcher。值得注意的是，只有 Watcher 触发的 getter 才会进行依赖收集，哪个 Watcher 触发了 getter，就把哪个 Watcher 收集到 Dep 中。当响应式数据发生改变的时候，就会把收集到的 Watcher 都进行通知。

**由于 Object.defineProperty 无法监听对象的变化，所以 Vue2 中设置了一个 Observer 类来管理对象的响应式依赖，同时也会递归侦测对象中子数据的变化。**

**Observer 类的作用就是把一个对象全部转换成响应式对象**，包括子属性数据，当对象新增或删除属性的时候负债通知对应的 Watcher 进行更新操作。

```javascript
// 定义一个属性
function def(obj, key, val, enumerable) {
    Object.defineProperty(obj, key, {
        value: val,
        enumerable: !!enumerable,
        writable: true,
        configurable: true
    })
}

class Observer {
    constructor(value) {
        this.value = value
        // 添加一个对象依赖收集的选项
        this.dep = new Dep()
        // 给响应式对象添加 __ob__ 属性，表明这是一个响应式对象
        def(value, '__ob__', this)
        if(Array.isArray(value)) {
           
        } else {
            this.walk(value)
        }
    }
    
    walk(obj) {
        const keys = Object.keys(obj)
        // 遍历对象的属性进行响应式设置
        for(let i = 0; i < keys.length; i ++) {
            defineReactive(obj, keys[i], obj[keys[i]])
        }
    }
}

// 更详细一些的defineReactive
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

所以响应式原理就是，我们通过递归遍历，把vue实例中data里面定义的数据，通过`Observer`包装，用`defineReactive`（`Object.defineProperty`）重新定义。每个数据内新建一个`Dep`实例，闭包中包含了这个 `Dep` 类的实例，用来收集 `Watcher` 对象。在对象被「读」的时候，会触发 reactiveGetter 函数把当前的 `Watcher` 对象（存放在 `Dep.target` 中）收集到 Dep 类中去。之后如果当该对象被「写」的时候，则会触发 reactiveSetter 方法，通知 Dep 类调用 notify 来触发所有 Watcher 对象的 update 方法更新对应视图。

### vue.$set以及原理

[Vue2.x | Vue.set和vm.$set](https://juejin.cn/post/6844904142507360269)

在`Vue`中，对象和数组在某些情况下无法触发响应式数据更新。比如:

```javascript
const vm = new Vue({
  el: '#root',
  data: {
    price: 10,
  },
});
vm.price = 20; // 重新渲染视图
vm.discount = 10; // 并不是响应式的数据
```

或者另一种情况，直接通过数组的下标修改数组的某一项:

```javascript
const vm = new Vue({
  el: '#root',
  data: {
    list: ['cat', 'dog', 'pig'],
  },
});
vm.list[1] = 'fish'; // 不会重新渲染视
```

为了解决上面的问题，`Vue`给出了两个`Api`，分别是`Vue.set`和`vm.$set`

[set源码](https://github.com/lgwebdream/FE-Interview/issues/139)

```javascript
export function set (target: Array<any> | Object, key: any, val: any): any {
  // target 为数组  
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 修改数组的长度, 避免索引>数组长度导致splcie()执行有误
    target.length = Math.max(target.length, key)
    // 利用数组的splice变异方法触发响应式  
    target.splice(key, 1, val)
    return val
  }
  // key 已经存在，直接修改属性值  
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = (target: any).__ob__
  // target 本身就不是响应式数据, 直接赋值
  if (!ob) {
    target[key] = val
    return val
  }
  // 对属性进行响应式处理
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}
```

`vm.$set` 的实现原理是：

1. 如果目标是数组，直接使用数组的 splice 方法触发相应式；
2. 如果目标是对象，会先判读属性是否存在、对象是否是响应式，
3. 最终如果要对属性进行响应式处理，则是通过调用 `defineReactive` 方法进行响应式处理

### Vue2 中是怎么监测数组的变化的？

通过上文我们知道如果使用 `Object.defineProperty` 对数组进行监听，当通过 Array 原型上的方法改变数组内容的时候是无发触发 getter/setter 的， Vue2 中是放弃了使用 `Object.defineProperty` 对数组进行监听的方案，而是通过对数组原型上的 7 个方法进行重写进行监听的。

原理就是使用拦截器覆盖 Array.prototype，之后再去使用 Array 原型上的方法的时候，其实使用的是拦截器提供的方法，在拦截器里面才真正使用原生 Array 原型上的方法去操作数组。

拦截器

```javascript
// 拦截器其实就是一个和 Array.prototype 一样的对象。
const arrayProto = Array.prototype
const arrayMethods = Object.create(arrayProto)
;[
    'push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
].forEach(function (method) {
    // 缓存原始方法
    const original = arrayProto[method]
    Object.defineProperty(arrayMethods, method, {
        value: function mutator(...args) {
            // 最终还是使用原生的 Array 原型方法去操作数组
            return original.apply(this, args)
        },
        eumerable: false,
        writable: false,
        configurable: true
    })
})
```

通过上文我们可以知道 Vue2 是在 `Observer` 类里面对对象的进行响应式处理，并且给对象也进行一个依赖收集。所以对数组的依赖处理也是在 `Observer` 类里面。

```javascript
class Observer {
    constructor(value) {
        this.value = value
        // 添加一个对象依赖收集的选项
        this.dep = new Dep()
        // 给响应式对象添加 __ob__ 属性，表明这是一个响应式对象
        def(value, '__ob__', this)
        // 如果是数组则通过覆盖数组的原型方法进来拦截操作
        if(Array.isArray(value)) {
        // 在这个地方 Vue2 会进行一些兼容性的处理，如果能使用 __proto__ 就覆盖原型，
        // 如果不能使用，则直接把那 7 个操作数组的方法直接挂载到需要被进行响应式处理的数组上
        // 因为当访问一个对象的方法时，只有这个对象自身不存在这个方法，才会去它的原型上查找这个方法。
          value.__proto__ = arrayMethods 
        } else {
            this.walk(value)
        }
    }
    // ...
}
```

数组如何收集依赖呢？

我们知道在数组进行响应式初始化的时候会在 Observer 类里面给这个数组对象的添加一个 `__ob__` 的属性，这个属性的值就是 Observer 这个类的实例对象，而这个 Observer 类里面有存在一个收集依赖的属性 dep，所以**在对数组里的内容通过那 7 个方法进行操作的时候，会触发数组的拦截器，那么在拦截器里面就可以访问到这个数组的 Observer 类的实例对象，从而可以向这些数组的依赖发送变更通知。**

```javascript
// 拦截器其实就是一个和 Array.prototype 一样的对象。
const arrayProto = Array.prototype
const arrayMethods = Object.create(arrayProto)
;[
    'push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
].forEach(function (method) {
    // 缓存原始方法
    const original = arrayProto[method]
    Object.defineProperty(arrayMethods, method, {
        value: function mutator(...args) {
            // 最终还是使用原生的 Array 原型方法去操作数组
            const result = original.apply(this, args)
            // 获取 Observer 对象实例
            const ob = this.__ob__
            // 通过 Observer 对象实例上 Dep 实例对象去通知依赖进行更新
            ob.dep.notify()
        },
        eumerable: false,
        writable: false,
        configurable: true
    })
})
```

因为 Vue2 的实现方法决定了在 Vue2 中对数组的一些操作无法实现响应式操作，例如：

```
this.list[0] = xxx
```

由于 **Vue2 放弃了 Object.defineProperty 对数组进行监听的方案**，所以通过下标操作数组是无法实现响应式操作的。

又例如：

```
this.list.length = 0
```

**这个动作在 Vue2 中也是无法实现响应式操作的。**

### Object.defineProperty 真的不能监听数组的变化吗？

其实 Object.defineProperty 是可以监听数组的变化的。

```javascript
const arr = [1, 2, 3]
arr.forEach((val, index) => {
  Object.defineProperty(arr, index, {
    get() {
      console.log('监听到了')
      return val
    },
    set(newVal) {
      console.log('变化了：', val, newVal)
      val = newVal
    }
  })
})
```

首先这种直接通过下标获取数组元素的场景就比较少，其次即便通过了 Object.defineProperty 对数组进行监听，但也监听不了 push、pop、shift 等对数组进行操作的方法，所以还是需要通过对数组原型上的那 7 个方法进行重写监听。所以为了性能考虑 Vue2 直接弃用了使用 Object.defineProperty 对数组进行监听的方案。

尤大回答为何不用这种方式: [为什么vue没有提供对数组属性的监听](https://github.com/vuejs/vue/issues/8562#issuecomment-408295547)

### vue3响应式原理

Vue3 是通过 `Proxy` 对数据实现 getter/setter 代理，从而实现响应式数据，然后在副作用函数中读取响应式数据的时候，就会触发 Proxy 的 getter，在 getter 里面把对当前的副作用函数保存起来，将来对应响应式数据发生更改的话，则把之前保存起来的副作用函数取出来执行。

具体是副作用函数里面读取响应式对象的属性值时，会触发代理对象的 getter，然后在 getter 里面进行一定规则的依赖收集保存操作。

简单代码实现：

```javascript
// 使用一个全局变量存储被注册的副作用函数
let activeEffect
// 注册副作用函数
function effect(fn) {
    activeEffect = fn
    fn()
}
const obj = new Proxy(data, {
    // getter 拦截读取操作
    get(target, key) {
        // 将副作用函数 activeEffect 添加到存储副作用函数的全局变量 targetMap 中
        track(target, key)
        // 返回读取的属性值
        return Reflect.get(target, key)
    },
    // setter 拦截设置操作
    set(target, key, val) {
        // 设置属性值
        const result = Reflect.set(target, key, val)
        // 把之前存储的副作用函数取出来并执行
        trigger(target, key)
        return result
    }
})
// 存储副作用函数的全局变量
const targetMap = new WeakMap()
// 在 getter 拦截器内追踪依赖的变化
function track(target, key) {
    // 没有 activeEffect，直接返回
    if(!activeEffect) return
    // 根据 target 从全局变量 targetMap 中获取 depsMap
    let depsMap = targetMap.get(target)
    if(!depsMap) {
       // 如果 depsMap 不存，那么需要新建一个 Map 并且与 target 关联
       depsMap = new Map()
       targetMap.set(target, depsMap)
    }
    // 再根据 key 从 depsMap 中取得 deps, deps 里面存储的是所有与当前 key 相关联的副作用函数
    let deps = depsMap.get(key)
    if(!deps) {
       // 如果 deps 不存在，那么需要新建一个 Set 并且与 key 关联
       deps = new Set()
       depsMap.set(key, deps)
    }
    // 将当前的活动的副作用函数保存起来
    deps.add(activeEffect)
}
// 在 setter 拦截器中触发相关依赖
function trgger(target, key) {
    // 根据 target 从全局变量 targetMap 中取出 depsMap
    const depsMap = targetMap.get(target)
    if(!depsMap) return
    // 根据 key 取出相关联的所有副作用函数
    const effects = depsMap.get(key)
    // 执行所有的副作用函数
    effects && effects.forEach(fn => fn())
}
```

通过上面的代码我们可以知道 Vue3 中依赖收集的规则，**首先把响应式对象作为 key，一个 Map 的实例做为值方式存储在一个 `WeakMap` 的实例中**，其中这个 Map 的实例又是以响应式对象的 key 作为 key, 值为一个 Set 的实例为值。而且这个 Set 的实例中存储的则是跟那个响应式对象 key 相关的副作用函数。

![image](https://user-images.githubusercontent.com/17645053/216752697-4bad10d2-6cb0-4ed0-84f2-efa9f302f0a3.png)

为什么 Vue3 的依赖收集的数据结构这里采用 WeakMap 呢？

所以我们需要解析一下 WeakMap 和 Map 的区别，首先 WeakMap 是可以接受一个对象作为 key 的，而 WeakMap 对 key 是弱引用的。所以当 WeakMap 的 key 是一个对象时，一旦**上下文执行完毕，WeakMap 中 key 对象没有被其他代码引用的时候，垃圾回收器就会把该对象从内存移除，我们就无法该对象从 WeakMap 中获取内容了**。

另外副作用函数使用 Set 类型，是因为 Set 类型能自动去除重复内容。

上述方法**只实现了对引用类型的响应式处理**，因为 Proxy 的代理目标必须是非原始值。原始值指的是 Boolean、Number、BigInt、String、Symbol、undefined 和 null 等类型的值。在 JavaScript 中，原始值是按值传递的，而非按引用传递。这意味着，如果一个函数接收原始值作为参数，那么形参与实参之间没有引用关系，它们是两个完全独立的值，对形参的修改不会影响实参。

Vue3 中是**通过对原始值做了一层包裹的方式来实现对原始值变成响应式数据的**。最新的 Vue3 实现方式是通过属性访问器 getter/setter 来实现的。

```javascript
class RefImpl{
    private _value
    public dep
    // 表示这是一个 Ref 类型的响应式数据
    private _v_isRef = true
    constructor(value) {
        this._value = value
        // 依赖存储
        this.dep = new Set()
    }
	// getter 访问拦截
    get value() {
        // 依赖收集
        trackRefValue(this)
        return this._value
    }
	// setter 设置拦截
    set value(newVal) {
        this._value = newVal
        // 触发依赖
        triggerEffect(this.dep)   
    }
}
```

ref 本质上是一个**实例化之后的 “包裹对象”**，因为 Proxy 无法提供对原始值的代理，所以我们需要**使用一层对象作为包裹，间接实现原始值的响应式方案**。 由于实例化之后的 “包裹对象” 本质与普通对象没有任何区别，所以**为了区分 ref 与 Proxy 响应式对象，我们需要给 ref 的实例对象定义一个 _v_isRef 的标识，表明这是一个 ref 的响应式对象。**

### Vue3 中是怎么监测数组的变化？

[Vue3 怎么监测数组的变化](https://juejin.cn/post/7124351370521477128#heading-6)

### Vue3响应式相对Vue2的优势

和 Vue2 进行一下对比，我们知道 Vue2 的响应式存在很多的问题，例如：

- 初始化时需要遍历对象所有 key，如果对象层次较深，性能不好
- 通知更新过程需要维护大量 dep 实例和 watcher 实例，额外占用内存较多
- 无法监听到数组元素的变化，只能通过劫持重写了几个数组方法
- 动态新增，删除对象属性无法拦截，只能用特定 set/delete API 代替
- 不支持 Map、Set 等数据结构

而 Vue3 使用 Proxy 实现之后，则以上的问题都不存在了。

