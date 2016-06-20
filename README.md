# Meteor/React Pre-Bind

Automatically binds methods defined within a component's Class to the current object's lexical `this` instance (similarly to the default behavior of React.createClass).

### Description

As you probably know, React ES6 class methods are not automatically bound to `this` in the same way that they are when using the classic `React.createClass` method. This led many people to bind their methods at the time of passing them as props, e.g.:

```jsx
<div onClick={this.doSomething.bind(this)} />
```

But, in some situations, this can be problematic as `someFunc.bind(this) !== someFunc.bind(this)`, and you may experience [needless re-rendering](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f).

The solution is to "pre-bind" your class methods:

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    
    this.method1 = this.method1.bind(this);
    this.method2 = this.method2.bind(this);
    // etc
  }
  // ...
```

Of course, this can become tedious, so it makes sense to have a single function to handle this for you.

### Installation

To install the stable version:

```
npm install --save meteor-react-prebind
```

Then import it:

```js
import prebind from 'meteor-react-autobind';
```

### Usage

You will generally call prebind from the class constructor, passing the `this` context:

```js
constructor(props) {
  super(props);
  prebind(this);
}
```

Prebind is smart enough to avoid binding React related methods (such as render and lifecycle hooks).

You can also explicitly specify certain methods to exclude from binding:

```js
constructor(props) {
  super(props);
  prebind(this, {
    wontBind: ['leaveAlone1', 'leaveAlone2']
  });
}
```

Or specify the only methods that should be auto-bound:

```js
constructor(props) {
  super(props);
  prebind(this, {
    bindOnly: ['myMethod1', 'myMethod2']
  });
}
```

Naturally, `wontBind` and `bindOnly` cannot be used together.

### Example

```jsx
import React, { Component } from 'react';
import prebind from 'meteor-react-autobind';

class Clicker extends Component {
  constructor(props) {
    super(props);

    prebind(this);
    this.state = {
      clickCounter: 0,
    };
  }

  increment() {
    this.setState({
      clickCounter: this.state.clickCounter + 1,
    });
  }

  // React's render and lifecycle hooks aren't bound

  componentDidMount() {
    console.info('Component is mounted.');
  }

  render() {
    return <div>
      <h1>Number of clicks: {this.state.clickCounter}</h1>
      <button onClick={this.increment}>Increment Counter</button>
    </div>
  }
}

```

### Prior Art

Thanks to Cássio Souza for [react-autobind](https://github.com/cassiozen/React-autobind), of which this project is a fork.

### Why Fork?

`react-autobind` worked well for the most part, except that when using higher-order functions (such as Radium) with Meteor/React, I noticed methods bound with autobind would stop working in production. There's something with Meteor's production build process, in combination with higher-order functions, that wraps the React class too deeply for autobind to find it. This package addresses that issue.