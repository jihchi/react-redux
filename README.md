React Redux
=========================

Official React bindings for [Redux](https://github.com/gaearon/redux).  
Performant and flexible.

[![npm version](https://img.shields.io/npm/v/react-redux.svg?style=flat-square)](https://www.npmjs.com/package/react-redux) 
[![npm downloads](https://img.shields.io/npm/dm/react-redux.svg?style=flat-square)](https://www.npmjs.com/package/react-redux)
[![redux channel on slack](https://img.shields.io/badge/slack-redux@reactiflux-61DAFB.svg?style=flat-square)](http://www.reactiflux.com)


>**Note: There is a project called `redux-react` on NPM that is [completely unrelated](https://github.com/cgarvis/redux-react/issues/1) to the official bindings. This documentation (and any other official Redux documentation) is for `react-redux`.**

## Table of Contents

- [React Native](#react-native)
- [Quick Start](#quick-start)
- [API](#api)
  - [`<Provider store>`](#provider-store)
  - [`connect([mapState], [mapDispatch], [mergeProps])(Component)`](#connectmapstate-mapdispatch-mergeprops)
- [License](#license)

## React Native

What you get from `react-redux` is for React.  
For React Native, import from `react-redux/native` instead.

## Quick Start

React bindings for Redux embrace the idea of [dividing “smart” and “dumb” components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

It is advisable that only top-level components of your app (such as route handlers, for example) are aware of Redux. Components below them should be “dumb” and receive all data via props.

<center>
<table>
    <thead>
        <tr>
            <th></th>
            <th>Location</th>
            <th>Use React-Redux</th>
            <th>To read data, they</th>
            <th>To change data, they</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <td>“Smart” Components</td>
          <td>Top level, route handlers</td>
          <td>Yes</th>
          <td>Subscribe to Redux state</td>
          <td>Dispatch Redux actions</td>
        </tr>
        <tr>
          <td>“Dumb” Components</td>
          <td>Middle and leaf components</td>
          <td>No</th>
          <td>Read data from props</td>
          <td>Invoke callbacks from props</td>
        </tr>
    </tbody>
</table>
</center>

### “Dumb” component is unaware of Redux

Let’s say we have a `<Counter />` “dumb” component with a number `counter` prop, and an `increment` function prop that it will call when user presses an “Increment” button:

```js
import { Component } from 'react';

export default class Counter extends Component {
  render() {
    return (
      <button onClick={this.props.increment}>
        {this.props.counter}
      </button>
    );
  }
}
```

### “Smart” component is `connect()`-ed to Redux

Here’s how we hook it up to the Redux Store.

We will use `connect()` function provided by `react-redux` to turn a “dumb” `Counter` into a smart component. The `connect()` function lets you specify *which exactly* state from the Redux store your component wants to track. This lets you subscribe on any level of granularity.

Passing action creator functions as the second parameter will bind them to the specific store instance, and they will be injected as props with the same names they were exported with. 

Why don’t we bind action creators to a store right away? This is because of the so-called “universal” apps that need to render on the server. They would have a different store instance for every request, so we don’t know the store instance during the definition!

##### `containers/CounterContainer.js`

```js
import { Component } from 'react';
import { connect } from 'react-redux';

// Action creators:
import { increment } from '../actionsCreators';
// “Dumb” component:
import Counter from '../components/Counter';

// Which part of the Redux global state does our component want to receive as props?
function mapState(state) {
  return {
    counter: state.counter
  };
}

// First argument tells which state fields it’s interested in.
// Second argument tells which action creators to bind and inject.
// You may also pass a `dispatch` => Object function as a second argument.

export default connect(mapState, { increment })(CounterContainer);
```

Whether to put `connect()` call in the same file as the “dumb” component, or separately, is up to you.  
Ask yourself whether you'd want to reuse this component but bind it to different data, or not.

### Nesting

You can have many `connect()`-ed components in your app at any depth, and you can even nest them. It is however preferable that you try to only `connect()` top-level components such as route handlers, so the data flow in your application stays predictable.

### Support for Decorators

You might have noticed that we used parens twice. This is called partial applications, and it lets people
use ES7 decorator proposal syntax:

```js
// Unstable syntax! It might change or break in production.
@connect(mapState)
export default class CounterContainer { ... }
```

Don’t forget decorators are experimental! And they desugar to function calls anyway as example above demonstrates.

### Additional Flexibility

This the most basic usage, but `connect()` supports many other different patterns: just passing the vanilla `dispatch()` function down, binding multiple action creators, putting them as `actions` prop, selecting parts of state and binding action creators depending on `props`, and so on. Check out `connect()` docs below to learn more.

### Injecting Redux Store

Finally, how do we actually hook it up to a Redux store? We need to create the store somewhere at the root of our component hierarchy. For client apps, the root component is a good place. For server rendering, you can do this in the request handler.

The trick is to wrap the whole view hierarchy into `<Provider>{() => ... }</Provider>` where `Provider` is imported from `react-redux`. One gotcha is that **the child of `Provider` must be a function**. This is to work around an issue with how context (undocumented feature we have to rely on to pass Redux data to components below) works in React 0.13. In React 0.14, you will be able to put your view hierarchy in `<Provider>` without wrapping it into a function.

```js
import { Component } from 'react';
import { Provider } from 'react-redux';

class App extends Component {
  render() {
    // ...
  }
}

const targetEl = document.getElementById('root');

React.render((
  <Provider store={store}>
    {() => <App />}
  </Provider>
), targetEl);
```

## API

### `<Provider store>`

Makes Redux store available to the `connect()` calls in the component hierarchy below.  
You can’t use `connect()` without wrapping the root component in `<Provider>`.

#### Props

* `store`: (*[Redux Store](http://gaearon.github.io/redux/docs/api/Store.html)*): The single Redux store in your application.
* `children`: (*Function*): Unlike most React components, `<Provider>` accepts a [function as a child](#child-must-be-a-function) with your root component. This is a temporary workaround for a React 0.13 context issue, which will be fixed when React 0.14 comes out.

#### Example

##### Vanilla React

```js
React.render(
  <Provider store={store}>
    {() => <MyRootComponent>}
  </Provider>,
  rootEl
);
```

##### React Router 0.13

```js
Router.run(routes, Router.HistoryLocation, (Handler) => {
  React.render(
    <Provider store={store}>
      {() => <Handler />}
    </Provider>,
    targetEl
  );
});
```

##### React Router 1.0

```js
React.render(
  <Provider store={store}>
    {() => <Router history={history}>...</Router>}
  </Provider>,
  targetEl
);
```

### `connect([mapState], [mapDispatch], [mergeProps])`

Connects a React component to a Redux store.

#### Arguments

* [`mapState`] \(*Function*): If specified, the component will subscribe to Redux store updates. Any time it updates, `mapState` will be called. Its result must be a plain object, and it will be merged into the component’s props. If you omit it, the component will not be subscribed to the Redux store.

* [`mapDispatch`] \(*Object* or *Function*): If an object is passed, each function inside it will be assumed to be a Redux action creator. An object with the same function names, but bound to a Redux store, will be merged into the component’s props. If a function is passed, it will be given `dispatch`. It’s up to you to return an object that somehow uses `dispatch` to bind action creators in your own way. (Tip: you may use [`bindActionCreators()`](http://gaearon.github.io/redux/docs/api/bindActionCreators.html) helper from Redux.) If you omit it, the default implementation just injects `dispatch` into your component’s props.

* [`mergeProps`] \(*Function*): If specified, it is passed the result of `mapState()`, `mapDispatch()`, and the parent `props`. The plain object you return from it will be passed as props to the wrapped component. You may specify this function to select a slice of the state based on props, or to bind action creators to a particular variable from props. If you omit it, `{ ...props, ...mapStateResult, ...mapDispatchResult }` is used by default.

#### Returns

A React component class that injects state and action creators into your component according to the specified options.

#### Remarks

* It needs to be invoked two times. First time with its arguments described above, and second time, with the component: `connect(mapState, mapDispatch, mergeProps)(MyComponent)`.

* The `mapState` function takes a single argument of the entire Redux store’s state and returns an object to be passed as props. It is often called a **selector**. Use [reselect](https://github.com/faassen/reselect) to efficiently compose selectors and [compute derived data](http://gaearon.github.io/redux/docs/recipes/ComputingDerivedData.html).

* **To use `connect()`, the root component of your app must be wrapped into `<Provider>{() => ... }</Provider>` before being rendered.**

#### Examples

##### Inject just `dispatch` and don't listen to store

```js
export default connect()(TodoApp);
```

##### Inject `dispatch` and every field in the global state (SLOW!)
```js
export default connect(state => state)(TodoApp);
```

##### Inject `dispatch` and `todos`

```js
function mapState(state) {
  return { todos: state.todos };
}

export default connect(mapState)(TodoApp);
```

##### Inject `todos` and all action creators (`addTodo`, `completeTodo`, ...)

```js
import * as actionCreators from './actionCreators';

function mapState(state) {
  return { todos: state.todos };
}

export default connect(mapState, actionCreators)(TodoApp);
```

##### Inject `todos` and all action creators (`addTodo`, `completeTodo`, ...) as `actions`

```js
import * as actionCreators from './actionCreators';
import { bindActionCreators } from 'redux';

function mapState(state) {
  return { todos: state.todos };
}

function mapDispatch(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) };
}

export default connect(mapState, actionCreators)(TodoApp);
```

#####  Inject `todos` and a specific action creator (`addTodo`)

```js
import { addTodo } from './actionCreators';
import { bindActionCreators } from 'redux';

function mapState(state) {
  return { todos: state.todos };
}

function mapDispatch(dispatch) {
  return { addTodo: bindActionCreators(addTodo, dispatch) };
}

export default connect(mapState, mapDispatch)(TodoApp);
```

##### Inject `todos`, todoActionCreators as `todoActions`, and counterActionCreators as `counterActions`

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapState(state) {
  return { todos: state.todos };
}

function mapDispatch(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch)
  };
}

export default connect(mapState, mapDispatch)(TodoApp);
```

##### Inject `todos`, and todoActionCreators and counterActionCreators together as `actions`

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapState(state) {
  return { todos: state.todos };
}

function mapDispatch(dispatch) {
  return {
    actions: bindActionCreators({ ...todoActionCreators, ...counterActionCreators }, dispatch)
  };
}

export default connect(mapState, mapDispatch)(TodoApp);
```

##### Inject `todos`, and all todoActionCreators and counterActionCreators directly as props

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapState(state) {
  return { todos: state.todos };
}

function mapDispatch(dispatch) {
  return bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch);
}

export default connect(mapState, mapDispatch)(TodoApp);
```

##### Inject `todos` of a specific user depending on props, and inject `props.userId` into the action

```js
import * as actionCreators from './actionCreators';

function mapState(state) {
  return { todos: state.todos };
}

function mergeProps(selectedState, boundActions, props) {
  return Object.assign({}, props, {
    todos: selectedState.todos[props.userId],
    addTodo: (text) => boundActions.addTodo(props.userId, text)
  });
}

export default connect(mapState, actionCreators, mergeProps)(TodoApp);
```

## License

MIT
