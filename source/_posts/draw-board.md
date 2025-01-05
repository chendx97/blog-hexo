---
title: 用canvas实现一个简单的画板
date: 2023-09-13 10:00:00
tags:
  - canvas
categories:
  - HTML
cover: 
---

## 1.画板的功能

* 修改画笔颜色；
* 修改画笔粗细；
* 橡皮擦；
* 重置画板；
* 撤销上一步；
* 保存成图片；
## 2.所需知识
`Element.getBoundingClientRect()` 方法返回元素的大小及其相对于视口的位置。
`ctx.moveTo(x, y)` 将一个新的子路径的起始点移动到(x，y)坐标
`ctx.lineTo(x, y)` 使用直线连接子路径的终点到x，y坐标
## 3.一步步实现
#### 第一步，实现基本功能，可以画出来鼠标路径；

```
<canvas id="myCanvas" width="400" height="400"></canvas>

class Board {
  constructor(id) {
    this.canvas = document.getElementById(id);
    this.context = this.canvas.getContext('2d');
    this.isDrawing = false;
    this.posX = 0;
    this.posY = 0;
    this.init();
  }
  init() {
    const bindDown = this.handleMouseDown.bind(this);
    const bindMove = this.handleMouseMove.bind(this);
    this.canvas.addEventListener('mousedown', bindDown);
    this.canvas.addEventListener('mousemove', bindMove);
    window.addEventListener('mouseup', () => {
      this.isDrawing = false;
    });
  }
  handleMouseDown(e) {
    const rect = this.canvas.getBoundingClientRect();
    this.posX = e.clientX - rect.left;
    this.posY = e.clientY - rect.top;
    this.isDrawing = true;
  }
  handleMouseMove(e) {
    if (this.isDrawing === true) {
      const rect = this.canvas.getBoundingClientRect();
      this.drawLine(this.context, this.posX, this.posY,
		 e.clientX - rect.left, e.clientY - rect.top);
      this.posX = e.clientX - rect.left;
      this.posY = e.clientY - rect.top;
    }
  }
  drawLine(context, x1, y1, x2, y2) {
    context.beginPath();
    context.strokeStyle = 'black';
    context.lineWidth = 1;
    context.moveTo(x1, y1);
    context.lineTo(x2, y2);
    context.stroke();
    context.closePath();
  }
}
new Board('myCanvas');
```
#### 第二步，可以修改画笔颜色；

```
<input id="colorPicker" type="color" />

document.getElementById('colorPicker').addEventListener('change', e => {
      b.changeColor(e.target.value);
    })
    
    class Board {
        constructor(id, color = '#000') {
              this.penColor = color; 
        }  
        drawLine(context, x1, y1, x2, y2) {
          context.strokeStyle = this.penColor;          
        }
        changeColor(color) {
        this.penColor = color;
      }              
    }
```
#### 第三步，修改画笔粗细；

```
ctx.lineWidth = number;
```
#### 第四步，橡皮擦；
`context.globalCompositeOperation = 'destination-out';`
参照刮刮乐功能。
#### 第五步，重置画板；
`context.clearRect(0, 0, width, height);`
#### 第六步，撤销上一步；
`this.canvas.toDataURL()`
将当前canvas保存为base64的图片，存放在数组中。再设置一个索引，撤销/恢复修改索引的值，从数组中取出对应的图片。
#### 第七步，保存为图片；
创建一个a标签，href为toDataURL()生成的图片，模拟点击事件，点击a链接。
### 4.完整代码
```
<div class="opera">
    <input id="colorPicker" type="color" />
    <select id="fontsizeSelect">
      <option value="1">1</option>
      <option value="2">2</option>
      <option value="3">3</option>
      <option value="4">4</option>
      <option value="5">5</option>
    </select>
    <button id="eraser">橡皮擦</button>
    <button id="reset">重置</button>
    <button id="revoke">撤销</button>
    <button id="recover">恢复</button>
    <button id="saveAsPic">保存为图片</button>
  </div>
  <canvas id="myCanvas" width="400" height="400"></canvas>
```
```
 class Board {
      constructor(id, color = '#000', fontsize = 1) {
        this.canvas = document.getElementById(id);
        this.context = this.canvas.getContext('2d');
        this.isDrawing = false;
        this.posX = 0;
        this.posY = 0;
        this.penColor = color;
        this.fontsize = fontsize;
        this.isErasing = false;
        this.step = 0;
        this.histroyList = [];
        this.init();
      }
      init() {
        const bindDown = this.handleMouseDown.bind(this);
        const bindMove = this.handleMouseMove.bind(this);
        this.canvas.addEventListener('mousedown', bindDown);
        this.canvas.addEventListener('mousemove', bindMove);
        window.addEventListener('mouseup', () => {
          this.isDrawing = false;
        });
        this.canvas.addEventListener('mouseup', () => {
          this.step++;
          if (this.step < this.histroyList.length) {
            this.histroyList.length = this.step;
          }
          this.histroyList.push(this.canvas.toDataURL());
        });
        this.histroyList.push(this.canvas.toDataURL());
      }
      handleMouseDown(e) {
        const rect = this.canvas.getBoundingClientRect();
        this.posX = e.clientX - rect.left;
        this.posY = e.clientY - rect.top;
        this.isDrawing = true;
      }
      handleMouseMove(e) {
        const rect = this.canvas.getBoundingClientRect();
        if (this.isErasing) {
          this.context.globalCompositeOperation = 'destination-out';
          this.context.beginPath();
          this.context.arc(e.clientX - rect.left, e.clientY - rect.top,
			 10, 0, Math.PI * 2);
          this.context.fill();
        } else if (this.isDrawing === true) {
          this.drawLine(this.context, this.posX, this.posY,
		 e.clientX - rect.left, e.clientY - rect.top);
          this.posX = e.clientX - rect.left;
          this.posY = e.clientY - rect.top;
        }
      }
      drawLine(context, x1, y1, x2, y2) {
        context.beginPath();
        context.strokeStyle = this.penColor;
        context.lineWidth = this.fontsize;
        context.moveTo(x1, y1);
        context.lineTo(x2, y2);
        context.stroke();
        context.closePath();
      }
      changeColor(color) {
        this.penColor = color;
      }
      changeFontSize(size) {
        this.fontsize = size;
      }
      switchEraseStatus() {
        this.isErasing = !this.isErasing;
      }
      clearBoard() {
        this.context.clearRect(0, 0, window.myCanvas.width,
	 window.myCanvas.height);
        this.step = 0;
        this.histroyList = [];
      }
      revoke() {
        if (this.step > 0) {
          this.step--;
          this.context.clearRect(0, 0, window.myCanvas.width, 
		window.myCanvas.height);
          let pic = new Image();
          pic.src = this.histroyList[this.step];
          pic.addEventListener('load', () => {
            this.context.drawImage(pic, 0, 0);
          })
        } else {
          console.log('不能继续撤销了')
        }
      }
      recover() {
        if (this.step < this.histroyList.length - 1) {
          this.step++;
          this.context.clearRect(0, 0, window.myCanvas.width,
       window.myCanvas.height);
          let pic = new Image();
          pic.src = this.histroyList[this.step];
          pic.addEventListener('load', () => {
            this.context.drawImage(pic, 0, 0);
          })
        } else {
          console.log('不能继续恢复了')
        }
      }
      saveAsPic() {
        const el = document.createElement('a');
        el.href = this.canvas.toDataURL();
        el.download = 'canvas';
        const event = new MouseEvent('click');
        el.dispatchEvent(event);
      }
    }
    const b = new Board('myCanvas');


    window.colorPicker.addEventListener('change', e => {
      b.changeColor(e.target.value);
    })


    window.fontsizeSelect.addEventListener('change', e => {
      b.changeFontSize(window.fontsizeSelect.value);
    })


    window.eraser.addEventListener('click', () => {
      b.switchEraseStatus();
    })


    window.reset.addEventListener('click', () => {
      b.clearBoard();
    })


    window.revoke.addEventListener('click', () => {
      b.revoke();
    })


    window.recover.addEventListener('click', () => {
      b.recover();
    })


    window.saveAsPic.addEventListener('click', () => {
      b.saveAsPic();
    })
```
