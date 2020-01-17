---
layout: post
title: 'React by Example: Stateless Functional Components'
date: '2016-05-24 02:21:10'
tags:
- react
- javascript
---

In v0.14 the React team introduced [stateless functional components](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions). They are implemented by functions that take a props argument and return JSX. They're simpler and offer performance benefits. The drawbacks are the lack of state and the inability to interact directly with the DOM.

#### Icon Component

In a typical app, most React components will be presentational only and should be stateless functional components. A good example is an Icon component I created to encapsulate [Font Awesome icons](https://fortawesome.github.io/Font-Awesome/icons/). The component renders an `<i></i>` HTML element with the proper Font Awesome classes. For example, the JSX declaration `<Icon name="envelope" fixedWidth />` renders the HTML `<i class="fa fa-envelope fa-fw"></i>` which displays an envelope icon with a fixed width.

<script src="https://gist.github.com/joebuschmann/00f5f5068b1bbcb84361d73d51139f45.js"></script>

The Icon source code is organized as a Common JS module. At the top, I import a few dependencies including React. The component is the `Icon` function with a single props argument. In the envelope example, props would contain two properties `name` and `fixedWidth` with the values *envelope* and *true* respectively.

```
var React = require('react');
var filterKeys = require('./filterKeys');
var classnames = require('classnames');

// The component is implemented as a function.
var Icon = function (props) {
    // Snip
};
```

The return value is a JSX snippet. [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) is React's way of describing how the component should be rendered. It's similar to HTML, but it needs to be [translated to Javascript](https://babeljs.io/blog/2015/02/23/babel-loves-react) before it will work in the browser.

```
return <i {...elementProps} className={className}></i>;
```

After the function declaration, a property named `propTypes` is added to the function. More on `propTypes` in the next section.

Finally, the `Icon` function is exported via `module.exports`.

```
module.exports = Icon;
```

#### PropTypes

React has the concept of `propTypes` to handle prop validation. [PropTypes](https://facebook.github.io/react/docs/reusable-components.html#prop-validation) ensure the correct prop values are being passed to a component. If a prop is of the wrong type or a required one is missing, React will log a warning.

In the case of the Icon component, the `propTypes` specification is added as a property of the component function. It is an object whose properties export validators from the React.PropTypes namespace.

```
Icon.propTypes = {
    name: React.PropTypes.string.isRequired,
    spin: React.PropTypes.bool,
    fixedWidth: React.PropTypes.bool,
    className: React.PropTypes.string
};
```

The Icon component has one required property `name` and three optional properties `spin`, `fixedWidth`, and `className`. The props `name`, `spin`, and `fixedWidth` are translated into Font Awesome CSS classes to provide the icon type, spin animation, and a fixed width. They are composed using [Classnames](https://www.npmjs.com/package/classnames), a simple utility for joining CSS classes.

```
    var className = classnames('fa', 'fa-' + props.name, props.className, {
        'fa-spin': props.spin,
        'fa-fw': props.fixedWidth
    });
```

#### Filtering PropTypes

The available props aren't limited to those defined in `propTypes`. Consumers of `Icon` can supply extra props that are applied directly to the `<i></i>` element using the [spread operator](https://facebook.github.io/react/docs/jsx-spread.html).

```
<i {...elementProps} className={className}></i>
```

Before they are added to the element, an additional step is needed to filter out the props specified by `propTypes`. This prevents name, spin, and fixedWidth from being included in the final HTML output as attributes.

```
var elementProps = filterKeys(props, Icon.propTypes);
```

The custom `filterKeys` method takes the keys defined by `propTypes` and removes them from props. It returns a new object containing only the items that apply to the `<i></i>` element. This approach de-clutters the rendered HTML and cuts down on the final script size.

<script src="https://gist.github.com/joebuschmann/4a74d87c3caade460eb2cf4e056fcc23.js"></script>

#### Component Composition

Now we have a component that renders icons, but how can it be used? React components can either be rendered directly into the HTML page like below:

```
// Rendering the Icon directly into the page isn't very useful.
ReactDOM.render(<Icon name="envelope" />, document.getElementById('icon'))
```

...or they can be composed into more complex components. In the example below, a Button component displays a contextual icon on the button itself. The `icon` prop indicates which image to render, and `waiting` displays a spinner.

<script src="https://gist.github.com/joebuschmann/a66349ec3bff45a567cdf0a50e66f5b6.js"></script>

The icon is conditionally created and saved to a local variable depending on the values of the `waiting` and `icon` props. Notice that JSX can be saved to a variable just like any other Javascript expression.

```
var icon = null;

if (props.waiting) {
    icon = <Icon name="spinner" fixedWidth spin style={iconStyle} />;
} else if (props.icon) {
    icon = <Icon name={props.icon} fixedWidth style={iconStyle} />;
}
```
Next the icon is added to the JSX returned by the function. The value of the icon variable may be null, but that's fine. React will ignore any null or undefined values.

```
return <button {...elementProps} type="button" onClick={onClick}>{icon}{props.children}</button>;
```

The snippet below creates a button with an envelope icon and sets the value of `waiting` to `this.state.waiting`. This state value comes from a stateful component that's hosting the button. As the parent component's value for `this.state.waiting` changes, the button will be re-rendered with the new prop values.

```
var Button = require('button');

<Button className="btn btn-primary" onClick={this.send} icon="envelope" waiting={this.state.waiting}>Send Message</Button>
```

#### Wrapping Up

Icon and Button are simple and carry no internal state. That makes them ideal candidates to be implemented as stateless functional components. They can be used to build more complex components which in turn can be composed into ever more complex components. Eventually state is thrown into the mix, and you have a React application.
