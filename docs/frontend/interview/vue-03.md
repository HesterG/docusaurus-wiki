# vue-03

## computed vs watcher

[**Vue中 computed 和 watch 区别及应用场景详解**](https://blog.51cto.com/u_15302032/3067114)

**计算属性 computed**

特点：

1. **支持缓存**，只有依赖数据发生改变，才会重新进行计算；

2. **不支持异步**，当 computed 内有异步操作时无效，无法监听数据的变化；
3. computed 属性值会默认走缓存，计算属性是**基于它们的响应式依赖进行缓存的**。也就是基于 data 中声明过或者父组件传递的 props 中的数据通过计算得到的值；
4. 如果一个属性是由其他属性计算而来的，这个属性依赖其他属性 是一个多对一或者一对一，一般用computed；
5. 如果 computed **属性值是函数，那么默认会走 get 方法**，函数的返回值就是属性的属性值；在computed中的，属性都有一个get和一个 set 方法，当数据变化时，调用 set 方法；

优点：

1. 当改变 data 变量值时，整个应用会重新渲染，vue 会被数据重新渲染到 dom 中。这时，如果我们使用 names ，随着渲染，方法也会被调用，而 computed 不会重新进行计算，从而性能开销比较小。当新的值需要大量计算才能得到，缓存的意义就非常大；
2. 如果 computed 所依赖的数据发生改变时，计算属性才会重新计算，并进行缓存；当改变其他数据时，computed 属性 并不会重新计算，从而提升性能；
3. 当拿到的值需要进行一定处理使用时，就可以使用 computed；

**侦听属性 watch**

特点：

1. 不支持缓存，数据变化，直接会触发相应的操作；

2. watch 支持异步操作；

3. 监听的函数接收两个参数，第一个参数是最新的值；第二个参数是输入之前的值；

4. 当一个属性发生变化时，需要执行对应的操作，一对多；

5. 监听数据必须是 data 中声明过或者父组件传递过来的 props 中的数据。当数据变化时触发其他操作，函数有两个参数：

   immediate：组件加载立即触发回调函数执行；

   deep: 深度监听；为了发现对象内部值的变化，复杂类型的数据时使用，例如：数组中的对象内容的改变

   注意：

   监听数组的变动不需要这么做。

   deep无法监听到数组的变动和对象的新增，参考vue数组变异,只有以响应式的方式触发才会被监听到；



## vue组件化的理解

回答总体思路：

组件化定义、优点、使用场景和注意事项等方面展开陈述，同时要强调vue中组件化的一些特点。

### 源码分析1：组件定义

定义方式一

```javascript
// 组件定义
Vue.component('comp', {
    template: '<div>this is a component</div>'
})
```

[源码src\core\global-api\assets.js](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/global-api/assets.ts)

```javascript
export function initAssetRegisters(Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  // ['component', 'filter', 'directive']
  ASSET_TYPES.forEach(type => {
    // @ts-expect-error function is not exact same type
    // Vue[component] = function(id, definition)
    Vue[type] = function (
      id: string,
      definition?: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (__DEV__ && type === 'component') {
          validateComponentName(id)
        }
    		// def是对象
        if (type === 'component' && isPlainObject(definition)) {
          // @ts-expect-error
          // 定义组件name
          definition.name = definition.name || id
          // extend创建组件构造函数时，def变成了构造函数
          // 研究一下extend函数
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && isFunction(definition)) {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

[extend函数](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/global-api/extend.ts#L20)

```javascript
...
// 创建VueComponent类
const Sub = function VueComponent(this: any, options: any) {
  this._init(options)
} as unknown as typeof Component
// 继承于Vue
Sub.prototype = Object.create(Super.prototype)
Sub.prototype.constructor = Sub
Sub.cid = cid++
// 选项合并
Sub.options = mergeOptions(Super.options, extendOptions)
Sub['super'] = Super

// For props and computed properties, we define the proxy getters on
// the Vue instances at extension time, on the extended prototype. This
// avoids Object.defineProperty calls for each instance created.
if (Sub.options.props) {
  initProps(Sub)
}
if (Sub.options.computed) {
  initComputed(Sub)
}

// allow further extension/mixin/plugin usage
Sub.extend = Super.extend
Sub.mixin = Super.mixin
Sub.use = Super.use

// create asset registers, so extended classes
// can have their private assets too.
ASSET_TYPES.forEach(function (type) {
  Sub[type] = Super[type]
})
// enable recursive self-lookup
if (name) {
  Sub.options.components[name] = Sub
}

// keep a reference to the super options at extension time.
// later at instantiation we can check if Super's options have
// been updated.
Sub.superOptions = Super.options
Sub.extendOptions = extendOptions
Sub.sealedOptions = extend({}, Sub.options)

// cache constructor
cachedCtors[SuperId] = Sub
return Sub
```

定义方式二(单文件组件)

```vue
<template>
    <div>
        this is a component
    </div>
</template>
```

> webpack调用vue-loader，会编译template为render函数，最终导出的依然是组件配置JS对象。



### 源码分析2：组件化优点

lifecycle.js - [mountComponent()](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/instance/lifecycle.ts#L146)

> 组件、Watcher、渲染函数和更新函数之间的关系

在执行$mount的时候会被调用

在这里建立组件和Watcher间一一对应的关系

```javascript
...
} else {
  // 用户$mount的时候，定义updateComponent
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
}

const watcherOptions: WatcherOptions = {
  before() {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}

if (__DEV__) {
  watcherOptions.onTrack = e => callHook(vm, 'renderTracked', [e])
  watcherOptions.onTrigger = e => callHook(vm, 'renderTriggered', [e])
}

// we set this to vm._watcher inside the watcher's constructor
// since the watcher's initial patch may call $forceUpdate (e.g. inside child
// component's mounted hook), which relies on vm._watcher being already defined
new Watcher(
  vm,
  updateComponent,
  noop,
  watcherOptions,
  true /* isRenderWatcher */
)
hydrating = false
```



### 源码分析3：组件化实现

构造函数，[src\core\global-api\extend.js](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/global-api/extend.ts)

- 全局调用的时候extend会立刻执行

实例化及挂载，src\core\vdom\patch.js - [createElm()](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/patch.ts#L121)

```javascript
......
// 如果要创建的是组件，走下面流程，如果是Vue就不会走了
if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
  return
}
......
```

createComponent

```js
// 这里createComponent是把前面的那个执行结果vnode转成真实dom
function createComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
    // 获取管理钩子函数
    let i = vnode.data
    if (isDef(i)) {
      const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
      // 存在init钩子，则执行之并挂载
      // 这个hooks在这里定义的
      // 挂载是从下而上
      // 实例化是从上而下
      if (isDef((i = i.hook)) && isDef((i = i.init))) {
        i(vnode, false /* hydrating */)
      }
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
      // 如果组件实例存在
      if (isDef(vnode.componentInstance)) {
        // 属性初始化
        initComponent(vnode, insertedVnodeQueue)
        // dom插入操作
        insert(parentElm, vnode.elm, refElm)
        if (isTrue(isReactivated)) {
          reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
        }
        return true
      }
    }
  }
```

[init钩子](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/create-component.ts#L37)

```javascript
const componentVNodeHooks = {
  init(vnode: VNodeWithData, hydrating: boolean): boolean | void {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      // 在这里创建组件实例
      const child = (vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      ))
      // mount自下而上，组件化自上而下
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  },


```

[createComponentInstanceForVnode](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/create-component.ts#L212)

```javascript
export function createComponentInstanceForVnode(
  // we know it's MountedComponentVNode but flow doesn't
  vnode: any,
  // activeInstance in lifecycle state
  parent?: any
): Component {
  const options: InternalComponentOptions = {
    _isComponent: true,
    _parentVnode: vnode,
    parent
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnode.componentOptions.Ctor(options)
}
```



### 总结

1. 组件是独立和可复用的代码组织单元。组件系统是 Vue 核心特性之一，它使开发者使用小型、独立和通常可复用的组件构建大型应用；
2. 组件化开发能大幅提高应用开发效率、测试性、复用性等；
3. 组件使用按分类有：页面组件、业务组件、通用组件；
4. vue的组件是基于配置的，我们通常编写的组件是**组件配置**而非组件，**框架后续会生成其构造函数，它们基于VueComponent，扩展于Vue**；
5. vue中常见组件化技术有：属性prop，自定义事件，插槽等，它们主要用于组件通信、扩展等；
6. 合理的划分组件，有助于提升应用性能；
7. 组件应该是高内聚、低耦合的；
8. 遵循单向数据流的原则。



## 谈一谈对vue设计原则的理解

vue[官网](https://cn.vuejs.org/)的定义和特点：

### 渐进式JavaScript框架

[官网解释](https://v2.cn.vuejs.org/v2/guide/?#Vue-js-%E6%98%AF%E4%BB%80%E4%B9%88)

与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://link.zhihu.com/?target=https%3A//cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://link.zhihu.com/?target=https%3A//github.com/vuejs/awesome-vue%23libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

![image](https://user-images.githubusercontent.com/17645053/216805487-cd11cffc-a9ce-4781-b60e-1ae53d00ae12.png)



### 易用、灵活和高效

**易用性**

vue提供数据响应式、声明式模板语法和基于配置的组件系统等核心特性。这些使我们只需要关注应用的核心业务即可，只要会写js、html和css就能轻松编写vue应用。

**灵活性**

渐进式框架的最大优点就是灵活性，如果应用足够小，我们可能仅需要vue核心特性即可完成功能；随着应用规模不断扩大，我们才可能逐渐引入路由、状态管理、vue-cli等库和工具，不管是应用体积还是学习难度都是一个逐渐增加的平和曲线。

**高效性**

超快的虚拟 DOM 和 diff 算法使我们的应用拥有最佳的性能表现。

追求高效的过程还在继续，vue3中引入Proxy对数据响应式改进以及编译器中对于静态内容编译的改进都会让vue更加高效。



## 谈谈你对MVC、MVP和MVVM的理解

答题思路：此题涉及知识点很多，很难说清、说透，因为mvc、mvp这些我们前端程序员自己甚至都没用过。但是恰恰反映了前端这些年从无到有，从有到优的变迁过程，因此沿此思路回答将十分清楚。

### Web1.0时代

在web1.0时代，并没有前端的概念。开发一个web应用多数采用[http://ASP.NET/Java/PHP](https://link.zhihu.com/?target=http%3A//ASP.NET/Java/PHP)编写，项目通常由多个aspx/jsp/php文件构成，每个文件中同时包含了HTML、CSS、JavaScript、C#/Java/PHP代码，系统整体架构可能是这样子的：

![image](https://user-images.githubusercontent.com/17645053/216805538-0bd010c7-fda1-4491-beaa-fb7dcb3debb4.png)

这种架构的好处是简单快捷，但是，缺点也非常明显：JSP代码难以维护

为了让开发更加便捷，代码更易维护，前后端职责更清晰。便衍生出MVC开发模式和框架，前端展示以模板的形式出现。典型的框架就是Spring、Structs、Hibernate。整体框架如图所示：

![image](https://user-images.githubusercontent.com/17645053/216805548-6d292866-de62-4e7c-aff6-8adca21090ef.png)

使用这种分层架构，职责清晰，代码易维护。但这里的MVC仅限于后端，前后端形成了一定的分离，前端只完成了后端开发中的view层。

但是，同样的这种模式存在着一些：

1. 前端页面开发效率不高
2. 前后端职责不清

### Web 2.0时代

自从Gmail的出现，ajax技术开始风靡全球。有了ajax之后，前后端的职责就更加清晰了。因为前端可以通过Ajax与后端进行数据交互，因此，整体的架构图也变化成了下面这幅图：

![image](https://user-images.githubusercontent.com/17645053/216805577-9e453709-4d66-4f8a-afdc-c1b938aba2f0.png)

通过ajax与后台服务器进行数据交换，前端开发人员，只需要开发页面这部分内容，数据可由后台进行提供。而且ajax可以使得页面实现部分刷新，减少了服务端负载和流量消耗，用户体验也更佳。这时，才开始有专职的前端工程师。同时前端的类库也慢慢的开始发展，最著名的就是jQuery了。

当然，此架构也存在问题：缺乏可行的开发模式承载更复杂的业务需求，页面内容都杂糅在一起，一旦应用规模增大，就会导致难以维护了。因此，前端的MVC也随之而来。

### 前后端分离后的架构演变

**MVC**

前端的MVC与后端类似，具备着View、Controller和Model。

Model：负责保存应用数据，与后端数据进行同步

Controller：负责业务逻辑，根据用户行为对Model数据进行修改

View：负责视图展示，将model中的数据可视化出来。

三者形成了一个如图所示的模型：

![image](https://user-images.githubusercontent.com/17645053/216805609-bafb81f1-c335-468e-ae01-ea433ee94c6f.png)

这样的模型，在理论上是可行的。但往往在实际开发中，并不会这样操作。因为开发过程并不灵活。例如，一个小小的事件操作，都必须经过这样的一个流程，那么开发就不再便捷了。

在实际场景中，我们往往会看到另一种模式，如图：

![image](https://user-images.githubusercontent.com/17645053/216805623-0e4b56aa-7503-4d07-9224-181e07efacab.png)

这种模式在开发中更加的灵活，backbone.js框架就是这种的模式。

但是，这种灵活可能导致严重的问题：

1. 数据流混乱。如下图：

   ![image](https://user-images.githubusercontent.com/17645053/216805641-05c69f64-df87-49d2-a80a-d28b5e0ffb48.png)

2. View比较庞大，而Controller比较单薄：由于很多的开发者都会在view中写一些逻辑代码，逐渐的就导致view中的内容越来越庞大，而controller变得越来越单薄。

既然有缺陷，就会有变革。前端的变化中，似乎少了MVP的这种模式，是因为AngularJS早早地将MVVM框架模式带入了前端。MVP模式虽然前端开发并不常见，但是在安卓等原生开发中，开发者还是会考虑到它。

**MVP**

MVP与MVC很接近，P指的是Presenter，presenter可以理解为一个中间人，它负责着View和Model之间的数据流动，防止View和Model之间直接交流。我们可以看一下图示：

![image](https://user-images.githubusercontent.com/17645053/216805669-84d9e953-4ab5-4982-b513-ee09664f3a73.png)

我们可以通过看到，presenter负责和Model进行双向交互，还和View进行双向交互。这种交互方式，相对于MVC来说少了一些灵活，VIew变成了被动视图，并且本身变得很小。虽然它分离了View和Model。但是应用逐渐变大之后，导致presenter的体积增大，难以维护。要解决这个问题，或许可以从MVVM的思想中找到答案。

**MVVM**

首先，何为MVVM呢？MVVM可以分解成(Model-View-VIewModel)。ViewModel可以理解为在presenter基础上的进阶版。如图所示：

![image](https://user-images.githubusercontent.com/17645053/216805688-3d655e8b-1b14-439c-845e-e7018daac7b8.png)

ViewModel通过实现一套数据响应式机制自动响应Model中数据变化；

同时Viewmodel会实现一套更新策略自动将数据变化转换为视图更新；

通过事件监听响应View中用户交互修改Model中数据。

这样在ViewModel中就减少了大量DOM操作代码。

MVVM在保持View和Model松耦合的同时，还减少了维护它们关系的代码，使用户专注于业务逻辑，兼顾开发效率和可维护性。

### 总结

- 这三者都是框架模式，它们设计的目标都是为了解决Model和View的耦合问题。
- MVC模式出现较早主要应用在后端，如Spring MVC、[http://ASP.NET](https://link.zhihu.com/?target=http%3A//ASP.NET) MVC等，在前端领域的早期也有应用，如Backbone.js。它的优点是分层清晰，缺点是数据流混乱，灵活性带来的维护性问题。
- MVP模式在是MVC的进化形式，Presenter作为中间层负责MV通信，解决了两者耦合问题，但P层过于臃肿会导致维护问题。
- MVVM模式在前端领域有广泛应用，它不仅解决MV耦合问题，还同时解决了维护两者映射关系的大量繁杂代码和DOM操作代码，在提高开发效率、可读性同时还保持了优越的性能表现。
- [vue的mvvm](https://www.kancloud.cn/dataoedu/vue/327305)



## 你了解哪些Vue性能优化方法

答题思路：根据题目描述，这里主要探讨Vue代码层面的优化

### 路由懒加载

```javascript
const router = new VueRouter({
 routes: [
    { path: '/foo', component: () => import('./Foo.vue') }
  ]
})
```

### keep-alive缓存页面

```javascript
<template>
 <div id="app">
 <keep-alive>
 <router-view/>
 </keep-alive>
 </div>
</template>
```

### 使用v-show复用DOM

```text
<template>
 <div class="cell">
 <!--这种情况用v-show复用DOM，比v-if效果好-->
 <div v-show="value" class="on">
 <Heavy :n="10000"/>
 </div>
 <section v-show="!value" class="off">
 <Heavy :n="10000"/>
 </section>
 </div>
</template>
```

### v-for 遍历避免同时使用 v-if

```text
<template>
 <ul>
 <li
 v-for="user in activeUsers"
 :key="user.id">
 {{ user.name }}
 </li>
 </ul>
</template>
<script>
 export default {
 computed: {
 activeUsers: function () {
 return this.users.filter(function (user) {
 return user.isActive
            })
          }
        }
    }
</script>
```

### 长列表性能优化

如果列表是纯粹的数据展示，不会有任何改变，就不需要做响应化

```javascript
export default {
 data: () => ({
 users: []
  }),
 async created() {
 const users = await axios.get("/api/users");
 this.users = Object.freeze(users);
  }
};
```

如果是大数据长列表，可采用虚拟滚动，只渲染少部分区域的内容

```javascript
<recycle-scroller
 class="items"
 :items="items"
 :item-size="24"
>
 <template v-slot="{ item }">
 <FetchItemView
 :item="item"
 @vote="voteItem(item)"
 />
 </template>
</recycle-scroller>
```

参考[vue-virtual-scroller](https://link.zhihu.com/?target=https%3A//github.com/Akryum/vue-virtual-scroller)、[vue-virtual-scroll-list](https://link.zhihu.com/?target=https%3A//github.com/tangbc/vue-virtual-scroll-list)

### 事件的销毁

Vue 组件销毁时，会自动解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。

```javascript
created() {
  this.timer = setInterval(this.refresh, 2000)
},
beforeDestroy() {
  clearInterval(this.timer)
}
```

### 图片懒加载
对于图片过多的页面，为了加速页面加载速度，所以很多时候我们需要将页面内未出现在可视区域内的图片先不做加载， 等到滚动到可视区域后再去加载。

```html
<img v-lazy="/static/img/1.png">
```

参考项目：[vue-lazyload](https://link.zhihu.com/?target=https%3A//github.com/hilongjw/vue-lazyload)

### 第三方插件按需引入

像element-ui这样的第三方组件库可以按需引入避免体积太大。

```javascript
import Vue from 'vue';
import { Button, Select } from 'element-ui';

 Vue.use(Button)
 Vue.use(Select)
```

### 无状态的组件标记为函数式组件

```vue
<template functional>
 <div class="cell">
 <div v-if="props.value" class="on"></div>
 <section v-else class="off"></section>
 </div>
</template>

<script>
export default {
 props: ['value']
}
</script>
```

### 子组件分割

```vue
<template>
 <div>
 <ChildComp/>
 </div>
</template>

<script>
export default {
 components: {
 ChildComp: {
 methods: {
 heavy () { /* 耗时任务 */ }
      },
 render (h) {
 return h('div', this.heavy())
      }
    }
  }
}
</script>
```

### 变量本地化

```vue
<template>
 <div :style="{ opacity: start / 300 }">
 {{ result }}
 </div>
</template>

<script>
import { heavy } from '@/utils'

export default {
 props: ['start'],
 computed: {
 base () { return 42 },
 result () {
 const base = this.base // 不要频繁引用this.base
 let result = this.start
 for (let i = 0; i < 1000; i++) {
     result += heavy(base)
 }
 return result
    }
  }
}
</script>
```

### SSR



## 你对Vue3.0的新特性有没有了解？

根据尤大的PPT总结，Vue3.0改进主要在以下几点：

- 更快

  虚拟DOM重写

  优化slots的生成

  静态树提升

  静态属性提升

- 基于Proxy的响应式系统

- 更小：通过摇树优化核心库体积

- 更容易维护：TypeScript + 模块化

- 更加友好

  跨平台：编译器核心和运行时核心与平台无关，使得Vue更容易与任何平台（Web、Android、iOS）一起使用

- 更容易使用

  改进的TypeScript支持，编辑器能提供强有力的类型检查和错误及警告

  更好的调试支持

  独立的响应化模块

  Composition API



### 虚拟 DOM 重写

期待更多的编译时提示来减少运行时开销，使用更有效的代码来创建虚拟节点。

组件快速路径 + 单个调用 + 子节点类型检测

- 跳过不必要的条件分支
- JS引擎更容易优化

<img width="1371" alt="Screen Shot 2023-02-05 at 6 26 22 PM" src="https://user-images.githubusercontent.com/17645053/216813700-a4dd279d-e8fc-4440-9f14-572821f528f6.png" />



### 优化slots生成

vue3中可以单独重新渲染父级和子级

- 确保实例正确的跟踪依赖关系
- 避免不必要的父子组件重新渲染

![image](https://user-images.githubusercontent.com/17645053/216812119-c325b824-d8ba-4426-9d0c-3b97112180ed.png)



### 静态树提升(Static Tree Hoisting)

使用静态树提升，这意味着 Vue 3 的编译器将能够检测到什么是静态的，然后将其提升，从而降低了渲染成本。

- **跳过修补整棵树，从而降低渲染成本**
- 即使多次出现也能正常工作

![image](https://user-images.githubusercontent.com/17645053/216812145-1ea13d64-c625-4823-bffd-6375065b79ee.png)



### 静态属性提升

使用静态属性提升，Vue 3打补丁时将跳过这些属性不会改变的节点。

![image](https://user-images.githubusercontent.com/17645053/216812195-bea77693-114d-4ba9-9462-789508c8d8e1.png)



### 基于 Proxy 的数据响应式

Vue 2的响应式系统使用 Object.defineProperty 的getter 和 setter。Vue 3 将使用 ES2015 Proxy 作为其观察机制，这将会带来如下变化：

- 组件实例初始化的速度提高100％
- 使用Proxy节省以前一半的内存开销，加快速度，但是存在低浏览器版本的不兼容
- 为了继续支持 IE11，Vue 3 将发布一个支持旧观察者机制和新 Proxy 版本的构建

![image](https://user-images.githubusercontent.com/17645053/216812219-402d616b-64cb-41da-9cdd-9b06e7879d8a.png)


### 高可维护性

Vue 3 将带来更可维护的源代码。它不仅会使用 TypeScript，而且许多包被解耦，更加模块化。

![image](https://user-images.githubusercontent.com/17645053/216812246-abf1a0f5-0f8f-466c-8440-4360b0254250.png)


## vue2 vue3 diff 区别

[vue2、vue3 diff 算法源码解析](https://juejin.cn/post/7100092670566989861)


## React vue diff 区别

[React vue diff 区别](https://juejin.cn/post/7053001279580143653)
