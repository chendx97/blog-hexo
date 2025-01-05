---
title: 刮刮乐效果原来如此简单
date: 2023-09-07 10:00:00
tags:
  - canvas
categories:
  - HTML
cover: 
---

## 1.刮刮乐（橡皮擦）效果的核心api

    ctx.globalCompositeOperation = type;

设置要在绘制新形状时应用的合成操作的类型。
我们这里需要用到的类型是 `destination-out`
![1206102951-61b0b351184c4_fix732.webp](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/e5c12874897c4199aabb4fd83d4bec25~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)  
此属性的详细信息：[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation)

## 2.基础版刮刮乐功能

canvs 覆盖在图片上

![1658915698-61b0b4e96c7e7.webp](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/0c8a90facb104af6b99b2ca35dfd941e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

    <style>
        body {
          margin: 0;
        }
        img {
          width: 400px;
          height: 300px;
          left: 200px;
          position: absolute;
          z-index: -1;
        }
        canvas {
          margin-left: 200px;
        }
      </style>
      
      <img src="./test.jpg" alt="pic"/>
      <canvas id="canvas" width="400" height="300"></canvas>

<!---->

    <script>
       let canvas = document.querySelector('#canvas');
       let context = canvas.getContext('2d');
       // 绘制涂层
       context.beginPath();
       context.fillStyle = 'grey';
       context.fillRect(0, 0, 400, 300);
       // 监听鼠标移动事件
       canvas.addEventListener('mousemove', (e) => {
        // 当鼠标左键按下&&移动鼠标时，清除鼠标附近涂层
        if (e.which === 1 && e.button === 0) {
          const x = e.clientX, y = e.clientY;
          context.globalCompositeOperation = 'destination-out';
          context.beginPath();
          // 清除以鼠标位置为圆心，半径为10px的圆的范围
          context.arc(x - 200, y, 10, 0, Math.PI * 2);
          context.fill();
        }
       })
      </script>

## 3.进阶版刮刮乐功能

进阶功能：
点击时，以当前位置为圆心刮开一部分区域；
刮开x百分比（可以自定义）后后显示全部，并且使用动画逐渐变淡；
调用第一次刮的回调方法和刮完的回调方法，可以传入或不传；
涂层上面可以显示自定义文字；


![14236041-61b0b553e0efc.webp](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/338e11a92b3741d18d5378f3f5481b76~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

首先改为class形式，方便多次创建刮刮乐。

    class Scratch {
          constructor(id, { maskColor = 'grey', cursorRadius = 10 } = {}) {
            this.canvas = document.getElementById('canvas');
            this.context = this.canvas.getContext('2d');
            this.width = this.canvas.clientWidth;
            this.height = this.canvas.clientHeight;
            this.maskColor = maskColor; // 涂层颜色
            this.cursorRadius = cursorRadius; // 光标半径
            this.init();
          }
          init() {
            // 添加涂层
            this.addCoat();
            let bindEarse = this.erase.bind(this);
            this.canvas.addEventListener('mousedown', (e) => {
              // 按下左键
              if (e.which === 1 && e.button === 0) {
                // 擦掉涂层
                this.canvas.addEventListener('mousemove', bindEarse);
              }
            })
            document.addEventListener('mouseup', () => {
              this.canvas.removeEventListener('mousemove', bindEarse);
            })
          }
          addCoat() {
            this.context.beginPath();
            this.context.fillStyle = this.maskColor;
            this.context.fillRect(0, 0, this.width, this.height);
          }
          erase(e) {
            const x = e.clientX, y = e.clientY;
            this.context.globalCompositeOperation = 'destination-out';
            this.context.beginPath();
            this.context.arc(x - this.width / 2, y, this.cursorRadius, 0, Math.PI * 2);
            this.context.fill();
          }
        }
        new Scratch('canvas');

然后，记录鼠标位置，mouseup时判断是点击还是点击&移动鼠标，如果是点击则以当前位置为圆心刮开一部分区域；

    this.canvas.addEventListener('mousedown', (e) => {
          this.posX = e.clientX;
          this.posY = e.clientY;
          ...
    })
     document.addEventListener('mouseup', (e) => {
        if (this.posX === e.clientX && this.posY === e.clientY) {
          this.erase(e);
        }
         ...
    })

然后，判断刮开面积是否超过一半，如果是清空涂层；

    ImageData ctx.getImageData(sx, sy, sw, sh);

sx：将要被提取的图像数据矩形区域的左上角 x 坐标。
sy：将要被提取的图像数据矩形区域的左上角 y 坐标。
sw：将要被提取的图像数据矩形区域的宽度。
sh：将要被提取的图像数据矩形区域的高度。

ImageData 对象，包含canvas给定的矩形图像数据。可以用来判断是否被刮开。
每4个元素表示一个像素点的rgba值，所以可以判断第4个的值是否小于256的一半即128，如果小于128即可视为透明（被刮开）。

清空指定区域内容：

    void ctx.clearRect(x, y, width, height);

<!---->

    document.addEventListener('mouseup', (e) => {
       this.getScratchedPercentage();
        if (this.currPerct >= this.maxEraseArea) {
            this.context.clearRect(0, 0, this.width, this.height);
        }
    })

    getScratchedPercentage() {
        const pixels = this.context.getImageData(0, 0, this.width, this.height).data;
        let transparentPixels = 0;
        for (let i = 0; i < pixels.length; i += 4) {
             if (pixels[i + 3] < 128) {
                transparentPixels++;
              }
        }
        this.currPerct = (transparentPixels / pixels.length * 4 * 100).toFixed(2);
    }

然后，设置第一次刮的回调方法和刮完的回调方法；

    constructor(id, { maskColor = 'grey', cursorRadius = 10, maxEraseArea = 50,
        firstEraseCbk = () => { }, lastEraseCbk = () => { } } = {}) {
        ...
        this.firstEraseCbk = firstEraseCbk; // 第一次刮的回调函数
        this.lastEraseCbk = lastEraseCbk; // 刮开的回调函数
    }

    this.canvas.addEventListener('mousedown', (e) => { 
        if (this.currPerct === 0) {
            this.firstEraseCbk();
        }
    })
    document.addEventListener('mouseup', (e) => {
        if (this.currPerct >= this.maxEraseArea) {
            this.context.clearRect(0, 0, this.width, this.height);
            this.lastEraseCbk();
        }
    })

然后，刮开全部时慢慢清空涂层，设置背景色渐变效果；
`requestAnimationFrame` 做出来的动画更流畅
回调函数用闭包形式可以给回调函数传参

    document.addEventListener('mouseup', (e) => {
        if (this.currPerct >= this.maxEraseArea) {
            this.done = true;
            requestAnimationFrame(this.fadeOut(255));
             this.lastEraseCbk();
        }
    })

    fadeOut(alpha) {
        return () => {
              this.context.save();
              this.context.globalCompositeOperation = 'source-in';
              this.context.fillStyle = this.context.fillStyle + (alpha -= 1).toString(16);
              this.context.fillRect(0, 0, this.width, this.height);
              this.context.restore();
              // 到210已经看不到涂层了
              if (alpha > 210) {
                requestAnimationFrame(this.fadeOut(alpha));
              }
         }
    }

然后，初始化涂层的时候，再涂层上显示自定义文字；

    addCoat() {
            ...
            if (this.text) {
              this.context.font = 'bold 48px serif';
              this.context.fillStyle = '#fff';
              this.context.textAlign = 'center';
              this.context.textBaseline = 'middle';
              this.context.fillText(this.text, this.width / 2, this.height / 2);
            }
    }

## 完整代码

    <!DOCTYPE html>
    <html lang="en">


    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      <style>
        body {
          margin: 0;
        }


        img {
          width: 400px;
          height: 300px;
          left: 200px;
          position: absolute;
          z-index: -1;
        }


        canvas {
          margin-left: 200px;
        }
      </style>
    </head>


    <body>
      <img src="./test.jpg" alt="pic" />
      <canvas id="canvas" width="400" height="300"></canvas>
      <script>
        class Scratch {
          constructor(id, { maskColor = 'grey', cursorRadius = 10, maxEraseArea = 50, text = '',
            firstEraseCbk = () => { }, lastEraseCbk = () => { } } = {}) {
            this.canvasId = id;
            this.canvas = document.getElementById(id);
            this.context = this.canvas.getContext('2d');
            this.width = this.canvas.clientWidth;
            this.height = this.canvas.clientHeight;
            this.maskColor = maskColor; // 涂层颜色
            this.cursorRadius = cursorRadius; // 光标半径
            this.maxEraseArea = maxEraseArea; // 刮开多少后自动清空涂层
            this.text = text;
            this.firstEraseCbk = firstEraseCbk; // 第一次刮的回调函数
            this.lastEraseCbk = lastEraseCbk; // 刮开的回调函数
            this.currPerct = 0; // 当前刮开多少百分比
            this.done = false; // 是否刮完
            this.init();
          }
          init() {
            // 添加涂层
            this.addCoat();
            let bindEarse = this.erase.bind(this);
            this.canvas.addEventListener('mousedown', e => {
              if (this.done) {
                return;
              }
              this.posX = e.clientX;
              this.posY = e.clientY;
              // 按下左键
              if (e.which === 1 && e.button === 0) {
                // 擦掉涂层
                this.canvas.addEventListener('mousemove', bindEarse);
              }
              if (this.currPerct === 0) {
                this.firstEraseCbk();
              }
            })
            document.addEventListener('mouseup', e => {
              if (this.done) {
                return;
              }
              if (e.target.id !== this.canvasId) {
                return;
              }
              if (this.posX === e.clientX && this.posY === e.clientY) {
                this.erase(e);
              }
              this.canvas.removeEventListener('mousemove', bindEarse);
              this.getScratchedPercentage();
              if (this.currPerct >= this.maxEraseArea) {
                this.done = true;
                requestAnimationFrame(this.fadeOut(255));
                this.lastEraseCbk();
              }
            })
          }
          // 添加涂层
          addCoat() {
            this.context.beginPath();
            this.context.fillStyle = this.maskColor;
            this.context.fillRect(0, 0, this.width, this.height);
            // 绘制涂层上的文字
            if (this.text) {
              this.context.font = 'bold 48px serif';
              this.context.fillStyle = '#fff';
              this.context.textAlign = 'center';
              this.context.textBaseline = 'middle';
              this.context.fillText(this.text, this.width / 2, this.height / 2);
            }
          }
          // 擦除某位置涂层
          erase(e) {
            const x = e.clientX, y = e.clientY;
            this.context.globalCompositeOperation = 'destination-out';
            this.context.beginPath();
            this.context.arc(x - this.width / 2, y, this.cursorRadius, 0, Math.PI * 2);
            this.context.fill();
          }
          // 计算被擦除的部分占全部的百分比
          getScratchedPercentage() {
            const pixels = this.context.getImageData(0, 0, this.width, this.height).data;
            let transparentPixels = 0;
            for (let i = 0; i < pixels.length; i += 4) {
              if (pixels[i + 3] < 128) {
                transparentPixels++;
              }
            }
            this.currPerct = (transparentPixels / pixels.length * 4 * 100).toFixed(2);
          }
          // 清空涂层时淡出效果
          fadeOut(alpha) {
            return () => {
              this.context.save();
              this.context.globalCompositeOperation = 'source-in';
              this.context.fillStyle = this.context.fillStyle + (alpha -= 1).toString(16);
              this.context.fillRect(0, 0, this.width, this.height);
              this.context.restore();
              // 到210已经看不到涂层了
              if (alpha > 210) {
                requestAnimationFrame(this.fadeOut(alpha));
              }
            }
          }
        }
        new Scratch('canvas', { text: '刮一刮', maxEraseArea: 10 });
      </script>
    </body>


    </html>
