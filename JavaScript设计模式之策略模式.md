### JavaScript设计模式之策略模式

​	策略模式对于我来说是一种比较陌生的设计模式，单从字面的意思我感觉这种设计模式应该是提供一些策略（解决方案或者方法）。哈哈，可能我理解的有些偏差，那么下面我们就来探究下什么是策略模式，以及在JavaScript中策略模式是什么样子的，优缺点等。

#### 定义

定义一系列的算法，把它们封装起来，并且使它们可以相互替换。其UML图如下图所示：

![](http://ww1.sinaimg.cn/mw690/d8f33188ly1fu7a3v3dzij20jh06240m.jpg)

由上面的UML图我们看出，策略模式中有两个重要的部分：

* **Strategy**

  策略部分，定义所有支持的算法的公共接口，**ConcreteStrategy**类实现该接口，实现具体算法。

* **Context**

  上下文部分，主要用来维护一个**Strategy**对象的引用，根据业务逻辑来决定调用哪个实现了**Strategy**接口的策略对象（例如：ConcreteStrategyA、ConcreteStrategyB、ConcreteStrategyC）

##### JavaScript中的策略模式

在JavaScrip中没有接口这一概念，因此在JavaScript中实现策略模式也是与上面传统的策略模式略有不同，但是大体上是一样的。JavaScrip实现策略模式程序也是由两部分组成，分别是策略类`Strategy`和环境类`Context`。策略类封装了具体的算法，并负责具体的计算过程；环境类接受用户的请求，随后把请求委托给某一个策略类。

#### 实现

下面我以一个简单的表单验证来展示一下在JavaScript中策略模式长什么样子。首先我们来实现表单验证中的策略类，代码如下：

```
// 表单验证策略类
class Strategy {
    /**
     * 是否为空校验
     * @param {*} value 校验对象
     * @param {*} errorMsg 错误信息
     */
    isNonEmpty (value, errorMsg) {
        if (value === '') return errorMsg
    }
    
    /**
     * 最小长度校验
     * @param {*} value 校验对象
     * @param {*} length 最小长度
     * @param {*} errorMsg 错误信息
     */
    minLength (value, length, errorMsg) { 
        if (value.length < length) return errorMsg
    }

    /**
     * 手机号码格式校验
     * @param {*} value 校验对象
     * @param {*} errorMsg 错误信息
     */
    isMobile (value, errorMsg){ // 手机号码格式
        if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) return errorMsg
    }
}
```

​	在表单校验策略类中提供了三种校验方法的实现。在策略模式中策略类`Strategy`只关注于每种策略函数的实现过程，而不关注于该策略函数何时被调用；而环境类`Context`则恰恰相反，其关注于根据用户请求来调用某种策略，而不关心该种策略是如何实现的，这正好满足了设计模式中的**开闭原则**。那么下面我们就来看下表单验证中环境类`Context`的实现，代码如下：

```
// 表单验证环境类
class Validator {
    constructor () {
        this.cache = [] // 保存校验规则
        this.strategies = new Strategy() //维护一个Strategy对象的引用
    }

    /**
     * 添加校验规则
     * @param {*} value 校验对象
     * @param {*} rule 以冒号隔开的字符串。冒号前面的代表客户挑选的strategy对象，冒号后面表示在校验过						 程中所必需的一些参数
     * @param {*} errorMsg 当校验未通过时返回的错误信息
     */
    add (value, rule, errorMsg) {
        var param = rule.split(':') // 把 strategy 和参数分开
        this.cache.push(() => { // 把校验的步骤用空函数包装起来,并且放入cache
            var strategy = param.shift() // 用户挑选的 strategy
            param.push(errorMsg) // 将errorMsg添加进参数列表
            return this.strategies[strategy].apply(value, param)
        })
    }

    /**
     * 检验
     */
    validate () {
        let errorMsgs = []
        for (let validatorFunc of this.cache) { // 遍历检验规则
            let msg = validatorFunc() // 调用校验方法
            if (msg) errorMsgs.push(msg) // 将错误信息存入到errorMsgs
        }
        if (errorMsgs.length > 0) return errorMsgs // 如果错误信息数组长度大于0则表明没有校验通过
    }
}
```

#### 调用

```
let validator = new Validator() // 创建一个 validator 对象
/***************添加一些校验规则****************/
validator.add('张三' , 'isNonEmpty', '用户名不能为空' )
validator.add('123', 'minLength:6', '密码长度不能少于6位' )
validator.add('135034', 'isMobile', '手机号码格式不正确' )
let errorMsg = validator.validate() // 获得校验结果
errorMsg.forEach(msg => { console.error(msg) }) // 输出：密码长度不能少于6位 手机号码格式不正确
```

#### 优缺点

* **优点**
  * 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句
  * 策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的 strategy 中，使得它们易于切换、易于理解、易于扩展
  * 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作
* **缺点**
  * 会在程序中增加许多的策略类和策略对象
  * 用户必须了解各个`Strategy`之间的不同点，这样才能选择一个合适的`Strategy`，因此`Strategy`要想用户暴露所有的实现，但这实际上是违反了最少知识原则

#### 总结

​	学习掌握了策略模式之后，我们代码中的if或switch判断会减少一部分，并且书写的代码的可维护度都在增高，也在逐渐满足开闭原则，可谓是优化代码的一个神器。