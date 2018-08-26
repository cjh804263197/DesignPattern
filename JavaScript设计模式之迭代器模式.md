### JavaScript设计模式之迭代器模式

	迭代器是我们平时编码中最常用的一个功能，通常由编程语言给我们提供，因此我们自定义迭代器的情况却很少，因此对于迭代器模式的具体实现也不是清楚，下面我们就一起来学习一下在JavaScript中如何实现一个自定义迭代器。

#### 定义

迭代器模式是指提供一种方法顺序的访问一个聚合对象中各个元素，而又不暴露该对象的内部表示。在面向对象语言中UML图如下所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1funesgvchxj20h40bpq3t.jpg)

*  聚集类：Aggregate(抽象类)和ConcreteAggregate(具体聚集类)表示聚集类，是用来存储迭代器的数据。在Aggregate(抽象类)中有一个CreateIterator方法，用来获取迭代器

* 迭代器：迭代器用来为聚集类提供服务，提供了一系列访问聚集类对象元素的方法。

#### JavaScript中的迭代器

	JavaScript中的迭代器可以分为两种，一种是**内部迭代器**，另一种是**外部迭代器**，单从字面上我们并不能区分出这两种迭代器有什么区别，下面我们从定义以及实现来让大家直观的看出这两种迭代器的区别以及其各自用途。

##### 内部迭代器

	内部迭代器在调用的时候非常方便，外界不用关心迭代器内部的实现，跟迭代器的交互也仅仅是一次初始调用，下面我们来实现一个内部迭代器：

```
// 定义内部迭代器
let forEach = (arry, callback) => {
  // 迭代规则
  for(let i = 0; i< arry.length; i++) {
    callback(arry[i], i)
  }
}
// 执行迭代器
forEach([a, b, c, d], (item, index) => {
  console.log(item) // 输出：a b c d
})
```

	如上代码所示，内部迭代器的迭代规则是提前设定好的，因此对于调用该迭代器的对象来说局限性也是很大的，这也正是内部迭代器的缺点。

##### 外部迭代器

	外部迭代器增加了一些调用的复杂度，但相对也增强了迭代器的灵活性，我们可以手工控制迭代的过程或者顺序。外部迭代器的示例代码如下：

```
// 定义外部迭代器类
class Iterator {
    // 构造函数
    constructor(arry) {
        this.currentIndex = 0;
        this.arry = arry
    }

    // 当前数组下标后移一位，若还有下一位，则返回true 若没有下一位，则范围false
    next() {
        if (this.currentIndex < (this.arry.length - 1)) {
            this.currentIndex++
            return true
        }
        return false
    }

    // 获取当前下标的对象
    getCurrentItem() {
        return this.arry[this.currentIndex]
    }
}

let iterator = new Iterator([1, 2, 3, 4, 5])
while(iterator.next()) {
    console.log(iterator.getCurrentItem()) // 最终输出1 2 3 4 5
}
```

外部迭代器虽然调用方式相对复杂，但它的适用面更广，也能满足更多变的需求。

##### 终止迭代器

终止迭代器就是提供一种可以跳出循环的方法，类似于for循环中的break，示例代码如下：

```
// 定义终止迭代器
let forEach = (arry, callback) => {
  // 迭代规则
  for(let i = 0; i< arry.length; i++) {
    if (!callback(arry[i], i)) break;
  }
}
// 执行迭代器
forEach([2, 4, 6, 7, 8], (item, index) => {
	console.log(item) // 输出：2 4 6 7
	return item%2 !== 0 // 当item不为偶数时终止
})
```

#### 总结

	迭代器模式是一种比较简单的设计模式，但是实际开发中使用频率也是非常高的，学会此模式有助于我们对迭代的更深层次的理解。