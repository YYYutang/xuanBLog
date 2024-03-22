---
title: Vue原理系列之五：虚拟节点 vnode 生成真实DOM
categories: Vue
tags: [Vue2, 前端]
cover: /picture/vue.jpg
date: 2024-03-22 10:56
---

首先，我们从前面的文章可以得知 `Virtual DOM` 渲染成真实的 `DOM` 实际上要经历 `VNode` 的定义、`diff`、`patch` 等过程。

[深入浅出虚拟 DOM 和 Diff 算法，及 Vue2 与 Vue3 中的区别 - 掘金 (juejin.cn)](https://juejin.cn/post/7010594233253888013#heading-8 "深入浅出虚拟 DOM 和 Diff 算法，及 Vue2 与 Vue3 中的区别 - 掘金 (juejin.cn)")

## Diff 算法

### **1. 只比较同一层级，不跨级比较**

vue中的diff算法有个特点，就是**只能在同级比较，不能跨级比较。** 即图中颜色相同部分进行比较。

![](image_yoohhv9_Vz.png)

举个例子:

```javascript
<!-- 之前 -->
<div>           <!-- 层级1 -->
  <p>            <!-- 层级2 -->
    <b> aoy </b>   <!-- 层级3 -->   
    <span>diff</Span>
  </P> 
</div>

<!-- 之后 -->
<div>            <!-- 层级1 -->
  <p>             <!-- 层级2 -->
      <b> aoy </b>        <!-- 层级3 -->
  </p>
  <span>diff</Span>
</div>

```

我们可能期望将`<span>`直接移动到`<p>`的后边，这是最优的操作。但是实际的diff操作是移除`<p>`里的`<span>`在创建一个新的`<span>`插到`<p>`的后边。因为新加的`<span>`在层级2，旧的在层级3，属于不同层级的比较。vue中的diff算法可能不是最优的操作，但是在一颗虚拟DOM树比较复杂的情况下是相对比较友好的。

### **2. 比较标签名**

如果同一层级的比较标签名不同，就直接移除老的虚拟 DOM 对应的节点，不继续按这个树状结构做深度比较，这是简化比较次数的第二个方面

![](image_EkwFSJGa1M.png)

### **3. 比较 key**

如果标签名相同，key 也相同，就会认为是相同节点，也不继续按这个树状结构做深度比较，比如我们写 v-for 的时候会比较 key，不写 key 就会报错，这也就是因为 Diff 算法需要比较 key

面试中有一道特别常见的题，就是让你说一下 key 的作用，实际上考查的就是大家对虚拟 DOM 和 patch 细节的掌握程度，能够反应出我们面试者的理解层次，所以这里扩展一下 key

#### key 的作用

比如有一个列表，我们需要在中间插入一个元素，会发生什么变化呢？先看个图

![](image_XkioJUzlhx.png)

如图的 `li1` 和 `li2` 不会重新渲染，这个没有争议的。而 `li3、li4、li5` 都会重新渲染

因为在不使用 `key` 或者列表的 `index` 作为 `key` 的时候，每个元素对应的位置关系都是 index，上图中的结果直接导致我们插入的元素到后面的全部元素，对应的位置关系都发生了变更，所以全部都会执行更新操作，这可不是我们想要的，我们希望的是渲染添加的那一个元素，其他四个元素不做任何变更，也就不要重新渲染

而在使用唯一 `key`  的情况下，每个元素对应的位置关系就是 `key`，来看一下使用唯一 `key` 值的情况下

![](image_MiIpiWxwE6.png)

这样如图中的 `li3` 和 `li4` 就不会重新渲染，因为元素内容没发生改变，对应的位置关系也没有发生改变。

这也是为什么 v-for 必须要写 key，而且不建议开发中使用数组的 index 作为 key 的原因

总结一下：

-   [ ] key 的作用主要是为了更高效的更新虚拟 DOM，因为它可以非常精确的找到相同节点，因此 patch 过程会非常高效
-   [ ] Vue 在 patch 过程中会判断两个节点是不是相同节点时，key 是一个必要条件。比如渲染列表时，如果不写 key，Vue 在比较的时候，就可能会导致频繁更新元素，使整个 patch 过程比较低效，影响性能
-   [ ] 应该避免使用数组下标作为 key，因为 key 值不是唯一的话可能会导致上面图中表示的 bug，使 Vue 无法区分它他，还有比如在使用相同标签元素过渡切换的时候，就会导致只替换其内部属性而不会触发过渡效果
-   [ ] 从源码里可以知道，Vue 判断两个节点是否相同时主要判断两者的元素类型和 key 等，如果不设置 key，就可能永远认为这两个是相同节点，只能去做更新操作，就造成大量不必要的 DOM 更新操作，明显是不可取的

## Vue源码的diff调用逻辑

`Vue.js` 源码实例化了一个 `watcher`，这个 \~ 被添加到了在模板当中所绑定变量的依赖当中，一旦 `model` 中的响应式的数据发生了变化，这些响应式的数据所维护的 `dep` 数组便会调用 `dep.notify()` 方法完成所有依赖遍历执行的工作，这包括视图的更新，即 `updateComponent` 方法的调用。`watcher` 和 `updateComponent`方法定义在  `src/core/instance/lifecycle.js` 文件中 。

```javascript
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  // 省略一系列其它代码
  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      // 生成虚拟 vnode   
      const vnode = vm._render()
      // 更新 DOM
      vm._update(vnode, hydrating)
     
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // 实例化一个渲染Watcher，在它的回调函数中会调用 updateComponent 方法  
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  return vm
}

```

完成视图的更新工作事实上就是调用了`vm._update`方法，这个方法接收的第一个参数是刚生成的`Vnode`，调用的`vm._update`方法定义在 `src/core/instance/lifecycle.js`中。

```javascript
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    vm._vnode = vnode
    if (!prevVnode) {
      // 第一个参数为真实的node节点，则为初始化
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // 如果需要diff的prevVnode存在，那么对prevVnode和vnode进行diff
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    restoreActiveInstance()
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
  }

```

在这个方法当中最为关键的就是 vm.\_\_**patch**\_\_ 方法，这也是整个 virtual-dom 当中最为核心的方法，主要完成了prevVnode 和 vnode 的 diff 过程并根据需要操作的 vdom 节点打 patch，最后生成新的真实 dom 节点并完成视图的更新工作。

## patch

先来介绍几个有用的api

-   [ ] `invokeDestroyHook`: 用来删除dom节点
-   [ ] `createElm`:用来创建一个节点
-   [ ] `patchVnode`: patch的核心方法，主要对比就发生在这个方法中。
-   [ ] `invokeInsertHook`: 用来插入节点。
-   [ ] `removeVnodes`: 用来移除旧节点。
-   [ ] `sameVnode`: 比较两个node的tag、isComment、inputType是否相同以及是否都有data属性。

patch的方式主要分为4步

-   [ ] 如果`oldVnode`存在,`vnode`不存在,则是要做**删除`oldVnode`节点**的操作。
-   [ ] 如果`oldVnode`不存在,`vnode`存在,则是要做**创建`vnode`节点**的操作。
-   [ ] 如果`oldVnode、vnode`都存在，且标签名相同、inputType属性(若有)相同且都存在data，则执行`patchVnode`方法。
-   [ ] 如果`oldVnode、vnode`都存在，但是不满足第三步条件，则\*\*删除`oldVnode`****节点,创建****`vnode`\*\***节点**

### patchVnode

```javascript
function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    // 完全相同，则什么都不做
    if (oldVnode === vnode) {
      return
    }

    const elm = vnode.elm = oldVnode.elm
    // 都是静态节点且key相同,且当vnode是克隆节点或是v-once指令控制的节点
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance
      return
    }

    const oldCh = oldVnode.children
    const ch = vnode.children
    // 不都是文本节点
    if (isUndef(vnode.text)) {
      if (isDef(oldCh) && isDef(ch)) {
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } else if (isDef(ch)) {
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
      } else if (isDef(oldCh)) {
        removeVnodes(oldCh, 0, oldCh.length - 1)
      } else if (isDef(oldVnode.text)) {
        nodeOps.setTextContent(elm, '')
      }
    } else if (oldVnode.text !== vnode.text) {  // 都是文本节点
      nodeOps.setTextContent(elm, vnode.text)
    }
  }


```

patchVnode方法主要分为以下步骤：

-   [ ] 若`vnode`和`oldVnode`完全相同，则不需要做任何事情
-   [ ] 若`vnode`和`oldVnode`都是静态节点，且具有相同的`key`,则当`vnode`是克隆节点或是`v-once`指令控制的节点时，只需要把`oldVnode.elm`和`oldVnode.child`都复制到`vnode`上，也不用再有其他操作。
-   [ ] 若`vnode`和`oldVnode`不是文本节点或注释节点时
    1.  [ ] 如果`oldVnode`和`vnode`都有子节点，且2方的子节点不完全一致，就执行更新子节点的操作（这一部分其实是在`updateChildren`函数中实现）。
    2.  [ ] 如果只有`oldVnode`有子节点，那就把这些节点都删除
    3.  [ ] 如果只有`vnode`有子节点，那就创建这些子节点
    4.  [ ] 如果`oldVnode`和`vnode`都没有子节点，但是`oldVnode`是文本节点或注释节点，就把`vnode.elm`的文本设置为空字符串
-   [ ] 如果`vnode`是文本节点或注释节点，但是`vnode.text != oldVnode.text`时，只需要更新`vnode.elm`的文本内容就可以.

### updateChildren

```javascript
  /**
  * parentElm:父级元素节点
  * oldCh: oldVnode的children
  * newCh: vnode的children
  **/
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    // 定义4个索引
    let oldStartIdx = 0  // 旧头
    let newStartIdx = 0  // 新头
    let oldEndIdx = oldCh.length - 1 // 旧尾
    let oldStartVnode = oldCh[0]  // oldStartIdx对应的node
    let oldEndVnode = oldCh[oldEndIdx] // oldEndIdx对应的node
    let newEndIdx = newCh.length - 1  // 新尾
    let newStartVnode = newCh[0] // newStartIdx对应的node
    let newEndVnode = newCh[newEndIdx] // newEndIdx对应的node
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

   
    // 当vnode和oldVnode在下标之间有node存在时
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }



```

首先我们定义 oldStartIdx、newStartIdx、oldEndIdx 以及 newEndIdx 分别是新老两个 VNode 的两边的索引，同时oldStartVnode、newStartVnode、oldEndVnode 以及 newEndVnode 分别指向这几个索引对应的 VNode 节点。

举个例子

![](image_S9DNivmLqQ.png)

假设现在oldch、newCh分别如上图所示。那么接下来就要执行

```javascript
while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx)

```

在这个循环中，`oldStartIdx、oldEndIdx`和`newStartIdx、newEndIdx`分别**从两边向中间移动，直到有其中一个存在交叉部分(startIdx>=endIdx)**

![](image_CRjnUH8iE3.png)

首先当 `oldStartVnode` 或者 `oldEndVnode` 不存在的时候，`oldStartIdx` 与 `oldEndIdx` 继续向中间靠拢，并更新对应的 `oldStartVnode` 与 `oldEndVnode` 的指向,这里需要**注意就是伴随着Idx移动，其对应的指向node也发生变化**

```javascript
if (isUndef(oldStartVnode)) {
    oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
} else if (isUndef(oldEndVnode)) {
    oldEndVnode = oldCh[--oldEndIdx]
} 

```

接下来就是新旧vnode的首首、首尾、尾尾、尾首对比的过程，即`oldStartVnode、newStartVnode`和`oldEndVnode、newEndVnode`两两之间执行`patchVnode`，同时`Idx`向中间移动

```javascript
else if (sameVnode(oldStartVnode, newStartVnode)) { // 
    patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
    oldStartVnode = oldCh[++oldStartIdx]
    newStartVnode = newCh[++newStartIdx]
} else if (sameVnode(oldEndVnode, newEndVnode)) {
    patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
    oldEndVnode = oldCh[--oldEndIdx]
    newEndVnode = newCh[--newEndIdx]
} else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
    patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
    canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
    oldStartVnode = oldCh[++oldStartIdx]
    newEndVnode = newCh[--newEndIdx]
} else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
    patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
    canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
    oldEndVnode = oldCh[--oldEndIdx]
    newStartVnode = newCh[++newStartIdx]
}

```

ok,接下来我们来分别分析

-   [ ] 首先如果`oldStartVnode、newStartVnode`符合 `sameVnode` 时，说明`oldVnode` 节点的头部与 `vNode` 节点的头部是相同的`VNode`节点，直接进行 `patchVnode`，同时 `oldStartIdx` 与 `newStartIdx` 向后移动一位。
-   [ ] `oldEndVnode、newEndVnode`同理，若两者符合 `sameVnode`，直接进行 `patchVnode`，同时 `oldEndIdx` 与 `newEndIdx` 向前移动一位。

![](image_hOQe2ysTpb.png)

-   [ ] 接下来比较`oldStartVnode、newEndVnode`,若两者符合 `sameVnode`，也就是老 `oldVnode` 节点的头部与新 `vNode` 节点的尾部是同一节点的时候，将 `oldStartVnode.elm` 这个节点直接移动到 `oldEndVnode.elm` 这个节点的后面即可。然后 `oldStartIdx` 向后移动一位，`newEndIdx` 向前移动一位。

![](image_iEj113L7rh.png)

-   [ ] 同理，`oldEndVnode` 与 `newStartVnode` 符合 `sameVnode` 时，也就是老 `oldVnode` 节点的尾部与新 `vNode` 节点的头部是同一节点的时候，将 `oldEndVnode.elm` 插入到 `oldStartVnode.elm` 前面。同样的，`oldEndIdx` 向前移动一位，`newStartIdx` 向后移动一位。

![](image_UxKr62qeWa.png)

-   [ ] 如果以上四种情况都没有命中，就不断拿新的开始节点的 key 去老的 children 里找

1.  [ ] 如果没找到，就创建一个新的节点
2.  [ ] 如果找到了，再对比标签是不是同一个节点

    如果是同一个节点，就调用 patchVnode 进行后续对比，然后把这个节点插入到老的开始前面，并且移动新的开始下标，继续下一轮循环对比

    如果不是相同节点，就创建一个新的节点&#x20;
3.  [ ] 如果老的 vnode 先遍历完，就添加新的 vnode 没有遍历的节点
4.  [ ] 如果新的 vnode 先遍历完，就删除老的 vnode 没有遍历的节点

```javascript
else {
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
      

// 
function createKeyToOldIdx (children, beginIdx, endIdx) {
  let i, key
  const map = {}
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key
    if (isDef(key)) map[key] = i
  }
  return map
}

```

`createKeyToOldIdx` 的作用是产生 `key` 与 `index` 索引对应的一个 map 表。比如说有这么一个oldChild(为举例，格式不正确)：

```javascript
[
    {xx: xx, key: 'key0'},
    {xx: xx, key: 'key1'}, 
    {xx: xx, key: 'key2'}
]


```

经过`createKeyToOldIdx`转换后就会变为

```javascript
{
    key0: 0, 
    key1: 1, 
    key2: 2
}

```

通过这种方式，就可以在`oldCh`中快速找到与当前节点(`newStartVnode`) `key`相同的节点的`索引idxInOld`.

```javascript
if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)

```

-   [ ] 如果没有找到这个`idxInOld`,则通过 createElm 创建一个新节点，并将 `newStartIdx` 向后移动一位。

```javascript
if (isUndef(idxInOld)) { // 创建新节点
  createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
}


```

-   [ ] 如果存在这个`idxInOld`,且符合`sameVnode`,则执行`patchVnode`并将`oldCh[idxInOld] = undefined`,最后将`newStartIdx` 向后移动一位。

```javascript
vnodeToMove = oldCh[idxInOld]
if (sameVnode(vnodeToMove, newStartVnode)) {
    patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
    oldCh[idxInOld] = undefined
    canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
}

```

![](image_zN18FsRp9g.png)

-   [ ] 如果不符合 `sameVnode`，只能创建一个新节点插入到 `parentElm` 的子节点中，`newStartIdx` 往后移动一位。

一张图总结

![](9b89cbc2280940038f5550bf9b7370aa~tplv-k3u1fbpfcp-z.webp)

最后，当`while`循环执行完成,会有两种情况

1.  [ ] 如果 `oldStartIdx > oldEndIdx`，说明老节点比对完了，但是新节点还有多的，需要将新节点插入到真实 DOM 中去，调用 `addVnodes` 将这些节点插入即可。
2.  [ ] 如果 `newStartIdx > newEndIdx` 条件，说明新节点比对完了，老节点还有多，将这些无用的老节点通过 `removeVnodes` 批量删除即可。
