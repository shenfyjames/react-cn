# Keyed Fragments


在大多数情况下，你可以使用 `key` prop 指定你从 `render` 返回的元素的 keys。然而，这在一个情况下会失败：如果你有两组你需要记录的子级，将没有办法在不使用包裹元素的情况下放置一个 key 到每组上。

即是，如果你有一个像这样的组件：

```js
var Swapper = React.createClass({
  propTypes: {
    // `leftChildren` and `rightChildren` can be a string, element, array, etc.
    leftChildren: React.PropTypes.node,
    rightChildren: React.PropTypes.node,

    swapped: React.PropTypes.bool
  },
  render: function() {
    var children;
    if (this.props.swapped) {
      children = [this.props.rightChildren, this.props.leftChildren];
    } else {
      children = [this.props.leftChildren, this.props.rightChildren];
    }
    return <div>{children}</div>;
  }
});
```

这些子级会在当你改变 `swapped` prop 时加载和卸载，因为没有任何的 key 标记在这两组子级上。

要解决这个问题，你可以使用 `createFragment` 插件来给予这两组子级 keys.

#### `Array<ReactNode> createFragment(object children)`

代替创建数组，我们这样写：

```js
var createFragment = require('react-addons-create-fragment');

if (this.props.swapped) {
  children = createFragment({
    right: this.props.rightChildren,
    left: this.props.leftChildren
  });
} else {
  children = createFragment({
    left: this.props.leftChildren,
    right: this.props.rightChildren
  });
}
```

被传入对象的 keys （即 `left` 和 `right`）被用作为整组子级的 keys，并且对象 keys 的顺序被用于决定渲染子级的顺序。通过这个改变，这两个子级将会恰当的在 DOM 里排序，而不被卸载。

`createFragment` 的返回值应该被对待为一个不透明的对象;你可以使用 `React.Children` 来遍历一个 fragment 但是不应该直接访问它。同样注意，我们依赖于 JavaScript 引擎保留了对象的枚举顺序，这点在 spec 上是不保证的，但是所有主要的浏览器和 VMs 都对非数字键的对象实现了这个特性。

> **注意:**
>
> 将来，`createFragment` 也许会被替换为如下的API：
>
> ```js
> return (
>   <div>
>     <x:frag key="right">{this.props.rightChildren}</x:frag>,
>     <x:frag key="left">{this.props.leftChildren}</x:frag>
>   </div>
> );
> ```
>
> 允许你直接在 JSX 里赋值 keys 而不用添加包裹元素。 
