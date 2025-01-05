---
title: 🔑输入密码时判断是否开启大写锁定
date: 2024-07-23 10:00:00
tags:
  - codeKey
categories:
  - js
cover: 
---
# 前言

任何人在任何时候都有可能悄无声息的把大写锁定打开。在大部分输入时，用户可以轻易地发现输入的是大写。但，在输入密码时，用户就不容易发现了。为了更好的用户体验，我们可以在用户输入密码时提示大写锁定已开启。你也一定见过这种场景吧。

# 提示用户大写锁定已开启

```javascript
const inputElement = document.querySelector('#your-input-element');

inputElement.addEventListener('keydown', function(event) {
    if (event.getModifierState('CapsLock')) {
        console.log('大写锁定是打开的');
    } else {
        console.log('大写锁定是关闭的');
    }
});
```

上面的代码是在监听用户在输入框输入时判断的，当然也可以在按下任何键的时候判断。

# getModifierState

```javascript
getModifierState(key)
```

KeyboardEvent.getModifierState() 方法返回指定修饰键的当前状态：如果修饰键处于活动状态（即修饰键被按下或锁定），则返回 true ，否则返回 false。

参数key还可以是Alt、NumLock等。对修饰符及不同平台的支持可见此[链接](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/getModifierState#modifier_keys_on_firefox)。
## 使用场景
### 检测快捷键组合
检查多个修饰键是否同时被按下。
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('Control') && event.getModifierState('Alt') && event.key === 'S') {
        // 执行Ctrl+Alt+S的快捷操作
    }
});

```
### 优化输入体验
```js
document.addEventListener('input', function(event) {
    if (event.getModifierState('NumLock') && event.target.type === 'number') {
        // 提示用户NumLock可能影响输入
    }
});
```
### 创建自定义键盘快捷操作
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('Shift') && event.getModifierState('Alt') && event.key === 'Z') {
        // 执行自定义操作
    }
});
```
## 仅能使用getModifierState的场景
### 检测大写锁定
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('CapsLock')) {
        console.log('Caps Lock is active');
    }
});
```
### 检测数字锁定
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('NumLock')) {
        console.log('Num Lock is active');
    }
});
```
某些情况下，可能误触数字锁定键，或者初始某些键盘初始就是数字锁定的。当你输入u，显示4；当你输入k，显示的是2。这时候就需要按一下F11解除数字锁定。
### 检测滚动锁定
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('ScrollLock')) {
        console.log('Scroll Lock is active');
    }
});
```
### 检测其他特殊修饰键
```js
document.addEventListener('keydown', function(event) {
    if (event.getModifierState('AltGraph')) {
        console.log('AltGraph is active');
    }
    if (event.getModifierState('OS')) {
        console.log('OS key (Windows key or Command key) is active');
    }
});
```
# 参考链接
[MDN: getModifierState](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/getModifierState)  
[David Walsh: detect-caps-lock](https://davidwalsh.name/detect-caps-lock)
