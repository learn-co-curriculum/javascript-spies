# JavaScript Spies

## Objectives

1. Explain why we use spies in JavaScript
2. Describe how to use Sinon spies
3. Practice using Sinon spies

## Cloak And Dagger
![A spy team](https://media.giphy.com/media/Dzmg5QAojwb6M/giphy.gif)

When testing our code, simple value assertations (e.g. `expect(foo).toEqual('bar');`) aren't enough.
Sometimes, we need to test our program to see if functions are being passed around and called
properly. The act of doing so is called _spying_. Pretty exciting, right? We can use
[Sinon.js](http://sinonjs.org/docs) to do this for us.

For example, let's say we have a basic data store. We can add subscribers to that data store to let
us know of any changes (so we can update our UI accordingly). A basic version would look something
like this:

```js
function createStore() {
  const listeners = [];
  const data = {};

  function emitChange() {
    listeners.forEach(listener => listener());
  }

  function subscribe(callback) {
    listeners.push(callback);
  }

  function setData(key, value) {
    data[key] = value;
    emitChange();
  }

  function getData() {
    return data;
  }

  return {
    subscribe,
    setData,
    getData,
  };
}
```

This store implementation lacks several features, but it's good enough for illustrative purposes.
Now, let's create a store and add a subscriber. Afterwards, we'll set some data on the store.

```js
const store = createStore();

store.subscribe(function () {
  const storeData = store.getData();
  console.log('Updated store data:', storeData);
});

store.setData('flavor', 'chocolate');
```

After `store.setData('flavor', 'chocolate');` has done its thing, our callback added above will fire
because the store is emitting a change (by calling all callbacks in the `listeners` array).

## Testing the `subscribe()` function
What we're interested in testing our store is the `subscribe()` function. We need to verify that our
callback gets called every time the store data changes. To do so, we'll write a fairly simple test
that makes use of spies. As mentioned above, we're using Sinon.js to create spies. There are other
spy libraries out there, but this is by far the most robust and popular one.

What we're going to do is create a store in our test, and then create a spy. We'll pass that spy to
the `subscribe()` function provided by the store. Afterwards, we'll set data on the store and verify
that our spy has been called the correct amount of times.

```js
describe('store', function () {
  it('should call a listener every time the store data updates', function () {
    const store = createStore();
    const spy = sinon.spy();

    store.subscribe(spy);
    store.setData('flavor', 'chocolate');
    expect(spy.calledOnce).toBeTruthy();
    store.setData('topping', 'sprinkles');
    expect(spy.calledTwice).toBeTruthy();
  });
});
```

As you can see, using spies is relatively straight-forward and makes writing tests a lot more
pleasant. See the [Sinon.js docs on spies](http://sinonjs.org/docs/#spies) to discover its full API.


## Where's the magic?
![Magic](https://media1.giphy.com/media/61uXIVNCSsTrq/200.gif)

Spies can seem a little magical at first, but their implementation is actually pretty simple. A
basic spy could look like this:

```js
function createSpy() {
  function spy() {
    spy.callCount++;
    spy.calledOnce = spy.callCount === 1;
    spy.calledTwice = spy.callCount === 2;
  }

  spy.callCount = 0;

  spy.wasCalled = () => this.callCount > 0;
  
  return spy;
}
```

All this (admittedly very rudimentary) spy does is increment its `callCount` every time it has been
called, along with setting the `calledOnce` and `calledTwice` properties every time it is called.

Obviously, the real Sinon.js spies are a lot more complicated than this implementation since they
have a lot more functionality, but this is basically how it works!
