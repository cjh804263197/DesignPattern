### JavaScript设计模式之代理模式

又到了一周设计模式学习及分享的时间了，本周我要给大家分享的是设计模式中大名鼎鼎的**代理模式**。代理模式可能大家都听过，并且代理模式在我们日常生活中的例子也有很多，比如明星和经纪人...
	

	代理模式的关键是，当客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象。

#### 定义

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。其UML图如下图所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1fuf1h9dymmj20fk07bgm9.jpg)

	RealImage真实图片类和ProxyImage代理图片类实现了Image接口和该接口中的display()方法，ProxyImage对象中保留了RealImage对象的引用，并在display()方法中封装了对RealImage对象display()方法的调用，当ProxyPatternDemo类中的main方法调用ProxyImage类的display()方法时，也就是相当于间接的调用了RealImage类的display()方法，这就是我所理解代理模式。

#### 职责划分

根据职责来进行划分，代理模式又分为以下8种：

1、远程代理 

2、虚拟代理

3、写时复制Copy-on-Write 代理

4、保护（Protect or Access）代理

5、Cache代理

6、防火墙（Firewall）代理

7、同步化（Synchronization）代理

8、智能引用（Smart Reference）代理

JavaScript开发中最常用的是**虚拟代理**和**缓存代理**。

* **虚拟代理**，
* **缓存代理**，

#### 虚拟代理

	根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象，比如浏览器的渲染的时候先显示问题，而图片可以慢慢显示（就是通过虚拟代理代替了真实的图片，此时虚拟代理保存了真实图片的路径和尺寸。用虚拟代理实现图片预加载代码实例：

```
// 图片加载函数
  var myImage = (function(){
      var imgNode = document.createElement("img");
      document.body.appendChild(imgNode);
  
      return {
          setSrc: function(src) {
              imgNode.src = src;
          }
     }
 })();
 
 // 引入代理对象
 var proxyImage = (function(){
     var img = new Image;
     img.onload = function(){
         // 图片加载完成，正式加载图片
         myImage.setSrc( this.src );
     };
     return {
         setSrc: function(src){
             // 图片未被载入时，加载一张提示图片
             myImage.setSrc("file://c:/loading.png");
             img.src = src;
         }
     }
 })();
 
 // 调用代理对象加载图片
 proxyImage.setSrc( "http://images/qq.jpg");
```

	proxyImage 间接地访问MyImage。proxyImage 控制了客户对MyImage 的访问，并且在此过程中加入一些额外的操作（真正的图片加载好之前，先把img 节点的src 设置为一张本地的loading 图片）。看完这段代码之后我们会立马思考一个问题，不使用代理模式我们照样可以实现图片的预加载功能，无非就是在MyImage的setSrc方法中加上图片加载完成监听以及加载本地提示图片这两部分代码，为什么要使用代理模式反而把实现变得复杂了？
	
	思考两个问题：
	
	一、对于MyImage的setSrc函数来说职责过多，既要给img设置src，又要负责预加载图片，违反了面向对象设计原则中的**单一职责原则**；
	
	二、如果后期我们考虑撤销图片预加载功能，就要去修改MyImage的setSrc方法，违反了**开闭原则**；

分析了上面两个问题后，代理模式恰好能规避掉这两个问题，此时就体现出了设计模式的优点。

#### 缓存代理

	缓存代理可以为一些开销大的运算结果提供暂时的存储，在下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。使用缓存代理实现运算结果缓存代码如下：

```
var add = function(){
    var sum = 0;
    for(var i = 0, l = arguments.length; i < l; i++){
        sum += arguments[i];
    }
    return sum;
};
var proxyAdd = (function(){
    var cache = {}; //缓存运算结果的缓存对象
    return function(){
        var args = Array.prototype.join.call(arguments);//把参数用逗号组成一个字符串作为“键”
        if(cache.hasOwnProperty(args)){//等价 args in cache
            console.log('使用缓存结果');
            return cache[args];//直接使用缓存对象的“值”
        }
        console.log('计算结果');
        return cache[args] = add.apply(this,arguments);//使用本体函数计算结果并加入缓存
    }
})();
console.log(proxyAdd(1,2,3,4,5));
console.log(proxyAdd(1,2,3,4,5));
console.log(proxyAdd(1,2,3,4,5));

// 输出结果
计算结果
15
使用缓存结果
15
使用缓存结果
15
```

	通过增加缓存代理的方式，add 函数可以继续专注于自身的职责——计算传入参数的和，缓存的功能是由代理对象实现的。

#### ES6中的代理Proxy

	ES6原生提供了Proxy构造函数，主要作用是在目标对象之前架设一层**拦截**，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。其主要的思想还是设计模式，下面我们就来学习一下如何使用Proxy。
	
	Proxy是一个构造函数，它可以接受两个参数：目标对象（target） 与句柄对象（handler） ，返回一个代理对象Proxy，主要用于从外部控制对对象内部的访问。

```
const target = {}, handler = {}
const proxy = new Proxy(target, handler)
```

* **Proxy、target、handler这三者之间有什么关系呢？**

  Proxy的行为很简单：将Proxy的所有内部方法转发至target 。即调用Proxy的方法就会调用target上对应的方法。

* **handler是用来干嘛的？**

  handler的方法可以覆写任意代理的内部方法。 外界每次通过Proxy访问`target` 对象的属性时，就会经过 `handler` 对象，因此，我们可以通过重写handler对象中的一些方法来做一些拦截的操作。 

##### 举个栗子

```
let user = {
	username: 'zhangsan',
	password: '123456',
}

var userProxy = new Proxy(user,{
	get: function(target, property, receiver){
		console.log(`你访问了user的${property}属性`)
		return target[prop]
	}
});
console.log(userProxy.username)
// 你访问了user的username属性 
// zhangsan
```

	如上面的代码，我们访问并输出了username属性，但是运行结果确又额外的输出了我们在`handler`的get方法中预先输出的一句话，这就是拦截器的作用。

##### handler的内建方法

	相信看了上面代码大家还有一个疑问，get方法是什么？get方法是handler对象的14个内建方法之一，我们可以通过重写这些内建方法来自定义拦截器的内容。handler对象拥有以下14个内建对象（我只举四个常用的，其他的请参考博客[深度揭秘ES6代理Proxy](https://blog.csdn.net/qq_28506819/article/details/71077788)）：

1. `handler.get(target, property, receiver)` 方法用于拦截对象的读取属性操作
   - target，目标对象
   - property，被获取的属性名
   - receiver，Proxy或者继承Proxy的对象
2. `handler.set(target, property, value, receiver)` 方法用于拦截设置属性值的操作 
   - target，目标对象
   - property，被设置的属性名
   - value，被设置的新值
   - receiver，最初被调用的对象。通常是proxy本身，但handler的set方法也有可能在原型链上或以其他方式被间接地调用（因此不一定是proxy本身）
3. `handler.apply(target, thisArg, argumentsList)` 方法用于拦截函数的调用 
   - target，目标对象（函数）
   - thisArg，被调用时的上下文对象
   - argumentsList，被调用时的参数列表
4. `handler.construct(target, argumentsList, newTarget)`用于来接new操作 
   - target，目标对象
   - argumensList，构造器参数列表
   - newTarget，最初调用的构造函数

**下面我们就来用ES6语法对虚拟代理例子进行重写**

* 虚拟代理

```
class MyImage {
    constructor(document) {
        this.imgNode = document.createElement('img')
      	document.body.appendChild(imgNode)
    }
    
    setSrc(src) {
        imgNode.src = src
    }
}

let myImg = new MyImage(document)

let myImageProxy = new Proxy(myImg.setSrc, {
    apply(target, ctx, arguments) {
        var img = new Image
        img.onload = function(){
            // 图片加载完成，正式加载图片
            Reflect.apply(target, ctx, arguments)
        }
        // 图片未被载入时，加载一张提示图片
        Reflect.apply(target, ctx, ['file://c:/loading.png'])
        img.src = arguments[0]
    }
})

// 调用
myImageProxy('http://images/qq.jpg')
```

* 缓存代理

```
class Add {
    constructor() {
        var sum = 0;
        for(var i = 0, l = arguments.length; i < l; i++){
            sum += arguments[i]
        }
        return {value: sum}
    }
}

let AddProxy = new Proxy(Add, {
    construct(target, arguments, newTarget) { // newTarget最初的构造函数
    	let cache = target.cache // 从Add类中取出静态属性cache(缓存运算结果的缓存对象)
        var args = Array.prototype.join.call(arguments);//把参数用逗号组成一个字符串作为“键”
        if(cache.hasOwnProperty(args)){//等价 args in cache
            console.log('使用缓存结果');
            return cache[args];//直接使用缓存对象的“值”
        }
        console.log('计算结果');
        return cache[args] = Reflect.construct(target, arguments);//使用本体函数计算结果并加入缓存
    }
})
```

#### 结束语

学习了代理模式之后，我的第一个想法就是赶快将我之前写的带有缓存的请求函数进行一个彻底的改造。