
​	单例模式算是设计模式中我们最常接触的一种设计模式。在实际开发中，系统中有些对象我们只需要一个，例如线程池，数据库操作对象，全局缓存等，这时候我就要用到单例模式。

#### 定义

单例模式的定义就是，保证一个类仅有一个实例，并提供一个访问它的全局访问点。其UML图如下所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1ftz68f0yl3j20e307zaas.jpg)

​	单例模式包含的角色只有一个，就是单例类——Singleton。单例类拥有一个私有构造函数，确保用户无法通过new关键字直接实例化它。除此之外，该模式中包含一个静态私有成员变量与静态公有的工厂方法，该工厂方法负责检验实例的存在性并实例化自己，然后存储在静态成员变量中，以确保只有一个实例被创建。

#### 实现

**ES5写法**

```
var Singleton = function(name) {
    this.name = name
    //一个标记，用来判断是否已将创建了该类的实例
    this.instance = null
}
// 提供了一个静态方法，用户可以直接在类上调用
Singleton.getInstance = function(name) {
    // 没有实例化的时候创建一个该类的实例
    if(!this.instance) {
        this.instance = new Singleton(name)
    }
    // 已经实例化了，返回第一次实例化对象的引用
    return this.instance
}

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')

console.log(a === b) // 输出true
console.log(a) // 输出Singleton {name: "sven1", instance: null}
console.log(b) // 输出Singleton {name: "sven1", instance: null}
```

**ES6写法**

```
class Singleton {
    constructor(name) {
        this.name = name
        this.instance = null
    }
    // 构造一个广为人知的接口，供用户对该类进行实例化
    static getInstance(name) {
        if(!this.instance) {
            this.instance = new Singleton(name)
        }
        return this.instance
    }
}

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')

console.log(a === b) // 输出true
console.log(a) // 输出Singleton {name: "sven1", instance: null}
console.log(b) // 输出Singleton {name: "sven1", instance: null}
```

​	我们在调用两次静态公有方法`getInstance`之后返回的对象都是同一个，因此达到了我们的目的（保证一个类仅有一个实例）。

​	相信大家看完这段代码之后和我一样都有一个疑问，那就是在输出a对象和b对象的时候name属性为sven1这在预料之内，但instance为null，就有些出乎意料了。明明在`getInstance`静态方法中都已经将`new Singleton(name)`赋值给了`this.instance`，为什么输出的时候还是null呢？这时候我们不妨将构造函数中的this与静态方法`getInstance`中的this进行输出

```
// constructor中输出
Singleton {name: "sven1", instance: null}

// getInstance中输出
function(name) {
    this.name = name;
    //一个标记，用来判断是否已将创建了该类的实例
    this.instance = null;
    console.log(`constructor:${Singleton}`)
}
```

​	  原来两个地方的this并不一样，构造函数中的this指向当前类的对象，而静态方法`getInstance`中的this则指向该静态方法。

#### 漏洞

​	虽然我们通过该类的静态工厂方法获取到的对象都是同一个对象，但是还记得我们一开始就介绍的单例类的必要条件之一就是私有的构造函数，为什么要有这样一个条件呢？试想一下，当用户拿到一个类之后的第一反应就是new，那么我们所设计的静态工厂方法`getInstance`此时并没有起到任何作用，下面我们来验证一下：

```

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')
var c = new Singleton('sven3')

console.log(a === b) // 输出true
console.log(a) // 输出Singleton {name: "sven1", instance: null}
console.log(b) // 输出Singleton {name: "sven1", instance: null}
console.log(a === c) // 输出false
console.log(c) // 输出Singleton  {name: "sven3", instance: null}
```

​	   通过new出来的对象c与通过静态工厂方法`getInstance`得到的对象（a, b）并不是同一个对象，因此这种单例模式是不是“单“的不够彻底呢。

#### 完善

​	在JavaScript中我暂时还没有查到如何将构造方法变为私有的，因此我们需要对构造函数做个手脚来堵住这个”漏洞“。

> constructor 方法是类的默认方法，通过 new 命令生成对象实例时会自动调用这个方法，类必须有 constructor 方法，如果一个类没有显式定义构造函数，那么一个空的 constructor 方法会自动添加到类中

**ES5写法**

```
var Singleton = function(name) {
	if (!Singleton.instance) {
        this.name = name
        //一个标记，用来判断是否已将创建了该类的实例
        Singleton.instance = this
	}
	return Singleton.instance
}
// 提供了一个静态方法，用户可以直接在类上调用
Singleton.getInstance = function(name) {
    // 没有实例化的时候创建一个该类的实例
    if(!this.instance) {
        this.instance = new Singleton(name)
    }
    // 已经实例化了，返回第一次实例化对象的引用
    return this.instance
}

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')
var c = new Singleton('sven3')

console.log(a === b) // 输出true
console.log(a) // 输出Singleton {name: "sven1"}
console.log(b) // 输出Singleton {name: "sven1"}
console.log(a === c) // 输出true
console.log(c) // 输出Singleton  {name: "sven1"}
```

**ES6写法**

```
class Singleton {
    constructor(name) {
        if (!Singleton.instance) {
            this.name = name
            //一个标记，用来判断是否已将创建了该类的实例
            Singleton.instance = this
		}
		return Singleton.instance
    }
    // 构造一个广为人知的接口，供用户对该类进行实例化
    static getInstance(name) {
        if(!this.instance) {
            this.instance = new Singleton(name)
        }
        return this.instance
    }
}

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')
var c = new Singleton('sven3')

console.log(a === b) // 输出true
console.log(a) // 输出Singleton {name: "sven1"}
console.log(b) // 输出Singleton {name: "sven1"}
console.log(a === c) // 输出true
console.log(c) // 输出Singleton  {name: "sven1"}
```

通过在构造函数中增加一个类似的判断之后，这个”漏洞“就被堵上了。

