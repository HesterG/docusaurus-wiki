# Vue 02

[vue面试中经常问到的问题](https://zhuanlan.zhihu.com/p/111310865)

[b站链接](https://www.bilibili.com/video/BV127411P7zi/?p=1&vd_source=2f792a955282ac02af1f8efe5510c76d)

## v-if和v-for优先级？如果两个同时出现，怎么优化

1. 显然v-for优先于v-if被解析

   [源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/compiler/codegen/index.ts#L74)

   ```javascript
   ...
   if (el.staticRoot && !el.staticProcessed) {
       return genStatic(el, state)
     } else if (el.once && !el.onceProcessed) {
       return genOnce(el, state)
     } else if (el.for && !el.forProcessed) {
       return genFor(el, state)
     } else if (el.if && !el.ifProcessed) {
       return genIf(el, state)
     } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
       return genChildren(el, state) || 'void 0'
     } else if (el.tag === 'slot') {
       return genSlot(el, state)
     } else {
   ...
   ```

2. 如果同时出现，每次渲染都会先执行循环再判断条件，无论如何循环都不可避免，浪费了性能

3. 要避免出现这种情况，则在外层嵌套template，在这一层进行v-if判断，然后在内部进行v-for循环

4. 如果条件出现在循环内部，可通过计算属性提前过滤掉那些不需要显示的项



## Vue组件data为什么必须是个函数而Vue的根实例则没有此限制

[initData源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/instance/state.ts#L122)

```javascript
function initData(vm: Component) {
  let data: any = vm.$options.data
  // 如果data是函数，则执行之并且将其结果作为data选项的值
  // 如果这里直接用Object, 那所有组件会共享这个data Obj
  data = vm._data = isFunction(data) ? getData(data, vm) : data || {}
  if (!isPlainObject(data)) {
    data = {}
    __DEV__ &&
      warn(
        'data functions should return an object:\n' +
          'https://v2.vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      )
  }
	...
}
  
```

[mergeOptions源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/util/options.ts#L395)

其中[strats.data](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/util/options.ts#L127)是下面的内容

```javascript
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): Function | null {
  // 如果不是vm，判定data是否是function
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      __DEV__ &&
        warn(
          'The "data" option should be a function ' +
            'that returns a per-instance value in component ' +
            'definitions.',
          vm
        )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```



Vue组件可能存在多个实例，**如果使用对象形式定义data，则会导致它们共用一个data对象**，那么状态变更将会影响所有组件实例，这是不合理的；采用函数形式定义，在initData时会**将其作为工厂函数返回全新data对象，有效规避多实例之间状态污染问题**。而在**Vue根实例**创建过程中则不存在该限制，也是因为根实例**只能有一个**，不需要担心这种情况。

[组件间data obj污染的例子](https://gaocarri.top/post/vue%E5%85%B3%E4%BA%8Edata%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF%E5%87%BD%E6%95%B0%E8%BF%99%E4%BB%B6%E4%BA%8B/)

因为obj是pass by reference的，如果一个instance里的改了，别的instance里的都会改



## vue中key的作用和工作原理

[Vue 中 key 的作用和工作原理](https://juejin.cn/post/7083079716046897165)

当数据发生变化时，会触发**渲染 watcher 的回调函数，进而执行组件的更新过程**。

在组件的**更新过程中**会对生成的虚拟 DOM **执行 patch方法**，代码在 src\core\vdom\patch.js -patch() 中。其中在比较两个节点时，会通过sameVnode方法判断判断是否时相同的 VNode 来决定不同的更新逻辑

```javascript
function sameVnode (a, b) {
  return (
    a.key === b.key &&
    a.asyncFactory === b.asyncFactory && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```

其中，会根据key进行判断！！

1. 如果新旧节点不相同时，会替换已存在的节点，分为3步：

- 创建新节点
- 更新父节点的占位符节点
- 删除旧节点

2. 如果**新旧节点相同**（当不设置key的时候，同类型节点也会走相同节点逻辑！），在 patch 方法，会**调用 patchVnode 方法**

   patchVnode 的作用就是把新的 vnode patch 到旧的 vnode 上，我们只关心核心逻辑，分为四步:

- 执行 prepatch 钩子函数，当更新的 vnode 是一个组件的时候，会执行 prepatch 方法， 在源码 src\core\vdom\create-component.js 中，prepatch 方法就是拿到新的 vnode 的组件配置及组件实例，去执行 updateChildComponent 方法，该方法在 src\core\instance\lifecycle.js -updateChildComponent() 中，updateChildComponent 方法逻辑用于更新 vnode 对应实例的一系列属性，包括占位符 vm.$vnode 的更新、slot、listeners、props更新等。

- 执行 update 钩子函数。在执行完新vnode的prepatch钩子函数后，会执行所有module的update钩子函数及用户自定义的update钩子函数。

- 完成 patch 过程

  一、如果 vnode 是文本节点，且新旧文本不同，则直接替换文本内容；

  二、如果不是文本节点，则判断他们的子节点，并分为了几种情况：

  1. oldCh 与 ch 都存在且不相同时，使用 **updateChildren** 函数来更新子节点。（**重点！！！**）
  2. 如果只有 ch 存在，表示旧节点不需要了。如果旧节点是文本则先将该节点的文本清除，然后通过 addVnodes 将 ch 批量插入到新节点 elm 下。
  3. 如果只有 oldCh 存在，表示更新的是空节点，则需要将旧的节点，则需要将旧的节点通过 removeVnodes 全部清除。
  4. 当只有旧节点是文字节点的时候，则清除其节点文本内容。

- 执行 postpatch 钩子函数

  执行完 patch 过程后，会执行 postpatch 钩子函数，它是组件自定义的钩子函数，有则执行。



在整个 patchVnode 过程中，最复杂的就是 updateChildren 方法

[updateChildren源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/patch.ts#L413)

updateChildren 的逻辑比较复杂，大致逻辑是：

- 将新旧节点的子节点，进行 “头头、尾尾、头尾、尾头” 4种方式进行比较。
- 如果上述方法没有匹配到，如果设置了key，就会用key进行比较。
  - 当其中两个能匹配上那么真实dom中的相应节点会移到Vnode相应的位置。
  - 如果是oldS和E匹配上了，那么真实dom中的第一个节点会移到最后
  - 如果是oldE和S匹配上了，那么真实dom中的最后一个节点会移到最前，匹配上的两个指针向中间移动
  - 如果四种匹配没有一对是成功的，那么遍历oldChild，S挨个和他们匹配，匹配成功就在真实dom中将成功的节点移到最前面，如果依旧没有成功的，那么将S对应的节点插入到dom中对应的oldS位置，oldS和S指针向中间移动。
- 在比较的过程中，使用双指针方式，指针会往中间靠近，一旦startIndex > endIndex，说明至少有一个已经遍历完了，比较就会结束，对剩下的vnode执行添加或删除vnode逻辑。

在这些节点 sameVnode(oldStartVnode, newStartVnode) 匹配成功后，就会执行 patchVnode 了，层层递归下去，直至 oldVnode 和 Vnode 中所有的子节点对比完。也将 DOM 的所有补丁打好了。

[patch流程](https://juejin.cn/post/7083079716046897165)

patch -> patchVnode -> updateChildren(也会调用patchVnode进行递归)

[patchVnode源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/patch.ts#L584)

[sameVnode源码](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/patch.ts#L36)

```javascript
function sameVnode(a, b) {
  return (
    a.key === b.key &&
    a.asyncFactory === b.asyncFactory &&
    ((a.tag === b.tag &&
      a.isComment === b.isComment &&
      isDef(a.data) === isDef(b.data) &&
      sameInputType(a, b)) ||
      (isTrue(a.isAsyncPlaceholder) && isUndef(b.asyncFactory.error)))
  )
}
```

1. key的作用主要是为了高效的更新虚拟DOM，其原理是vue在patch过程中通过key可以精准判断两个节点是否是同一个，从而避免频繁更新不同元素，使得整个patch过程更加高效，减少DOM操作量，提高性能。
2. 另外，若不设置key还可能在列表更新时引发一些隐蔽的bug
3. vue中在使用相同标签名元素的过渡切换时，也会使用到key属性，其目的也是为了让vue可以区分它们，否则vue只会替换其内部属性而不会触发过渡效果。



## 怎么理解diff算法

源码分析1：必要性，lifecycle.js - [mountComponent()](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/instance/lifecycle.ts#L146)

- $mount调用了mountComponent

- 每次组件实例创建会创建一个Watcher, Watcher和组件实例一一对应

  ```javascript
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

- 组件中可能存在很多个data中的key使用，引入diff才能精确找到发生变化的地方

源码分析2：执行方式，patch.js - [patchVnode()](https://github.com/vuejs/vue/blob/6d9aac8bd38f6d30a6db70b51fa46f27cbeac559/src/core/vdom/patch.ts#L584)

- **patchVnode是diff发生的地方**，整体策略：深度优先，同层比较

  ```javascript
  const oldCh = oldVnode.children
  const ch = vnode.children
  if (isDef(data) && isPatchable(vnode)) {
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef((i = data.hook)) && isDef((i = i.update))) i(oldVnode, vnode)
  }
  // 判断是否元素
  if (isUndef(vnode.text)) {
  	// 双方都有孩子  
    if (isDef(oldCh) && isDef(ch)) {
      // 比孩子，如果不一样，updateChildren进行递归
      if (oldCh !== ch)
        updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      // 新节点有孩子
      if (__DEV__) {
        checkDuplicateKeys(ch)
      }
      // 清空老节点文本
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      // 创建孩子并追加
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      // 老节点有孩子，删除即可
      removeVnodes(oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      // 老节点存在文本，清空
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    // 双方都是文本节点，更新文本
    nodeOps.setTextContent(elm, vnode.text)
  }
  if (isDef(data)) {
    if (isDef((i = data.hook)) && isDef((i = i.postpatch))) i(oldVnode, vnode)
  }
  ```

源码分析3：高效性，patch.js - updateChildren()

1. diff算法是虚拟DOM技术的必然产物：通过新旧虚拟DOM作对比（即diff），将变化的地方更新在真实DOM上；另外，也需要diff高效的执行对比过程，从而降低时间复杂度为O(n)。

2. vue 2.x中为了降低Watcher粒度，每个组件只有一个Watcher与之对应，只有引入diff才能精确找到发生变化的地方。

3. 当我们修改一个数据的时候，由于数据响应式，会调用setter，setter中会触发通知。就会触发Dep里Watcher的更新函数。更新函数执行的时候调用了组件的渲染和更新函数。vue中diff执行的时刻是组件实例执行其更新函数时，它会比对上一次渲染结果oldVnode和新的渲染结果newVnode，此过程称为patch。

4. diff过程整体遵循深度优先、同层比较的策略；两个节点之间比较会根据它们是否拥有子节点或者文本节点做不同操作；比较两组子节点是算法的重点，首先假设头尾节点可能相同做4次比对尝试，如果没有找到相同节点才按照通用方式遍历查找，查找结束再按情况处理剩下的节点(批量新增或删除)；借助key通常可以非常精确找到相同节点，因此整个patch过程非常高效。

