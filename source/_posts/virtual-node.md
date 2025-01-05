---
title: 【翻译】200行JS实现的虚拟DOM
date: 2024-07-04 10:00:00
tags:
  - vue
  - VDOM
categories:
  - 源码
cover: 
---
# 前言

在这篇文章中，作者将演示如何用200多行JS实现一个虚拟DOM。而且实现的是一个功能齐全、性能足够的npm库。主要目标是说明react、vue等框架背后的基本技术。react、vue、Elm都允许用户描述网页外观，从而简化了交互式网页创建，不必使用添加/删除元素来实现该页面，它们都是通过虚拟DOM来做到的。

Elm是一个纯函数式的语言，它声明式的创建网页。

# 虚拟DOM的目标

它与性能无关。

虚拟DOM是一种抽象，它用于简化修改UI的操作。

用户描述期待的外观，库负责将DOM从当前状态转换成用户期望的状态。

# 关键思想

该库将接管单个DOM并在其中进行操作。

该元素最初应该是空的，而且假设除了该库没有任何东西能修改它，这是用户应用程序的根。

如果只有我们能修改它，那么我们不需要检查就可以知道元素里都有什么。如何知道？通过追踪我们对它的所有修改。

我们将通过保留包含每个HTML元素简化表示的结构，来追踪根节点的内容。准确的说，是每个DOM节点。

因为这个表示是真实DOM的映射，不在真实的DOM中，所以称其为虚拟节点。它们组成虚拟DOM。

用户永远不会创建真实的 DOM 节点，只会创建那些虚拟的节点。他们会告诉我们通过使用虚拟节点整个页面应该是什么样子。然后我们的库将负责修改真实的 DOM，使其符合我们的预期。

为了知道要修改什么，我们的库将获取用户创建的虚拟 DOM，并将其与表示页面当前外观的虚拟 DOM 进行比较。这个过程称为比较。它将记录差异，例如应添加或删除哪些元素以及应添加或删除哪些属性。比较的输出是虚拟 DOM 之间的差异。

然后我们将该差异的更改应用到真实的 DOM 中。一旦我们完成修改，用户创建的虚拟 DOM 就成为真实 DOM 的当前真实表示。

所以，对于UI部分，我们需要以下3点：

- 创建DOM的虚拟表示；

- 对比虚拟DOM节点；

- 将虚拟DOM的差异更新到真实的HTML元素上；

构建之后，我们将看到如何通过在短短几行代码中添加状态处理来充分利用这种虚拟DOM作为一个强大的库。

> 总的来说，用一个对象表示一个DOM节点，对象之间构成树状关系。这种对象只有当前库能够修改，追踪库对这种对象的所有修改。发生变化时，创建新的对象，对比新旧对象差异，然后按差异修改真实DOM。

# 代表DOM

我们希望这个结构包含尽可能少的信息，以忠实的表示页面中的内容。

DOM节点具有标签名、属性和子节点。所以表示DOM的对象的结构如下所示。
```js
const exampleButton = { 
    tag : "button", 
    properties: { 
        class: "primary", 
        disabled: true, 
        onClick: doSomething 
    }, 
    children : [] // 数组元素类型与当前对象相同 
};
```
文本节点只有文字内容，需要单独表示，如下所示。
```js
const exampleText = { text: "Hello World" };
```
我们可以通过检查tag或text属性是否存在来区分DOM节点和文本节点。

方便起见，我们可以创建一个函数来创建这些节点，如下所示。
```js
function h(tag, properties, children) { 
    return { tag, properties, children); 
} 
function text(content) { 
    return { text : content }; 
};
```
现在，我们可以轻松的创建出复杂的嵌套结构，如下所示。
```js
const pausedScreen = h("div", {}, [ 
    h("h2", {}, text("Game Paused")), 
    h("button", { onClick: resumeGame }, [ text("Resume") ]), 
    h("button", { onClick: quitGame }, [ text("Quit") ]) 
])
```
# 比较

在开始比较之前，先思考一下比较操作的期望输出是什么。

比较操作描述如何修改元素，我们可以根据修改类型来判断输出。

- 创建：应返回要添加的虚拟DOM节点；

- 删除：无返回；

- 替换：返回新的DOM节点；

- 修改某节点：要添加或删除的属性、对子节点的修改数组；

- 无变化：不需要任何操作。

你可能想知道为什么有替换，因为除非用户为每个虚拟DOM节点提供一个唯一标识符，否则我们无法确定子元素的顺序是否发生变化。

比如，初始DOM描述如下：
```js
{ 
    tag: "div", 
    properties: {}, 
    chidlren: [ { text: "One" }, { text: "Two" }, { text: "Three" } ] 
}
```
发生修改后的DOM描述如下：
```js
{ 
    tag: "div", 
    properties: {}, 
    chidlren: [ { text: "Three" } { text: "Two" }, { text: "One" }, ] 
}
```
一和三交换了位置，我们必须将第一个对象的每个子对象与第二个对象的每个子对象进行比较，这效率太低了。所以，我们通过children数组的索引来识别元素。这意味着我们仅需交互第一个和第三个子对象。

所以，当插入一个元素作为最后一个子元素时，使用create。否则，使用replace。

如果没有唯一ID标识子元素，我们将通过O(n²)的比较才知道交换了位置。

现在，让我们来直接深入并实现这个diff函数。
```js
function diffOne(l, r) { 
    const isText = l.text !== undefined; 
    // 处理文本节点 
    if (isText) { 
        // replace替换 noop无变化 
        return l.text !== r.text ? {replace: r} : {noop: true}; 
    } 
    // 替换整个节点 
    if (l.tag !== r.tag) { 
        return {replace: r}; 
    } 
    // 接下来是修改 
    // 获取删除的属性 
    const remove = []; 
    for(const prop in l.properties) { 
        if (r.properties[prop] === undefined) { 
            remove.push(prop); 
        } 
    } 
    // 获取新增和修改的属性 
    const set = {}; 
    for(const prop in r.properties) { 
        if (r.properties[prop] !== l.properties[prop]) { 
            set[prop] = r.properties[prop]; 
        } 
    } 
    // 对比children 
    const children = diffList(l.children, r.children); 
    return {modify: {remove, set, children}}; 
}
```
作为一种优化，我们发现，当没有属性更改而且所有子元素修改都是noop时，可以设置该元素为noop。

对比children的方法如下所示。
```js
function diffList(ls, ls) { 
    const length = Math.max(ls.length, rs.length); 
    return Array.from({length}).map((_, i) => { 
        if (ls[i] === undefined) { 
            return {create: rs[i]}; 
        } else if (rs[i] === undefined) { 
            return {remove: true}; 
        } else { 
            diffOne(ls[i], rs[i]); 
        } 
    }) 
}
```
# 应用差异

我们已经可以创建一个虚拟 DOM 并对其进行比较。现在是时候将 diff 应用于真实的 DOM 了。

apply函数接收2个参数，第一个参数是一个真实的DOM节点，第二个参数是上一步中的子节点的差异数组。

apply函数没有任何返回值，它的主要目的是执行修改DOM的副作用。
```js
function apply(el, childrenDiff) { 
    const children = Array.from(el.childNodes); 
    childrenDiff.forEach((diff, i) => { 
        const action = Object.keys(diff)[0]; 
        switch (action) { 
            case 'remove': 
                children[i].remove(); 
                break; 
            case 'modify': 
                modify(children[i], diff.modify); 
                break; 
            case 'create': 
                // 添加{}是为了让下面创建的child在块级作用域中 
                { 
                    const child = create(diff.create); 
                    el.appendChild(child); 
                    break; 
                } 
            case 'replace': 
                { 
                    const child = create(diff.replace); 
                    children[i].replaceWith(child); 
                    break; 
                } 
            case 'noop': 
                break; 
        } 
    }) 
}
```
## 事件监听器

在处理创建和修改之前，先考虑一下如何处理事件监听器。

期望方便添加和删除监听器，不会留下监听器未绑定，一个节点的每个事件只有一个监听器，事件名作为key且不能重复。

我们可以向DOM对象添加一个自定义的特殊属性，如下所示：
```js
element["_ui"] = { listeners : { click: doSomething } };
```
创建一个listener函数，一旦触发事件，该函数就会获取此事件，并分配给对应的用户自定义函数来处理，如下所示。
```js
function listener(event) { 
    const el = event.currentTarget; 
    const handler = el._ui.listeners[event.type]; 
    handler(event); 
}
```
采用这种办法，在修改监听器函数时，不需要调用addEventListener和removeEventListener，只需要修改listeners中对象的值。

接下来，我们创建一个函数来将事件监听器添加到DOM节点上。
```js
function setListener(el, event, handle) { 
    if (el._ui.listeners[event] === undefined) { 
        el.addEventListener(event, listener); 
    } 
    el._ui.listeners[event] = handle; 
}
```
另外，我们还需要判断properties中的属性名是否是事件监听器。
```js
function eventName(str) {
  if (str.indexOf("on") == 0) {
    return str.slice(2).toLowerCase();
  }
  return null;
}
```
## 属性

一部分属性需要调用setAttribute设置，另一部分属性需要直接在DOM节点上设置。比如，给复选框设置是否选中，如下所示：
```js
checkboxDom.checked = true; // √ 
checkboxDom.setAttribute("checked", true); // × 不生效
```
可以将其中一部分属性放在列表中，根据是否存在于列表中来决定设置属性的方式。
```js
const props = new Set([ "autoplay", "checked", "checked", "contentEditable", "controls",
  "default", "hidden", "loop", "selected", "spellcheck", "value", "id", "title",
  "accessKey", "dir", "dropzone", "lang", "src", "alt", "preload", "poster",
  "kind", "label", "srclang", "sandbox", "srcdoc", "type", "value", "accept",
  "placeholder", "acceptCharset", "action", "autocomplete", "enctype", "method",
  "name", "pattern", "htmlFor", "max", "min", "step", "wrap", "useMap", "shape",
  "coords", "align", "cite", "href", "target", "download", "download",
  "hreflang", "ping", "start", "headers", "scope", "span" ]);

function setProperty(prop, value, el) {
  if (props.has(prop)) {
    el[prop] = value;
  } else {
    el.setAttribute(prop, value);
  }
}
```
## 创建和修改

根据虚拟DOM对象创建真实DOM对象的方法如下所示：
```js
function create(vnode) {
  // 文本节点
  if (vnode.text!== undefined) {
    const el = document.createTextNode(vnode.text);
    return el;
  }
  // 元素节点
  const el = document.createElement(vnode.tag);
  el._ui = {listeners: {}};
  for(const prop in vnode.properties) {
    const event = eventName(prop);
    const value = vnode.properties[prop];
    if (event !== null) {
      setListener(el, event, value);
    } else {
      setProperty(prop, value, el);
    }
  }
  for(const childVNode of vnode.children) {
    const child = create(childVNode);
    el.appendChild(child);
  }
  return el;
}
```
修改如下所示：

```js
function modify(el, diff) {
  for(const prop in diff.remove) {
    const event = eventName(prop);
    if (event === null) {
      el.removeAttribute(prop);
    } else {
      el._ui.listeners[event] = undefined;
      el.removeEventListener(event, listener);
    }
  }
  for(const prop in diff.set) {
    const value = diff.set[prop];
    const event = eventName(prop);
    if (event === null) {
      setProperty(prop, value, el);
    } else {
      setListener(el, event, value);
    }
  }
  apply(el, diff.children);
}
```

在上面的代码中，modify和apply存在递归调用。

# 处理状态

我们现在有了一个完整的虚拟DOM渲染实现。使用h和text可以创建一个虚拟DOM，使用apply和diffList将其具体化为真实DOM并更新它。

我们可以到此为止，但我认为如果没有结构化的方法来处理状态更改，那么实现是不完整的。总之，虚拟 DOM 的全部意义在于，当状态发生变化时，您可以重复地重新创建它。

## API

有两种类型的用户定义值：

- 应用程序状态**state**：包含渲染VDOM所需的所有信息的值；

- 应用程序消息**msg**：包含有关如何更改状态的信息的值；

我们会要求用户实现两个功能：

- **view**函数：参数是应用程序状态，返回值是VDOM；

- **update**函数：参数是应用程序状态和一条应用程序消息，返回值是一个新的应用程序状态；

用户在程序开始时提供这两个函数，VDOM 库将控制何时调用它们。用户从不直接调用它们。

另外，用户根据我们返回的**enqueue**函数来发出要通过 update 函数处理的消息。即，调用enqueue时传的参数会是update函数的第2个参数。

最后，用户还需要提供初始状态和VDOM挂载的节点。

我们定义一个名为 init 的函数，它将获取用户所需的所有输入并启动应用程序。它将返回该应用程序的 enqueue 函数。这种设计允许我们在同一页面中运行多个 VDOM 应用程序，并且每个应用程序都有自己的 enqueue 功能。

以实现一个计数器为例，如下所示：
```js
function view(state) {
    return [
        h("p", {}, [ text(`Counter: ${state.counter}`) ])
    ];
}

function update(state, msg) {
    return { counter : state.counter + msg }
}

const initialState = { counter: 0 };

const root = document.querySelector(".my-application");

const { enqueue } = init(root, initialState, update, view);

setInterval(() => enqueue(1), 1000);
```
## init函数

思考一下如何实现init函数。

我们肯定会为每条消息调用一次 update 。但我们不需要每次状态更改时都调用 view ，因为这可能会导致我们更新 DOM 的频率超过浏览器能够显示 DOM 更新的频率。我们希望*每个动画帧*最多调用一次 view 。

此外，我们希望用户能够根据需要多次调用 enqueue ，并且可以*从任何地方调用*，而不会导致我们的应用程序崩溃。这意味着我们应该接受 enqueue 被调用，即使是在 update 函数中。

我们将通过解耦消息队列、更新状态和更新 DOM 来实现这一点。

调用 enqueue 只会将消息添加到数组中。然后，在每个动画帧上，我们将获取所有排队的消息，并通过在每个动画帧上调用 update 来处理它们。处理完所有消息后，我们将使用 view 函数呈现结果状态。

具体实现如下所示：
```js
function init(root, initialState, update, view) {
  let state = initialState;
  let nodes = []; // 虚拟DOM
  let queue = []; // 消息队列

  function enqueue(msg) {
    queue.push(msg);
  }

  // 根据当前状态渲染页面
  function draw() {
    let newNodes = view(state); // 根据初始状态返回虚拟node
    apply(root, diffList(nodes, newNodes)); // 对比新旧虚拟node差异，然后更新真实DOM
    nodes = newNodes; // 将虚拟DOM设置为新的
  }

  function updateState() {
    if (queue.length > 0) {
      let msgs = queue;
      // 将队列替换为空数组，以便我们在本轮中不处理新排队的消息
      queue = [];
      for (msg of msgs) {
        state = update(state, msg);
      }
      draw();
    }
    window.requestAnimationFrame(updateState);
  }

  draw();         // 根据初始状态渲染
  updateState();  // 启动状态更新周期

  return { enqueue };
}
```
目前，不能从update和view中调用enqueue，因为enqueue从init返回，而执行init时期望已定义update和view。

想在update中调用enqueue，可以将enqueue作为update的第3个参数，如下所示：
```js
state = update(state, msg, enqueue)
```
接下来思考如何在view中调用enqueue。

用户在渲染期间不会调用 enqueue 。他们会调用它来响应某些事件，例如 onClick 或 onInput 。因此，用户为这些事件创建的处理函数接收 enqueue 作为参数以及事件对象是有意义的。
```js
const button = h(
    "button",
    { onClick : (_event, enqueue) => { enqueue(1) } },
    [text("Increase counter")]
);
```
我们可以通过将事件处理程序返回的与 undefined 不同的任何值都视为消息来使其变得更加容易。所以可以简写为如下所示：
```js
const button = h(
    "button",
    { onClick : () => 1 },
    [text("Increase counter")]
);
```
我们调度事件的单个 listener 函数需要访问 enqueue 。传递它的最简单方法是通过已保存用户定义侦听器的
**_ui ** 对象。
```js
function listener(event) {
  const el = event.currentTarget;
  const handler = el._ui.listeners[event.type];
  const enqueue = el._ui.enqueue;
  const msg = handler(event);
  if (msg !== undefined) {
    enqueue(msg);
  }
}
```
要在节点创建时将 enqueue 添加到 _ui ，我们需要将其传递给 apply、 modify 和 create 。
```js
function apply(el, enqueue, childrenDiff) { ... }
function modify(el, enqueue, diff) { ... }
function create(enqueue, vnode) { ... }
```
完整代码
```js
const props = new Set([ "autoplay", "checked", "checked", "contentEditable", "controls",
  "default", "hidden", "loop", "selected", "spellcheck", "value", "id", "title",
  "accessKey", "dir", "dropzone", "lang", "src", "alt", "preload", "poster",
  "kind", "label", "srclang", "sandbox", "srcdoc", "type", "value", "accept",
  "placeholder", "acceptCharset", "action", "autocomplete", "enctype", "method",
  "name", "pattern", "htmlFor", "max", "min", "step", "wrap", "useMap", "shape",
  "coords", "align", "cite", "href", "target", "download", "download",
  "hreflang", "ping", "start", "headers", "scope", "span" ]);
// 给真实DOM添加属性
function setProperty(prop, value, el) {
  if (props.has(prop)) {
    el[prop] = value;
  } else {
    el.setAttribute(prop, value);
  }
}

// 当DOM触发点击等事件时调用这个方法。执行绑定的监听回调函数，把监听回调函数的返回值作为消息传递给enqueue，enqueue来更新消息队列。
function listener(event) {
  const el = event.currentTarget;
  const handler = el._ui.listeners[event.type];
  const enqueue = el._ui.enqueue;
 
  const msg = handler(event);
  if (msg !== undefined) {
    enqueue(msg);
  }
}

// 在虚拟DOM创建真实DOM时绑定监听回调函数
function setListener(el, event, handle) {
 
  if (el._ui.listeners[event] === undefined) {
    el.addEventListener(event, listener);
  }

  el._ui.listeners[event] = handle;
}

// 判断event是否是可以监听的事件
function eventName(str) {
  if (str.indexOf("on") == 0) {
    return str.slice(2).toLowerCase();
  }
  return null;
}

// 比较虚拟DOM
function diffOne(l, r) {
  
  const isText = l.text !== undefined;
  if (isText) {
    return l.text !== r.text
      ? { replace: r }
      : { noop : true };
  }

  if (l.tag !== r.tag) {
    return { replace: r };
  }

  const remove = [];
  const set = {};

  for (const prop in l.properties) {
    if (r.properties[prop] === undefined) {
      remove.push(prop);
    }
  }

  for (const prop in r.properties) {
    if (r.properties[prop] !== l.properties[prop]) {
      set[prop] = r.properties[prop];
    }
  }

  const children = diffList(l.children, r.children);
  const noChildrenChange = children.every(e => e.noop);
  const noPropertyChange =
        (remove.length === 0) &&
        (Array.from(Object.keys(set)).length == 0);

  return (noChildrenChange && noPropertyChange)
    ? { noop : true }
    : { modify: { remove, set, children } };
}

function diffList(ls, rs) {
  
  const length = Math.max(ls.length, rs.length);
  return Array.from({ length })
    .map((_,i) =>
      (ls[i] === undefined)
      ? { create: rs[i] }
      : (rs[i] == undefined)
      ? { remove: true }
      : diffOne(ls[i], rs[i])
    );
}

// 根据新旧虚拟DOM比较结果，创建新的真实DOM
function create(enqueue, vnode) {
  
  if (vnode.text !== undefined) {
    const el = document.createTextNode(vnode.text);
    return el;
  }

  const el = document.createElement(vnode.tag);
  el._ui = { listeners : {}, enqueue };

  for (const prop in vnode.properties) {
    const event = eventName(prop);
    const value = vnode.properties[prop];
    (event === null)
      ? setProperty(prop, value, el)
      : setListener(el, event, value);
  }

  for (const childVNode of vnode.children) {
    const child = create(enqueue, childVNode);
    el.appendChild(child);
  }

  return el;
}

// 根据单个新旧虚拟DOM比较结果，修改真实DOM
function modify(el, enqueue, diff) {
  for (const prop of diff.remove) {
    const event = eventName(prop);
    if (event === null) {
      el.removeAttribute(prop);
    } else {
      el._ui.listeners[event] = undefined;
      el.removeEventListener(event, listener);
    }
  }

  for (const prop in diff.set) {
    const value = diff.set[prop];
    const event = eventName(prop);
    (event === null)
      ? setProperty(prop, value, el)
      : setListener(el, event, value);
  }

  
  apply(el, enqueue, diff.children);
}

// 遍历子虚拟DOM差异，修改真实DOM
// apply和modify互相递归调用
function apply(el, enqueue, childrenDiff) {
  const children = Array.from(el.childNodes);

  childrenDiff.forEach((diff, i) => {
    const action = Object.keys(diff)[0];
    switch (action) {
      case "remove":
        children[i].remove();
        break;

      case "modify":
        modify(children[i], enqueue, diff.modify);
        break;

      case "create": {
       
        const child = create(enqueue, diff.create);
        el.appendChild(child);
        break;
      }

      case "replace": {
        const child = create(enqueue, diff.replace);
        children[i].replaceWith(child);
        break;
      }

      case "noop":
        break;

      default:
        throw new Error("Unexpected diff option: " + Object.keys(diff));
    }
  });
}

// 创建虚拟DOM
function h(tag, properties, children) {
 
  return new VirtualNode({ tag, properties, children });
}


function text(content) {
  return new VirtualNode({ text: content });
}

// 初始化
function init(root, initialState, update, view) {
  let state = initialState;
  let nodes = [];
  let queue = [];

  function enqueue(msg) {
    queue.push(msg);
  }

  
  function draw() {
    let newNodes = view(state);
    apply(root, enqueue, diffList(nodes, newNodes));
    nodes = newNodes;
  }

  function updateState() {
    if (queue.length > 0) {
      let msgs = queue;
      queue = [];

      msgs.forEach(msg => {
        try {
          state = update(state, msg, enqueue);
        } catch (e) {
          console.error(e);
        }
      });

      draw();
    }

    window.requestAnimationFrame(updateState);
  }

  draw();
  updateState();

  return { enqueue };
}

 // 导出init、h、text
```
# 例子

计数器：
```js
<html>
  <body>
    <div id="container"></div>
    <script src="./smvc.js"></script>
    <script>
      const { init, h, text } = SMVC;
      const root = document.querySelector("#container");
      const initialState = 0;

      const update = (state, msg, enqueue) => state + msg;

      const view = (state) => [
        h("div", { style: "color: red", onClick: () => 2 }, [
          h("p", {}, [
            text(`The count is ${state}. Click here to increment.`)
          ])
        ])
      ];

      const { enqueue } = init(root, initialState, update, view);
      enqueue(1);
    </script>
  </body>
</html>
```
[todo MVC](https://github.com/lazamar/smvc/blob/main/demos/todoMVC.js)

[大量node](https://github.com/lazamar/smvc/blob/main/demos/million.js)

# 最后

[原文链接](https://lazamar.github.io/virtual-dom/)

[该文章所实现的库](https://github.com/lazamar/smvc/tree/main)
