---
id: reusable-components-zh-CN
title: 可重用组件
permalink: reusable-components-zh-CN.html
prev: multiple-components-zh-CN.html
next: transferring-props.html
---

在设计接口的时候，将常用元素（按钮，表单字段，布局组件等）分解成定义好接口的可重用组件。这样下次你在创建同样界面就可以少写很多代码，这意味着更快的开发时间，更少的bug，和更少的内容下载。


## 属性验证

当你的应用程序变得越来越大，保证你的组件能被正确的使用是很有用的功能。通过提供`propTypes` 可以做到这点。'React.PropTypes' 提供了一系列的验证函数来保证组件收到的数据是合法的。当一个不合法的数据提供给属性，在JavaScript console 会有警告信息。注意因为性能的原因，`propTypes` 仅仅在开发的时候检查。这里有个使用不同验证函数的例子：

```javascript
React.createClass({
  propTypes: {
    // 你可以申明一个属性是JS 里面的一种元类型，缺省是可选的
    optionalArray: React.PropTypes.array,
    optionalBool: React.PropTypes.bool,
    optionalFunc: React.PropTypes.func,
    optionalNumber: React.PropTypes.number,
    optionalObject: React.PropTypes.object,
    optionalString: React.PropTypes.string,

    // 任何类型的: numbers, strings, elements 或者任何这种类型的数组
    optionalNode: React.PropTypes.node,

    // React 元素
    optionalElement: React.PropTypes.element,

    // 你也可以申明属性是一个类的实例这个使用JS 的instanceof 操作符
    optionalMessage: React.PropTypes.instanceOf(Message),

    // 你也可以通过定义enum来保证属性限定在某些具体的值
    optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),

    // 任何下面一种类型中的对象
    optionalUnion: React.PropTypes.oneOfType([
      React.PropTypes.string,
      React.PropTypes.number,
      React.PropTypes.instanceOf(Message)
    ]),

    // 某种类型的数组
    optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),

    // 具有某类属性值的对象
    optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),

    // 定义某种形状的对象
    optionalObjectWithShape: React.PropTypes.shape({
      color: React.PropTypes.string,
      fontSize: React.PropTypes.number
    }),

    // 你可以在上面任何一个类型后面加上 `isRequired`， 这样当属性没有提供的时候就会显示警告信息
    requiredFunc: React.PropTypes.func.isRequired,

    // 任何数据类型的值
    requiredAny: React.PropTypes.any.isRequired,

    // 你也可以提供自定义的验证函数，这个函数当验证失败的情况下返回Error 对象。不要用`console.warn` 或者 throw, 因为它在`oneOfType` 的情况下不工作。
    customProp: function(props, propName, componentName) {
      if (!/matchme/.test(props[propName])) {
        return new Error('Validation failed!');
      }
    }
  },
  /* ... */
});
```


## 缺省属性值

React 让你可以通过描述的方法定义缺省属性值。

```javascript
var ComponentWithDefaultProps = React.createClass({
  getDefaultProps: function() {
    return {
      value: 'default value'
    };
  }
  /* ... */
});
```

`getDefaultProps()` 的值会被缓存下来，如果父组件没有提供属性的值，这个值就会被使用。这样你就可以很安全的使用属性，而不用重复这样的情况。


## 传递属性：一个捷径

一个常见React 组件类型是简单扩展基本HTML 标签。很多情况下，你需要拷贝组件的属性，将它传递给HTML 属性。为了少敲几个键，你可以使用JSX _spread_ 语法来实现：

```javascript
var CheckLink = React.createClass({
  render: function() {
    // 这个会将所有传递给CheckLink 的属性拷贝给<a>
    return <a {...this.props}>{'√ '}{this.props.children}</a>;
  }
});

React.render(
  <CheckLink href="/checked.html">
    Click here!
  </CheckLink>,
  document.getElementById('example')
);
```

## 单个子组件

使用 `React.PropTypes.element` 你可以指明只有单个组件可以传给组件作为子组件。

```javascript
var MyComponent = React.createClass({
  propTypes: {
    children: React.PropTypes.element.isRequired
  },

  render: function() {
    return (
      <div>
        {this.props.children} // 有且仅有一个元素，否则会抛出异常
      </div>
    );
  }

});
```

## Mixins

在React 中，组件是最好的重用代码的方法，但是有些时候很多不同的组件可能共享同样的功能。这在有些时候叫 [cross-cutting concerns](http://en.wikipedia.org/wiki/Cross-cutting_concern). React 提供 `mixins` 来解决这个问题。

一个常见的例子是一个组件需要间隔一段时间更新自己。使用`setInterval()` 会很简单，但是很重要的是当你不需要的时候取消它来减少内存使用。React 提供了生命周期有关的方法 [lifecycle methods](/react/docs/working-with-the-browser.html#component-lifecycle) 让你有机会知道一个组件将要被创建或者销毁。让我们创建一个简单的mixin 来使用这些方法，提供简单的可以在组件销毁的时候自动回收的`setInterval()` 函数。

```javascript
var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.map(clearInterval);
  }
};

var TickTock = React.createClass({
  mixins: [SetIntervalMixin], // 使用mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // 使用 mixin 提供的函数
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

React.render(
  <TickTock />,
  document.getElementById('example')
);
```

mixins 一个很好的功能是如果组件使用多个mixins 并且多个mixins定义在同一个生命周期方法中（比如：几个mixins 都希望在组件销毁的时候做一些清理工作），所有的生命周期方法都会被调到。mixin列出的顺序就是方法被调到得顺序，最后会调用组件的方法。

