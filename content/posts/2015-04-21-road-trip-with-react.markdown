---
type: post
title: "Road Trip with React"
date: 2015-04-21 22:07:01 -0400
comments: true
share: true
tags: [Web, Javascript, React]
image:
  feature: /images/abstract-5.jpg
---

React is a javascript UI rendering engine for building user interfaces from Facebook and the Instagram team. Most likely use as the **V** in the **MVC** design pattern, React makes no assumptions about the rest of your technology stack. This is a short tutorial on how to create a React component.

<!--more-->

## Intoduction

### What is React?

>React is a library for building composable user interfaces. It encourages the creation of reusable UI components which present data that changes over time.
>
Traditionally, web application UIs are built using templates or HTML directives. These templates dictate the full set of abstractions that you are allowed to use to build your UI.
>
React approaches building user interfaces differently by breaking them into components. This means React uses a real, full featured programming language to render views, which we see as an advantage over templates for a few reasons:
>
* JavaScript is a flexible, powerful programming language with the ability to build abstractions. This is incredibly important in large applications.
* By unifying your markup with its corresponding view logic, React can actually make views easier to extend and maintain.
* By baking an understanding of markup and content into JavaScript, there's no manual string concatenation and therefore less surface area for XSS vulnerabilities.
>
We've also created JSX, an optional syntax extension, in case you prefer the readability of HTML to raw JavaScript.
>
-- [React blog]

### A Hello World example!

Look at the code below

{{< jsfiddle fiddle="ym2bqz78" tabs="js,html,css,result" >}}

Nothing unfamiliar, pure javacscript code with some fancy thing. You may notice some unwrapped **«html»** in the code

``` html
<|-- omitted code -->
return <div>Hello {this.props.name}</div>;
<|-- omitted code -->
```

You may say «the guy sucks, he can write a simple code right...» HA HA HA and no this is valid code in React called JSX Syntax.

### JSX Syntax

[JSX] is a JavaScript syntax extension that looks similar to XML. You can use a simple JSX syntactic transform with React.

### Why JSX?

You don't have to use JSX with React. You can just use plain JS. However, we recommend using JSX because it is a concise and familiar syntax for defining tree structures with attributes.

It's more familiar for casual developers such as designers.

XML has the benefit of balanced opening and closing tags. This helps make large trees easier to read than function calls or object literals.

It doesn't alter the semantics of JavaScript.

## A little bit more funny _Hello World_ component

Lets make an Hello World widget with:

- A Logo represented by an image
- A label: to display Hello World **your name**
- An input box: to get the user input

So lets decompose this component with a picture:

{{< figure class="center" src="/images/preview-react-helloworld.png" alt="React Hello World" >}}

{{< alert class="alert-warning" >}}
This is a naive exanple don't take to much attention to the naming convention of the components.
{{< /alert >}}

So Lets go !!!

First clone the Github repo on your computer

{{< figure class="btn btn-lg btn-primary fa fa-github fa-2x pull-left" link=https://github.com/guillaumemaka/HelloWorldReact" title="React example<br>Repository" >}}

Create an `index.html` with the content of the following snippet  

```html examples/starter/index.html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv='Content-type' content='text/html; charset=utf-8'>
    <title>Basic Example with JSX</title>
    <link rel="stylesheet" href="../shared/css/base.css" />
  </head>
  <body>
    <h1>Basic Example with JSX</h1>
    <div id="container">
      <p>
        To install React, follow the instructions on
        <a href="http://www.github.com/facebook/react/">GitHub</a>.
      </p>
      <p>
        If you can see this, React is not working right.
        If you checked out the source from GitHub make sure to run <code>grunt</code>.
      </p>
    </div>
    <h4>Example Details</h4>
    <p>This is written with JSX and transformed in the browser.<p>
    <p>
      Learn more about React at
      <a href="http://facebook.github.io/react" target="_blank">facebook.github.io/react</a>.
    </p>
    <script src="../shared/thirdparty/es5-shim.min.js"></script>
    <script src="../shared/thirdparty/es5-sham.min.js"></script>
    <script src="../shared/thirdparty/shell-polyfill.js"></script>
    <script src="../../build/react.js"></script>
    <script src="../../build/JSXTransformer.js"></script>
    <script type="text/jsx">

    </script>
  </body>
</html>
```

### The Logo component

Lets build the `Logo` component that consist of rendering an `<img />` tag

```javascript
// 1. 2.
var Logo = React.createClass({
  // 3.
  propTypes:{
    src: React.PropTypes.string.isRequired
  },
  // 4.
  render: function(){
    // 5.
    return(
        <img className="center" {...this.props} />
      );
    }
});

```

1. First we create a local variable `Logo`
2. Next we pass some methods in a JavaScript object to `React.createClass()` to create a new React component.
3. Next we create a `propTypes` object to handling validation
4. Next we create the most important method for a component: `render`
5. Next we return an image tag in JSX format with some attributes:
 * `className`: represent the `class` attribute in HTML, because JSX is written directly in the javascript code you can't use the `class` because is a reserved keyword
 * `{...this.props}`: this is a feature in React called _**[Transffering Props]**_. Sometimes is tedious to pass every property along, so the `...<other>` say to React to merge all the property in `other` to the current component property

{{< alert class="alert-info" >}}
**Note:** Order matter, `{...this.props}` is the last attribute that means all attribute before will be overwritten. You will see another example for the `Input` component.
{{< /alert >}}

{% comment %}
{{< alert class="alert-warning" >}}
**Warning:** The _**Transfering Props**_ use the `Destructuring assignment` feature in _**[ES6]**_
{% endalert%}
{% endcomment %}

### The Label component

The `label` component consist of rendering a `<p></p>` element

```javascript
// 1.
var Label = React.createClass({
  // 2.
  render: function() {
    // 3.
    return (
      <p className="default-label" {...this.props}>
        Hello World <span className="name">{ this.props.name}</span>
      </p>
    );
  }
});
```

No big change from the previous section
1. Create a Label component
2. Implementing the `render` method
3. Return the UI composing the element


### The Input component

The `Input` component consits of rendering an `<input />` element

```javascript
var Input = React.createClass({
  render: function() {
    return (
      <input
        className="default-input"
        placeholder="Enter your name"
        {...this.props}
        type="text" />
    );
  }
});
```
Anything new but look at the `{...this.props}`. In the previous component the statement is the last attribute. Why ? In fact transfering property merge all attribute of an element if they are define when instanciating the component. Putting `{...this.props}` before `type="text"` ensure that the input is always a text input even if the component redefined the property.

### Putting all together: Wrapping the Logo, Label and the Input

Lets wrap ours components

```javascript
var HelloWidget = React.createClass({
  // 2.
  getInitialState: function() {
    return {
      name: ''
    };
  },
  // 2.
  handleChange: function(e) {
    // 3.
    this.setState({
      name: e.target.value
    });
  },
  // 4.
  render: function() {
    // 8.
    return (
      <div className="widget">
        // 5.
        <Logo src="http://goo.gl/fx5Zwn" />
        // 6.
        <Label className="green-label" name={this.state.name} />
        // 7.
        <Input onChange={this.handleChange} />
      </div>
    );
  }
});
```

Some new stuff:

1. A new method: `getInitialState`. In React component are state machines. React thinks of UIs as simple state machines. By thinking of a UI as being in various states and rendering those states, it's easy to keep your UI consistent.

{{< alert class="alert-info" >}}
In React, you simply update a component's state, and then render a new UI based on this new state. React takes care of updating the DOM for you in the most efficient way.

So we declare `name` as a state, now whenever name change, the UI is re-rendered to reflect the new state.
{{< /alert >}}

2. We declare the `handleChange` method that will update the state whenever you type in the `Input` component.
3. Here its where the magic happen, `this.setState()` will set the new state of the component.
4. We implement the render method
5. We declare our `<Logo />` component, and pass the `src` property to set the image source url.
6. We declare our `<Label />` component, and pass some styling and binding the `name` property to `{this.state.name}`.
7. We declare our `<Input />` component, we bind our `onChange` property event handler to our component `handleChange` method.
8. Finally we return our component markup.

### Render the component

```JavaScript
React.render(<HelloWidget />, document.getElementById('container'));
```

Finally we render the component inside our `<div id=container></div>` declared element in the `index.html` file.

### The result

{{< jsfiddle fiddle="20mbecfu" tabs="result" >}}

## ES6 ?

Since ``React 0.13.1 beta 1`` React ship with ES6 support.

>In React 0.13.0 you no longer need to use React.createClass to create React components. If you have a transpiler you can use ES6 classes today. You can use the transpiler we ship with react-tools by making use of the harmony option: jsx --harmony.

### Refactor our component to ES6 classes

So the code will look like this:

{% jsfiddle jwm6k66c js %}


## Where to go from here

Read the documentation, fork the jsfiddle code snippets and play with them. Visit the links in the resources section below.

## Resources

- [React website][react]
- [Egghead React lessons][egghead]  
- [Github source code repository]


[Github source code repository]: https://github.com/guillaumemaka/HelloWorldReact "Github Repo"
[JSX]: https://facebook.github.io/jsx/ "JSX"
[react]: https://facebook.github.io/ "React"
[egghead]: https://egghead.io/technologies/react
[Transffering Props]: https://facebook.github.io/react/docs/transferring-props.html
[ES6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
[React blog]: http://facebook.github.io/react/blog/2013/06/05/why-react.html
