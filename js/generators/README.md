# Javascript Generators

** A Brief Introduction**

Generators are a new feature introduced in ES6. They're functions that can be paused at any time during their execution (an arbitrary amount of times), allowing other code to run during these paused times. Generators can then be un-paused at a later point in time and continue where they left off.

A generator function can only be paused from within itself, it cannot be paused from the outside. You pause a generator function by using the new `yield` keyword from within the function body. This new `yield` keyword can be used an arbitrary amount of times from within a generator function. Once a generator is paused though, it cannot un-pause itself; something from the outside has to tell it to un-pause. 

Everytime a generator is paused, you can have it send messages out with each `yield`. Everytime a generator is un-paused you can pass messages into it as well. 

Generators therefore allow you to control the execution of the function as it progresses, as well as, 2-way messaging into and out of them. 

Generator functions are written like normal functions except they have an asterisk (`*`). 

*Example*:

```
function *foo() {
  // ...
}
```

or 

```
function* foo() {
  // ...
}
```

The way in which we control the execution of a generator function from the outside is by constructing a "*Generator Iterator*". Building on the example above, this is how we would construct an iterator. 

*Example*:

```
function *foo() {
  // ...
}

var myGenerator = foo();
```

It's important to note that unlike regular functions, calling generator functions `()` won't execute any of the code within the function; it simply constructs the iterator. 

Now to start executing the code within our generator or to start *iterating on our generator function*, we simply call `next()` on it. Each time we call `next()`, the iterator returns an object with a `value` property for the yield-ed out value (if any) and a `done` property which returns a boolean indicating whether the function has completed or not. 

*Example*:

```
function *foo() {
  console.log('Before pausing the function');
  var x = 2 + (yield 3)
  console.log('After un-pausing the function');
  console.log(x)
}

var myGenerator = foo();

myGenerator.next()
// Before pausing the function
// { value: 3, done: false }

myGenerator.next(2)
// After un-pausing the function
// 4
// { value: undefined, done: true }
```

Here's a more complicated example borrowed from [Kyle Simpson](http://davidwalsh.name/es6-generators) in a more in-depth series of posts about ES6 generators (definitely check it out). 

*Example*:

```
function *foo(x) {
    var y = 2 * (yield (x + 1));
    var z = yield (y / 3);
    return (x + y + z);
}

var myGenerator = foo(5); // You can pass parameters into generators like in normal functions

myGenerator.next(3)
// { value: 6, done: false }

myGenerator.next(12)
// { value:8, done:false }

myGenerator.next(13)
// { value:42, done:true }
```

Notice how the first time we call `next()`, the value 6 is returned (5 + 1). But what happened to the 3? When we passed in the 3, there was no `yield` expression to to receive what we passed in. The first `next()` paused the generator, and we can only pass values in when un-pausing a generator.

The second time we called `next()`, we passed in 12; which was multiplied by 2 and set to `y`. Then `y` was divided by 3 and sent back out. The generator was once again paused. 

The third time we called `next()`, we passed in 13, which was set to `z`. Then `x` (5), `y` (24), and `z` (13) are added together and returned. As you can see, the return value is set as the `value` in the object the iterator returns. 








