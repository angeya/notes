虽然这里面的代码在实际开发中不会用到，但是还是要了解的，原因如下：

1. 加深对java语法的理解
2. 万一有其他人写出这种代码，不至于被坑的很惨

## 注释中使用Unicode隐藏代码

我打赌你肯定想不到，有人居然会在注释里下了毒。看看下面的代码，简单到`main`方法中只有一行注释。

```java
public static void main(String[] args) {
    // \u000d System.out.println("coder sunny");
}
```

猜猜看，这段程序运行结果如何？执行后它居然会在控制台打印：

```
coder sunny
```

看到这你是不是一脸懵逼，为什么注释中的代码会被执行？

其实原理就在于大家熟悉的`unicode`编码，上面的`\u000d`就是一个`unicode`转义字符，它所表示的是一个换行符。而java中的编译器，不仅会编译代码，还会解析`unicode`编码将它替换成对应的字符。所以说，上面的代码解析完后实际是这样的：

```java
public static void main(String[] args) {
    //
    System.out.println("coder sunny");
}
```

这样，就能解释为什么能够执行注释中的语句了。当然，如果你觉得上面的代码不够绝，想要再绝一点，那么就可以把代码写成下面这个样子。

```java
public static void main(String[] args) {
    int a=1;
    // \u000d \u0061\u002b\u002b\u003b
    System.out.println(a);
}
```

执行结果会打印`2`，同理，因为后面的`unicode`编码的转义后表示的是`a++;`。

使用这种方法，可以实现在代码中藏一些奇葩操作或者后门，哈哈哈。

## 把简单的东西复杂化

要想写出别人看不懂的代码，很重要的一个小技巧就是**把简单的东西复杂化** 。例如，判断一个`int`型数字的正负时明明可以写成这样：

```java
public void judge(int x){
    if (x>0){
        //...
    }else if (x<0){
        //...
    }
}
```

但是我偏不，放着简单的代码不用，我就是玩，非要写成下面这样：

```java
public void judge2(int x){
    if (x>>>31==0){
        //...
    }else if (x>>>31==1){
        //...
    }
}
```

怎么样，这么写的话是不是逼格一下子就支棱起来了！别人看到这多少得琢磨一会这块到底写了个啥玩意。

其实原理也很简单，这里用到的`>>>`是无符号右移操作。举个简单的例子，以`-3`为例，移位前先转化为它的补码：

```
11111111111111111111111111111101
```

无符号右移一位后变成下面的形式，这个数转化为十进制后是`2147483646`。

```
01111111111111111111111111111110
```

所以，当一个`int`类型的数字在无符号右移31位后，其实在前面的31位高位全部是0，剩下的最低位是原来的符号位，因此可以用来判断数字的正负。

基于这个小知识，我们还能整出不少活来。例如，放着好好的0不用，我们可以通过下面的方式定义一个0：

```java
int ZERO=Integer.MAX_VALUE>>31>>1;
```

通过上面的知识，相信大家可以轻易理解，因为在将一个数字无符号右移32位后，二进制的所有位上全部是0，所以最终会得到0。那么问题来了，我为什么不直接用`Integer.MAX_VALUE>>32`，一次性右移32位呢？

这是因为在对`int`型的数字进行移位操作时，会对操作符右边的参数进行模32的取余运算，因此如果直接写32的话，那么相当于什么都不做，得到的还是原数值。

## 使用反射修改基础类

阻碍同事阅读你代码的有力武器之一，就是让他在遇到条件判断时失去基本判断能力，陷入云里雾里，不知道接下来要走的是哪一个分支。

下面的代码，我说会打印`fasle`，是不是没有人会信？

```java
public class TrueTest {
    public static void main(String[] args) {
        Boolean reality = true;
        if(reality) {
            System.out.println("true");
        } else {
            System.out.println("false");
        }
    }
}
```

没错，只要大家了解布尔类型就知道这不符合逻辑，但是，经过下面的改造就可以让它变为现实。

首先，在类中找个**隐蔽的位置** 插入下面这段代码（这里为好理解，加入了注释）：

```java
static {
    try {
    	// 获取到Boolean的TRUE静态属性
        Field trueField = Boolean.class.getDeclaredField("TRUE");
        trueField.setAccessible(true);

        // 获取修饰符
        Field modifiersField = Field.class.getDeclaredField("modifiers");
        modifiersField.setAccessible(true);
        // 取到trueField的修饰符的整数表示。然后使用位操作符 & 和 ~ 对其进行修改，将 Modifier.FINAL 修饰符从中移除
        modifiersField.setInt(trueField, trueField.getModifiers() & ~ Modifier.FINAL);
        // 静态属性传null
        trueField.set(null, false);
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```

然后再运行上面的程序，你就会发现神奇地打印了`false`。

其实原理也很简单，首先通过反射拿到`Boolean`类中定义的`TRUE`这个变量：

```java
public static final Boolean TRUE = new Boolean(true);
```

接着使用反射，去掉它的`final`修饰符，最后再将它的值设为`false`。而在之后再使用`true`进行定义`Boolean`类型的变量过程中，会进行**自动装箱** ，调用下面的方法：

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

这时的`b`为`true`，而`TRUE`实际上是`false`，因此不满足第一个表达式，最终会返回`false`。

这样一来就能解释上面的打印结果了，不过切记，这么写的时候一定要找一个代码中隐蔽的角落，不要被人发现，否则容易被打的很惨…

## 串行化的条件分支

将原有的一段串行逻辑改写成判断逻辑中的不同分支，并且保证最后能够正常执行。

在开始前先提一个问题，有没有一种方法，可以让`if`和`else`中的语句都能执行，就像下面的这个例子中：

```java
public static void judge(String param){
    if (/*判断条件*/){
        System.out.println("step one");
    }else {
        System.out.println("step two");
    }
}
```

如果我说只调用一次这个方法，就能同时输出`if`和`else`中的打印语句，你肯定会说不可能，因为这违背了java中判断逻辑的基本常识。

没错，在限定了上面的修饰语**只调用『一次』方法** 的条件下，谁都无法做到。但是如果在判断条件中动一点点手脚，就能够实现上面提到的功能。看一下改造后的代码：

```java
public class IfTest {
    public static void main(String[] args) {
        judge("sunny");
    }

    public static void judge(String param){
        // 使用匿名内部类，两个大括号之间可以执行对象的方法或者类的静态方法
        if (param == null || new IfTest(){{ IfTest.judge(null); }}.equals("sunny")){
            System.out.println("step one");
        }else {
            System.out.println("step two");
        }
    }
}
```

运行后控制台打印了：

```
step one
step two
```

惊不惊喜、意不意外？其实它能够执行的秘密就在`if`的判断条件中。

当第一次调用`judge()`方法时，不满足或运算中的第一个条件，因此执行第二个条件，会执行匿名内部类内的实例化初始块代码，再次执行`judge()`方法，此时满足`if`条件，因此执行第一句打印语句。

而实例化的新对象不满足后面的`equals()`方法中的条件，所以不满足`if`中的任意一个条件，因此会执行`else`中的语句，执行第二句打印语句。

这样就实现了**表面上** 调用一次方法，同时执行`if`和`else`中的语句块的功能。怎么样，用这种方式把一段整体的逻辑拆成两块，让你的同事迷惑去吧。

## 使用底层操作

在程序员的世界里，不同语言之间一直存在鄙视链，例如写c的就看不起写java的，因为直接操作内存啥的看上去就很高大上不是么？那么我们今天就假装自己是一个c语言程序员，来在java中操作一把内存。

具体要怎么做呢，还是要使用java中的魔法类`Unsafe`。看这个名字也可以明白，这玩意如果使用不当的话不是非常安全，所以获取`Unsafe`实例也比较麻烦，需要通过反射获取：

```java
Field unsafeField = Unsafe.class.getDeclaredField("theUnsafe");
unsafeField.setAccessible(true);
Unsafe unsafe =(Unsafe) unsafeField.get(null);
```

在拿到这个对象后，我们就可以对内存为所欲为了。例如，我们在实现`int a=1;`这样的简单赋值时，就可以搞复杂点，像下面这样绕一个弯子：

```java
void test(){
    long addr = unsafe.allocateMemory(4);
    unsafe.putInt(addr,1);
    int a=unsafe.getInt(addr);
    System.out.println(a);
    unsafe.freeMemory(addr);
}
```

首先通过`allocateMemory`方法申请4字节的内存空间后，然后通过`putInt`方法写入一个1，再从这个地址读取一个`int`类型长度的变量，最终实现了把1赋值给`a`的操作。

Unsafe除了能直接操作内存空间外，还有线程调度、对象操作、CAS操作等实用的功能。