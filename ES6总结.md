## ES6总结

### let与const命令

>`let`命令用于变量的声明，但是不同于`var`，`let`所声明的变量只在所在的代码块有效，类似`java`语言
>
>ES5（`var`命令）没有代码块的概念，而且`var`命令会发生变量提升现象：即在代码执行之前，解释器会将变量提升到代码的最前面。这会导致在变量声明之前使用变量不会报错，只不过此时变量的值为`undefined`
>
>在浏览器中，var为全局变量，可以通过window对象访问到，let和const有自己的作用域，所以不能通过window对象访问到



> `const ` 命令用于声明常量，一旦声明，常量的值就不可以修改（因此声明时必须立即初始化），作用域与let一样
>
> `const` 声明的简单数据类型常量无法改变初始值，而声明的引用类型变量是可以改变内部的属性值，因为声明的引用类型变量只保存了指向实际引用对象的内存地址，所以我们一般改变的只是引用对象的内容而没改变存储引用对象的内存地址。
> 扩展：简单数据类型一般存储在栈里，引用类型一般存储在堆里。
>
> ```javascript
> const foo = {}
>  // 为 foo 添加一个属性，可以成功,因为改变的是foo引用对象的值，而foo保存的是对象的地址
>  foo.prop = 123
>  console.log(foo.prop) //123
>  
>  // 将 foo 指向另外一个对象，就会报错, 因为对象的保存地址改变了
>  foo = {} // TypeError: "foo" is read-only
> ```

### 变量的解构赋值

> ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这就是解构赋值
>
> ```javascript
> // 以前的变量赋值
> let a = 2, b = 3, c = 4
> 
> // ES6允许以下赋值
> [a, b, c] = [2, 3, 4]
> ```
>
> 如果右边没有待解构对象没有左边对应的值，即 **解构不成功** 则左边变量的值就等于`undefined`
>
> ```javascript
> 以下两种情况都属于解构不成功，test的值都会等于undefined
> let [test] = []
> let [bar, test] = [1]
> ```
>
> 解构赋值允许指定默认值，只有当对应的数值成员严格等于（===）`undefined` ，默认值才会生效，否则默认值不生效，类似函数默认值。

> 数组的解构赋值是根据顺序来的，而对象的属性没有顺序，解构赋值是根据属性来判断的
>
> ```javascript
> let {bar, foo, other} = {foo: 'a', bar: 'b'}
> foo // 'a'
> bar //'b'
> other // undefined 解构不成功
> ```
>
> 如果变量名`baz`与属性名`foo`不一致，必须写成下面这样。
>
> ```javascript
> let {foo: baz} = {foo: 'a', bar: 'bbb'}
> baz //'a'
> //对象的解构赋值的内部机制，是先找到同名属性，然后在赋值给对应的变量。真正被赋值的是后者，而不是前者
> ```
>
> 嵌套结构的解构赋值
>
> ```javascript
> let obj = {
>  p: [
>  'arr',
>  { y: 'obj' }
>  ]
> }
> let { p: [x, { y }] } = obj
> x // "arr"
> y // "obj"
> //对于嵌套解构的解构赋值，就是剥洋葱的过程，只要根据右边待解构对象的数据结构，一层一层往下找到对应的值再赋值给左边同样结构的变量即可
> ```
>
> 解构赋值的一般用途
>
> 1. 交换变量的值
>
>    ```javascript
>    let a = 1, b = 2
>    [a, b] = [b, a]
>    ```
>
> 2. 解构从函数返回多个值（即数组赋值给变量）
>
>    ```javascript
>    function test(){
>        return [1, 2, 3]
>    }
>    let [a, b, c] = test()
>    //返回对象也可以解构
>    ```
>
> 3. 提取`Json`数据，与嵌套解构一样
>
> 4. 遍历`Map`解构
>
>    ```javascript
>    for(let [k, v] of map){
>    	//每次都可以拿到相应的key与value    
>    }
>    ```

### Module 语法

> `Es6`模块的设计思想是尽量静态化，使得编译时就能确定模块的依赖关系，以及输入输出的变量。
>
> 模块不是对象，而是通过`export`命令输出模块中的变量，再通过`import`命令导入变量
>
> `export` 命令用于规定模块的对外接口， `import` 命令用来输入其他模块提供的功
>
> 1. `export`命令
>
>    一个模块就是一个独立文件。该文件内部的所有变量，外部无法获取。如果希望通过外部能够读取模块中的某个变量，就必须使用`export`关键字导出。
>
>    ```javascript
>    // test.js
>    // 导出简单变量
>    export let cat = "Tom"
>    export let mouse = "Jerry"
>    
>    // 导出对象
>    export {cat, mouse}
>    
>    // 导出函数
>    export function add(x, y){
>        return x + y
>    }
>    
>    // 使用别名再导出
>    export{ cat as oldCat }
>    ```
>    
> 2. import命令
>
>    使用export命令定义接口之后，其他JS文件就可以通过import命令加载这个模块中的接口了
>
>    ```javascript
>    // 可以写相对路径或者绝对路径，可以省略文件后缀名
>    import {cat, mouse} from "./test"
>    // 以上等价于
>    import cat from "./test"
>    import mouse from "./mouse"
>    // 使用别名
>    import {cat as oldcat} from "./test"
>    // 模块的整体加载,但是要使用别名（可以不加大括号）
>    import * as test from "./test"
>    ```
>
>    import命令是在编译器执行，即在代码之前执行（import不能使用表达式或者变量）
>
> 3. export default命令
>
>    使用export default命令导出模块变量时，表示导出一个叫做default的变量，并且在导入时，可以直接指定任意名字
>



### 函数的扩展

>1. 函数参数的默认参数
>
>   ES6以前，不能为函数参数指定默认参数，只能通过其他方法变通实现，但是比较繁琐，可读性性不高。
>
>   ```javascript
>   // 如果y没有赋值，则默认值‘world’生效，缺点是y为布尔值值false时，也将会使用默认值
>   function log(x, y) {
>       y = y || 'World';
>       console.log(x, y);
>   }
>    // 改进方法，在函数体内在判断y是否为undefined
>   if (typeof y === 'undefined') {
>   	y = 'World';
>   }
>   ```
>
>   ES6的写法很简洁，易懂
>
>   ```javascript
>   function log(x, y = 'world') {
>       console.log(x, y)
>   }
>   ```
>
>   默认参数的位置应该在函数的尾参数，否则不可省略（需要显示地传undefined）
>
>   函数的length属性返回参数个数，但是如果函数中有默认参数，则length属性不再计算默认参数以及后面的参数
>
>2. rest参数
>
>   ES6引入rest参数，其形式为(...变量名)，该变量用于获取函数的多余参数。rest参数搭配的变量是一个数组，该变量将多余参数都存放到数组中
>
>   ```javascript
>   function add(...values) {
>    let sum = 0;
>    for (var val of values) {
>    sum += val;
>    }
>    return sum;
>   }
>   add(2, 5, 3) // 10
>   // 这是一个求和函数，可以随便传多少个参数，函数的length参数不包含rest参数
>   ```
>
>3. 严格模式
>
>   函数内部可以设置为严格模式，但是只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数将不能显示设置严格模式。
>
>4. name属性
>
>   函数的name属性返回该函数的函数名
>
>   ```javascript
>   // 对于匿名函数  ES5返回空字符串，ES6返回实际函数名
>   var f = function(){}
>   // ES5
>   f.name // ""
>   // ES6
>   f.name // "f"
>   
>   // 对于具名函数  ES5与ES6都返回函数名称
>   var f = function hi(){}
>   // ES5
>   f.name // "hi"
>   // ES6
>   f.name // "hi"
>   ```
>
>   
>
>5. 箭头函数
>
>   基本用法：
>
>   ```javascript
>   // 函数只有一个参数，而且函数体只有一个语句时，可以写成
>   let f = v => 2 * v
>   // 等价于
>   let f =function(v){
>       return 2 * v
>   }
>   // 函数没有参数或者有多个参数时，使用()代表参数部分
>   let f = (a, b) => a + b
>   // 由于{}被解释为代码块，因此箭头函数返回对象时，应该加上()
>   let getPerson = name => ({name: name, age: 23})
>   ```
>
>   箭头函数注意点：
>
>   1，箭头函数没有自己的this。函数体内this对象，就是定义时所在的对象，而不是使用时所在的对象
>
>   2，不可以当做构造函数
>
>   3，函数体不存在arguments对象。可以使用rest参数代替
>
>   4，不可以使用yield命令



### Set和Map 数据结构

> 1. Set
>
>      Set对象类似于数组，但是成员的值是唯一的，与java类似
>
>      ```javascript
>      // Set构造函数可以接受一个数组（具有iterable接口的数据结构）
>      const set = new Set([1, 2, 3, 3])
>      // 由于set不会保存重复的值，同时set对象可以使用扩展运算符...(这是因为扩展运算符内部使用了for...of结构)，因此set又可以转化为数组
>      let a = [...set] // [1, 2, 3]，使用此特性可以实现数组去重
>      let a = Array.from(set) // 与上面一致
>           
>      // 常用属性
>      size // 返回set对象的长度
>           
>      // 常用方法
>      set(value) // 添加新元素
>      has(value) // 查看元素是否在set中
>      delete(value) // 删除元素
>      clear() // 清空set对象
>           
>      // 遍历操作 前三种使用类似 for(let item of set.keys()){} 方式遍历
>      keys() // 返回键名遍历器 set的value和key相等
>      values() // 返回键值遍历器 set的value和key相等
>      entries() // 返回键值对遍历器 每次item一个数组，它的两个成员相等
>      forEach() // 使用回调函数遍历每个成员，第二个参数可以用来绑定this指针（可选）
>      ```
>
>      WeakSet
>
>      WeakSet与Set类似，但是有三点不同
>
>      1，WeakSet对象只可以保存对象，不能是其他类型的值
>
>      2，WeakSet中的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象的引用，直接回收
>
>      3，WeakSet没有iterable接口，没有size属性，因为内部的某些对象可能已经被垃圾回收器清理了
>
>      *这些特点同样适用于本章后面要介绍的 WeakMap 结构*
>
>      
>
> 2. Map
>
>     Map对象是键值对的集合，Object也可以实现（因为js是动态类型语言），但是Object的键只能是字符串，而Map的键可以是各类型的值（包括对象）。
>
>     如果Map的键是引用类型，那么只有当地址一致的时候，Map才认为它们是一个键（这解决了属性名相同的问题），如果Map的键是基本数据类型，那么只要键严格相等，Map则认为是同一个键
>
>     Map实例的属性和操作方法
>
>     ```javascript
>     // 数组、Set、Map都可以作为Map构造函数的参数进行初始化(具有iterable接口、且每个成员都是一个双元素素组的数据结构够可以)
>     const items = [['name', '张三'],['title', 'Author']]
>     let map1 = new Map(items)
>     
>     const set = new Set(items)
>     let map2 = new Map(set)
>     
>     let map3 = new Map(map1)
>        
>     // 常用属性
>     size // 返回map对象的长度
>     
>     // 常用方法
>     set(key, value) // 添加新元素
>     has(key) // 查看元素是否在map中
>     get(key) // 返回key对应的value
>     delete(key) // 删除元素
>     clear() // 清空set对象
>     
>     // 遍历操作与set一致，此处略...
>     // map可以使用扩展运算符...转化为数组结构
>     [...map.keys()]    // [1, 2, 3]
>     [...map.values()]  // ['one', 'two', 'three']
>     [...map.entries()] // [[1, 'one'], [2, 'two'], [3, 'three']]
>     [...map] // [[1, 'one'], [2, 'two'], [3, 'three']]
>     ```
>
>     WeakMap
>
>     WeakMap键名是弱引用，键值的正常引用。WeakMap 应用的典型场合就是 DOM 节点作为键名，它可以有效防止内存泄漏。
>
> 3. Set与Map有很大的相似性，只是Set只有键没有值，或者说set的值等于它的键

### Proxy和Reflect

> 1. Proxy用于修改某些操作的默认行为，等同在语言层面做出修改。可以理解成在目标对象之前架设了一层拦截，外界对该对象的访问，都必须通过这层拦截，因此可以在拦截器中对访问进行过滤和改写
>
>     ```javascript
>     // 实例代码
>     var handler = {
>          get: function(target, name) {
>            if (name === 'prototype') {
>     			return Object.prototype;
>             }
>     		return 'Hello, ' + name;
>          },
>          apply: function(target, thisBinding, args) {
>          	return args[0];
>          },
>          construct: function(target, args) {
>             return {value: args[1]};
>          }
>     };
>     var fproxy = new Proxy(function(x, y) {
>          return x + y;
>     }, handler);
>     fproxy(1, 2) // 1
>     new fproxy(1, 2) // {value: 2}
>     fproxy.prototype === Object.prototype // true
>     fproxy.foo === "Hello, foo" // true
>     ```
>
> 2. 几个常用Proxy支持的拦截操作
>
>     `get(target, propKey, receiver)`：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo'] 。
>     `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v ，返回一个布尔值。
>
>     `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args) 、proxy.call(object, ...args) 、proxy.apply(...) 。
>     `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)
>
>     `has(target, propKey)`：拦截 propKey in proxy 的操作，返回一个布尔值。
>     `deleteProperty(target, propKey)`：拦截 delete proxy[propKey] 的操作，返回一个布尔值。
>
> 3. Reflect
>
>     Reflect是ES6为对象操作提供的新API。可以将Object对象的一些明显的属于语言内部的方法放到Reflect对象上，使从Reflect对象上可以拿到语言内部的方法。同时使得Object操作变成函数行为。比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)则让它们变成函数行为。
>
>     Reflect对象的方法和Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。
>
>     Reflect对象一共有13个静态方法，常用的方法如下
>
>     `Reflect.apply(target, thisArg, args)
>     `
>
>     `Reflect.construct(target, args)`
>
>     ``Reflect.get(target, name, receiver)`
>
>     `Reflect.set(target, name, value, receiver)`

### Iterator和for of

> 1. JavaScript中原有的表示集合的数据结构，主要有Arary、Object，后来ES6又添加了Map和Set，这时候就需要一种统一中机制统一处理（遍历）它们，Iterator就是这样的机制
>
> 2. Iterator接口体现在可以通过next()方法不断获取集合中的元素，每次next()方法返回包含value的done属性的对象，value为当前元素的值，done标志是否超过最后一个，超过之后返回undefined和true
>
> 3. 只要具有Iterator接口的数据结构就可以使用for of进行遍历，一个数据结构只要具有Symbol.iterator属性，皆可以认为是可遍历的（iterable）
>
> 3. 原生具备Iterator接口的数据结构如下：Array, Map, Set, String, TypeArray, 函数的arguments对象，NodeList对象 

### Promise

在 ES6 中，`Promise` 是用于处理异步操作的对象，通过链式调用 `then()`、`catch()` 和 `finally()` 方法，来编排异步操作的执行顺序和错误处理。下面是 `Promise` 的基础使用方式。

#### 1. 基本使用

一个 `Promise` 有三种状态：

- `pending`：初始状态，未完成；
- `fulfilled`：操作成功完成；
- `rejected`：操作失败。

创建一个 `Promise` 需要传入一个执行器函数，该函数接受两个参数：`resolve`（成功时调用）和 `reject`（失败时调用）。

示例代码

```javascript
// 创建一个Promise对象
const myPromise = new Promise((resolve, reject) => {
  // 模拟异步操作，比如网络请求
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve("操作成功");
    } else {
      reject("操作失败");
    }
  }, 1000);
});
```

#### 2. 使用 `then`、`catch` 和 `finally`

- `then()`：接收 `resolve` 的结果，并返回一个新的 `Promise`，允许链式调用。
- `catch()`：捕获 `reject` 的结果。
- `finally()`：无论 `Promise` 的状态是成功还是失败，都会执行。

**示例代码**

```javascript
myPromise.then(result => {
    console.log(result); // 输出 "操作成功"
  }).catch(error => {
    console.error(error); // 输出 "操作失败"
  }).finally(() => {
    console.log("操作结束"); // 无论成功或失败都会执行
  });
```

#### 3. 链式调用

每次 `then()` 或 `catch()` 返回一个新的 `Promise`，可以进行链式调用来处理多步异步操作。

**示例代码**

```javascript
myPromise
  .then(result => {
    console.log(result); // 输出 "操作成功"
    return "下一步操作"; // 传递给下一个 then
  })
  .then(nextResult => {
    console.log(nextResult); // 输出 "下一步操作"
  })
  .catch(error => {
    console.error("错误：", error); // 捕获所有错误
  });
```

#### 4. 使用 `Promise.all` 和 `Promise.race`

**Promise.all**

`Promise.all` 接收一个 `Promise` 数组，所有 `Promise` 都完成后返回一个包含所有结果的数组，如果任意一个 `Promise` 失败则返回错误。

```javascript
javascript复制代码const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) => setTimeout(resolve, 100, 'foo')); // setTimeout第三个参数为第一个参数的方法的实参
const promise3 = 42;

Promise.all([promise1, promise2, promise3])
  .then(values => console.log(values)) // 输出：[3, "foo", 42]
  .catch(error => console.error(error));
```

**Promise.race**

`Promise.race` 接收一个 `Promise` 数组，只要有一个 `Promise` 处理完成，立即返回该 `Promise` 的结果。

```javascript
const promise1 = new Promise((resolve, reject) => setTimeout(resolve, 500, 'one'));
const promise2 = new Promise((resolve, reject) => setTimeout(resolve, 100, 'two'));

Promise.race([promise1, promise2])
  .then(value => console.log(value)) // 输出："two"
  .catch(error => console.error(error));
```

#### 5. 使用 `async` / `await`

在 ES8 中，`async` / `await` 是对 `Promise` 的语法糖，使代码更清晰。

```javascript
javascript复制代码async function asyncFunction() {
  try {
    const result = await myPromise;
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}
asyncFunction();
```

#### 总结

`Promise` 通过链式调用简化了异步操作，避免了传统的回调嵌套问题，是现代 JavaScript 进行异步编程的重要工具。

### Generator函数

> 1. Generator函数是ES6提供的一种异步编程的解决方案，可以理解为函数是一个状态机，封装多个内部状态，一个状态一个状态的执行
>
> 2. Generator函数与普通函数相比有两个主要特征：一是function后面有一个星号；二是函数体内部使用yield表达式定义不同的内部状态
>
> 3. 调用Generator函数不是得到执行结果（有点类似构造函数），而是得到一个遍历器对象，通过遍历器对象调用next()方法，逐一返回yield表达式后的值（包含value和done属性的对象），next()方法可以传入参数，该参数将作为上一个yield表达式的结果，如果没有传参，则上个表达式的结果为undefined
>
>     ```javascript
>     // 示例代码
>     function* Generator() {
>         yield 'hello';
>         yield 'world';
>         return 'ending';
>     }
>     var ge = Generator();
>     ge.next() // { value: 'hello', done: false }
>     ge.next() // { value: 'word', done: false }
>     ge.next() // { value: 'ending', done: true }
>     ge.next() // { value: 'undefined', done: true }
>     ```
>
> 4. 如果Generator函数内部想嵌套另一个Generator函数，则使用yield*表达式，如yield\* ge2()

### async函数

> 1. async是Generator函数的语法糖，可以使得一步操作变得更加简单方便（使异步操作看起来像是同步操作）
>
> 2. 在一async异步函数操作中，需要同步的地方需要使用await关键字，且await后面返回的应该是一个Promise对象，
>
> 3. async函数返回值是一个Promise对象，如果内部的异步操作中被reject，则该Promise也是reject的
>
>     ```javascript
>     function read() {
>         return new Promise(function(resolve, reject){
>             // 异步操作
>         })
>     }
>     function write(data) {
>         return new Promise(function(resolve, reject){
>             // 异步操作
>         })
>     }
>     async function asyncRead() {
>         // read的结果是write的输入
>         let data = await read()
>         await write(dir2, data)
>     }
>     asyncRead()
>     ```

### class的基本语法

> 1. ES6的class基于ES5的构造函数，是构造函数的语法糖，类内部的属性是属于属于构造函数的，但是类的方法是属于prototype的属性。在class中，方法不需要function关键字，方法之间也无需逗号分号分割
> 2. 类名和函数类似，可以使用类声明或者类表达式（使用表达式的时候，类名为左值，而不是class后面的名称），而且类名不具有变量提升，所以先声明后使用、继承
> 3. class使用#来表示私有属性，如#name；私有属性实例外是无法访问的。ES6没有提供私有方法，可以变通实现
> 4. class的static关键字实现静态方法，静态方法不属于类的实例，仅仅属于类（ES6目前不允许使用静态属性）

### class的继承

> 1. class可以通过extends关键字显示继承，子类的构造函数中必须调用super()方法创建父类对象的this
> 2. super关键字可以当做方法来使用，也可以当做对象来使用。super()方法虽然代表了父类的构造函数，但是返回的却是子类的实例，即super内部的this指向子类。如果super当做对象来使用，则super指向父类的prototype，所以定义在父类实例上的方法或者属性，是无法通过super调用的。
> 3. 大多数浏览器的ES5实现中，每一个对象都有\_\_prototype\_\_属性，指向构造函数的prototype。class作为构造函数的语法糖，同时有prototype属性和\_\_prototype\_\_属性，所以同时存在两条继承链。即子类的\_\_prototype\_\_表示构造函数的继承，指向父类，子类的prototype属性的\_\_prototype\_\_属性表示方法的继承，总是指向父类的prototype属性
