<!-- {"created_at": "2014-11-19"} -->

JabJS - Introduction
====================

**JabJS** allows you to bind any JavaScript model to any DOM element using the following idiomatic format:

```js
jab.bind(model, 'property', domElement[s]);
```

Example:

```js
user = {name: 'John Lennon'}; //any JavaScript object
jab.bind(user, 'name', document.getElementByid('input')); //user is now binded (two-way) with #input
```

#### Demo Page
This tutorial is accompanied by [the JabJS demo page](http://www.sellarafaeli.com/jabjs-demo/index.html), which includes some HTML and the JabJS library. Every example given in this document can (and should) be executed on that page via the JavaScript console, to demonstrate JabJS usage.  

#### Bind To Element
Suppose you have an HTML element `<div id="div"></div>` and a POJS (plain old JavaScript object) 

```js
user = {name: 'John Lennon'}
```

After including JabJS, to bind the object to the element you would run:

```js
jab.bind(user, 'name', document.getElementById('div'));
```

After this, user.name would be *binded* to the `#div` element: that is, changing user.name would implicitly change the contents of #div. Try this for yourself in the JS console in the demo page. 

#### Two-Way Binding
For `div`s, binding is a one-way process: changes in the object update the HTML. However, for some elements we want a two-way binding: changes in the model should update the view (HTML), and changes in the view (HTML) should update the model. That way, for example, if we use an `<input id='input'>` element, we could access its value in the JS (without having to manually extract it every time we needed it). Form submissions can work immediately on the JS, without parsing the DOM upon submission. 

JabJS immediately performs two-way data-bindings on appropriate HTML elements, so suppose you have an HTML element `<input id="input">` and a JavaScript object `user = {name: 'John Lennon'}`. After including JabJS, to bind the object to the input you would run the same idiomatic JabJS binding:

```js
jab.bind(user, 'name', document.getElementById('input'));
```

Now user.name's value is binded to #input's value. Changes in one update the other. Try this for yourself in the JS console in the demo page. 

#### Three-Way Binding (Multiple Elements)
JabJS supports binding to multiple elements, enabling three-way (actually, *n*-way) changes between the model and each of the binded models. This is achieved using the idiomatic syntax, supplying an array of elements rather than a singular element.

```js
jab.bind(user, 'name', [document.getElementById('input'), document.getElementById('textarea'), document.getElementById('div')]);
```

Now #input, #textarea, #div and user.name are all bounded to each other. 

#### Select, Checkbox
Usage is as follows (assuming a `select` and `checkbox` elements with those Ids):

```js
user = {selectIndex=0; checked=false};
jab.bind(user, 'selectIndex', document.getElementById('select'));
jab.bind(user, 'checked', document.getElementById('checkbox'));
```

#### Special Bindings
By default, JabJS binds by value. In our examples, `user.name`'s value is input as the element's appropriate value. These are binded by using the following syntax (note the new fourth parameter):

```js
jab.bind(model, 'property', domElem, {opts: bindingName})
```

or

```js
jab.bind(model, 'property', domElem, 'bindingName')
```

JabJS ships by default with a few special bindings:

One special binding is the `show` binding:
```js
jab.bind(user, 'name', document.getElementById('input'), 'show')
```

which 'shows' the element (makes it visible) if and only if user.name is a truthy value.

Another default special binding is the `click` binding, which adds the value as an 'onclick' handler to the binded elements. Notice this requires the `value` to be a **function**.

```js
alertClicked = function(){ alert('clicked') };
user.clickHandler = alertClicked;
jab.bind(user, 'clickHandler', document.getElementById('input'), {func: 'click'});
```

#### Custom Bindings
Adding your own binding is as easy as pie, using the following pattern:

```js
myFunc = function(elem, value) {
  //do something with element and value
};
jab.bind(user, 'name', document.getElementById('input'), {func: myFunc});
```

The above binds `user.name` to #input, by running ``myFunc`` on the binded element (#input) and applying `myFunc` on the element, using the value in `user.name`. 

Let's observe a concrete example:

```js
setBorderWidth = function(elem, value) {
  elem.style.borderWidth = value+'px';
};
borderData = {width: 20};
jab.bind(borderData, 'width', document.getElementById('input'), {func: setBorderWidth});
```

Now, whenever ```borderData.width``` is set, ```setBorderWidth``` will be execute on the binded element (and will set its width).

#### Binding sub-elements
If your model is 

```js
user = {details: {name: 'Abraham', age: 900}};
```
You can jab.bind it by using the idiomatic syntax on whatever sub-object you wish to bind:
```js
jab.bind(user.details, 'name', document.getElementById('input'));
```

#### After-Hooks (reacting to changes in binded elements)
Sometimes binding your elements to the view is not enough - you want to perform some additional actions whenever they change. This is often referred to as [reactive programming](http://en.wikipedia.org/wiki/Reactive_programming). For example, consider two input elements binded each to its model, and a third element reflecting some computation on the values of the first two. (Say, an input for speed and an input for time, and a third <p> element to display distance covered.)

After-hooks are used by JabJS by supplying an `afterHook` callback function to the `opts` parameter. This callback function is called whenever the element is updated. The function is given the element's new value, but that's often not needed. For example:

```js
car = {speed: "10", time: "3"};
computeDistance = function(){ car.distance = car.speed * car.time; };
jab.bind(car, 'speed', document.getElementById('input'), {afterHook: computeDistance} ); //#input.value == 10
jab.bind(car, 'time', document.getElementById('textarea'), {afterHook: computeDistance}); //#textarea.value == 3
jab.bind(car, 'distance', document.getElementById('p')); //#p.innerHTML == 30
//now, changing the speed or time via the input elements or JS models will also update #p and distance.
car.speed = 20 //#input.value == 20, textarea.value == 3, #p.value == 60
```

#### Binding to JavaScript variables - Pure Reactive JavaScript

JabJS also enables you to performing binding on pure JavaScript objects, with no DOM elements. This can be used to hold computed variables of a user (composed of other variables). Along with DOM bindings, this can be utilized to bind computed, reactive properties to the DOM itself. 

Binding pure JS vars is done with the following syntax: `jab.bindVar(model, property/ies, callback)`. For example:

```js
man = {firstName: 'Bill', lastName: 'Clinton', fullName: ''};
jab.bindVar(man, ['firstName', 'lastName'], function() { man.fullName = man.firstName + " " +man.lastName } ); //whenever firstName or lastName change, fullName will also change...
jab.bind(man, 'fullName', div); //...and so will its binded view. 
```

#### Technical Points
* Stand-alone & dependency-free, simply include and run.
* Tiny: ~1.5K compressed. 
* Pure JS, creates bindings without changing HTML markup: Keep your JS out of your markup.
* Source code is easily readable and modifiable - understand exactly what is happening, customize to your own needs. 
* Orthogonal to other JS libraries - use it anywhere, with any other library or templating, without depending or modifying anything else. 

#### Etymology
A *jab* is a [type of punch](http://en.wikipedia.org/wiki/Jab) used in the martial arts.

#### Contact
* Source code is on [GitHub](https://github.com/SellaRafaeli/jabjs/).
* Demo page which works with all the examples in this tutorial is [here](/jabjs-demo/index.html).
* For any questions, help or pull requests please contact sella.rafaeli@gmail.com. 
