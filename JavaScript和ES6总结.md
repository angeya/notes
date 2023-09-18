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

// 迭代方法 每一个方法都是对数组中的每一项给指定的函数，有返回值的不会改变原本数组
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

   如果想计算某个元素在页面的偏移量，则需要将这个元素的offsetLeft和offersetRight与其offsetParent的相同属性相加，如此循环直至根元素。

2. 客户区大小

3. 滚动大小

   指包含滚动内容的元素的大小

   如果元素没有滚动条，则scrollWidth 和 scrollHeight 与 clientWidth 和 clientHeight 之间的关系并不十分清晰，各个浏览器不同，它们有可能相等也有可能不相等。

   元素滚动量属性scrollLeft和scrollTop

4. getBoundingClientRect

   该方法返回元素各个边与视图边界的距离

   返回对象的属性有left、top、right、bottom

   （使用这个方法的距离加上滚动偏移可以获取元素相对body的偏移）



事件坐标

1. 客户区坐标位置

   event.clientX和event.clientY。所有的浏览器都支持这两个属性。

2. 页面坐标位置

   event.pageX和event.pageY，在页面没有滚动的情况下，pageX和pageY的值与clientX和clientY的值相等。

3. 屏幕坐标位置


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



## ES6
### let与const命令

`let`命令用于变量的声明，但是不同于`var`，`let`所声明的变量只在所在的代码块有效，类似`java`语言

ES5（`var`命令）没有代码块的概念，而且`var`命令会发生变量提升现象：即在代码执行之前，解释器会将变量提升到代码的最前面。这会导致在变量声明之前使用变量不会报错，只不过此时变量的值为`undefined`

在浏览器中，var为全局变量，可以通过window对象访问到，let和const有自己的作用域，所以不能通过window对象访问到

`const ` 命令用于声明常量，一旦声明，常量的值就不可以修改（因此声明时必须立即初始化），作用域与let一样

`const` 声明的简单数据类型常量无法改变初始值，而声明的引用类型变量是可以改变内部的属性值，因为声明的引用类型变量只保存了指向实际引用对象的内存地址，所以我们一般改变的只是引用对象的内容而没改变存储引用对象的内存地址。
扩展：简单数据类型一般存储在栈里，引用类型一般存储在堆里。

```javascript
const foo = {}
 // 为 foo 添加一个属性，可以成功,因为改变的是foo引用对象的值，而foo保存的是对象的地址
 foo.prop = 123
 console.log(foo.prop) //123
 // 将 foo 指向另外一个对象，就会报错, 因为对象的保存地址改变了
 foo = {} // TypeError: "foo" is read-only
```

### 变量的解构赋值

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这就是解构赋值

```javascript
// 以前的变量赋值
let a = 2, b = 3, c = 4
// ES6允许以下赋值
[a, b, c] = [2, 3, 4]
```

如果右边没有待解构对象没有左边对应的值，即 **解构不成功** 则左边变量的值就等于`undefined`

```javascript
以下两种情况都属于解构不成功，test的值都会等于undefined
let [test] = []
let [bar, test] = [1]
```

解构赋值允许指定默认值，只有当对应的数值成员严格等于（===）`undefined` ，默认值才会生效，否则默认值不生效，类似函数默认值。

数组的解构赋值是根据顺序来的，而对象的属性没有顺序，解构赋值是根据属性来判断的

```javascript
let {bar, foo, other} = {foo: 'a', bar: 'b'}
foo // 'a'
bar //'b'
other // undefined 解构不成功
```

如果变量名`baz`与属性名`foo`不一致，必须写成下面这样。

```javascript
let {foo: baz} = {foo: 'a', bar: 'bbb'}
baz //'a'
//对象的解构赋值的内部机制，是先找到同名属性，然后在赋值给对应的变量。真正被赋值的是后者，而不是前者
```

嵌套结构的解构赋值

```javascript
let obj = {
 p: [
 'arr',
 { y: 'obj' }
 ]
}
let { p: [x, { y }] } = obj
x // "arr"
y // "obj"
//对于嵌套解构的解构赋值，就是剥洋葱的过程，只要根据右边待解构对象的数据结构，一层一层往下找到对应的值再赋值给左边同样结构的变量即可
```

解构赋值的一般用途

1. 交换变量的值

   ```javascript
   let a = 1, b = 2
   [a, b] = [b, a]
   ```

2. 解构从函数返回多个值（即数组赋值给变量）

   ```javascript
   function test(){
       return [1, 2, 3]
   }
   let [a, b, c] = test()
   //返回对象也可以解构
   ```

3. 提取`Json`数据，与嵌套解构一样

4. 遍历`Map`解构

   ```javascript
   for(let [k, v] of map){
   	//每次都可以拿到相应的key与value    
   }
   ```

### Module 语法

`Es6`模块的设计思想是尽量静态化，使得编译时就能确定模块的依赖关系，以及输入输出的变量。

模块不是对象，而是通过`export`命令输出模块中的变量，再通过`import`命令导入变量

`export` 命令用于规定模块的对外接口， `import` 命令用来输入其他模块提供的功

1. `export`命令

   一个模块就是一个独立文件。该文件内部的所有变量，外部无法获取。如果希望通过外部能够读取模块中的某个变量，就必须使用`export`关键字导出。

   ```javascript
   // test.js
   // 导出简单变量
   export let cat = "Tom"
   export let mouse = "Jerry"
   
   // 导出对象
   export {cat, mouse}
   
   // 导出函数
   export function add(x, y){
       return x + y
   }
   
   // 使用别名再导出
   export{ cat as oldCat }
   ```
   
2. import命令

   使用export命令定义接口之后，其他JS文件就可以通过import命令加载这个模块中的接口了

   ```javascript
   // 可以写相对路径或者绝对路径，可以省略文件后缀名
   import {cat, mouse} from "./test"
   // 以上等价于
   import cat from "./test"
   import mouse from "./mouse"
   // 使用别名
   import {cat as oldcat} from "./test"
   // 模块的整体加载,但是要使用别名（可以不加大括号）
   import * as test from "./test"
   ```

   import命令是在编译器执行，即在代码之前执行（import不能使用表达式或者变量）

3. export default命令

   使用export default命令导出模块变量时，表示导出一个叫做default的变量，并且在导入时，可以直接指定任意名字




### 函数的扩展

1. 函数参数的默认参数

  ES6以前，不能为函数参数指定默认参数，只能通过其他方法变通实现，但是比较繁琐，可读性性不高。

  ```javascript
  // 如果y没有赋值，则默认值‘world’生效，缺点是y为布尔值值false时，也将会使用默认值
  function log(x, y) {
      y = y || 'World';
      console.log(x, y);
  }
   // 改进方法，在函数体内在判断y是否为undefined
  if (typeof y === 'undefined') {
  	y = 'World';
  }
  ```

  ES6的写法很简洁，易懂

  ```javascript
  function log(x, y = 'world') {
      console.log(x, y)
  }
  ```

  默认参数的位置应该在函数的尾参数，否则不可省略（需要显示地传undefined）

  函数的length属性返回参数个数，但是如果函数中有默认参数，则length属性不再计算默认参数以及后面的参数

2. rest参数

  ES6引入rest参数，其形式为(...变量名)，该变量用于获取函数的多余参数。rest参数搭配的变量是一个数组，该变量将多余参数都存放到数组中

  ```javascript
  function add(...values) {
      let sum = 0;
      for (var val of values) {
          sum += val;
      }
      return sum;
  }
  add(2, 5, 3) // 10
  // 这是一个求和函数，可以随便传多少个参数，函数的length参数不包含rest参数
  ```

3. 严格模式

  函数内部可以设置为严格模式，但是只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数将不能显示设置严格模式。

4. name属性

  函数的name属性返回该函数的函数名

  ```javascript
  // 对于匿名函数  ES5返回空字符串，ES6返回实际函数名
  var f = function(){}
  // ES5
  f.name // ""
  // ES6
  f.name // "f"
  
  // 对于具名函数  ES5与ES6都返回函数名称
  var f = function hi(){}
  // ES5
  f.name // "hi"
  // ES6
  f.name // "hi"
  ```

5. 箭头函数

  基本用法：

  ```javascript
  // 函数只有一个参数，而且函数体只有一个语句时，可以写成
  let f = v => 2 * v
  // 等价于
  let f =function(v){
      return 2 * v
  }
  // 函数没有参数或者有多个参数时，使用()代表参数部分
  let f = (a, b) => a + b
  // 由于{}被解释为代码块，因此箭头函数返回对象时，应该加上()
  let getPerson = name => ({name: name, age: 23})
  ```

  箭头函数注意点：

  1，箭头函数没有自己的this。函数体内this对象，就是定义时所在的对象，而不是使用时所在的对象

  2，不可以当做构造函数

  3，函数体不存在arguments对象。可以使用rest参数代替

  4，不可以使用yield命令

### Set和Map 数据结构

1. Set

     Set对象类似于数组，但是成员的值是唯一的，与java类似

     ```javascript
     // Set构造函数可以接受一个数组（具有iterable接口的数据结构）
     const set = new Set([1, 2, 3, 3])
     // 由于set不会保存重复的值，同时set对象可以使用扩展运算符...(这是因为扩展运算符内部使用了for...of结构)，因此set又可以转化为数组
     let a = [...set] // [1, 2, 3]，使用此特性可以实现数组去重
     let a = Array.from(set) // 与上面一致
          
     // 常用属性
     size // 返回set对象的长度
          
     // 常用方法
     set(value) // 添加新元素
     has(value) // 查看元素是否在set中
     delete(value) // 删除元素
     clear() // 清空set对象
          
     // 遍历操作 前三种使用类似 for(let item of set.keys()){} 方式遍历
     keys() // 返回键名遍历器 set的value和key相等
     values() // 返回键值遍历器 set的value和key相等
     entries() // 返回键值对遍历器 每次item一个数组，它的两个成员相等
     forEach() // 使用回调函数遍历每个成员，第二个参数可以用来绑定this指针（可选）
     ```

     WeakSet

     WeakSet与Set类似，但是有三点不同

     1，WeakSet对象只可以保存对象，不能是其他类型的值

     2，WeakSet中的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象的引用，直接回收

     3，WeakSet没有iterable接口，没有size属性，因为内部的某些对象可能已经被垃圾回收器清理了

     *这些特点同样适用于本章后面要介绍的 WeakMap 结构*

     

2. Map

    Map对象是键值对的集合，Object也可以实现（因为js是动态类型语言），但是Object的键只能是字符串，而Map的键可以是各类型的值（包括对象）。

    如果Map的键是引用类型，那么只有当地址一致的时候，Map才认为它们是一个键（这解决了属性名相同的问题），如果Map的键是基本数据类型，那么只要键严格相等，Map则认为是同一个键

    Map实例的属性和操作方法

    ```javascript
    // 数组、Set、Map都可以作为Map构造函数的参数进行初始化(具有iterable接口、且每个成员都是一个双元素素组的数据结构够可以)
    const items = [['name', '张三'],['title', 'Author']]
    let map1 = new Map(items)
    
    const set = new Set(items)
    let map2 = new Map(set)
    
    let map3 = new Map(map1)
       
    // 常用属性
    size // 返回map对象的长度
    
    // 常用方法
    set(key, value) // 添加新元素
    has(key) // 查看元素是否在map中
    get(key) // 返回key对应的value
    delete(key) // 删除元素
    clear() // 清空set对象
    
    // 遍历操作与set一致，此处略...
    // map可以使用扩展运算符...转化为数组结构
    [...map.keys()]    // [1, 2, 3]
    [...map.values()]  // ['one', 'two', 'three']
    [...map.entries()] // [[1, 'one'], [2, 'two'], [3, 'three']]
    [...map] // [[1, 'one'], [2, 'two'], [3, 'three']]
    ```

    WeakMap

    WeakMap键名是弱引用，键值的正常引用。WeakMap 应用的典型场合就是 DOM 节点作为键名，它可以有效防止内存泄漏。

3. Set与Map有很大的相似性，只是Set只有键没有值，或者说set的值等于它的键

### Proxy和Reflect

1. Proxy用于修改某些操作的默认行为，等同在语言层面做出修改。可以理解成在目标对象之前架设了一层拦截，外界对该对象的访问，都必须通过这层拦截，因此可以在拦截器中对访问进行过滤和改写

    ```javascript
    // 实例代码
    var handler = {
         get: function(target, name) {
           if (name === 'prototype') {
    			return Object.prototype;
            }
    		return 'Hello, ' + name;
         },
         apply: function(target, thisBinding, args) {
         	return args[0];
         },
         construct: function(target, args) {
            return {value: args[1]};
         }
    };
    var fproxy = new Proxy(function(x, y) {
         return x + y;
    }, handler);
    fproxy(1, 2) // 1
    new fproxy(1, 2) // {value: 2}
    fproxy.prototype === Object.prototype // true
    fproxy.foo === "Hello, foo" // true
    ```

2. 几个常用Proxy支持的拦截操作

    `get(target, propKey, receiver)`：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo'] 。
    `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v ，返回一个布尔值。

    `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args) 、proxy.call(object, ...args) 、proxy.apply(...) 。
    `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)

    `has(target, propKey)`：拦截 propKey in proxy 的操作，返回一个布尔值。
    `deleteProperty(target, propKey)`：拦截 delete proxy[propKey] 的操作，返回一个布尔值。

3. Reflect

    Reflect是ES6为对象操作提供的新API。可以将Object对象的一些明显的属于语言内部的方法放到Reflect对象上，使从Reflect对象上可以拿到语言内部的方法。同时使得Object操作变成函数行为。比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)则让它们变成函数行为。

    Reflect对象的方法和Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。

    Reflect对象一共有13个静态方法，常用的方法如下

    `Reflect.apply(target, thisArg, args)`
    

`Reflect.construct(target, args)`
    
`Reflect.get(target, name, receiver)`
    
`Reflect.set(target, name, value, receiver)`

### Iterator和for of

1. JavaScript中原有的表示集合的数据结构，主要有Arary、Object，后来ES6又添加了Map和Set，这时候就需要一种统一中机制统一处理（遍历）它们，Iterator就是这样的机制

2. Iterator接口体现在可以通过next()方法不断获取集合中的元素，每次next()方法返回包含value的done属性的对象，value为当前元素的值，done标志是否超过最后一个，超过之后返回undefined和true

3. 只要具有Iterator接口的数据结构就可以使用for of进行遍历，一个数据结构只要具有Symbol.iterator属性，皆可以认为是可遍历的（iterable）

3. 原生具备Iterator接口的数据结构如下：Array, Map, Set, String, TypeArray, 函数的arguments对象，NodeList对象 

### Promise

1. promise对象保存着未来才会结束的事件，结束时状态由pending变为fulfilled（resolve方法）或者rejected（reject方法）

2. 对象的then()方法将在状态改变之后调用，方法中获取异步事件的返回的结果

    ```javascript
    // 示例代码
    function read(dir){
        let promise = new Promise(function (resolve, reject){
            fs.readFile(dir, function(err, data){
                if (err) {
                    reject(err)
                }else{
                    resolve(data)
                }
            })
        })
        return promise
    }
    read().then(function(data){}).catch(function(err){}).then()
    ```

3. then方法仍然返回一个新的Promise对象，该对象的resolve数据为then方法中return的值

### Generator函数

1. Generator函数是ES6提供的一种异步编程的解决方案，可以理解为函数是一个状态机，封装多个内部状态，一个状态一个状态的执行

2. Generator函数与普通函数相比有两个主要特征：一是function后面有一个星号；二是函数体内部使用yield表达式定义不同的内部状态

3. 调用Generator函数不是得到执行结果（有点类似构造函数），而是得到一个遍历器对象，通过遍历器对象调用next()方法，逐一返回yield表达式后的值（包含value和done属性的对象），next()方法可以传入参数，该参数将作为上一个yield表达式的结果，如果没有传参，则上个表达式的结果为undefined

    ```javascript
    // 示例代码
    function* Generator() {
        yield 'hello';
        yield 'world';
        return 'ending';
    }
    var ge = Generator();
    ge.next() // { value: 'hello', done: false }
    ge.next() // { value: 'word', done: false }
    ge.next() // { value: 'ending', done: true }
    ge.next() // { value: 'undefined', done: true }
    ```

4. 如果Generator函数内部想嵌套另一个Generator函数，则使用yield*表达式，如yield\* ge2()

### async函数

1. async是Generator函数的语法糖，可以使得操作变得更加简单方便（使异步操作看起来像是同步操作）

2. 在一async异步函数操作中，需要同步的地方需要使用await关键字，且await后面返回的应该是一个Promise对象，

3. async函数返回值是一个Promise对象，如果内部的异步操作中被reject，则该Promise也是reject的

    ```javascript
    function read() {
        // 返回Promise对象这一步是同步的，只是对象内部可以存放未来才执行完成的方法
        return new Promise( function (resolve, reject){
            // 异步操作
        })
    }
    function write(data) {
        return new Promise( function (resolve, reject){
            // 异步操作
        })
    }
    async function asyncRead() {
        // read的结果是write的输入
        // data是promise中resolve的数据
        let data = await read()
        await write(dir2, data)
    }
    // async函数是异步的，只有内部是同步的
    // 所以这段代码会先打印123，然后456，再是异步函数中的异步方法
    console.log("123")
    asyncRead()
    console.log("456")
    ```

### class的基本语法

1. ES6的class基于ES5的构造函数，是构造函数的语法糖，类内部的属性是属于属于构造函数的，但是类的方法是属于prototype的属性。在class中，方法不需要function关键字，方法之间也无需逗号分号分割
2. 类名和函数类似，可以使用类声明或者类表达式（使用表达式的时候，类名为左值，而不是class后面的名称），而且类名不具有变量提升，所以先声明后使用、继承
3. class使用#来表示私有属性，如#name；私有属性实例外是无法访问的。ES6没有提供私有方法，可以变通实现
4. class的static关键字实现静态方法，静态方法不属于类的实例，仅仅属于类（ES6目前不允许使用静态属性）

### class的继承

1. class可以通过extends关键字显示继承，子类的构造函数中必须调用super()方法创建父类对象的this
2. super关键字可以当做方法来使用，也可以当做对象来使用。super()方法虽然代表了父类的构造函数，但是返回的却是子类的实例，即super内部的this指向子类。如果super当做对象来使用，则super指向父类的prototype，所以定义在父类实例上的方法或者属性，是无法通过super调用的。
3. 大多数浏览器的ES5实现中，每一个对象都有\_\_prototype\_\_属性，指向构造函数的prototype。class作为构造函数的语法糖，同时有prototype属性和\_\_prototype\_\_属性，所以同时存在两条继承链。即子类的\_\_prototype\_\_表示构造函数的继承，指向父类，子类的prototype属性的\_\_prototype\_\_属性表示方法的继承，总是指向父类的prototype属性

## 开发案例

### 获取本地文件内容

FileReader 是 JavaScript 提供的内置对象，用于读取本地文件内容。它可以将文件内容读取为文本或二进制数据，提供了一些异步方法来处理文件的读取操作。

使用 FileReader，你可以通过以下步骤来读取本地文件：

1. 创建 FileReader 对象：使用 `new FileReader()` 创建一个新的 FileReader 实例。
2. 监听文件加载事件：使用 `onload` 事件监听器来处理文件加载完成后的操作。该事件在文件加载成功后触发。
3. 选择文件并读取内容：通过用户的操作（例如 `<input type="file">`），选择要读取的文件，并将文件传递给 FileReader 实例。
4. 调用 FileReader 方法来读取文件内容：
   - 对于文本文件，使用 `readAsText(file)` 方法来读取文件内容为文本。
   - 对于二进制文件，使用 `readAsArrayBuffer(file)` 方法来读取文件内容为二进制数据。

下面是一个简单的示例，演示如何使用 FileReader 读取本地文本文件的内容：

```html
<input type="file" id="file-input" onchange="handleFileSelect(event)">
<script>
  // JavaScript
  function handleFileSelect(e) {
    // 获取文件
    let file = e.target.files[0]
    let reader = new FileReader()

    reader.onload = function(e) {
      let fileContent = e.target.result
      // 在控制台打印文件内容
      console.log(fileContent)
    }
    // 读取文件内容为文本
    reader.readAsText(file)
    // readAsArrayBuffer(file) 读取二进制文件
  }
</script>
```

在上面的示例中，我们添加了一个文件选择框 `<input type="file">`，当用户选择文件后，触发 `change` 事件。在事件处理函数中，创建了一个 FileReader 对象，并设置其 `onload` 事件处理函数来处理文件加载完成后的操作。然后，调用 `readAsText(file)` 方法将文件内容读取为文本。

需要注意的是，由于 FileReader 方法是异步执行的，因此需要在 `onload` 事件处理函数中处理文件内容或进行后续操作。

### 获取静态文件内容

在浏览器中直接打开静态文件的地址会变成文件预览或者下载，如果使用axios访问，获取到的文件内容被修改了，不知道是不是目前项目中做了拦截的原因（可以进一步去验证）。

其实我们可以使用`fetch`api来请求静态文件地址，是可以获取到文件内容的，这里以文本文件为例：

```js
function getPlainText(link) {
    // 在Vue项目中时开发状态下是可以走代理的
    fetch('/ossProxy' + link)
      .then(response => {
        console.log(response)
        // 将文件的请求结果转为文本，json字符串还可以转为json()
        return response.text()
      })
      .then(data => {
        console.log(data)
      })
      .catch(error => {
        console.error(error)
      })
  }
```

### 前端使用代码编辑器

可以使用开源项目Vue-Codemirror，它基于开源项目Codemirror，Codemirror支持很多种语言语法。
Vue-Codemirror地址：<https://github.com/surmon-china/vue-codemirror>

Codemirror配置参数：<https://codemirror.net/5/doc/manual.html#config>

Vue-Codemirror 6版本只支持Vue3，因为项目是Vue2，所以这里的使用以Vue-Codemirror4为例：

1. 安装

   npm install vue-codemirror@4.x --save

2. main.js引入

   ```js
   import VueCodemirror from 'vue-codemirror'
   // require styles
   import 'codemirror/lib/codemirror.css'
   
   // you can set default global options and events when use
   Vue.use(VueCodemirror, /* {
     options: { theme: 'base16-dark', ... },
     events: ['scroll', ...]
   } */)
   ```

3. 在组件中使用

   ```vue
   <template>
       <codemirror
         ref="cmEditor"
         :value="content"
         :options="cmOptions"
         @ready="onCmReady"
         @focus="onCmFocus"
         @input="onCmCodeChange" />
   </template>
   
   <script>
   // import language js (java，c#等是clike)
   import 'codemirror/mode/javascript/javascript.js'
   import 'codemirror/mode/clike/clike.js'
   
   // import theme style
   import 'codemirror/theme/base16-dark.css'
   import 'codemirror/theme/idea.css'
   
   export default {
     name: 'CodeEditor',
     props: {
       : {
         type: String,
         default: '文件预览'
       }
     },
     data () {
       return {
         content: ''
         cmOptions: {
           tabSize: 4,
           mode: 'text/javascript',
           theme: 'idea',
           lineNumbers: true,
           line: true,
           readonly: true,
           // more CodeMirror options see https://codemirror.net/5/doc/manual.html#config
         },
   
       }
     },
     methods: {
       onCmReady(cm) {
         console.log('the editor is readied!', cm)
       },
       onCmFocus(cm) {
         console.log('the editor is focused!', cm)
       },
       onCmCodeChange(newCode) {
         console.log('this is new code', newCode)
         this.code = newCode
       }
     },
     computed: {
       codemirror() {
         return this.$refs.cmEditor.codemirror
       }
     },
     mounted() {
       console.log('the current CodeMirror instance object:', this.codemirror)
       // you can use this.codemirror to do something...
     }
   }
   </script>>
   <style scoped lang="scss">
   </style>
   ```

   

