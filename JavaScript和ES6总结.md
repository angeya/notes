## JavaScript

### 数据类型（七种）

1. 5种基本数据类型：Number、String、Boolean、undefined、null

   （not defined是报错信息，和undefined没有什么关系）

2. 引用类型（Object）

   对象、数组、函数等

3. Symbol

   ES6中引入，表示独一无二的值

### 栈内存与堆内存

1. 基本数据类型存储于栈内存中

   栈内存大小比较固定，一般用于存储已知存储空间大小的数据

   每个严格相等的值只存储一遍（推测）

2. 引用类型存储于堆内存中，指针存放于栈内存

   堆内存大小不固定，一般用于存储大小容易发生改变的对象类型

3. 判断相等

   严格等于：=== 是判断地址是否相等，如栈堆内存中的同一个值，或者指向同一个堆内存对象的指针 它们都相等

   不严格等于：== 如果两个值类型不相等，则会做一次类型的转换再进行比较

### 面向对象编程

1. javascript中使用构造函数创建一个实例，构造函数如果不使用new关键字执行，则与普通函数一样

2. 构造函数与原型组合是创建对象很好的模式，构造函数中有属性，而原型中定义方法

3. 注意点1：原型中添加方法要在实例创建之前，否则方法对实例无效

   注意点2：构造函数中的this指向可能会改变，如addEventListener绑定事件时调用的方法，方法中的this将属于事件对象而不再属于构造函数。一些情况下使用self=this能解决大部分问题
   
   ```javascript
   function Person() {
       this.age = 123
       this.name = 'sunny'
   }
   Person.prototype.log = function () {
       console.log("my name is " + this.name)
   }
   const p = new Person()
   p.log() // my name is sunny
   console.log(p.name) // sunny
   ```

### Global对象

Global对象 所有的全局变量和函数都是Global对象的属性和方法，浏览器中是window对象

### hasProperty和HasOwnProperty的区别

hasProperty包括继承的属性

hasOwnProperty不包括继承得来的属性

### 函数声明和函数表达式

```javascript
// 函数声明
function hi() {
    ...
}
// 函数表达式
let hi = function() {
    ...
}
// 二者在使用上最大的区别就是函数声明会会使声明提升，即可以在函数声明之前调用函数
// 函数表达式则不可以，将函数赋值给变量，函数调用必须在函数表达式之后
// 函数表达式不会引起变量提升=>在构造函数中容易别忽略而出错     
```

### 四种遍历方式

1. for 循环

   ```javascript
   // js基础循环语法
   for (let i = 0; i < 100){
       ...
   }
   ```

2. forEach

   ```javascript
   // 数组的遍历方法
   array.forEach(function(i) {
       // 将数组中每一项i作为参数在函数中执行
   })
   // map方法是返回所有函数的返回值组成的数组，forEach方法没有返回值
   ```

3. for in

   ```javascript
   // 不仅可以对数组遍历，还可以对enumerable对象进行操作（只可以获取键名，不可以获取键值）
   for (let i in array) {
       // 仅仅返回数组中的下标0,1.2等，因为只获取键名
   }
   let obj = {a: 1, b: 2}
   for (let i in obj) {
       console.log(i, obj[i])
       // i 为属性名，obj[i]为属性值
   }
   ```

4. for of（ES6中新增的循环）

   ```javascript
   // 可以遍历具有Iterator接口的数据机构，如数组、字符串等
   for (let i of array) {
       ...
   }
   let str = 'hello world'
   for (let i of str) {
       ...
   }
   
   ```

### 深拷贝和浅拷贝

浅拷贝：对象B拷贝对象A后，如果A内部的值改变，B内部的值也会跟着改变

深拷贝：对象B拷贝对象A后，AB相互独立，如果A内部的值改变，B内部的值不会跟着改变

原因：浅拷贝只是复制了对象地址，AB只是对堆内存中对象的引用

以下为深拷贝的两种方法

1. JSON方法

   JSON.parse(JSON.stringify(obj))；先使用stringify方法变为字符串，再使用parse方法转换回来

   缺点：对象中的方法会被忽略

2. 递归复制

   ```javascript
   rawMethod (obj) {
       let result = {}
       for (let key in obj) {
           if (obj.hasOwnPropertkey && typeof obj[key] === 'object') {
               result[key] = rawMethod(obj)
           } else {
               result[key] = obj[key]
           }
       }
       return result
   }
   ```

### 数组常用方法

```javascript
// 栈方法
push() // 将参数添加到数组末尾
pop() //删除并返回数组的最后一项
// 队列方法
shift() // 删除并返回数组第一项，可以与push（）方法结合实现队列
unshift() // 将参数添加到数组的第一项，可以与pop（）方法结合实现逆序的队列
// 重排序方法
reverse() // 将数组反转
sort() // 对数组进行排序，排序方法作为参数传入
// 操作方法
concat() // 创建数组的一个副本，并将参数加入到副本的末尾，最后返回这个副本
slice(start, end) // 切片，返回下标之间的项组成的数组，不影响原数组。end默认为数组长度
splice(start, num, newItem...) // 从下标start开始删除num项，并从start处添加newItem...
             //可以实现数组的删除，插入，替换等功能。返回值是删除掉的项组成的数组
// 位置方法
// 两个方法都返回要查找的项所在的位置，或者没有返回-1，参数一是查找项，参数二是从哪里开始
indexOf() // 从头至尾
lastIndexOf() // 从尾至头
// 迭代方法
// 每一个方法都是对数组中的每一项给指定的函数，有返回值的不会改变原本数组
every() // 如果每一项在函数中都返回true，则返回true
filter() // 返回每一项在函数中返回true的项组成的数组
forEach() // 数组中每一项都给函数，无返回值
map() // 返回每一项在函数运行后返回结果组成的数组
some() // 只要函数对任意一项返回true，则返回true
find() // 返回满足条件的第一个值
```



### ajax（Asynchrnous JavaScript And Xml）

1. XMLHttpRequest对象：js原生的ajax请求对象，axios和jquery等都是在它的基础上封装

   （浏览器限制跨域请求，如果请求的域与当前域不一致，则会限制或者直接拒接请求）

   ```javascript
   // 使用post方法做示范
   function post () {
       let xhr = new XMLHttpRequest()
       let form = new FormData
       form.append('name', 'sunny')
       form.append('age', 21)
       xhr.onreadystatechange = () => { // 状态改变的时候调用
           // 0:未调用open()，1:调用open(),但是还没有调用send(),2:调用send()还没有响应
           // 3:接收到部分数据了， 4：数据接收完成
           if (xhr.readyState === 4) {
               if (xhr.status >= 200 && xhr.status < 300) { // http状态码
                   let resultCon = document.querySelector('#result')
                   resultCon.innerText = xhr.responseText // 响应数据
               } else {
                   console.info('请求失败！', xhr.statusText)
               }
           }
       }
       // 参数： 方法名  URL 是否为异步
       xhr.open('post', 'http://127.0.0.1:8888/upload', true)
       // 使用FormData对象可以不设置请求头部，浏览器会自动匹配设置
       xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
       xhr.send(form) // 开始发送请求，get方法没有请求体，send()参数为null
   }
   ```

2. node.js接收请求

   ```javascript
   const express = require('express')
   const path = require('path')
   const app = express()
   const bodyParser = require('body-parser') // 用于接收、解析post请求体 req.body
   const multer = require('multer') // 用于接收、解析文件 req.files
   app.use(express.static('./file')) //设置静态路径
   // 设置post请求体接收容量
   let urlencodedParser = bodyParser.urlencoded(
       { limit: 1024 * 1024 * 10, extended: false, parameterLimit: 100 })
   app.use(urlencodedParser)
   // 设置文件上传后临时存放位置，如果不设置则放在内存中
   app.use(multer({ dest: './tmp/'}).any())
   app.get('/index', (req, res) => {
       res.sendFile(path.join(__dirname, './xhr.html')) // 返回html页面
   })
   app.post('/upload', (req, res) => {
       console.log(req.files) // 获取接收的文件，可通过file[x]的path属性获取到文件
       let form = req.body
       console.log(form.name) // 获取接收的FromData中的对象
       res.send('post is OK')
   })
   app.listen(8888)
   ```




### BOM相关

浏览器对象模型以window对象作为依托，window表示浏览器窗口以及页面课件区域。window还是ECMAScript中的Global对象，因而所有的全局变量和函数都是它的属性和方法，它们在浏览器控制台中可直接访问到。且所有的原生构造函数以及其他函数都是存在于window对象的命名空间下。

window对象常见方法：

1. moveto()，moveBy()，resizeTo()，resizeBy()等调整窗口的大小和位置
2. open()方法：打开新的窗口
3. alert()，confirm()，prompt()打开系统对话框

window对象的常用属性有

1. location对象：保存当前页面的url、处理页面跳转等。同时它也是document对象的属性
2. navigator对象：保存浏览器信息
3. screen对象：获取屏幕信息
4. history：管理浏览器的浏览记录，可以通过go()方法进行跳转、向前或向后

### DOM(DOM1)

Node类型：在DOM中，每一个部分都是节点，一共有12个节点，使用1-12个常数表示（nodeType属性），如Node.ELEMENT_NODE(1)，Node.ATTRIBUTE_NODE(2)， Node.TEXT_NODE(3)；在DOM的任意一个部分都属于节点，并对应12种节点中的一个。

节点的关系如图所示<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201224163629363.png" alt="image-20201224163629363" style="zoom:60%;" />

Document类型：表示文档，在浏览器中document对象是HTMLDocument类型（继承自Document，可能xml对应XMLDocument类型）的一个实例。常用方法有createElement()、getElementByxx()、querySelector()等。document对象属于window的属性，所以可以直接全局访问。

Element类型：实例表示DOM元素，如<div></div>，nodeType为1，元素一定是节点，但是节点不一定是元素。比如文本节点，注释节点，元素属性节点等。

因为元素一定是节点，所以元素具有Node类型和Element的方法和属性。如parentElement何parentNode属性（虽然是不同属性，但是返回的都是父元素/节点）；使用Node类型的方法获取多个节点时返回的是Nodelist、而Element类型返回的是HtmlCollection，不过它们都是类数组结构，因此操作方法与数组相似。

其他：还有Text、Attr等其他类型，只是不常用。为DOM元素设置属性时，一般直接使用对象属性方式设置，而不是使用setAttribute()方法。

### DOM扩展

虽然DOM的API已经很完善了，但是为了实现更多的功能，Selector API和HTML5已经被写入标准中，它们源于开发社区。

1. Selector API：

   querySelector()、querySelectorAll()等

2. 元素遍历：

   与Node类型的方法firstChild()、lastChild()、nextSibling()方法类似，firstElementChild()、lastElementChild()、nextElementSibling()等方法返回返回的是元素，而不是节点，因此它们返回的结果不包括文本节点、注释节点等，它们只返回Element类型

3. HTML5：

   与class相关的扩充：getElementByClassName()方法可以获取元素，classList属性可以获取或设置类名

   插入标记：innerHTML和outerHTML可以将字符串序列化为HTML

   滚动到视图：scrollIntoView()方法可以将当前元素滚动到视图中，focus()方法可能也会达到同样目的

   自定义数据属性：当需要在某个元素上绑定一些数据时，可以使用ele.dataset.name = 'sunny'设置值，通过ele.dataset.name获取数据值

4. 专有扩展：浏览器开发商自己定义的扩展功能，不一定通用，不过在未来可能会加入W3C标准。如innetText、outerText等属性

注意点：在使用innerHTML等属性覆盖元素的同时，需要注销被覆盖元素的事件和JavaScript对象，因为它们不会自动注销，这样会导致多余的内存占用。

### DOM2和DOM3

1. DOM2级和3级的目的在于扩展DOM API，以满足操作XML的所有需求。增加了针对xml命名空间的设置。
2. 可以操作DOM元素的样式，获取元素大小等（见下一节）
3. 提高元素遍历的类型NodeIterator和TreeWalker
4. 范围：获取Dom片段



### DOM元素大小相关

`event.target`与`event.currentTarget`的区别：

因为事件的传递机制有捕获和冒泡两个阶段，因此事件发生时相关的元素可能不止一个

`event.target`指向触发事件的元素

`event.currentTarget`指向的是事件绑定的元素

当点击绑定事件的元素本身时，`event.target`与`event.currentTarget`相等

属性访问！！！

使用 js 访问或者设置element属性时，如果是元素属性，可以直接通过属性名或者getAttribute()访问，但是如果是样式属性等，需要加上style再加上属性名！！！

元素大小

1. 偏移量

   相对于父元素的偏移，属性有element.offsetTop,element.offsetWith,element.offsetParent...

   ![image-20200825144111475](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200825144111475.png)

   如果想计算某个元素在页面的偏移量，则需要将这个元素的offsetLeft和offersetRight与其offsetParent的相同属性相加，如此循环直至根元素。

2. 客户区大小

   ![image-20200825144519658](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200825144519658.png)

3. 滚动大小

   指包含滚动内容的元素的大小

   ![image-20200825145046115](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200825145046115.png)

   如果元素没有滚动条，则scrollWidth 和 scrollHeight 与 clientWidth 和 clientHeight 之间的关系并不十分清晰，各个浏览器不同，它们有可能相等也有可能不相等。

   元素滚动量属性scrollLeft和scrollTop

4. getBoundingClientRect

   该方法返回元素各个边与视图边界的距离

   返回对象的属性有left、top、right、bottom

   （使用这个方法的距离加上滚动偏移可以获取元素相对body的偏移）



事件坐标

1. 客户区坐标位置

   event.clientX和event.clientY。所有的浏览器都支持这两个属性。

   ![image-20200825150120672](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200825150120672.png)

2. 页面坐标位置

   event.pageX和event.pageY，在页面没有滚动的情况下，pageX和pageY的值与clientX和clientY的值相等。



3. 屏幕坐标位置

   ![image-20200825150526514](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200825150526514.png)



iframe

每一个“窗口”都是一个JS runtime，如果只有一个窗口，那么就只有一个runtime，如果一个窗口内部含有一个iframe，那么就有两个runtime

runtime之间互相操作（或者是通信）是有域限制的，跨域不可以操作（通信）。因此不允许跨域，在一定程度上避免跨站脚本攻击

### 定时器 setTimeout与setInterval

`JavaScript`是运行在单线程的环境中的，定时器仅仅是计划代码在未来的某个时间执行，但是并不能保证准时执行。

`JavaScript`有一个代码队列，定时器按照规定好的时间将代码放入队列中，如果队列中没有其他代码，则定时器代码得到执行，否则需等待其他代码执行完成。

1. `setTimeout`

   `setTimeout`是规定时间后执行一次（其实是将代码放入队列中，会尽快得到执行），可以通过以下代码实现循环定时的功能（实现`setInterval`）

   ```javascript
   setTimeout(function(){ 
    // 在定时器内部调用定时器，或者定时器所在的方法
    setTimeout(arguments.callee, interval); 
   }, interval); 
   ```



2. `setInterval`是重复定时，缺点是当上一次定时代码还没有执行完成甚至还在代码队列中的时候，又将要把本次的定时代码加入到代码队列中（`JavaScript`会避免这样，因此会导致某些间隔的定时器代码会被跳过），但是这样会导致定时器的时间间隔发生很大的改变，有可能之前的代码刚执行完又要执行本次代码了（小于设定的时间），所以可以使用`setTimeout`代替`setInterval`

### Canvas绘图

`<canvas>`是html5新增的元素，其由几组API构成，但是并非所有浏览器都支持这些API

1. 基本用法

   首先创建`<canvas>`标签，并设置其宽高（在绘制内容之前，没有实际内容）

   然后通过`js`获取创建的画布元素，使用`getContext()`方法获取画布元素的上下文

   最后使用上下文进行绘制图形

   画布元素可以通过toDataURL("image/png")方法返回图像对象，该对象可以放入img标签的src中

   ```javascript
   let drawing = document.getElementById("drawing"); 
   //确定浏览器支持<canvas>元素
   if (drawing.getContext){ 
       let context = drawing.getContext("2d"); 
       //绘制红色矩形
       context.fillStyle = "#ff0000"; 
       context.fillRect(10, 10, 50, 50); 
       //绘制半透明的蓝色矩形
       context.fillStyle = "rgba(0,0,255,0.5)"; 
       context.fillRect(30, 30, 50, 50); 
   }
   ```



2. 2D上下文

   2D上下文对象是可以绘制简单的2D图形，如矩形，弧形和路径。坐标原点为`<canvas`元素的左上角。2D上下文对象的两种基本绘图操作是填充和描边。

   填充是使用指定样式填充图形

   描边是在图像的边缘划线（路径绘制完成以后需要描边才可见，使用`stroke()`方法）

   ```javascript
   // 填充和描边的样式设置
   let context = drawing.getContext("2d")
   context.strokeStyle = "red"
   context.fillStyle = "#0000ff"
   ```

   2D上下文常用API

   ```javascript
   //绘制矩形、描边矩形和清除矩形区域
   context.fillRect(x, y, width, height)
   context.StrokeRect(x, y, width, height)
   context.clearRect(x, y, width, height)
   // 绘制路径: 弧线，移动画笔，直线 
   arc(x, y, radius, startAngle, endAngle, counterclockwise)
   moveTo(x, y)
   lineTo(x, y)
   rect(x, y, width, height)
   // 绘制文本
   context.font = "bold 14px Arial"; 
   context.textAlign = "center"; 
   context.textBaseline = "middle"; 
   context.fillText("12", 100, 20); 
   // 变换: 旋转、伸缩、位移
   rotate(angle)
   scale(scaleX, scaleY)
   translate(x, y) // 调整原点坐标
   
   还有其他绘制图形的API，如：阴影、渐变、获取图像数据、设置图层先后顺序等
   ```

3. WebGL 是针对 Canvas 的 3D 上下文，多数浏览器还不支持

### 避免错误的一些编程知识点



1. 在函数中对函数参数进行类型校验

  ```javascript
  // typeof来判断是否为想要的类型
  function concat(str1, str2, str3){ 
      var result = str1 + str2; 
      if (typeof str3 == "string"){ // 如果不是字符串直接往后执行
          result += str3; 
      } 
      return result; 
  } 
  // 可以使用instance of判断是否为想要的实例
  function reverseSort(values){ 
      if (values instanceof Array){ //如果不是数组直接过滤掉
          values.sort(); 
          values.reverse(); 
      } 
  }
  ```

  2. 在发送请求之前使用encodeURIComponent()对数据进行编码。否则可能因为有URL不允许的字符而导致通信出错。

### Number()和parseInt()

```javascript
// 当字符串是由数字组成的时候 他们转换的数字一样的没有差别  
let numStr = '123'
console.log(parseInt(numStr))   //123
console.log(Number(numStr))		//123

// 当字符串是由字母组成的时候 
let numStr = 'abc'
console.log(parseInt(numStr))   //NaN
console.log(Number(numStr))		//NaN

// 当字符串是由数字和字母组成的时候 
let numStr = '123a'
console.log(parseInt(numStr))   //123
console.log(Number(numStr))		//NaN

// 当字符串是由0和数字
let numStr = '0123'
console.log(parseInt(numStr))   //123
console.log(Number(numStr))		//123

// **当字符串包含小数点**
let numStr = '123.456'
console.log(parseInt(numStr))		//123
console.log(Number(numStr))			//123.456

// **当字符串为null时**
let numStr = null
console.log(parseInt(numStr))		//NaN
console.log(Number(numStr))			//0

// **当字符串为''(空)时**
let numStr = ''
console.log(parseInt(numStr))		//NaN
console.log(Number(numStr))			//0

// 实例

parseInt('16', 8)  = 14
parseInt('10', 8)  = 8

parseInt('16', 10)  = 16
parseInt('10', 10)  = 10

parseInt('16', 16)  = 22
parseInt('10', 16)  = 16
```

