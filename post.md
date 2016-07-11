##  Introduction
This is a quick reference for React UI Component development & testing.  
Goes over basic patterns in React to get most component functionality done.  Further, we illustrate a way to test each of these patterns.  The patterns covered are:

1. Basic Components
* Passing in Props
* Browser Events and setState
* Communicating with Server

## 1. Components

We name components using `CamelCase.`  The extension is `.jsx`  Our example will be `MyComponent.jsx`

Every React component starts the same way.  Copy and paste the following into a new file.

### Component Starter

[Live Example](http://codepen.io/lawwantsin/pen/EyKgoz)

```javascript
// MyComponent.jsx
import React from 'react';

export default class MyComponent extends React.Component {

  constructor(props, context) {
    super(props, context);
    this.state = {
    }
  }

  render() {
    return (
                const label = this.props.label
  		<div className="my-component">{label}</div>
  	)
  }
}
```

From the top, we have an `import` call.  This is ES6 syntax to include other modules in your code.  In this case, we're importing only `React`.  `export default class` follows the same logic as import.  It's an E6 convention for declaring a class.  Just like a ruby class, JS classes namespace their functions and can inherit functionality from a parent.  In this case, `MyComponent` inherits from `React.Component`.

The constructor function is like `def initialize` in Ruby. An ES6 convention to set up the class with initial values.  Constructors always have the `super(props, context)`  We can also set up initial state by setting `this.state` to an object.  This is the only place where you'll ever want to use the object itself instead of `setState()` to change the state.

ES6 has a variable keyword `const` which is like `var` in ES5.  So whenever you see const, it's just our way to set a variable.  Except it will throw a TypeError if you try and change it's value.  `const` are set once.  This is an attempt by the javascript spec to move towards more [functional programming](https://www.sitepoint.com/introduction-functional-javascript/)techniques and [more immutable variables](https://mathiasbynens.be/notes/es6-const). 

`Render()/return()` is the last and most important part of a React component.  Every component has a `render` function and every function in javascript needs a `return` statement.  In the case of render, we want to return some JSX, which is React's flavor of HTML.  I know, gross.  HTML in the JS...haven't we been here before?  It's different this time.  In the end it's all compiled into javascript.  JSX is a more human readable facade.

[Babel](https://babeljs.io), the JS transpiler, does a JSX pass first.  This turns JSX into the functions that look like this.

```javascript

// From JSX
<div style={textAlign: 'left'}>
  <h1></h1>
  <p></p>
</div>

// To Javascript in the browser
React.createElement('div', {style: {textAlign: 'left'}}, [ 
   React.createElement('h1', {...}),
   React.createElement('p', {...})
]);
```

We can code in the JS version, but it isn't as sexy as JSX, so we'll stick with something we can all read.  The point is, it's not HTML, it just looks like it.  We're not concatenating strings to build up DOM elements.  We're transpiling anything in the file that starts with `<abc` into a function that returns a Virtual DOM representation of the application.

And once we have a Virtual DOM representation, not only does React take care of updating the real DOM for us, we also have painless, super fast frontend testing without fixtures or headless browsers.  Finally.

### Test Starter with Enzyme & Chai 

Corresponding test files have the same name as the component except the extension is `.test.js` instead of `.jsx`  We use mocha to run them with `npm test`

```javascript
// MyComponent.test.js
import React from 'react';
import { mount, shallow } from 'enzyme';
import { expect } from 'chai';
import MyComponent from './MyComponent';

describe('<MyComponent />', () => {

  it('Renders the proper class', () => {
    const component = shallow(<MyComponent />);
    expect(component.find('.my-component').length).to.equal(1);
  });

  it('Renders the label prop', () => {
    const component = shallow(<MyComponent label="Hello" />);
    expect(component).to.have.text('Hello');
  });

});

```

At the top of the file, we're importing React and the component itself.  
We're also importing `shallow` and `mount` from `enzyme`.  
Enzyme is a test helper that renders the Virtual DOM and lets us use a jQuery 
like syntax to find elements in the Virtual DOM and act on them.  
Browser events and such.  Mount is like the more expensive version of `shallow`  
It renders all children of the component, while `shallow` only renders one 
level of depth, so the unit tests can be more decoupled.  
There's also `render` which returns actual DOM for deeper testing.  
Shallow works for most everything.

The `describe/it` syntax is much like BDD.  
New ES6 arrow `() => {}` functions like ruby `do blocks`.  The `expect` line comes from the [Chai](http://chaijs.com/) assertion library.  It has a nice [chaining syntax](http://chaijs.com/api/bdd/#method_language-chains) that makes the tests really read nicely.  In this example we pass in a label prop, "Hello"  And we check that the component is actually outputting the passed in value with the `to.have.text()` function.  `chai` and `enzyme` have a lot of matchers for tests. 

### Composability

React components can act like any XML tag and be nested.  
The only thing you need to know is that `{this.props.chlidren}` 
is equivalent to `<%= yeild %>` in ERB.  Notice the end tag has a forward slash 
at the front.  props.children returns an Array of child React elements.

```javascript
// Usage

<Parent>
 <Child />
</Parent>

// Component Definition
export default class Parent extends React.Component {

  constructor(props, context) {
    super(props, context);
  }

  render() {
    return (
      <div className="parent">
        {this.props.children}
      </div>
    }
  }
}
```

### Testing for the Presence of Child Components

If we use `shallow` and render one level, that means we can see the immediate 
children as their HTML representations.  So all we need to do to check that the children are being passed thru is `find` by their JS component name ie `MyComponent` or by the CSS selector.

```javascript
// MyComponent.test.js
import React from 'react';
import { mount, shallow } from 'enzyme';
import { expect } from 'chai';
import Parent from './Parent';
import Child form './Child';

describe('<Parent />', () => {

  it('Thinks of the children using the CSS selector', () => {
    const component = shallow(<Parent><div id="child"></div></Parent>);
    expect(component.find('#child').length).to.equal(1);
  });

  it('Thinks of the children using component name', () => {
    const component = shallow(<Parent><Child /></Parent>);
    expect(component.find(Child).length).to.equal(1);
  });

});

```

## 2. Passing in Props

If the metaphor for a component is a `function` then it's props are like it's `signature`.  The part between the parenthesis.  Props are read only and they allow components communicate with each other.  They can be any structure in javascript.  String, Arrays, Objects, Functions, etc.  We use props to pass in data or functions that will be called by the child component, without it ever knowing what that data is or what that function does.  In this way, we make "smart Parents" and "dumb Children."

The analogy in HTML is the attribute and they can include normal html attributes.  The only gotchas are use `className` instead of `class` and `labelFor` instead of `for` as they are both reserved words in javascript.  So, don't forget to use `className` for your CSS classes.  It fails silently and is usually the cause of an element not being styled.  `class=` should never appear in JSX.

```javascript
// Usage
<Image src="image.jpg" className="photo" alt="SEO Descriptor" />

// Component Definition
export default class Image extends React.Component {

  constructor(props, context) {
    super(props, context);
  }

  render() {
    const cdnPath = "//s3.amazonws.com/bucket/";
    const { src, ...otherProps } = this.props;
    const cdnSrc = cdnPath + src;
    const styles = { 
      float: 'left',
      height: '100px',
    };
    return (
       <img style={styles} src={cdnSrc} {...otherProps} />
    )
  }
}
```

In the above example we are creating a simple Image component that adds a path to our CDN in the render function. The value is placed within the JSX using single brackets. `{}`

We use the ES6 splat `{...otherProps}` to indicate any additional stuff in the `otherProps` object we'd like to pass thru to the `<img>` tag.  In the above example `className` and `alt` are part of otherProps.

The `style` attribute is special as well, in that it must be passed in as an object, as shown above.  Not a string.  2 word CSS rules are camelCased.

## 3. Browser Events and setState

In React, events are declared on the JSX itself, with `onEventName.`  React uses a cross-browser [synthetic event system](https://facebook.github.io/react/docs/events.html). The handling function is either passed in as a prop from a parent or declared on the component itself.

### Passing in Event handlers from a Parent Component
[Live example](http://codepen.io/lawwantsin/pen/ZOWpJq/)

```javascript
// Usage
<Button onClick={this.props.clickHandler} />

// Child Component Definition
export default class Button extends React.Component {

  constructor(props, context) {
    super(props, context);
  }

  render() {
    return (
       <a className="button" onClick={this.props.onClick}>{this.props.label}</a>
    )
  }
}

// Parent Component Definition

import Button from '../button/Button';
export default class ButtonParent extends React.Component {

  constructor(props, context) {
    super(props, context);
    this.state = { numClicks: 0 }
    this.clickHandler = this.clickHandler.bind(this);
  }

  clickHandler() {
    this.setState({
      numClicks: this.state.numClicks + 1
    });
  }

  render() {
    return (
      <div class="counter">
       <p>Clicked {this.state.numClicks} times</p>
       <Button onClick={this.clickHandler} label="Click Me!" />
      </div>
    )
  }
}
```

In the above example, we import the Button component for use in the parent's render function.  Button gets a prop called onClick (but can be named anything).  That props's value is the `clickHandler` function in the ButtonParent class.  In this way we allow for dumb or "pure" components like `Button` to not have to define their own click handlers or labels, so they can be used for different purposes and still deliver consistent results and remain testable.

This is the preferred way to handle events and state in React. The top most parent passes in all the handling functions as props that the children call, keeping the state as high up in the hierarchy as possible.

### Handling Events on the Component Itself

Sometimes you just need to call a function on the component itself on some browser event.  Like an accordion widget that controls when it's collapsed or open via a user's click.  It works the same way.

```javascript
// Usage
<Accordion collapsed={false} />

// Component Definition
export default class Accordion extends React.Component {

  constructor(props, context) {
    super(props, context);
    this.state = { collapsed: props.collapsed };
    this.togglePanel = this.togglePanel.bind(this);
  }

  togglePanel(event) {
    console.log(event.target);
    this.setState({
      collapsed: !this.state.collapsed
    });
  }

  renderPanel(classes) {
    return <div className={classes}></div>
  }

  classes() {
    const collapsed = (this.state.collapsed ? 'accordion__panel--collapsed' : '');
    return "accordion__panel " + collapsed;
  }

  render() {
    const label = (this.state.collapsed ? 'Open' : 'Close');
    const classes = this.classes();
    const panel = this.renderPanel();
    return (
      <div className="accordion">
        <a className="accordion__link" onClick={this.togglePanel} />{label}</a>
        {panel}
      </div>
    )
  }
}
```

In this example Accordion handles it's own clicking with `this.togglePanel()`.  We `setState` and reverse the last collapsed state from true to false.  `state.collapsed` is used in the render function to calculate the label and the classes every time `setState` re-runs the `render`.  

This is a crucial pattern in React.  If it can be calculated from read only props and state, then don't use a new variable.  Instead calculate it above the `return()` in the render function.  

Variables can be set and JSX can be returned from other function besides render, as long as it's called from within render.  In this example `renderPanel` is a function that returns  a JSX div.  Use this pattern to break up your component and make them more readable.

There is one gotcha!  Because of how scoping is different in ES6 when translated to ES5 using Babel, if you don't `bind` the event handling function to the correct `this` scope, you'll get an error along the lines of `Can't call setState of undefined`.  This is only for event handling functions.  Functions that just return HTML do not have to be `bind`ed in the constructor.

You'll also notice you can pass in the event as the first argument and can call `event.target` to get the HTML node that was clicked in the browser.  Useful for getting an input's value.

### Faking User Interaction Events with Enzyme

```javascript
import React from 'react';
import { mount, shallow } from 'enzyme';
import { expect } from 'chai';
import Accordion from './Accordion';

describe('<Accordion />', () => {
  
  it('Collapses an open panel', () => {
    const component = shallow(<Accordion collapsed={false} />);
    expect(component).to.have.state('collapsed', false);
    component.find('.accordon__link').simulate('click');
    expect(component).to.have.state('collapsed', true);
    expect(component.find('.accordion__panel')).to.have.class('accordion__panel--collapsed');
  });

});
```

Here we're pulling in the Accordion component and `shallow` rendering them with `enzyme` with the collapsed prop set to `false`.  The beauty of Props (and functional programming in general) is it allows us to set up our component's state, so we know that the click actually changes the state and adds a new class.  The collapsing animation itself happens in the CSS of that class.

## 4. Communicating with the Server

Sometimes we need to send to the server on an event like a `keyPress.`  React handles this like any event with a function.

When a component loads for the first time, it may require data on the fly.  The component can [fetch this data](https://facebook.github.io/react/tips/initial-ajax.html) itself in the `componentDidMount` function.  Which is analogous to the `$(document).ready()` function in jQuery.  This function can also be used to load legacy jQuery code that will work on React code. (though React will not manage it for you on extensive re-renders).

```javascript
export default class Search extends React.Component {

     constructor(props, context) {
       super(props, context);
       this.state = {
         results: []
       }
       this.fetchResults = this.fetchResults.bind(this);
     }

    fetchResults(event) {
      const http = new XMLHttpRequest({ responseType: "json" });
      http.open("GET", '/search?query=' + event.target.value, true);
      http.onload = () => {
        const resultsArray = JSON.parse(http.responseText);
        this.setState({results: resultsArray});
      };
      http.send();
    }

    render() {
      return (
        <div className="search">
          <input type="text" className="search__input" 
                 onKeyPress={this.fetchResults} />
        </div>
      )
    }
}

```

In the above example we want to make a call to the server every time the user presses a key in an input.  fetchResults uses a plain ol XMLHttpRequest to quickly get the job done, tho React will work with anything, `fetch` `$.ajax()` or more involved stores like [Redux](http://redux.js.org/).  For most things, the less external dependencies you can use the better.  XMLHttpRequest is [crossbrowser](http://caniuse.com/#search=XMLHttpRequest) as of IE 8. And their are `fetch` [polyfills](https://github.com/github/fetch).

### Mocking XMLHttpRequest with Sinon

We don't have to actually use a server when testing requests.  That would be too slow.  Mocking it out is much easier.  We've been using Sinon to recreate the request in our tests.

```javascript
import React from 'react';
import { mount, shallow } from 'enzyme';
import { expect } from 'chai';
import sinon from 'sinon';
import Search from './Search';

describe('<Search />', () => {

  describe('onKeyPress', () => {

    var xhr, requests = [];

    before(() => {
      xhr = sinon.useFakeXMLHttpRequest();
      xhr.onCreate = (req) => { requests.push(req) };
    });

    after(() => {
      xhr.restore();
    });

    it('fetches to the Server', () => {
      requests = [];
      const component = shallow(<Search />);
      component.find('.search__input')
               .simulate('keyPress', {target: {value: 'test'}});
      expect(requests.length).to.equal(1);
      expect(requests[0].url).to.equal('/search?query=test');
    });
  });
});
```

In the above example we're setting up the mock `sinon.useFakeXMLHttpRequest()` in a before block.  Which work the same as Minitest or rSpec.  `before` runs once for the describe `beforeEach` runs before every `it`.  Same with after.  And in this case we're cancelling the mock to clean up after ourselves.

The way that it works is, every time our component tries to call `XMLHttpRequest` it is intercepted by our mock in the `xhr.onCreate` function which adds the request to an array.  We can then use the request in our `expect`.

In this case we test if a request was created so the array length = 1.  And we inspect that first request url to make sure it includes the input from our component.  In this case "test."

## Running Your Tests

From the command line.  At the root of the project.  `cd client`  then run `npm test` to run once or `npm run watch` to watch the test files for changes and rerun.

## Conclusion

This should serve as a reference for the most common basic patterns in React.  Props and state, events and server requests, each with a way to test.  Tests run extremely quickly, so more tests will not be expected to make the suite noticeably slower.  With React, enzyme and sinon, tests run at about 1 per 2 milliseconds.  So 500 tests per second.

Please add to and change this document freely or reach out to **@lawwantsin** on github **@law** on Slack for more clarification on these patterns.

## References

* [React Docs](https://facebook.github.io/react/)
* [Forms](https://facebook.github.io/react/docs/forms.html)
* [Enzyme Docs](http://airbnb.io/enzyme/)
* [Sinon Docs](http://sinonjs.org/)
* [Chai Docs](http://chaijs.com/)
* [Mocha Docs](https://mochajs.org/)
* [Testing How to](https://medium.freecodecamp.com/react-unit-testing-with-mocha-and-enzyme-77d18b6875cb)
