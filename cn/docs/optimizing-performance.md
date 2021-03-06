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
* [**`性能优化`**](/cn/docs/optimizing-performance.md)
* [不使用 ES6 的 React](/cn/docs/react-without-es6.md)
* [不使用 JSX 的 React](/cn/docs/react-without-jsx.md)
* [一致性比较（Reconciliation）](/cn/docs/reconciliation.md)
* [上下文（Context）](/cn/docs/context.md)
* [Web Components](/cn/docs/web-components.md)
* [高阶组件](/cn/docs/higher-order-components.md)
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

# 优化性能

在内部，React使用几种巧妙的技术来最大限度地减少更新 UI 所需的昂贵的 DOM 操作的数量。对于大多数应用而言，使用 React 可以达到一个快速的用户界面，而不需要做太多的工作来专门优化性能。 然而，有几种方法可以加快你的React应用。

## 使用生产版本

如果您在 React 应用中进行基准测试或遇到性能问题，请确保您正在使用压缩过的生产版本进行测试。

默认情况下，React 包含一些错误帮助信息。这些错误信息在开发环境中是非常有帮助的。然而，它们使得 React 又大又慢，因此你应该确保部署你的应用时使用生产版本。

如果你不能确定你的构建过程是否正确设置，你可以通过Chrome安装 [React 开发者工具](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) 来检查。如果你在生产模式下查看一个用 React 创建的站点，这个工具的 icon 将有一个深色的背景。

<img src="/cn/img/docs/devtools-prod.png" style="max-width:100%" alt="React DevTools on a website with production version of React">

如果你在开发模式下查看一个用 React 创建的站点，这个工具的 icon 将有一个红色的背景。
If you visit a site with React in development mode, the icon will have a red background:

<img src="/cn/img/docs/devtools-dev.png" style="max-width:100%" alt="React DevTools on a website with development version of React">

最好在开发应用时使用开发模式，部署应用时换为生产模式。

您可以在下面找到构建您的应用程序的说明。

### Create React App

如果你的应用是通过 [Create React App](https://github.com/facebookincubator/create-react-app) 创建的, 运行:

```
npm run build
```

这将在你项目的 `build/` 目录为你的应用创建一个生产构建。

记住这只在部署到生产环境前才是必须的。在一般的开发环境中，使用 `npm start`.

### 单文件构建

我们提供单个文件生产版本的 React 和 React DOM：

```html
<script src="https://unpkg.com/react@15/dist/react.min.js"></script>
<script src="https://unpkg.com/react-dom@15/dist/react-dom.min.js"></script>
```

记住只有以 `.min.js` 结尾的 React 文件才是适用于生产环境的。

### Brunch [一个构建工具](http://brunch.io/)

为了创建更高效的 Brunch 生产构建，安装 [`uglify-js-brunch`](https://github.com/brunch/uglify-js-brunch) 插件：

```
# 如果你使用 npm
npm install --save-dev uglify-js-brunch

# 如果你使用 Yarn
yarn add --dev uglify-js-brunch
```
然后，去创建一个生产构建，添加 `-p` 参数到 `build` 命令：

```
brunch build -p
```

记住你只需要在生产构建的时候这么做。你不应该在开发环境中传递 `-p` 参数或使用这个插件，因为它将隐藏有用的 React 警告信息，县城会让构建更慢。

### Browserify [一个构建工具](http://browserify.org/)

为了创建更高效的 Browserify 生产构建，需要安装一些插件：

```
# 如果你使用 npm
npm install --save-dev bundle-collapser envify uglify-js uglifyify 

# 如果你使用 Yarn
yarn add --dev bundle-collapser envify uglify-js uglifyify 
```

要创建一个生产构建，请确保你添加了这些转换器 **(这点很重要)**:

* [`envify`](https://github.com/hughsk/envify) 转换器确保正确的编译环境被设置。全局安装 (`-g`).
* [`uglifyify`](https://github.com/hughsk/uglifyify) 转换器移除开发接口. 全局安装 (`-g`).
* [`bundle-collapser`](https://github.com/substack/bundle-collapser) 插件用数字替换长长的模块ID.
* 最终, 产生的结果被管理传送到 [`uglify-js`](https://github.com/mishoo/UglifyJS2) 来整合 ([了解原因](https://github.com/hughsk/uglifyify#motivationusage)).

示例:

```
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  -p bundle-collapser/plugin \
  | uglifyjs --compress --mangle > ./bundle.js
```

>**提示:**
>
>包名是 `uglify-js`, 但是它提供的文件名叫 `uglifyjs`.
>这不是一个印刷错误.

记住你只需要在生产构建中这么做。你不应该把这些插件用在开发环境中，因为它将隐藏有用的 React 警告信息，并且使得构建变慢。

### Rollup [一个构建工具](http://rollupjs.org/)

要创建更高效的 Rollup 生产构建，需要安装一些插件：

```
# 如果你使用 npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 

# 如果你使用 Yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 
```

要创建一个生产构建，请确保你添加了这些插件 **(这点很重要)**:

* [`replace`](https://github.com/rollup/rollup-plugin-replace) 插件确保设置了正确的构建环境
* [`commonjs`](https://github.com/rollup/rollup-plugin-commonjs) 插件在 Rollup 内提供对 CommonJS 的支持
* [`uglify`](https://github.com/TrySound/rollup-plugin-uglify) 插件压缩生成最终版本

```js
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-uglify')(),
  // ...
]
```

一个完整的安装示例 [看这里](https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0).

记住你只需要在生产构建时这么做。在开发环境中不应该使用 `uglify` 或 `replace` 插件的 `'production'` 值，因为它们将隐藏有用的 React 警告信息，并使构建更慢。

### Webpack [一个构建工具](https://webpack.github.io/)

>**提示:**
>
>如果你正在使用 Create React App, 请拉到 [上述文档](#create-react-app).
>这个章节只适用于直接配置 Webpack 的情况

为了更高效的 Webpack 生产构建，请确保在你的生产配置里包含了这些插件：

```js
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

你能在 [Webpack 文档](https://webpack.js.org/guides/production-build/) 里了解到更多

记住你需要在生产构建里这么做，你不应该在开发环境中使用 `UglifyJsPlugin` 或 `DefinePlugin` 的 `'production'` 值，因为它们将隐藏有用的 React 警告信息，并使构建更慢。

## 通过 Chrome Timeline 分析组件性能

在 **开发模式** 中, 你能在支持的浏览器里使用性能工具可视化组件如何装载(mount) ，更新(update) 和 卸载(unmount) 的过程，例如:

<center><img src="/cn/img/blog/react-perf-chrome-timeline.png" style="max-width:100%" alt="React components in Chrome timeline" /></center>

在Chrome里这么做:

1. 在你的请求字符串里通过 `?react_perf` 加载你的应用 (例如, `http://localhost:3000/?react_perf`).

2. 打开Chrome开发者工具 **[Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool)** 标签并点击 **Record**（译者：新版Chrome已经改用 [Performance](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)）.

3. 执行你想要分析的操作. 不要记录超过20秒，否则Chrome可能会挂起.

4. 停止记录.

5. React 事件将被分组在 **User Timing** 标签下

注意 **这些数字是相对的，组件在生产环境中将更快** 。然而，这对你分析由于错误导致不相关的组件的更新、分析组件更新的深度和频率很有帮助。

最新的 Chrome, Edge, 和 IE 是唯一支持这个特性的浏览器，但是我们使用标准的  [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) ，因此我们期待更多浏览器对它添加支持。

## 避免更新

React 构建和维护渲染 UI 的内部表现。它包含从你的组件中返回的 React 元素。这个表现让 React 避免创建 DOM 节点和访问已经存在的 DOM 节点，因为这比操作 JavaScript 对象更慢。有时它被称为"虚拟DOM"，但是它在 React Native 中以一样的方式运行。

当一个组件的属性或状态变动，React 通过比较新返回的元素 和 之前渲染的元素来决定是否有必要更新DOM元素，当它们不相等时，React将更新DOM。

在一些的情况下，你的组件可以通过重写生命周期函数 `shouldComponentUpdate` 来优化性能，它在重新渲染进程开始前触发。这个函数的默认实现返回 `true`，使得 React 执行更新：

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果你知道在一些你的组件不需要更新的情况，你能在 `shouldComponentUpdate` 中返回 `false` ，用来跳过整个渲染进程，包含在组件上调用 `render()` 和之后的流程。

## shouldComponentUpdate

这是一个组件树。这里面，`SCU` 表示 `shouldComponentUpdate` 返回结果,  `vDOMEq` 表示已渲染的 React 元素是否相等。最后，这个圆形的颜色表示组件是否需要更新。

![](/cn/img/docs/should-component-update.png)

由于以 C2 为根节点的子树 `shouldComponentUpdate` 返回 `false` ，React 不会试图渲染 C2，因此也不会在 C4 和 C5 上调用 `shouldComponentUpdate`

对于 C1 和 C3, `shouldComponentUpdate` 返回 `true`, 因此 React 需要向下遍历检查叶子节点。 C6 `shouldComponentUpdate` 返回 `true`, 并且由于需要渲染的元素不相同，因此 React 需要更新 DOM。

最后一个值得关注的例子是 C8。React 必须渲染这个组件，但由于返回的 React 元素跟之前渲染的一样，因此它不需要更新 DOM。

注意 React 仅需要修改 C6 的 DOM，这是必须的。对于 C8 ，通过比较已渲染的 React 元素被剔除，对于 C2 的子树和 C7，它甚至不需要比较元素，我们在 `shouldComponentUpdate` 里就已经剔除了，并且 `render` 不会被调用。

## 示例

如果你的组件变化的唯一方式是通过 `props.color` 或 `state.count` 变量改变，你可以通过 `shouldComponentUpdate` 来检查:

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

在这段代码里， `shouldComponentUpdate` 只需检查`props.color`或`state.count`是否有任何变化。如果这些值没有变化，组件不会更新。如果你的组件更复杂，你可以使用类似的模式，在“props”和“state”的所有字段之间进行“浅比较”，以确定组件是否应该更新。这个模式很普遍，React提供了一个帮助程序来使用这个逻辑 - 只是继承自'React.PureComponent'。因此下面这段代码是一种更简单的方式来做同样的事情：

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

大部分时间，你能使用 `React.PureComponent` 代替你自己写 `shouldComponentUpdate`.它只做一个浅比较，因此如果 props 或 state 可能在浅比较中失效的话，你就不能使用它。

如果有更复杂的数据结构这可能有一些问题，假设你想要一个“ListOfWords”组件来呈现一个逗号分隔的单词列表，在父组件 `WordAdder` 中当你点击一个按钮添加一个单词到列表。这段代码 *不能* 正确的执行：

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // 这个部分是不好的风格，造成一个错误
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

这里的问题在于 `PureComponent` 将在 `this.props.words` 旧的值和新的值之间做一个简单的比较。由于这段代码在 `WordAdder` 的 `handleClick` 方法里改变了 `words` 数组，虽然在数组里的单词实际已经有变化，但 `this.props.words` 旧的值和新的值是相等的。虽然它有应该被渲染的新的单词，但 `ListOfWords` 不会更新。

## 不可变数据的力量

避开这个问题最简单的方式是避免使用变化的值作为属性(props)或状态(state)。例如，上面的 `handleClick` 方法能使用`concat` 重写：

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6 支持数组 [展开语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)，可以使这个问题变得简单。如果你正在使用 Create React App，默认支持该语法。

```js
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

你也能以一种简单的方式重写代码，使得改变对象的同时不会突变对象。例如，假设我们有一个名为 `colormap` 的对象，并且我们要写一个函数来将 `colormap.right` 的值变为 `'blue'`。我们可以这么写：

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

要不改变原始对象来实现，我们能使用 [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 方法：

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap` 现在返回一个新的对象，而不是改变之前的对象。 `Object.assign` 属于 ES6 的语法，需要一个 polyfill.

有一个 JavaScript 提案添加了 [对象展开符](https://github.com/sebmarkbage/ecmascript-rest-spread) 来让更新对象而不改变对象变得更简单：

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

如果你正在使用 Create React App， `Object.assign` 和对象展开语法默认都是可用的。

## 使用不可变（Immutable）数据结构

[Immutable.js](https://github.com/facebook/immutable-js) 是解决这个问题的另外一种方式。其提供了通过结构共享实现(Structural Sharing)地不可变的(Immutable)、持久的(Persistent)集合:

* *不可变（Immutable）*: 一旦创建，集合在其它时间不能被改变。
* *持久的(Persistent)*: 可以从之前的集合和诸如 set 这样的突变中创建新的集合. 新的集合创建后之前的集合依旧是可用的.
* *结构共享(Structural Sharing)*: 新的集合尽可能使用之前的集合相同的结构创建，将复制降至最低，以提高性能。

不可变性使得追踪变得容易。改变会产生新的对象，因此我们仅需要检查对象的引用是否改变。例如，下面是普通的JavaScript代码：

```javascript
const x = { foo: 'bar' };
const y = x;
y.foo = 'baz';
x === y; // true
```

尽管 `y` 已经被编辑, 但由于它引用的是相同的对象 `x`, 这个比较返回 `true`. 你能用 immutable.js 写类似的代码:

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
x === y; // false
```

在种情况下，由于当改变 `x` 时返回一个新的引用，我们可以放心的假设 `x` 已经改变了

其它两个能帮助我们使用不可变数据的库分别是 [seamless-immutable](https://github.com/rtfeldman/seamless-immutable) 和 [immutability-helper](https://github.com/kolodny/immutability-helper).

不可变数据结构提供了一种简单的方式追踪对象的变化，这正是我们实现 `shouldComponentUpdate` 所需要的. 这经常能给你带来可观的性能提升.


---

* 上一篇：[不可控组件](/cn/docs/uncontrolled-components.md)
* 下一篇：[不使用 ES6 的 React](/cn/docs/react-without-es6.md)
