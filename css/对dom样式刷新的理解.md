### GUI渲染线程和JS引擎线程
首先两者是互斥的。   

GUI渲染线程负责：   
* dom的修改(包括样式，内容等)
   
JS引擎线程负责：  
* js脚本的运行  

### GUI渲染线程刷新的时机    
在宏任务之间发生，即在当前宏任务之后，下一个宏任务之前，调用GUI渲染线程对修改的dom样式进行页面渲染。   
并不代表宏任务之间一定发生GUI渲染，一旦发生了重绘重排，才会导致GUI进行渲染。     

过程如下：       

  **宏任务1 => GUI渲染 => 宏任务2 => GUI渲染**
  
例如：
```html
  <a href="#" class="btn">点击</a>
	<div class="box" style=" background: #ddd;">
	</div>
```

```js
       const btn = document.querySelector('.btn');
       const box = document.querySelector('.box');
       // 当前宏任务
       btn.onclick= function() { 
       	box.style.background= 'green';
        box.style.left = '30px';
        console.log(box.style.left) // 30px, 能获取到因为操作的是dom对象
        
        // 下一个宏任务
        setTimeout(() => {     
           box.style.background= 'pink'; // 重置样式
           box.style.left= '50px';
           console.log(box.style.left) // 50px, 能获取到因为操作的是dom对象
        },5000)
       }
```
具体流程： **宏任务1(点击事件) => GUI渲染(渲染背景色green) => 宏任务2(定时器) => GUI渲染(重置背景色为pink)**   

在比如：
```js
       // 当前宏任务
       btn.onclick = function() {
           box.style.transform = 'translateX(500px)';
           box.style.background = 'green';
           
           // 下一个宏任务
       	   box.addEventListener('transitionend', function () {
       		 box.style.transform = 'translateX(0)';
       		 box.style.background = '#ddd';
       	   })
       }
```

所以，如果要重置样式，需要放在下一个宏任务中，下一个宏任务之后，就开始GUI渲染。      

如果所有样式放在一个宏任务内，样式能覆盖的覆盖，GUI渲染只渲染一次最终的样式     

例如：
```js
       box.style.left = '30px';
       box.style.top = '80px';
       box.style.left = '50px';
       box.style.background = 'green';
```
GUI只会渲染一次，渲染的结果为 left : '50px', top : '80px', background : 'green'
           
### 宏任务：      
**定时器，事件，ajax请求**
