## JavaScript设计模式之命令模式

命令模式对于我来说是一种较为陌生的设计模式，我们用一个生活中的一个例子来引出今天的主角**命令模式**。假设有一个快餐店，而我是该餐厅的点餐服务员，每天当客人点餐或者打来订餐电话后，我会把他的需求都写在清上，然后交给厨房，客人不用关心是哪些厨师帮他炒菜。这些记录着订餐信息的清单，便是命令模式的命令对象。

### 定义

有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是什么。此时希望用一种松耦合的方式来设计程序，使得请求发送者和请求接收者能够消除彼此的耦合关系。其UML图如下所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1fv3eqa1r96j20pi0aoju1.jpg)

* **Command**

  定义命令的接口，声明执行的方法。

* **Receiver**

  接收者，真正执行命令的对象，任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能。

* **ConcreteCommand**

  命令接口实现对象，通常会持有接收者，并调用接收者的功能来完成命令要执行的操作。

* **Invoker**

  要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。

* **Client**

  注意不是常规意义上的客户端，其主要作用是创建具体的命令对象，并且设置命令对象的接收者。

#### JavaScript中命令模式的实现思路

* 命令模式的接受者Receiver被当成命令对象Command的属性保存起来
* 约定执行命令的操作默认调用execute方法

### 实现

#### 基础命令

首先我们按照命令模式的实现思路来实现一个最简单的命令模式

```
// 定义接受者类
class Receive {
    constructor(name) {
        this.name = name
    }
    /**
     * 保存方法
     */
    save() {
        console.log(`我是${this.name}的保存操作`)
    }

    /**
     * 删除方法
     */
    delete() {
        console.log(`我是${this.name}的删除操作`)
    }
}

/**
 * 定义保存命令
 */
class SaveCommand {
    constructor(receiver) {
        this.receiver = receiver
    }

    /**
     * 默认的命令执行方法
     */
    execute() {
        this.receiver.save()
    }
}

/**
 * 定义删除命令
 */
class DeleteCommand {
    constructor(receiver) {
        this.receiver = receiver
    }

    /**
     * 默认的命令执行方法
     */
    execute() {
        this.receiver.delete()
    }
}

// 创建接受者对象
const receiver = new Receive('哈哈哈')
// 创建保存命令对象
const saveCommand = new SaveCommand(receiver)
// 创建删除命令对象
const deleteCommand = new SaveCommand(receiver)
// 执行保存命令
saveCommand.execute() //输出：我是哈哈哈的保存操作
// 执行删除命令
deleteCommand.execute() //输出：我是哈哈哈的删除操作
```

#### 宏命令

宏命令是一组命令的集合，通过执行宏命令的方式可以批量执行命令。例如我们在保存一个人员信息（包含有该人员的联系人信息）的时候，该人员与其相关联系人是一对多的关系，因此这两种信息我们要分别进行保存，示例代码如下：

```
// 人员信息类
class Person {
    constructor(name, age, gender) {
        this.name = name
        this.age = age
        this.gender = gender
    }
    /**
     * 保存方法
     */
    save() {
        // 具体保存方法省略
        console.log(`${this.name}的信息保存成功`)
    }
}

// 人员联系人信息类
class PersonContact {
    constructor(person) {
        this.person = person
    }
    /**
     * 保存方法
     */
    save() {
        // 具体保存方法省略
        console.log(`${this.person.name}的联系人信息保存成功`)
    }
}

/**
 * 定义保存命令
 */
class SaveCommand {
    constructor(receiver) {
        this.receiver = receiver
    }

    /**
     * 默认的命令执行方法
     */
    execute() {
        this.receiver.save()
    }
}

/**
 * 定义宏命令
 */
class MacroCommand {
    constructor() {
        this.commandsList = []
    }

    addCommand(command) {
        this.commandsList.push(command)
    }

    execute() {
        this.commandsList.forEach(command => {
            command.execute()
        })
    }
}

// 创建人员信息对象
const person = new Person('张三', 21, '男')
// 创建人员联系人信息对象
const pContact = new PersonContact(person)
// 创建保存命令对象
const saveCommandPerson = new SaveCommand(person)
const saveCommandContact = new SaveCommand(pContact)
// 创建宏命令对象
const macroCommand = new MacroCommand()
macroCommand.addCommand(saveCommandPerson)
macroCommand.addCommand(saveCommandContact)
// 执行宏命令
macroCommand.execute() //输出：张三的信息保存成功 张三的联系人信息保存成功
```

上面的例子我们还可以在命令中加入撤销函数，从而在保存方法有异常错误的时候执行撤销函数就可以实现事务回滚，借助命令模式，可以简单地实现一个具有原子事务的行为。当一个事务失败时，往往需要回退到执行前的状态，可以借助命令对象保存这种状态，简单地处理回退操作。