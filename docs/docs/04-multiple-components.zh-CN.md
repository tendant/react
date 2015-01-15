---
id: multiple-components-zh-CN
title: 多个组件
permalink: multiple-components-zh-CN.html
prev: interactivity-and-dynamic-uis-zh-CN.html
next: reusable-components-zh-CN.html
---

到目前为止，我们看到了怎么编写单个的组件用来显示数据和处理用户输入。下面让我们来看看一个React 最酷的功能：可组合性。


## 动机: 解耦

通过构建模块化的组件，你可以获得很多类似于使用函数或者类的好处。这样的组件可以使用定义好的接口来重用其他组件。具体的讲，你可以通过你可以通过构建新组件的方式来解耦你的应用程序。通过构建你自己应用程序的组件库，你可以用最适合你的领域的方法来表现用户界面。


## 组合实例

让我们来创建一个用户头像组件，这个组件使用Facebook Graph API 用来显示个人资料图片和用户名。

```javascript
var Avatar = React.createClass({
  render: function() {
    return (
      <div>
        <ProfilePic username={this.props.username} />
        <ProfileLink username={this.props.username} />
      </div>
    );
  }
});

var ProfilePic = React.createClass({
  render: function() {
    return (
      <img src={'http://graph.facebook.com/' + this.props.username + '/picture'} />
    );
  }
});

var ProfileLink = React.createClass({
  render: function() {
    return (
      <a href={'http://www.facebook.com/' + this.props.username}>
        {this.props.username}
      </a>
    );
  }
});

React.render(
  <Avatar username="pwh" />,
  document.getElementById('example')
);
```


## 从属关系

在上面的例子中，`Avatar`实例拥有`ProfilePic` and `ProfileLink`实例. 在React 中, **谁设置其他组件的属性`props`，它就是其他组件的所有者**。更正式一点讲，如果组件`Y` 的`render()` 方法创建了组件`X`，那么就说组件`X` 属于组件`Y`。就像前面谈到的，一个组件不能修改它的属性`props` - 他们永远保持和他的所有者设置的值一样。这个关键的属性保证了用户界面的一致性。

很重要的一点是要分清楚组件的从属关系和父子关系。从属关系是React 中特有的，而父子关系简单来讲就是DOM 里的标签的关系。在上面的例子中，`Avatar` 拥有`div`，`ProfilePic` 和`ProfileLink` 实例，而`div` 是`ProfilePic` 和`ProfileLink` 实例的父节点，但是`div`并不是它们的所有者。


## 子节点

当你创建React 组件的实例的时候，你可以在开始标签和结束标签之间引用在React 组件或者Javascript 表达式。例子如下：

```javascript
<Parent><Child /></Parent>
```

`Parent` 可以通过`this.props.children` 特殊属性访问子组件。**`this.props.children` 是一个不可见的数据结构:** 可以使用 [React.Children utilities](/react/docs/top-level-api.html#react.children) 来操作它们。


### 子组件同步

** 同步是React 在每一个render 步骤中更新DOM。** 一般来讲，子组件按照它们render的顺序来同步更新。例如，如果有下面两次render。

```html
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p>Paragraph 2</p>
</Card>
```

看起来像是 `<p>Paragraph 1</p>` 被删除了。实际上，React 会通过改变第一个子标签的内容和删掉第二个子标签的方式来实现同步。React 通过子组件的顺序来实现同步。


### 子组件状态管理

对于大多数的组件状态管理不是什么问题。然而有状态的组件在多次显示中，要使用`this.state` 来保持数据，这个可以很麻烦。

大多数情况下，这个可以通过隐藏某些元素，而不是销毁的方式来绕开。

```html
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p style={{'{{'}}display: 'none'}}>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
```


### 动态子组件

有些情况可以比较复杂，比如子组件会变换位置（例如查询结果），或者新的组件呗添加到列表的最前面。在这种情况下，标签和每一个子组件需要在多次render 中保持状态，你可以通过给每个子组件分配一个唯一标识的`key`。

```javascript
  render: function() {
    var results = this.props.results;
    return (
      <ol>
        {results.map(function(result) {
          return <li key={result.id}>{result.text}</li>;
        })}
      </ol>
    );
  }
```

当React 重新同步含有标识的子组件，它将会保证所有有标识的组件的顺序或者销毁它们（而不是重用它们）。

`key` 应该*永远*直接提供给列表里的每一个组件，而不是提供给列表里每一个组件的HTML 标签。

```javascript
// 错误!
var ListItemWrapper = React.createClass({
  render: function() {
    return <li key={this.props.data.id}>{this.props.data.text}</li>;
  }
});
var MyComponent = React.createClass({
  render: function() {
    return (
      <ul>
        {this.props.results.map(function(result) {
          return <ListItemWrapper data={result}/>;
        })}
      </ul>
    );
  }
});

// 正确 :)
var ListItemWrapper = React.createClass({
  render: function() {
    return <li>{this.props.data.text}</li>;
  }
});
var MyComponent = React.createClass({
  render: function() {
    return (
      <ul>
        {this.props.results.map(function(result) {
           return <ListItemWrapper key={result.id} data={result}/>;
        })}
      </ul>
    );
  }
});
```

你也可以通过传递对象的方式来提供键(key) 给子组件。对象的键值对将会被使用。然而，很重要的一点需要记住的是JavaScript 不保证属性的顺序。在某些浏览器里会保留顺序，除了那些可以被转换成32位无符号整数以外。 数字属性会按照顺序排在其他属性之前。在这种情况下，React 组件会显示错误的顺序。这可以通过添加一个前缀字符串的方式来避免。

```javascript
  render: function() {
    var items = {};

    this.props.results.forEach(function(result) {
      // If result.id can look like a number (consider short hashes), then
      // object iteration order is not guaranteed. In this case, we add a prefix
      // to ensure the keys are strings.
      items['result-' + result.id] = <li>{result.text}</li>;
    });

    return (
      <ol>
        {items}
      </ol>
    );
  }
```

## 数据流

在React 中，如上所述，数据是由组件通过`props` 流向从属组件。这是一种很邮箱的单向数据绑定：所有者使用它自己的属性和状态来绑定其从属组件的属性。因为这是一个递归的过程，数据改变会自动的反映在所有用到它得地方。


## 关于性能

你可能会觉得改变一个有很多节点的组件的数据会花很长时间。好消息是JavaScript 运行很快，`render()` 方法一般会很简单，所以大多数的应用程序应该会非常快。再加上性能瓶颈往往在DOM 的操作，而不是执行JavaScript 的时间。React 专门使用批量处理和检测改动的方式来优化。

然后，有些时候你还是很想要更细粒度的控制程序的性能。这种情况下，你可以通过重载`shouldComponentUpdate()` 方法，在你希望React 跳过处理子树的情况下直接返回false。更多信息请见 [the React reference docs](/react/docs/component-specs.html)

> 注意:
>
> 如果在数据真的改变的情况下 `shouldComponentUpdate()` 还是返回 false，React 不能保证你的UI 和数据同步. 在用这个方法的时候，你要保证自己知道在做什么。只在你知道有显著的性能问题的时候才使用它。相比较DOM，不要小看了JavaScript 的执行速度。
