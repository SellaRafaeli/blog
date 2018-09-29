<!-- {"created_at": "2014-11-10"} -->

## Native JavaScript Data-Binding 

Two-way data-binding is such an important feature - align your JS models with your HTML view at all times, to reduce boilerplate coding and enhance UX. We will observe two ways of doing this using native JavaScript, with no frameworks - one with revolutionary technology (Object.observe), and one with an original concept (overriding get/set). Spoiler alert - the second one is better. See TL;DR at bottom.
 
### 1: Object.observe && DOM.onChange

*Object.observe()* is the [new kid on the block](http://www.html5rocks.com/en/tutorials/es7/observe/). This native JS ability - well, actually it's a *future* ability since it's only proposed for ES7, but it's already[!] available in the [current stable Chrome](http://kangax.github.io/compat-table/es7/#Object.observe) - allows for reactive updates to changes to a JS object. Or in simple English - a callback run whenever an object('s properties) change(s). 

An idiomatic usage could be:

~~~ javascript
log = console.log
user = {}
Object.observe(user, function(changes){    
    changes.forEach(function(change) {
        user.fullName = user.firstName + " " + user.lastName;         
    });
});

user.firstName = 'Bill';
user.lastName = 'Clinton';
user.fullName // 'Bill Clinton'
~~~

This is already pretty cool and allows reactive programming within JS - keeping everything up-to-date by *push*. 

But let's take it to the next level:

~~~ javascript
//<input id="foo">
user = {};
div = $("#foo");
Object.observe(user, function(changes){    
    changes.forEach(function(change) {
        var fullName = (user.firstName || "") + " " + (user.lastName || "");         
        div.text(fullName);
    });
});

user.firstName = 'Bill';
user.lastName = 'Clinton';

div.text() //Bill Clinton
~~~
*[JSFiddle](http://jsfiddle.net/v2bw6658/)*

Cool! We just got model-to-view databinding! Let's DRY ourselves with a helper function. 

~~~javascript
//<input id="foo">
function bindObjPropToDomElem(obj, property, domElem) { 
  Object.observe(obj, function(changes){    
    changes.forEach(function(change) {
      $(domElem).text(obj[property]);        
    });
  });  
}

user = {};
bindObjPropToDomElem(user,'name',$("#foo"));
user.name = 'William'
$("#foo").text() //'William'
~~~
*[JSFiddle](http://jsfiddle.net/v2bw6658/2/)*

Sweet! 

Now for the other way around - binding a DOM elem to a JS value. A pretty good solution could be a simple use of jQuery's *.change* (http://api.jquery.com/change/):

~~~ javascript
//<input id="foo">
$("#foo").val("");
function bindDomElemToObjProp(domElem, obj, propertyName) {  
  $(domElem).change(function() {
    obj[propertyName] = $(domElem).val();
    alert("user.name is now "+user.name);
  });
}

user = {}
bindDomElemToObjProp($("#foo"), user, 'name');
//enter 'obama' into input
user.name //Obama. 
~~~
*[JSFiddle](http://jsfiddle.net/v2bw6658/4/)*

That was pretty awesome. To wrap up, in practice you could combine the two into a single function to create a two-way data-binding:

~~~ javascript 
function bindObjPropToDomElem(obj, property, domElem) { 
  Object.observe(obj, function(changes){    
    changes.forEach(function(change) {
      $(domElem).text(obj[property]);        
    });
  });  
}

function bindDomElemToObjProp(obj, propertyName, domElem) {  
  $(domElem).change(function() {
    obj[propertyName] = $(domElem).val();
    console.log("obj is", obj);
  });
}

function bindModelView(obj, property, domElem) {  
  bindObjPropToDomElem(obj, property, domElem)
  bindDomElemToObjProp(obj, propertyName, domElem)
}
~~~

Take note to use the correct DOM manipulation in case of a two-way binding, since different DOM elements (input, div, textarea, select) answer to different semantics (text, val). Also take note that two-way data-binding is not always necessary -- "output" elements rarely need view-to-model binding and "input" elements rarely need model-to-view binding. But wait -- there's more: 

### 2: Go deeper: Changing 'get' and 'set' 

We can do even better than the above. Some issues with our above implementation is that using .change breaks on modifications that don't trigger jQuery's "change" event - for example, DOM changes via the code, e.g. on the above code the following wouldn't work:

~~~javascript 
$("#foo").val('Putin')
user.name //still Obama. Oops. 
~~~

We will discuss a more radical way - to override the definition of getters and setters. This feels less 'safe' since we are not merely observing, we will be overriding the most basic of language functionality, get/setting a variable. However, this bit of metaprogramming will allow us great powers, as we will quickly see. 

So, what if we *could* override getting and setting values of objects? After all, that's exactly what data-binding is. Turns out that using `Object.defineProperty()` [we can in fact do exactly that](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). 

We used to have the [old, non-standard, deprecated way](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/__defineGetter__) but now we have the new cool (and most importantly, standard) way, using `Object.defineProperty`, as so: 

~~~javascript 
user = {}
nameValue = 'Joe';
Object.defineProperty(user, 'name', {
  get: function() { return nameValue }, 
  set: function(newValue) { nameValue = newValue; },
  configurable: true //to enable redefining the property later
});

user.name //Joe 
user.name = 'Bob'
user.name //Bob
nameValue //Bob
~~~
OK, so now user.name is an alias for nameValue. But we can do more than just redirect the variable to be used - we can use it to create an alignment between the model and the view. Observe: 

~~~javascript
//<input id="foo">
Object.defineProperty(user, 'name', {
  get: function() { return document.getElementById("foo").value }, 
  set: function(newValue) { document.getElementById("foo").value = newValue; },
  configurable: true //to enable redefining the property later
});
~~~

`user.name` is now binded to the input `#foo`. This is a very concise expression of 'binding' at a native level - by defining (or extending) the native get/set. Since the implementation is so concise, one can easily extend/modify this code for custom situation - binding only get/set or extending either one of them, for example to enable binding of other data types. 

As usual we make sure to DRY ourselves with something like:

~~~javascript 
function bindModelInput(obj, property, domElem) {
  Object.defineProperty(obj, property, {
    get: function() { return domElem.value; }, 
    set: function(newValue) { domElem.value = newValue; },
    configurable: true
  });
}
~~~

usage:

~~~javascript
user = {};
inputElem = document.getElementById("foo");
bindModelInput(user,'name',inputElem);

user.name = "Joe";
alert("input value is now "+inputElem.value) //input is now 'Joe';

inputElem.value = 'Bob';
alert("user.name is now "+user.name) //model is now 'Bob';
~~~

*[JSFiddle](http://jsfiddle.net/v2bw6658/6/)*

Note the above still uses 'domElem.value' and so will still work only on `<input>` elements. (This can be extended and abstracted away within the bindModelInput, to identify the appropriate DOM type and use the correct method to set its 'value'). 

Discussion:

* DefineProperty is available in [pretty much every browser](http://kangax.github.io/compat-table/es5/#Object.defineProperty).
* It is worth mentioning that in the above implementation, the *view* is now the 'single point of truth' (at least, to a certain perspective). This is generally unremarkable (since the point of two-way data-binding means equivalency). However on a principle level this may make some uncomfortable, and in some cases may have actual effect - for example in case of a removal of the DOM element, would our model would essentially be rendered useless? The answer is no, it would not. Our `bindModelInput` creates a closure over `domElem`, keeping it in memory - and preserving the behavior a la binding with the model - even if the DOM element is removed. Thus the model lives on, even if the view is removed. Naturally the reverse is also true - if the model is removed, the view still functions just fine. Understanding these internals could prove important in extreme cases of refreshing both the data and the view.

Using such a bare-hands approach presents many benefits over using a framework such as Knockout or Angular for data-binding, such as:

* Understanding: Once the source code of the data-binding is in your own hands, you can better understand it and modify it to your own use-cases. 
* Performance: Don't bind everything and the kitchen sink, only what you need, thus avoiding performance hits at large numbers of observables. 
* Avoiding lock-in: Being able to perform data-binding yourself is of course immensely powerful, if you're not in a framework that supports that. 

One weakness is that since this is not a 'true' binding (there is no 'dirty checking' going on), some cases will fail - updating the view will not 'trigger' anything in the model, so for example trying to 'sync' two dom elements *via the view* will fail. That is, binding two elements to the same model will only refresh both elements correctly when the model is 'touched'. This can be amended by adding a custom 'toucher':

~~~javascript 

//<input id='input1'>
//<input id='input2'>
input1 = document.getElementById('input1')
input2 = document.getElementById('input2')
user = {}
Object.defineProperty(user, 'name', {
  get: function() { return input1.value; }, 
  set: function(newValue) { input1.value = newValue; input2.value = newValue; },
  configurable: true
});
input1.onchange = function() { user.name = user.name } //sync both inputs.
~~~

### TL;DR:

Create a two way data-binding between model and view with native JavaScript as such:

~~~javascript 
function bindModelInput(obj, property, domElem) {
  Object.defineProperty(obj, property, {
    get: function() { return domElem.value; }, 
    set: function(newValue) { domElem.value = newValue; },
    configurable: true
  });
}

//<input id="foo">
user = {}
bindModelInput(user,'name',document.getElementById('foo')); //hey presto, we now have two-way data binding.
~~~

Thanks for reading. Previously published on [JavaScript Weekly](http://javascriptweekly.com/issues/207), comments at [reddit](http://redd.it/2manfb) or at sella.rafaeli@gmail.com.