---
title: Mocking Constructors With Jest
date: 2023-06-05T06:58:40.125Z
draft: false
---
Mocks are probably the most difficult feature of Jest to use and understand. It doesn’t help that the Jest documentation on mocking constructors makes use of a relatively obscure JavaScript feature. 

See this [example from the official Jest docs](https://jestjs.io/docs/es6-class-mocks#calling-jestmock-with-the-module-factory-parameter):

```js
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

Which mocks this class:

```
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }
}
```

Note that the mock implementation is mocking a constructor function. The function passed to `mockImplementation` is an arrow function, but arrow functions can’t be used as constructors. How come the mock works when used as a constructor? The answer is that the arrow function passed to `mockImplementation` doesn’t become the entirety of the resulting mocked function. The final mocked function is a combination of the functionality that comes with `jest.fn()` as well as the custom code in the arrow function passed to `mockImplementation`. And the final mocked function returned from `mockImplementation` (and made available to the code under test) is a `function` function, which can be used with the new keyword.

But the example is not just a little confusing; it comes with a major pitfall that has tripped up several poor Jest users, including me ([#2982](https://github.com/jestjs/jest/issues/2982), [#8431](https://github.com/jestjs/jest/issues/8431), [#10965](https://github.com/jestjs/jest/issues/10965), [#11316](https://github.com/jestjs/jest/issues/11316)). If you try to read the `instances` property of the mocked function, it won’t work as expected! 

```js
test("get mocked instance", () => {
  const myInstance = new SoundPlayer();
  expect(myInstance).toBe(SoundPlayer.mock.instances[0]);
});
```

That test fails:

```
    expect(received).toBe(expected) // Object.is equality

    - Expected  - 1
    + Received  + 3

    - mockConstructor {}
    + Object {
    +   "playSoundFile": [Function mockConstructor],
    + }
```

To fully understand why that is, we need to know a little about the new keyword in JavaScript. MDN explains what happens when you invoke a function with new:

> When a function is called with the new keyword, the function will be used as a constructor. new will do the following things:
1. Creates a blank, plain JavaScript object. For convenience, let's call it `newInstance`.
2. Points `newInstance`’s [[Prototype]] to the constructor function's `prototype` property, if the `prototype` is an `Object`. Otherwise, `newInstance` stays as a plain object with `Object.prototype` as its [[Prototype]]. Note: Properties/objects added to the constructor function's `prototype` property are therefore accessible to all instances created from the constructor function.
3. Executes the constructor function with the given arguments, binding `newInstance` as the `this` context (i.e. all references to `this` in the constructor function now refer to `newInstance`).
4. If the constructor function returns a non-primitive, this return value becomes the result of the whole `new` expression. Otherwise, if the constructor function doesn't return anything or returns a primitive, `newInstance` is returned instead. (Normally constructors don't return a value, but they can choose to do so to override the normal object creation process.)


Notice how the `mockImplementation` function returns an object (`{playSoundFile: mockPlaySoundFile}`). That object will, of course, be returned by the mocked function. And that mocked function is being called with new. That means the `this` value created in step 1, `newInstance`, is not what’s being returned. The tricky part is that Jest stores `newInstance` into `instances`, not whatever you return from your constructor! You can think of the internals of `jest.fn()` as looking like this:

```

/**
 * Create a mock function. A partial simulation of `jest.mock`.
 */
function fn() {
  // This will be the `mock` property of the returned function
  const mock = {
    // ... other fields such as .calls, .results, .contexts, etc.
    instances: [],
  };

  // This is the mocked function that we will return
  function mockConstructor() {
    if (new.target) {
      // Note how the customImplementation return value is not
      // used for the `instances` array!
      mock.instances.push(this);
    }
    // ... other logic to append to the other fields in `mock`
    if (mockConstructor._customImplementation) {
      return mockConstructor._customImplementation.call(this);
    }
  }
  mockConstructor.mock = mock;
  mockConstructor.mockImplementation = (implementation) => {
    mockConstructor._customImplementation = implementation;

    // Return the same function that `fn()` returns to allow
    // for chaining
    return mockConstructor;
  };
  return mockConstructor;
}
```

Here are some examples to demonstrate the consequences of using an arrow function and returning an object instead of modifying `this`:

```
// No custom implementation; default mock function.
const mocked0 = fn();
console.log(new mocked0(), mocked0.mock.instances);
// mockConstructor {} [ mockConstructor {} ]
// The two are the same.

const mocked1 = fn().mockImplementation(
  // Custom implementation: arrow function returning
  // non-primitive value
  () => {
    this.c = "d";
    return {
      a: "b",
    };
  }
);
console.log(new mocked1(), mocked1.mock.instances);
// { a: 'b' } [ mockConstructor {} ]
// The two are not the same. The instance in the `instances`
// array remains unmodified by the line `this.c = "d"` because
// the arrow function is unable to modify `this`.

const mocked2 = fn().mockImplementation(
  // Custom implementation: arrow function returning undefined
  () => {
    this.c = "d";
  }
);
console.log(new mocked2(), mocked2.mock.instances);
// mockConstructor {} [ mockConstructor {} ]
// The two are the same, but unmodified by the line `this.c = "d"`
// because the arrow function is unable to modify `this`.

const mocked3 = fn().mockImplementation(
  // Custom implementation: `function` function returning
  // non-primitive value
  function () {
    this.c = "d";
    return {
      a: "b",
    };
  }
);
console.log(new mocked3(), mocked3.mock.instances);
// { a: 'b' } [ mockConstructor { c: 'd' } ]
// The two are not the same. The instance in the `instances`
// array was successfully modified by the line `this.c = "d"`.

const mocked4 = fn().mockImplementation(
  // Custom implementation: `function` function returning undefined
  function () {
    this.c = "d";
  }
);
console.log(new mocked4(), mocked4.mock.instances);
// mockConstructor { c: 'd' } [ mockConstructor { c: 'd' } ]
// The two are the same, and successfully modified by the
// line `this.c = "d"`.
```

For all of these examples, replacing `fn()` with `jest.fn()` will result in the same output.

My take is that, to avoid tripping up users, the example in the Jest documentation should probably look like this:

```
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock(‘./sound-player’, () => {
  return jest.fn().mockImplementation(function () {
    this.playSoundFile = mockPlaySoundFile;
  });
});
```

And that would suffice to pass the earlier `"get mocked instance"` test.
