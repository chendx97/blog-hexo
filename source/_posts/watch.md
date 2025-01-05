---
title: 📢彻底了解Vu3的watchEffect和watch
date: 2024-11-19 10:00:00
tags:
  - vue
  - watch
categories:
  - 源码
cover: 
---

# 前言
我经常使用`watch`和`watcheffect`，但前几天遇到一个稍微复杂点的功能，就出现了意料之外的情况，感觉对它们了解的还不够。所以，在此详细探索一下它们。
# watch vs watchEffect
它们的相同点是都可以监听**响应式数据**的变化，并执行回调函数。不同点比较多。
不同点如下：
- watch需要显示指定要监听的数据，watchEffect会**自动**收集依赖；
- watchEffect会**立即**执行一次，watch在设置`{immediate: true}`时才会立即执行一次；
- watch可以获取到**旧值**，watchEffect不可以；
- watch可以通过选项设置为**仅执行一次**，watchEffect不可以；
- watch可以设置监听对象的**层数**，watchEffect不可以；

官方文档也有指出，watch相对于watchEffect的优点：
> 与 watchEffect() 相比，watch() 使我们可以：
> - 懒执行副作用；
> - 更加明确是应该由哪个状态触发侦听器重新执行；
> - 可以访问所侦听状态的前一个值和当前值

综上所述，使用watchEffect虽然比较省事，但如果功能复杂很容易出现问题，灵活性也不如watch。比如，在watchEffect内调用一个函数，但这个函数内读取了某个响应式数据，导致这个数据被意外监听。所以，我推荐==任何情况都使用**watch**==。
# watch详解
## 第一个参数：侦听源
watch的第一个参数指定要监听哪些响应式数据。这些数据必须是响应式的，但可以包括以下几种：
- 一个ref
- 一个响应式对象
- 一个函数，这个函数返回一个值
- 由以上类型的值组成的数组

当第一个参数是响应式对象时，默认开启深度监听。
当第一个参数是函数时，该函数会立即执行。
当第一个参数是getter函数时，有以下三种情况：
- 如果该函数返回响应式对象，回调函数不触发；
```js
const state = reactive({ count: 0 });
watch(
  () => state,
  (newValue, oldValue) => {
    console.log(newValue === oldValue);
  },
);
```
- 如果返回响应式对象的某个属性，回调函数触发；
```js
const state = reactive({ count: 0 });
watch(
  () => state.count,
  (newValue, oldValue) => {
    console.log(newValue === oldValue);
  },
);
```
- 如果返回响应式对象，但开启`{deep: true}`，回调函数触发。但，此时新值和旧值相同。
```js
const state = reactive({ count: 0 });
watch(
  () => state,
  (newValue, oldValue) => {
    console.log(newValue === oldValue);
  },
  {
    deep: true,
  },
);
```
## 第二个参数：回调函数
watch的第二个参数会在监听的数据发生变化时被调用。如果监听的多个数据都发生变化，在一个事件循环周期内回调只执行一次。
该回调函数有3个参数：新值、旧值、一个用来**清理副作用**的方法。
当监听数组时，新值和旧值也都是数组，与监听源一一对应。
### 副作用清理
副作用清理的使用场景：异步未返回时，watch的回调就再次执行。
使用方法：把清理副作用的方法传入到`onCleanup`中，如下所示，`cancel`并不会马上执行，而是下一次执行watch的回调函数时执行。
```js
watch(id, async (newId, oldId, onCleanup) => {
  const { response, cancel } = doAsyncWork(newId) // !!!注意：没有await
  onCleanup(cancel)
  data.value = await response
})
```
vue3.5+引入了onWatcherCleanup，
```js
import { onWatcherCleanup } from 'vue'

watch(id, async (newId) => {
  const { response, cancel } = doAsyncWork(newId)
  onWatcherCleanup(cancel)
  data.value = await response
})
```
那么，onCleanup和onWatcherCleanup有什么不同呢？
1. 语义友好，onWatcherCleanup明确表示这个方法与watch的清理操作相关；
2. onWatcherCleanup更好的和vue3.5+的新特性结合，vue3.5+新增了watch选项、优化了watcher的管理机制；
3. 出现错误时，vue会为onWatcherCleanup返回更详细的错误信息；

==总结：如果你的vue版本是3.5+，请选择onWatcherCleanup。==
## 第三个参数：配置对象
第三个可选的参数是一个对象，支持以下选项：
- immediate: 立即触发回调，第一次调用时旧值是`undefined`；
- deep: 如果源是对象，进行深度遍历。在3.5+中可以设置为最大遍历深度的数字；
- once: 回调函数只执行一次；
- flush: 设置回调函数的刷新时机；
- onTrack/onTrigger: 调试侦听器的依赖；

前3个选项比较好理解也经常用，接下来了解一下后面2个。
### flush
flush的值有2个：`post`和`sync`。
默认情况下，watch的回调函数会在父组件更新之后、所属组件的DOM更新**之前**被调用。当你在回调函数中访问所属组件的DOM时，获取到的是**更新前的状态**。
当设置flush为post时，在回调函数中获取到的DOM是更新之后的状态。

默认情况下，回调函数会被批量处理，一个事件循环内只执行一次。
当设置flush为sync时，监听的数据变化时就马上触发回调函数。所以要慎重使用，可以监听布尔值，避免监听可能多次同步修改的数据源。
### onTrack/onTrigger
注意：这两个方法仅在开发模式下生效。
```js
watch(source, callback, {
  onTrack(e) {
    debugger
  },
  onTrigger(e) {
    debugger
  }
})
```
onTrack在响应式数据被追踪时调用，onTrigger在响应式数据发生变化时被调用。
可以用这2个方法在开发环境进行调试。
## 返回值：一个停止监听的函数
watch的返回值可以用来停止、暂停、恢复监听，类型定义如下所示：
```js
interface WatchHandle {
  (): void // 可调用，与 `stop` 相同
  pause: () => void
  resume: () => void
  stop: () => void
}
```
使用场景：在组件销毁时，或达到某个条件时，停止监听。

# watch源码解析
首先解释几个名词，假设监听源是source，监听的回调函数是cb，
- **traverse**: 一个深度优先遍历对象的方法。当source是**响应式对象**时，用traverse遍历source，那么响应式对象的每一个属性都执行了**读取操作**，所以当响应式对象任一属性发生变化时都能触发回调函数的执行。
- **getter**: 一个获取source的函数。根据source的类型，对source进行读取操作，使source的变化能被监听。
- **effect**: 副作用函数。 当响应式对象某个属性的值发生变化时，需要重新执行的函数。
- **onWatcherCleanup**: 一个注册清理副作用的函数，接收一个函数作为参数，比如把一个清除定时器的函数传给它。注意这个副作用不是指effect，而是指定时器、异步等，它们会影响下一次cb的执行。
- **boundCleanup**: 是onWatcherCleanup的绑定版本，把effect传给onWatcherCleanup。
- **cleanup**: 执行传给onWatcherCleanup的那些清理副作用的函数。
- **ReactiveEffect**: 一个类，它的实例是effect，它具有的属性和方法如下所示，
  - active: 布尔值，表示副作用函数是否处于活动状态，source变化时，当active是true，cb才重新执行；
  - deps: 数组，包含副作用函数的依赖项；
  - fn: 副作用函数本身；
  - scheduler: 一个调度器函数，控制副作用函数的执行时机；
  - scope: 副作用函数的作用域；
  - run: 一个方法，用于执行副作用函数；
  - stop: 一个方法，用于停止副作用函数；
  - pause: 一个方法，用于暂停副作用函数的执行；
  - resume: 一个方法，用于恢复副作用函数的执行；
  - onStop: 一个方法，stop被调用时执行；



完整源码：
```js
export function watch(source, cb, options) {
  /**
   * immediate: 是否立即执行回调函数
   * deep: 是否深度监听
   * once: 是否只监听一次
   * scheduler: 调度器，用于控制回调函数的执行时机
   * augmentJob: 对job函数进行增强，即执行job函数时做些额外操作
   * call: 传入一个方法，处理方法执行中出现的异常。第一个参数是要被调用的函数fn，第二个参数表示fn的类型，
   *       第三个参数是传递给fn的参数
   */
  const { immediate, deep, once, scheduler, augmentJob, call } = options;

  // 根据deep配置和isShallow(source)确定如何读取source
  const reactiveGetter = (source) => {
    if (deep) return source
    if (isShallow(source) || deep === false || deep === 0)
      return traverse(source, 1)
    return traverse(source)
  }

  let effect; // 副作用函数实例
  let getter; // 读取数据
  let cleanup;
  let boundCleanup;
  // 当数据源是(或包含)响应式对象时，由于引用类型的特性，新旧对象相同，无法确定是否发生了改变，所以需要强制触发回调函数的调用
  let forceTrigger = false;
  let isMultiSource = false; // 是否监听多个数据源

  if (isRef(source)) {
    getter = () => source.value
    forceTrigger = isShallow(source)
  } else if (isReactive(source)) {
    getter = () => reactiveGetter(source)
    forceTrigger = true
  } else if (isArray(source)) {
    isMultiSource = true
    forceTrigger = source.some(s => isReactive(s) || isShallow(s))
    getter = () =>
      source.map(s => {
        if (isRef(s)) {
          return s.value
        } else if (isReactive(s)) {
          return reactiveGetter(s)
        } else if (isFunction(s)) {
          return call ? call(s, WatchErrorCodes.WATCH_GETTER) : s()
        } else {
          // warnInvalidSource 打印警告信息
          __DEV__ && warnInvalidSource(s)
        }
      })
  } else if (isFunction(source)) {
    // 当source是函数时，cb可以不传，source作为cb，相当于watchEffect
    if (cb) {
      getter = call
        ? () => call(source, WatchErrorCodes.WATCH_GETTER)
        : source
    } else {
      getter = () => {
        // 当source作为cb时，source 函数可能会创建一些需要在函数执行结束后清理的资源，例如定时器、事件监听器等，
        // 因此，这种情况需要调用cleanup清除之前的副作用。
        if (cleanup) {
          // 在清理副作用时，需要暂停依赖收集，以避免在清理过程中触发不必要的副作用。
          pauseTracking()
          try {
            cleanup()
          } finally {
            resetTracking()
          }
        }
        // activeWatcher：vue需要知道响应式数据是被哪个watcher访问的，以便这些数据变化时通知对应watcher
        // 在副作用函数执行前，暂存原来的activeWatcher，并将当前副作用effect赋值给activeWatcher
        const currentEffect = activeWatcher
        activeWatcher = effect
        try {
          return call
            ? call(source, WatchErrorCodes.WATCH_CALLBACK, [boundCleanup])
            : source(boundCleanup)
        } finally {
          // 在副作用函数执行结束后，恢复原来的activeWatcher
          activeWatcher = currentEffect
        }
      }
    }
  } else {
    getter = NOOP // NOOP是一个空函数
    __DEV__ && warnInvalidSource(source)
  }

  // 处理deep选项
  if (cb && deep) {
    const baseGetter = getter
    const depth = deep === true ? Infinity : deep
    getter = () => traverse(baseGetter(), depth)
  }

  // scope：当前作用域
  const scope = getCurrentScope()
  // watchHandle是watch的返回值，可以暂停、恢复、停止监听，直接调用就是停止监听
  const watchHandle = () => {
    effect.stop()
    if (scope && scope.active) {
      remove(scope.effects, effect)
    }
  }

  // 处理once选项
  if (once && cb) {
    const _cb = cb
    cb = (...args) => {
      _cb(...args)
      watchHandle()
    }
  }

  // INITIAL_WATCHER_VALUE 是{}
  // 初始化旧值
  let oldValue = isMultiSource
    ? new Array(source.length).fill(INITIAL_WATCHER_VALUE)
    : INITIAL_WATCHER_VALUE

  // job的作用是在数据源发生变化时，执行回调函数，并更新旧值。
  const job = (immediateFirstRun) => {
    // 如果effect没有处于活动状态，或者effect的dirty标志位为false且immediateFirstRun为false，则直接返回。
    if (
      !(effect.flags & EffectFlags.ACTIVE) ||
      (!effect.dirty && !immediateFirstRun)
    ) {
      return
    }
    if (cb) {
      const newValue = effect.run()
      // 如果deep选项为true，或者需要强制触发回调，或者新值和旧值不相等，则执行回调函数。
      if (
        deep ||
        forceTrigger ||
        (isMultiSource
          ? newValue.some((v, i) => hasChanged(v, oldValue[i]))
          : hasChanged(newValue, oldValue))
      ) {
        if (cleanup) {
          cleanup()
        }
        const currentWatcher = activeWatcher
        activeWatcher = effect
        try {
          // watch的第二个参数cb，cb接受3个参数：新值、旧值、用来清理副作用的cleanup函数
          const args = [
            newValue,
            oldValue === INITIAL_WATCHER_VALUE
              ? undefined
              : isMultiSource && oldValue[0] === INITIAL_WATCHER_VALUE
                ? []
                : oldValue,
            boundCleanup,
          ]
          call
            ? call(cb, WatchErrorCodes.WATCH_CALLBACK, args)
            :
            cb(...args)
          oldValue = newValue
        } finally {
          activeWatcher = currentWatcher
        }
      }
    } else {
      // 执行source函数
      effect.run()
    }
  }

  // 如果用户自定义了augmentJob函数，则调用该函数对job进行增强。
  if (augmentJob) {
    augmentJob(job)
  }

  effect = new ReactiveEffect(getter)

  // 如果用户自定义了scheduler函数，则将该函数作为effect的调度器
  effect.scheduler = scheduler
    ? () => scheduler(job, false)
    : job

  // boundCleanup是对onWatcherCleanup的包裹
  boundCleanup = fn => onWatcherCleanup(fn, false, effect)

  cleanup = effect.onStop = () => {
    const cleanups = cleanupMap.get(effect)
    if (cleanups) {
      if (call) {
        call(cleanups, WatchErrorCodes.WATCH_CLEANUP)
      } else {
        for (const cleanup of cleanups) cleanup()
      }
      cleanupMap.delete(effect)
    }
  }

  if (__DEV__) {
    effect.onTrack = options.onTrack
    effect.onTrigger = options.onTrigger
  }

  if (cb) {
    if (immediate) {
      job(true)
    } else {
      // 执行source函数，并更新旧值
      oldValue = effect.run()
    }
  } else if (scheduler) {
    scheduler(job.bind(null, true), true)
  } else {
    effect.run()
  }

  watchHandle.pause = effect.pause.bind(effect)
  watchHandle.resume = effect.resume.bind(effect)
  watchHandle.stop = watchHandle

  return watchHandle
}

```