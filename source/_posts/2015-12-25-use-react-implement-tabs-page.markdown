---
layout: post
title: "用React实现点击切换的标签页"
date: 2015-12-25 11:25:48 +0800
comments: true
categories: 
---
### 一、首先是Showcase
<p data-height="268" data-theme-id="0" data-slug-hash="KVgNeK" data-default-tab="result" data-user="charleyw" class='codepen'>See the Pen <a href='http://codepen.io/charleyw/pen/KVgNeK/'>react-tabs</a> by Wang Chao (<a href='http://codepen.io/charleyw'>@charleyw</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

### 二、如何实现
既然用React写，那么它就必然是一个组件，首先考虑你怎么使用这个组件，也就是这个组件的接口是怎么样的。

```
<TabsControl>
  <Tab name="red">
    <div className="red"/>
  </Tab>
  <Tab name="blue">
    <div className="blue"/>
  </Tab>
  <Tab name="yellow">
    <div className="yellow"/>
  </Tab>
</TabsControl>
```
这个`TabsControl`作为父组件，它来控制`Tab`的如何切换，`Tab`是用来包裹真正要显示的内容的，它的`name`属性是这个标签页的名字，会被显示在标签页的标题栏上。
### 三、创建基本元素
按照之前的想法，我们用`Tab`定义了很多个标签页，我们需要根据这些定义生成标签页的标题栏和内容。
#### 1. 遍历`this.props.children`动态生成标题栏
`this.props.children`是React内建的一个属性，用来获取组件的子元素。因为子元素有可能是Object或者Array，所以React提供了一些处理children的辅助方法用来遍历：`React.Children.map`

那么动态生成标题的代码可能是这样子的：

```
React.Children.map(this.props.children, (element, index) => {
	return (<div className="tab-title-item">{element.props.name}</div>)
```
#### 2. 再用同样方法生成标签页内容
```
React.Children.map(this.props.children, element => {
	return (element)
})
```
组合起来就是`TabsControl`的实现：

```
let TabsControl = React.createClass({
  render: function () {
    let that = this;
    let {baseWidth} = this.props;
    let childrenLength = this.props.children.length;
    return (
      <div>
        <nav className="tab-title-items">
          {React.Children.map(this.props.children, (element, index) => {
            return (<div className="tab-title-item">{element.props.name}</div>)
          })}
        </nav>
        <div className="tab-content-items">
          {React.Children.map(this.props.children, element => {
            return (element)
          })}
        </div>
      </div>
    )
  }
});
```
加上一些css就能看到一个标签页的雏形了。

### 三、实现标签页切换
这里要出现React最重要的概念了`state`，`state`是一个Javascript的Object，它是用来表示组件的当前状态的，如果用`TabsControl`举例的话，它的`state`可以是当前处于激活状态的标签页编号（当然，如果你想的话也可以保存标签页的内容）。
React提供了一个方法`setState()`让你可以改变`state`的值。每次调用`setState()`都会触发组件的`render()`，也就是说会把组件所代表的DOM更新到`state`所代表的状态。

所以实现切换的关键如下：
1. `state`保存当前处于激活状态的标签页的编号
1. 点击标题的时候调用`setState()`更新激活的标签页编号
2. `render()`的时候，在遍历`this.props.children`的时候把编号与`state`中编号一致的元素标记为`active`
3. 用css将非`active`的元素隐藏起来

所以代码是这样的：

```
let TabsControl = React.createClass({
  getInitialState: function(){
    return {currentIndex: 0}
  },
  
  getTitleItemCssClasses: function(index){
    return index === this.state.currentIndex ? "tab-title-item active" : "tab-title-item";
  },
  
  getContentItemCssClasses: function(index){
    return index === this.state.currentIndex ? "tab-content-item active" : "tab-content-item";
  },
  
  render: function(){
    let that = this;
    let {baseWidth} = this.props;
    let childrenLength = this.props.children.length;
    return (
      <div>
        <nav className="tab-title-items">
          {React.Children.map(this.props.children, (element, index) => {
            return (<div onClick={() => {this.setState({currentIndex: index})}} className={that.getTitleItemCssClasses(index)}>{element.props.name}</div>)
          })}
        </nav>
        <div className="tab-content-items">
          {React.Children.map(this.props.children, (element, index) => {
            return (<div className={that.getContentItemCssClasses(index)}>{element}</div>)
          })}  
        </div>
      </div>
    )
  }
});
```

`getInitialState`：是组件的初始化状态，默认是第一个标签页处于激活状态。  
`getTitleItemCssClasses`：判断当前标签和`state`中保存的标签编号是否一直，是则标识为`active`。  
`getContentItemCssClasses`：同上。  
`onClick={() => {this.setState({currentIndex: index})}}`：标签页标题绑定了点击事件，每次点击都会更新`state`保存的标签页编号，然后触发`render()`方法重绘标签页。  

### 四、总结
上面一系列的操作最终的结果都需要用`render()`来反应出来，所以关键点是如何在`render()`中使用`state`来动态生成DOM.
#### 接下来的改进
实现可以滑动的标签页