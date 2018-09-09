## JavaScript设计模式之发布订阅模式

	发布订阅模式无论是在现实生活中还是在程序的世界中应用都非常之广泛。举个简单的例子，微博是大部分年轻人都会接触使用的一种社交软件，假设在微博中我们关注了一个大V(通常把“粉丝”在50万以上的称为网络**大V**)，当这个大V发布了一条动态后，就会向关注他的所有粉丝推送这条动态，而关注他的粉丝登录微博后就会收到这条动态的推送，这就是一种发布订阅模式。

### 定义

	发布订阅模式又称观察者模式，它定义对象间一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。其UML图如下所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1fuws64657tj20dw07fq3u.jpg)

* **Subject**

  作为发布订阅模式中的发布者，其维护了一个所有订阅它的引用集合，并拥有添加、删除观察者以及通知所有观察者的方法

* **Observer**

  作为发布订阅模式中的订阅者，为所有具体订阅者定义一个接口，在接到发布者的通知时来更新自己

**JavaScript发布订阅模式的实现步骤：**

* 首先要指定好谁充当发布者

* 然后给发布者添加一个缓存列表，用于存放回调函数以便通知订阅者
*  最后发布消息的时候，发布者会遍历这个缓存列表，依次触发里面存放的订阅者回调函 

数 

### 实现

首先我们来实现发布者的基类，所有继承该发布者基类的子类都会成为发布者。

```
// 定义发布者基类
class Subject {
    // 构造函数
    constructor() {
        this.observers = []
    }

    /**
     * 添加订阅者
     */
    add(observer) {
        this.observers.push(obj)
    }

    /**
     * 移除订阅者
     */
    remove(observer) {
        this.observers.forEach((item, index) => {
            if (item === observer) this.observers.splice(index, 1);
        })
    }

    /**
     * 通知消息
     * @param {*} msg 
     */
    notify(msg) {
        this.observers.forEach(item => {
            item.receive(this, msg)
        })
    }
}
```

然后我们来定义一个大V类，继承自Subject

```
/**
 * 定义大V类
 */
class BigV extends Subject {
    constructor(name) {
        super()
        this.name = name
    }

    /**
     * 获取姓名
     */
    getName() {
        return this.name
    }

    /**
     * 添加粉丝
     * @param {} fans 
     */
    addFans(fans) {
        this.add(fans)
    }

    /**
     * 向粉丝推送消息
     * @param {*} msg 
     */
    pushMsg(msg) {
        this.notify(msg)
    }
}
```

接着我们来定义粉丝类

```
/**
 * 定义粉丝类
 */
class Fans {
	constructor(name) {
        this.name = name
    }
   /**
	*接受推送消息方法
	*/
    receive(bigV, msg) {
        console.log(`我是粉丝${this.name}接收到了大V${bigV.getName}推送的消息：${msg}`)
    }
}

let bigV = new BigV('Eason')
let fans1 = new Fans('Jack')
let fans2 = new Fans('Tom')
bigV.addFans(fans1)
bigV.addFans(fans2)
bigV.pushMsg('祝大家新年快乐！')// 输出： 我是粉丝Jack接收到了大VEason推送的消息：祝大家新年快乐！
									  我是粉丝Tom接收到了大VEason推送的消息：祝大家新年快乐！
```

### 优缺点

	通过上面的代码我们可以看出发布订阅模式的优点是**解耦**，作为发布者来说不用关心订阅方接收到推送消息之后究竟要做什么，而订阅方则不用关心发布方会何时发布消息。缺点就是创建订阅者本身就会**消耗一部分时间和内存**，有可能订阅消息到最终都未发生，并且过度使用发布订阅模式会导致程序**难以跟踪和理解**。

### 总结

	我们学习设计模式的最终目的不是会使用某种设计而已，而是能判断出在哪种情况下可以使用设计模式来进行**解耦**和**优化代码**，这将是我接下来学习设计模式的重点。