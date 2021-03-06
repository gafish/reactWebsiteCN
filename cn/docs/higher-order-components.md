[文档](/cn/docs/hello-world.md) | [教程](/cn/tutorial/tutorial.md) | [社区](/cn/community/support.md) | [博客](/cn/_posts/2017-04-07-react-v15.5.0.md) | [React Github](https://facebook.github.io/react/)

---
<details>
  <summary>导航</summary>

#### 快速入门

* [安装](/cn/docs/installation.md)
* [Hello World](/cn/docs/hello-world.md)
* [JSX 介绍](/cn/docs/introducing-jsx.md)
* [渲染元素](/cn/docs/rendering-elements.md)
* [组件和Props](/cn/docs/components-and-props.md)
* [State和生命周期](/cn/docs/state-and-lifecycle.md)
* [事件处理](/cn/docs/handling-events.md)
* [条件渲染](/cn/docs/conditional-rendering.md)
* [列表和键](/cn/docs/lists-and-keys.md)
* [表单](/cn/docs/forms.md)
* [状态提升](/cn/docs/lifting-state-up.md)
* [组合 vs 继承](/cn/docs/composition-vs-inheritance.md)
* [用 React 思考](/cn/docs/thinking-in-react.md)

#### 高级教程

* [深入JSX](/cn/docs/jsx-in-depth.md)
* [使用 PropTypes 做类型检查](/cn/docs/typechecking-with-proptypes.md)
* [Refs 和 DOM](/cn/docs/refs-and-the-dom.md)
* [不可控组件](/cn/docs/uncontrolled-components.md)
* [性能优化](/cn/docs/optimizing-performance.md)
* [不使用 ES6 的 React](/cn/docs/react-without-es6.md)
* [不使用 JSX 的 React](/cn/docs/react-without-jsx.md)
* [一致性比较（Reconciliation）](/cn/docs/reconciliation.md)
* [上下文（Context）](/cn/docs/context.md)
* [Web Components](/cn/docs/web-components.md)
* [**`高阶组件`**](/cn/docs/higher-order-components.md)
* [与其它类库集成](/cn/docs/integrating-with-other-libraries.md)

#### 参考

* [React](/cn/docs/react-api.md)
* [React.Component](/cn/docs/react-component.md)
* [ReactDOM](/cn/docs/react-dom.md)
* [ReactDOMServer](/cn/docs/react-dom-server.md)
* [DOM 元素](/cn/docs/dom-elements.md)
* [合成事件（SyntheticEvent）](/cn/docs/events.md)

#### 贡献

* [如何贡献](/cn/contributing/how-to-contribute.md)
* [代码库概述](/cn/contributing/codebase-overview.md)
* [实现说明](/cn/contributing/implementation-notes.md)
* [设计原则](/cn/contributing/design-principles.md)


</details>

# 高阶组件(Higher-Order Components)

高阶组件(HOC)是 React 中可复用组件逻辑的高级技术。HOCs 不是 React API 的一部分。它源自 React 生态体系。

具体地来说，**高阶组件是一个函数，能够接收一个组件并且返回一个组件**

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

组件是将属性转换成 UI，而高阶组件是将一个组件转换成另一个组件。

HOCs 在第三方 React 库中比较常见，像 Redux's [`connect`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) 和 Relay's [`createContainer`](https://facebook.github.io/relay/docs/api-reference-relay.html#createcontainer-static-method).

在这篇文档里，我们将讨论为什么高阶组件是有用的，并且如何编写你自己的高阶组件。

## 在横切关注点中使用高阶组件

> **注意**
>
> 我们以前推荐 mixins 作为处理横切关注点的一种方式。我们已经意识到 mixins 相比它创造的价值会带来更多麻烦。[阅读更多](/cn/blog/2016/07/13/mixins-considered-harmful.md) 关于为什么我们已经放弃了 mixins，并且你如何过渡你目前的组件。

组件是 React 中代码复用的主要单位。然而，你将发现某些模式不是简单适用于传统的组件。

例如，假设你有一个`CommentList` 组件可以订阅一个外部的数据源，用来渲染一个评论列表：

```js
class CommentList extends React.Component {
  constructor() {
    super();
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" 是一些全局数据源
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // 更改订阅
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // 清除监听
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // 当数据源变化时更新组件
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

之后，你写了一个组件用于订阅一个简单的博客贴子，它类似于下面的模型：

```js
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

`CommentList` 和 `BlogPost` 不是完全相同的，它们调用`DataSource`上不同的方法, 并且它们渲染不同的输出。但它们大多数实现是相同的：

- 在挂载阶段，添加一个变化监听到 `DataSource`.
- 在监听内部，每当数据源变化时调用 `setState` .
- 在卸载阶段，移除变化监听.

你能想像一下在大型应用中，这种订阅 `DataSource` 和调用 `setState` 的模式将反复不断地发生。我们需要一个抽象允许我们在一个地方去定义这个逻辑，并且将它们分享到更多组件。这是高阶组件擅长的地方。

我们能写一个创建组件的函数，像`CommentList` 和 `BlogPost` ，在那订阅 `DataSource`。该函数将接受作为其参数之一的子组件，该子组件接收订阅的数据作为属性。让我们叫这个函数为 `withSubscription`：

```js
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
});
```

第一个参数是容器组件。第二个参数检索我们感兴趣的数据，给出一个 `DataSource` 和当前的属性。

当 `CommentListWithSubscription` 和 `BlogPostWithSubscription` 被渲染时， `CommentList` 和 `BlogPost` 将被传递一个 `data` 属性，并从 `DataSource` 检索最新的数据：

```js
// 此函数需要一个组件...
function withSubscription(WrappedComponent, selectData) {
  // ...并且返回另外一个组件...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... 它负责订阅 ...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... 从最新的数据渲染容器组件!
      // 注意我们传递一些附加的属性
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

请注意， HOC 不会修改输入组件，也不使用继承来拷贝它的行为。相反，HOC 通过 *包裹* 在一个容器组件里来 *组成* 原始组件。HOC 是一个没有副作用的纯函数。

就是这样！容器组件接收所有的容器属性，以及一个新的属性 `data`，它用于渲染输出。HOC 不关心如何或为什么数据被使用，容器组件不关心数据从哪里来。

因为 `withSubscription` 是一个普通的函数，所以你能添加任意数据的参数。例如，你可能希望可以配置 `data` 的名字，以进一步将 HOC 和 容器组件隔离。或你可能接受配置 `shouldComponentUpdate` 参数，或配置数据源的参数。这些都是可能，因为 HOC 完全控制组件的定义。

与组件一样， `withSubscription` 和容器组件之间的联系完全是基于属性的。 这样可以方便地将一个HOC替换为不同的HOC，只要它们为容器组件提供相同的属性。如果更改数据获取库，这可能很有用，例如。

## 不要改变原始组件，而是使用组合

抵制在HOC内部修改组件原型（或以其他方式修改）的诱惑。

```js
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  }
  // 事实上，我们返回的原始输入意味着它已经被修改
  return InputComponent;
}

// EnhancedComponent 将在接收到属性时记录
const EnhancedComponent = logProps(InputComponent);
```

这会有一些问题。一是输入组件不能与 enhanced 组件分开复用。更关键的是，如果你应用其它 HOC 到 `EnhancedComponent` ，那么*也*将修改 `componentWillReceiveProps`， 第一个 HOC 的功能将被覆盖！ 这个HOC也不适用于没有生命周期方法的功能组件。

具有修改功能的 HOC 是一种有漏洞的抽象 - 用户必须知道如何实施它们，以避免与其他HOC的冲突。

相比较修改，HOCs 应该使用组合，通过包裹输入组件在一个容器组件里：

```js
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // 包裹输入组件在一个容器组件里, 没有修改它，非常好!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

这个 HOC 有跟修改版本同样的功能，同时避免了潜在的冲突。它在 class 和 functional 组件里同样运行的非常好。并且因为它是纯函数，它可以和其它 HOCs 甚至它自己组合。

你可能已经注意到 HOCs 和 **容器组件**模式 有相似之处。容器组件是将责任分为高级和低级关注点策略的一部分。容器管理类似于订阅和状态的东西，传递属性到处理类似于渲染UI之类的组件。HOCs 使用容器作为它们实现的一部分。你可以将 HOCs 作为参数容器组件来定义。

## 约定: 给包裹组件传递不相关的属性(Props)

HOCs 向组件功能功能。它们不应该彻底的修改它的约定。它预期组件从 HOC 返回一个相似的界面到包裹组件。

HOCs 应该传递与其具体关切无关的属性(props)。大多数HOC包含一个如下所示的渲染方法：

```js
render() {
  // 过滤特定于本 HOC 的额外属性(props)，不应该被传递
  const { extraProp, ...passThroughProps } = this.props;

  // 注入属性(props)到包裹组件。这些通常是状态值或实例方法
  const injectedProp = someStateOrInstanceMethod;

  // 传递属性(props)到包裹组件
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

这个约定有助于确保 HOCs 尽可能灵活的和可复用

## 约定: 最大化组合

不是所有的 HOCs 都看起来一样。有时它们只接受包裹组件作为单个参数：

```js
const NavbarWithRouter = withRouter(Navbar);
```

通常来说，HOCs 接受额外的参数。在 Relay 这个示例中，配置对象用于指定组件的数据依赖关系：

```js
const CommentWithRelay = Relay.createContainer(Comment, config);
```

HOCs 最常见的签名如下：

```js
// React Redux 的 `connect`
const ConnectedComment = connect(commentSelector, commentActions)(Comment);
```

*什么？！*如果你把它分开，很容易看到发生了什么。

```js
// connect 是一个返回另一个函数的函数
const enhance = connect(commentListSelector, commentListActions);
// 返回的函数是一个 HOC，它返回一个连接到 Redux 存储的组件
const ConnectedComment = enhance(CommentList);
```

换句话说，`connect` 是一个返回高阶组件的高阶函数！

这张表可能看起来很混乱或不必要，但它有一个有用的属性。 `connect` 函数返回的单参数HOC具有 `Component => Component`的签名。 输出类型与其输入类型相同的函数真的很容易组合在一起。

```js
// 而不是这样做
const EnhancedComponent = connect(commentSelector)(withRouter(WrappedComponent))

// ... 你能使用功能组合实用程序
// compose(f, g, h) 和 (...args) => f(g(h(...args))) 是相同的
const enhance = compose(
  // 它们都是单个参数的 HOCs
  connect(commentSelector),
  withRouter
)
const EnhancedComponent = enhance(WrappedComponent)
```

(相同的属性与允许 `connect` 和其它的增加风格 HOCs 作为装饰器使用，一个实验性的提议)

`compose` 功能函数通过一些包含 lodash 的第三方库提供（像 [`lodash.flowRight`](https://lodash.com/docs/#flowRight)), [Redux](http://redux.js.org/docs/api/compose.html), 和 [Ramda](http://ramdajs.com/docs/#compose）

## 约定：为了方便调试包裹显示名称

通过 HOCs 创建的容器组件像其它任意组件一样在 [React Developer Tools](https://github.com/facebook/react-devtools) 里显示。为便于调试，选择一个表示它是 HOC 的显示名称。

大多数常用的技巧是包裹包裹组件的显示名。所以如果你的高阶组件名叫 `withSubscription`，并且包裹组件显示名是 `CommentList`，显示这个显示名 `WithSubscription(CommentList)`：

```js
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```


## 警告

高阶组件带有一些警告，如果您刚接触到React，则不会立即显现。

### 不要在 render 方法里使用 HOCs

React 的差异算法（又称 reconciliation）使用组件特性来决定是否它应该更新目前的子树，或放弃它挂载一个新的。如果从 `render` 返回的组件跟之前渲染的组件完全相等(`===`)。React 通过新的组件递归更新子树。如果它们不相等，上一个子树将完全卸载。

一般来说，你不需要关心这些。但是对于HOC来说很重要，因为它意味着你不能将 HOC 应用到组件的render方法中的组件：

```js
render() {
  // 在每一个 render 上创建一个新版本的 EnhancedComponent 
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 这会导致整个子树每次都会执行 卸载/重新装载（unmount/remount）！
  return <EnhancedComponent />;
}
```

这里的问题不仅是关于性能，重新装载一个组件会导致组件的状态和所有的子节点丢失。

相反，在组件定义之外应用HOC，以便生成的组件只能创建一次。 那么，它的身份在渲染中是一致的。 无论如何，这通常是你想要的。

在那些你需要动态应用 HOC 的极少案例中，你也能在组件的生命周期方法或它的 constructor 里处理它。

### 静态方法必须复制

有时在 React 组件上定义一个静态方法是非常有用的。例如，Relay 容器暴露一个静态方法 `getFragment` ，用于构建 GraphQL 片断。

当你应用 HOC 到一个组件，但是，原始组件被容器组件包裹。这意味着新的组件没有任何原始组件的静态方法。

```js
// 定义一个静态方法
WrappedComponent.staticMethod = function() {/*...*/}
// 现在应用一个 HOC
const EnhancedComponent = enhance(WrappedComponent);

// 增强的组件没有静态方法
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

要解决这个问题，你可以在返回它之前复制一个方法到容器

```js
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 必须知道要复制的方法 :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

然而，这要求你知道需要复制哪个方法。你能使用 [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) 来自动的复制所有的非 React 静态方法： 

```js
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

另一种可能的解决方法是从组件自身输出静态方法

```js
// 代替这种做法
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...分别输出方法...
export { someFunction };

// ...并在使用的模块里，导入两者
import MyComponent, { someFunction } from './MyComponent.js';
```

### Refs 不会被传递

虽然高阶组件的约定是传递所有的属性到包裹组件，但它不可能传递 refs 。那是因为 `ref` 不是一个真正的属性，就像 `key` 一样。它通过 React 特殊处理。如果你添加 ref  到 HOC 结果的组件元素中，ref 引用一个最外层容器组件的实例，而不是包裹组件。

如果你发现你自己正面临这个问题，最理想的解决方案是想出如何从根本上避免使用 `ref` 。有时，刚开始使用 React 的用户依赖 refs ，在这种情况下，属性将做的更好。

也就是说，有些时候，refs 是必要的，否则 React 将不会支持它们。聚焦输入字段是你可能需要控制组件一个例子。在这种情况下，一种解决方案是传递一个 ref 回调作为一个普通的属性，通过一个不同的名字来获取：

```js
function Field({ inputRef, ...rest }) {
  return <input ref={inputRef} {...rest} />;
}

// 在一个高阶组件里包裹字段 Field
const EnhancedField = enhance(Field);

// 在 class 组件的 render 方法里...
<EnhancedField
  inputRef={(inputEl) => {
    // 这个回调作为普通的属性获取传递
    this.inputEl = inputEl
  }}
/>

// 现在你能调用需要的方法了
this.inputEl.focus();
```

这不是一个完美的解决方案。我们更喜欢 refs 保留在图书馆里，而不是需要你手动的处理它们。我们正在探索解决这个问题的方法，以便使用 HOC 是无感知的。


---

* 上一篇：[Web Components](/cn/docs/web-components.md)
* 下一篇：[与其它类库集成](/cn/docs/integrating-with-other-libraries.md)
