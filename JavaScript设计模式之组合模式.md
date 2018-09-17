## JavaScript设计模式之组合模式

上周学习了命令模式，本周我们来学习一个类似于宏命令的一种设计模式——组合模式。什么是组合模式？组合模式可以让我们使用树形方式创建对象的结构。我们可以把相同的操作应用在组合对象和单个对象上。在大多数情况下，我们都 可以忽略掉组合对象和单个对象之间的差别，从而用一致的方式来处理它们。

### 定义

组合模式，将对象组合成树形结构以表示“部分-整体”的层次结构，组合模式使得用户对单个对象和组合对象的使用具有一致性。其UML图如下所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1fvcw2pvs2dj21560hegpj.jpg)

* **Component** 是组合中的对象声明接口，在适当的情况下，实现所有类共有接口的默认行为。声明一个接口用于访问和管理Component子部件。
* **Leaf **在组合中表示叶子结点对象，叶子结点没有子结点。
* **Composite** 定义有枝节点行为，用来存储子部件，在Component接口中实现与子部件有关操作，如增加(add)和删除(remove)等。

#### JavaScript中的组合模式

JavaScript 中实现组合模式的难点在于要保证组合对象和叶对象对象拥有同样的方法，这通常需要用鸭子类型的思想对它们进行接口检查。在 JavaScript 中实现组合模式，看起来缺乏一些严谨性，我们的代码算不上安全，但能更快速和自由地开发，这既是 JavaScript 的缺点，也是它的优点。

### 实现

下面我们用组合模式来实现一个常见的例子，文件夹和文件之间的关系，非常适合用组合模式来描述。文件夹里既可以包含文件，又可以包含其他文件夹，最终可能组合成一棵树。首先我们需要构造的文件结构如下图所示：

![](http://ww1.sinaimg.cn/large/d8f33188ly1fvcwndybsdj20s607et99.jpg)

代码实现如下：

```
// 文件夹类
class Folder {
    constructor(name) {
        this.name = name
        this.parent = null
        this.components = []
    }
    /**
     * 添加方法
     */
    add(component) {
        component.parent = this
        this.components.push(component)
    }

    /**
     * 扫描方法
     */
    scan() {
        console.log('开始扫描文件夹' + this.name)
        this.components.forEach(component => {
            component.scan()
        })
    }

    /**
     * 移除方法
     */
    remove() {
        if (!this.parent) return //根节点或者树外的游离节点
        this.parent.components.forEach((component, index) => {
            if (component === this) this.parent.components.split(index, 1)
        })
    }
}

// 文件类
class File {
    constructor(name) {
        this.name = name
        this.parent = null
    }
    /**
     * 添加方法
     */
    add() {
        throw new Error('不能添加在文件下面');
    }

    /**
     * 扫描方法
     */
    scan() {
        console.log('开始扫描文件' + this.name)
    }

    /**
     * 移除方法
     */
    remove() {
        if (!this.parent) return //根节点或者树外的游离节点
        this.parent.components.forEach((component, index) => {
            if (component === this) this.parent.components.split(index, 1)
        })
    }
}

// 创建文件夹
const myFolder = new Folder('我的学习资料')
const javaFolder = new Folder('Java')
const jsFolder = new Folder('JavaScript')
const designPatternFolder = new Folder('设计模式')
const javaWebFolder = new Folder('JavaWeb')
const reactFolder = new Folder('React')
const nodeFolder = new Folder('Node')

// 创建文件
const javaFile = new File('Java从入门到精通.pdf')
const springFile = new File('Spring从入门到精通.pdf')
const reactFile = new File('React学习笔记.md')
const jsFile = new File('JavaScript设计模式与开发实践.pdf')

// 向文件夹中添加文件夹以及文件
myFolder.add(javaFolder)
myFolder.add(jsFolder)
myFolder.add(designPatternFolder)

javaFolder.add(javaFile)
javaFolder.add(javaWebFolder)
javaWebFolder.add(springFile)

jsFolder.add(reactFile)

designPatternFolder.add(jsFile)

myFolder.scan()
```

输出

![](http://ww1.sinaimg.cn/large/d8f33188ly1fvcxn4quwpj21cg0ban0m.jpg)

移除操作

```
// 移除
jsFolder.remove()
myFolder.scan()
```

输出

![](http://ww1.sinaimg.cn/large/d8f33188ly1fvcxomjtb7j21cu08omzw.jpg)

### 使用场景

组合模式如果运用得当，可以大大简化客户的代码。当有以下两种情况出现的时候我们就可以使用组合模式：

* **表示对象的部分-整体层次结构** 组合模式可以方便地构造一棵树来表示对象的部分整 体结构。特别是我们在开发期间不确定这棵树到底存在多少层次的时候。 
* **统一对待树种所有的对象 **组合模式使客户可以忽略组合对象和叶对象的区别， 客户在面对这棵树的时候，不用关心当前正在处理的对象是组合对象还是叶对象，也就 不用写一堆 if、else 语句来分别处理它们。 

