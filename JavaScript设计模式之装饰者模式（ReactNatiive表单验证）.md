> 在任何系统的开发工作中，表单的验证都是一项必不可少的工作，在ReactNative的开发过程中也不过如此。

### 现状

​         由于ReactNative原生并没有提供表单验证功能，因此只能求助于第三方的一些插件，目前比较常用的ReactNative表单验证插件有以下几种：

1. [**react-native-gifted-form**](https://github.com/FaridSafi/react-native-gifted-form#readme)

   react-native-gifted-form是一款非常棒的ReactNative表单验证插件，页面效果非常酷炫，上手也不是很难；但是该项目作者已经很长时间没有维护，并且表单控件只能使用该插件提供的一些控件。

2. [**tcomb-form-native**](https://github.com/gcanti/tcomb-form-native)

   tcomb-form-native是一款非常容易上手的ReactNative表单验证插件，star也达到了将近3k；同样它的缺点也是表单控件只能使用该插件提供的一些控件，并且表单控件的效果与我们的需求差异很大。

3. [**rc-form**](https://github.com/react-component/form)

   rc-form是antd所推荐的一款表单验证插件，它支持ReactNative，可以使用自己封装的表单控件；但是看完官方给出的demo之后，完全懵逼，瞬间感觉遇到了一块硬骨头。

### 取舍

​        以上三种表单验证插件都是非常棒的，各有优缺点，关于要使用哪一种完全取决于自己的业务需求，由于目前的业务需求，我选择了**rc-form**。

### rc-from for ReactNative

先来看下官方给出的[demo](https://github.com/react-component/form/blob/master/examples/react-native/App.js)，代码如下：

```
import React from 'react'
import PropTypes from 'prop-types'
import {
  StyleSheet,
  Button,
  Dimensions,
  TextInput,
  Text,
  View,
  Alert,
} from 'react-native'

import { createForm } from 'rc-form'

const { width } = Dimensions.get('window')
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    padding: 50,
    justifyContent: 'center',
  },
  inputView: {
    width: width - 100,
    paddingLeft: 10,
  },
  input: {
    height: 42,
    fontSize: 16,
  },
  errorinfo: {
    marginTop: 10,
  },
  errorinfoText: {
    color: 'red',
  },
})

class FromItem extends React.PureComponent {
  getError = error => {
    if (error) {
      return error.map(info => {
        return (
          <Text style={styles.errorinfoText} key={info}>
            {info}
          </Text>
        )
      })
    }
  }
  render() {
    const { label, onChange, value, error } = this.props
    return (
      <View style={styles.inputView}>
        <TextInput
          style={styles.input}
          value={value || ''}
          label={`${label}：`}
          duration={150}
          onChangeText={onChange}
          highlightColor="#40a9ff"
          underlineColorAndroid="#40a9ff"
        />
        <View style={styles.errorinfo}>{this.getError(error)}</View>
      </View>
    )
  }
}

class App extends React.Component {
  static propTypes = {
    form: PropTypes.object.isRequired,
  }

  checkUserNameOne = (value, callback) => {
    setTimeout(() => {
      if (value === '15188888888') {
        callback('手机号已经被注册')
      } else {
        callback()
      }
    }, 2000)
  }
  submit = () => {
    this.props.form.validateFields((error) => {
      if (error) return
      Alert('通过了所有验证') // eslint-disable-line new-cap
    })
  }
  render() {
    const { getFieldDecorator, getFieldError } = this.props.form
    return (
      <View style={styles.container}>
        <Text>简单的手机号验证</Text>
        {getFieldDecorator('username', {
          validateFirst: true,
          rules: [
            { required: true, message: '请输入手机号!' },
            {
              pattern: /^1\d{10}$/,
              message: '请输入正确的手机号!',
            },
            {
              validator: (rule, value, callback) => {
                this.checkUserNameOne(value, callback);
              },
              message: '手机号已经被注册!',
            },
          ],
        })(
          <FromItem
            autoFocus
            placeholder="手机号"
            error={getFieldError('username')}
          />
        )}
        <Button color="#40a9ff" onPress={this.submit} title="登陆" />
      </View>
    )
  }
}

export default createForm()(App)
```

​        看完这段代码，相信大部分人跟我的感觉是一样的（不就是一个表单验证吗，怎么使用起来这么复杂，让我瞬间怀念起使用Vue的那段日子）。之前做过Vue相关的开发，elementUI和iView中的表单组件使用起来是多么的简洁易懂，怎么到了ReactNative中就变的这么的复杂难懂。先不吐槽了~，来梳理一下这个demo中的思路吧，毕竟思路清晰以后我们还是可以将其再次封装的（毕竟再好用的插件也都是人家进行多次封装之后供我们使用的，这也是一次技术提升的好机会）。

​	我先来讲一下大致的思路，在代码的最后一句，导出了`createForm()(App)`，createForm是一个React高阶函数，它接收一个React组件（App），在函数内部对该组件进行加工改造，最后返回加工改造之后的新组件。在createForm函数中其将form对象传递给App组件，因此通过this.props.form就可以获取到这个form对象。

```
const { getFieldDecorator, getFieldError } = this.props.form
....
{getFieldDecorator('username', {
	validateFirst: true,
	rules: [
		{ required: true, message: '请输入手机号!' },
		{
			pattern: /^1\d{10}$/,
			message: '请输入正确的手机号!',
         },
		{
        	validator: (rule, value, callback) => {
		    	this.checkUserNameOne(value, callback);
			},
			message: '手机号已经被注册!',
         },
	],
})(
	<FromItem
		autoFocus
		placeholder="手机号"
		error={getFieldError('username')}
	/>
)}
...
```

#### getFieldDecorator()函数

form提供了`getFieldDecorator`函数。其函数定义形式为 

```
getFieldDecorator(fieldName, fieldOption) => (Component) => ComponentWithExtraProps
```

getFieldDecorator函数接收两个参数，分别是

* fieldName：字段名称
* fieldOption：字段操作对象

**源码**

```
/** 
 *  {getFieldDecorator(name,fieldOption)(<FormItem {...props}/>)}将表单项包装为高阶组件后返回
 *  实现功能同getFieldProps方法，内部也调用getFieldProps方法
 *  与getFieldProps方法不同的是，被封装表单项的props作为this.fieldMeta[name]的originalProps属性
 *  originalProps属性的主要目的存储被封装表单项的onChange事件，fieldOption下无同类事件时，执行该事件
 *  不推荐将value、defaultValue作为表单项组件如FormItem的props属性
 */
getFieldDecorator: function getFieldDecorator(name, fieldOption) {
	var _this = this;
	// 获取需要传递给被修饰元素的属性。包括onChange,value等
    // 同时在该props中设定用于收集元素值得监听事件(onChange)，以便后续做双向数据。
	var props = this.getFieldProps(name, fieldOption);
     return function (fieldElem) {
     	  // 此处fieldStore存储字段数据信息以及元数据信息。
          // 数据信息包括value,errors,dirty等
          // 元数据信息包括initValue,defaultValue，校验规则等。
          var fieldMeta = _this.getFieldMeta(name);
          var originalProps = fieldElem.props;
          ...
          fieldMeta.originalProps = originalProps;
          fieldMeta.ref = fieldElem.ref;
          return _react2["default"].cloneElement(fieldElem, (0, _extends3["default"])({}, props, _this.getFieldValuePropValue(fieldMeta)));
      };
};
```

其返回值是一个React高阶函数，接收参数是FormItem自定义组件，在该高阶函数中将onChange以及value传递给FormItem，然后在FormItem组件中

```
...
const { label, onChange, value, error } = this.props
return (
	<View style={styles.inputView}>
		<TextInput
			style={styles.input}
			value={value || ''}
			label={`${label}：`}
			duration={150}
			onChangeText={onChange}
			highlightColor="#40a9ff"
			underlineColorAndroid="#40a9ff"
		/>
		<View style={styles.errorinfo}>{this.getError(error)}</View>
	</View>
)
...
```

获取到onChange以及value后，将其绑定在TextInput组件的onChangeText和value上。（TextInput可以替换为其他的组件，但是替换的组件一定要有value和onChange属性）

#### getFieldError()函数

form同样还提供了`getFieldError`函数。其函数定义形式为 

```
getFieldError(fieldName) => errors数组
```

getFieldError函数接收一个参数：

* fieldName：字段名称（要与getFieldDecorator函数的第一个参数fieldName保持一致）

**源码**

```
// 获取this.fields[name]["errors"]错误数据，并剔除没有error.message的错误数据
getFieldError: function getFieldError(name) {
	return (0, _utils.getErrorStrs)(this.getFieldMember(name, 'errors'));
},
// 获取this.fields[name][member]属性数据
getFieldMember: function getFieldMember(name, member) {
	var field = this.getField(name);
	return field && field[member];
},
```

其返回值是一个数组，数组中存储的是错误信息，拿到错误信息之后，将其传递给FormItem的error属性

```
...
<FromItem
	...
	error={getFieldError('username')}
/>
...
// 在FromItem.js中
...
const { label, onChange, value, error } = this.props
return (
	<View style={styles.inputView}>
		...
		<View style={styles.errorinfo}>{this.getError(error)}</View>
	</View>
)
...
// 返回error信息组件函数
getError = error => {
    if (error) {
      return error.map(info => {
        return (
          <Text style={styles.errorinfoText} key={info}>
            {info}
          </Text>
        )
      })
    }
  }
```

​	在FromItem组件中，获取到error数组对象之后，调用error信息展示函数，该函数返回错误信息展示组件，从而将错误信息展示给用户。

### 表单组件的封装

​	要想对render函数进行瘦身，我们首先要从自定义表单项组件入手。上一节我们讲到`getFieldDecorator`函数接收fieldName字段名称和fieldOption字段操作对象两个参数，并返回一个React高阶函数（该高阶函数的参数就是自定义表单组件），在该高阶函数中将onChange以及value传递给自定义表单项组件并返回一个加工后的组件对象。因此，我们可以将`getFieldDecorator`函数放到自定义表单组件中去，代码如下所示：

```
// FormItem.js

import BaseFormItem from './base-form-item' // 表单项组件基类

class FromItem extends BaseFormItem {
  render() {
    const { label, onChange, value, prop, rules, form } = this.props;
    const { getFieldDecorator, getFieldError } = form;
    return (
      <View style={styles.inputView}>
      	{getFieldDecorator(prop, {rules})(
      		<TextInput
              style={styles.input}
              value={value || ''}
              label={`${label}：`}
              duration={150}
              onChangeText={onChange}
              highlightColor="#40a9ff"
              underlineColorAndroid="#40a9ff"
            />
            <View style={styles.errorinfo}>{this.getError(getFieldError(prop))}</View>
      	)}
      </View>
    );
  }
}
```

​	我们可以看到，上面的代码将`getFieldDecorator`放到了FormItem组件中去，在页面组件中将form对象，prop和rules传递给FormItem组件，然后在FormItem组件中从页面组件传递过来的form对象中取出`getFieldDecorator`和`getFieldError`函数，`getFieldDecorator`函数接收的两个参数fieldName（页面组件传递过来的prop）、fieldOption（页面组件传递过来的rules）。

```
/**
 * 表单项基类
 */
class BaseFormItem extends Component {
  getError = (error) => {
    if (error) {
      return error.map(info => <Text style={{color: 'red'}} key={info}>{info}</Text>)
    }
    return null
  }
}
```

​	每一个表单子项都会有显示错误信息的方法，为了避免写重复代码，我们将显示错误信息的方法抽离出来，并创建一个基类，将该方法放到基类中，让所有表单项来继承这个基类。

```
class App extends React.Component {
	constructor(props) {
        super(props)

        this.state = {
          rules: { // 定义校验规则
            username: [
              	{ required: true, message: '请输入手机号!' },
                {
                  pattern: /^1\d{10}$/,
                  message: '请输入正确的手机号!',
                },
                {
                  validator: (rule, value, callback) => {
                    this.checkUserNameOne(value, callback);
                  },
                  message: '手机号已经被注册!',
                },
            ],
          },
        }
    }
    ...
    render() {
    	const { form } = this.props
        return (
        	<View>
              <FromItem
              	form={form}
              	prop="username"
              	rules={this.state.rules.username}
                autoFocus
                placeholder="手机号"
              />
              <Button color="#40a9ff" onPress={this.submit} title="登陆" />
      		</View>
        )
    }
}
```

​	经过改造之后，页面的render函数代码是不是瞬间减少了。但是这个时候我们要考虑另外一个问题，页面中render函数确实被瘦身了，但是表单组件中的render函数却又开始变的臃肿了。如果我们需要来封装一个选择器表单组件，那FormItem中的那段臃肿的代码是不是还要再被重写一遍呢；如果我们后续开发的过程中**rc-form**的实现形式改变了，或者是换了另外一个表单校验插件，那么我们之前封装的表单组件就全部要进行修改；我们封装的表单组件只能用于表单验证，如果想在非表单验证页面上使用的话也是不可以的。综上所述组件与表单验证是强耦合的，我们能不能将**组件**与**表单验证**这两个东西给分开呢？试想一下，组件如果不披表单验证这件外衣它就是一个单纯的组件，如果披了这件外衣，它就是带有表单验证功能的组件，表单验证实现的改变不会影响到组件。怎么才能达到这种理想状态呢？

### 装饰者模式

​	绕了一大圈子，终于把本文的重点**装饰者模式**引了出来。

#### 什么是装饰者模式？

​	装饰模式指的是在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。通常情况下要给对象扩展功能使用继承的方式，但是继承的方式并不灵活，还会带来许多问题：一方面会导致超类和子类之间存在强耦合性，当超类改变时，子类也会随之改变。

```
@startuml
Decorator <|-- ConcreteDecoratorA
Decorator <|-- ConcreteDecoratorB

Component <|-- Decorator
Component <|-- ConcreteComponent

Decorator o-- Component

Component: +Operation()
ConcreteComponent: +Operation()
Decorator: +Operation()
class ConcreteDecoratorA {
 {field} addedState
 {method} Operation()
}
ConcreteDecoratorB: +Operation()
ConcreteDecoratorB: AddedBehavior()

@enduml
```



```
/**
 * 表单项装饰函数（详情可参考React高阶组件）
 * @param {*} WrappedComponent 传入React组件对象
 * @returns 经过加工后新的React组件
 */
const formItemDecorator = function formItemDecorator(WrappedComponent) {
  return class extends Component {
    componentWillMount() {
      const { form, prop, rules } = this.props
      const { getFieldDecorator } = form
      this.fieldDecorator = getFieldDecorator(prop, {rules: [...rules]})
    }

    render() {
      const {form, prop} = this.props
      const {getFieldError} = form
      return (
        <View>
          {this.fieldDecorator(<WrappedComponent {...this.props} error={getFieldError(prop)} />)}
        </View>
      )
    }
  }
}


//InputItem.js中使用
export default formItemDecorator(InputItem)

```
