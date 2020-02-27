---
layout: post
title:  "React's Shallow Render - Tread Carefully"
date:   2017-01-27
---

Before I say anything about shallow render testing with in React, we need
some baseline testing principles.

1. Tests should exercise public interfaces.

   Tests should use functions and objects as they are used in your application.
   This means that tests should not directly call private functions or access
   private variables.

2. Testing should influence design.

   Tests are easier to write when modules are focused and easy to use.
   Difficulty in testing often exposes implicit dependencies and mixed concerns.

If these principles seem new then I recommend reading [Growing Object Oriented
Software Guided by Tests](http://amzn.to/2jyjJYF) and [Practical Object-Oriented
Design in Ruby](http://amzn.to/2jynsFx).

If you're aware of these ideas but disagree with them then that's great and I'd
love to read about why. However, you might not get much out of this article.

Let's look at two problems with shallow rendering.

## Problem 1: Mocking implementation details is brittle and creates low value tests.

Let's look at module called `subtract.js`.

```js
export default function subtract(a, b) {
  return add(a, b * -1);
}

function add(a, b) {
  return a + b;
}
```

Because JavaScript modules have a strict public interface, there's really only
one way we can test this.

```js
import subtract from 'subtract';

it('subtracts', () => {
  expect(subtract(5, 3)).toBe(2);
});
```

But if we turned our simple function into an object or wrapped the `add` in an
another function call we could use a mock.

```js
export default class Subtractor {
  subtract(a, b) {
    return add(a, b * -1);
  }

  _add() {
    return a + b;
  }
}
```

```js
import subtractor from 'subtract';

it('subtract calls _add and flips the sign of the second argument', () => {
  let addCalls = [];
  subtractor.add = (...args) => {
    addCalls.push(args);
    return 'subtract-response';
  }

  const result = subtractor.subtract(5, 3);

  expect(addCalls).toEqual([5, -3]);
  expect(result).toBe('subtract-response');
});

it('has an _add method that adds numbers', () => {
  expect(subtractor._add(5, -3)).toBe(2);
});
```

Notice that:

1. We need more assertions to make sure the code is working.
2. It's hard to tell what the intention of `subtract` is because we're mired in
   its implementation details. It's like we just wrote the inverse of the code
   under test.

Also it's very brittle. What if we refactor our code?

```js
export default class Subtractor {
  subtract(a, b) {
    return a - b;
  }
}
```

Seems like a nice improvement and it doesn't change the public interface, but
the tests are broken.

Shallow Render
--------------

Let's do the same thing with shallow rendering.

```jsx
const MyComponent = ({ todos }) => (
  <ul>
    {todos.map(todo => <ToDo key={todo.id} todo={todo} />)}
  </ul>
);

const ToDo = ({ todo }) => (
  <li>{todo.title}</li>
);

describe('<MyComponent />', () => {
  it('should render todos', () => {
    const todos = [{ id: 1, title: 'ToDo 1' }];
    const wrapper = shallow(<MyComponent todos={todos} />);

    expect(wrapper.equals(
      <ul>
        <ToDo todo={todos[0]} />
      </ul>
    )).toBe(true);
  });
});

describe('<ToDo />', () => {
  it('should show the title', () => {
    const todo = { title: 'ToDo 1' };
    const wrapper = shallow(<ToDo todo={todo} />);

    expect(wrapper.equals(
      <li>ToDo 1</li>
    )).toBe(true);
  });
});
```

Just like in our previous example we're testing the interactions between two
functions rather than just testing the output. Also like our previous example
this test breaks when refactoring.

```jsx
const MyComponent = ({ todos }) => (
  <ul>
    {todos.map(todo => <li key={todo.id}>{todo.title}</li>)}
  </ul>
);
```

If we just render our components in the test the problem goes away:

```jsx
describe('<MyComponent />', () => {
  it('should render todos', () => {
    const todos = [{ id: 1, title: 'ToDo 1' }]
    const html = render(<MyComponent todos={todos} />).html();
    expect(wrapper.html()).toEqual('<ul><li>ToDo 1</li></ul>');
  });
});
```

At this point you might be thinking, "These are ridiculous, trivial cases!
There is a _reason_ we use shallow rendering. We have complicated stuff to test
and we only want to test one component at a time."

This brings me to the second problem.

## Problem 2: Mocking private functions stifles design feedback.

I'm trying to keep these examples clear so bear with me and pretend we've got
some very complicated functions that may have side effects.

```js
export default function callIt(fn) {
  return mapIt(fn());
}

function mapIt(array) {
  return array.map(decorateIt);
}

function decorateIt(item) {
  return `*${item}*`;
}
```

We want to test each function directly because they each represent a different
concern. We'll say that testing all this behavior by only calling `callIt` won't
cut it.

If we don't change the code then we have no option but to start adding mocks.

```js
it('callIt calls the passed function and then returns the result of mapIt', () => {
  // We need to mock `mapIt` here.
});

it('mapIt maps over each items and decorates it with decorateIt', () => {
  // We need to mock `decorateIt` here.
});

it('decorateIt adds asterixes around a string', () => {
});
```

Ugh. I can't even bring myself to write the details of these hypothetical tests
because they're going to be so painful.

Actually there is one I don't mind writing. `decorateIt` is just a pure
function.

```js
it('decorateIt adds asterixes around a string', () => {
  expect(decorateIt('hi')).toBe('*hi*');
});
```

`mapIt` is almost a pure function, but it is pulling in `decorateIt` from its
outer scope. `callIt` is not a pure function, it has a side effect because it
calls the function that was passed in and it implicitly depends on `mapIt`. If
only we could untangle these dependencies, this would be a lot easier to test.

We'll untangle by moving the co-ordination of these functions into a new
function that composes them.

```js
export default function compose(fn) {
  return mapIt(callIt(fn), decorateIt);
}

export default function callIt(fn) {
  return fn();
}

function mapIt(array, fn) {
  return array.map(fn);
}

function decorateIt(item) {
  return `*${item}$`;
}
```

Now testing `callIt` and `mapIt` is simple.

```js
it('callIt calls the function and returns the result', () => {
  const someFn = () => 'result';

  expect(callIt(someFn)).toBe('result');
});

it('mapIt maps over each item with a function', () => {
  const fn = (i) => `${i}-mapped`;

  expect(mapIt(['a', 'b'], fn).toEqual(['a-mapped', 'b-mapped']);
});
```

We probably want a light test for compose that makes sure everything is wired up
correctly, but we don't need to test edge cases there.

```js
it('compose does all the things', () => {
  const someFn = () => ['a', 'b'];

  expect(compose(someFn)).toEqual(['*a*', '*b*']);
});
```

This refactoring pattern shows up all the time and it goes by many names
including "dependency injection", "matchmaker", "strategy pattern",
"composition" and "container vs presentation components".

The point is, we made our code better by listening to the test pain.  Does this
have any bearing on React components?

Shallow Render
--------------

This pattern is so common when building components that I'm pretty sure
most developers would wouldn't raise an eyebrow.

```jsx
class ToDoApp extends React.Component {
  constructor(props) {
    super(props);
    this.state = { todos: request() };
  }

  render() {
    return <ToDoList todos={this.state.todos} />;
  }
};

const ToDoList = ({ todos }) => (
  <ul>
    {todos.map(todo => <ToDo key={todo.id} todo={todo} />)}
  </ul>
);

const ToDo = ({ todo }) => (
  <li>{todo.title}</li>
);
```

Using shallow render here would make a lot of unnecessary testing work for
ourselves. We can create a test that calls `<ToDoApp />` and asserts that the
result looks correct. No test pain here---seems good.

Now let's say that the `ToDoApp` actually makes an AJAX call and shows a spinner
while it loads and an error if it fails. The `ToDoList` component filters
`ToDo`s based on a search field.  The `ToDo` component responds to different
events from user input (typing, clicks, etc). Now we want to test each of
these responsibilities on their own.

Shallow rendering provides a terse API for asserting on the internal
interactions between components and the components that they own. The APIs are
so nice that we would probably just write the tests and not change our code at
all.

Here's what the test for the `ToDoApp` would look like:

```jsx
describe('<ToDoLoader />', () => {
  it('should render todos', () => {
    const todos = [{ id: 1, title: 'ToDo 1' }];
    const request = () => todos;
    const wrapper = shallow(<ToDoLoader request={request} />);
    expect(wrapper.equals(
      <ToDoList todos={todos} />
    )).toBe(true);
  });
});
```

And we would write similar tests for `ToDoList` and `ToDo`. The `wrapper.equals`
assertion (made possible by React's shallow rendering) is a concise way to make an
assertion on the seam between the `render` function output and React's rendering
system. Unfortunately this test is still brittle and tied to implementation
details.

Let's do the same thing we did with our simple functions earlier and create a
component to compose the others.

```jsx
// Now responsible for composing.
const ToDoApp = () => (
  <ToDoLoader request={request}>
    {todos => <ToDoList todos={todos} ToDoComponent={ToDo} />}
  </ToDoLoader>
);

// New component only responsible for loading behavior.
class ToDoLoader extends React.Component {
  constructor(props) {
    super(props);
    this.state = { todos: request() };
  }

  render() {
    return <ToDoList todos={this.state.todos} />;
  }
};

const ToDoList = ({ todos, ToDoComponent }) => (
  <ul>
    {todos.map(todo => <ToDoComponent key={todo.id} todo={todo} />)}
  </ul>
);

const ToDo = ({ todo }) => (
  <li>{todo.title}</li>
);
```

If we have shallow rending tests in place they'll fight us during this
refactoring. Now we can write straight-forward tests.

```jsx
describe('<ToDoApp />', () => {
  it('loads and renders todos', () => {
    const request = () => [{ id: 1, title: 'ToDo 1' }];
    const wrapper = render(<ToDoApp request={request} />);
    expect(wrapper.html()).toEqual(
      '<ul><li>ToDo 1</li></ul>'
    );
  });
});

describe('<ToDoLoader />', () => {
  it('loads a request and provides the response', () => {
    const request = () => 'response';
    const wrapper = render(
      <ToDoLoader request={request}>
        {(response) => <p>response</p>}
      </ToDoLoader>
    );
    expect(wrapper.html()).toEqual('<p>response</p>');
  });
});

describe('<ToDoList />', () => {
  it('renders an item for each passed todo', () => {
    const todos = [{ id: 1 }];
    const ToDoComponent = ({ todo }) => <p>{todo.id}</p>;
    const wrapper = render(
      <ToDoList todos={todos} ToDoDecorator={ToDoComponent}/>
    );
    expect(wrapper.html()).toEqual('<ul><p>1</p></ul>');
  });
});

describe('<ToDo />', () => {
  it('renders a todo', () => {
    const todo = { title: 'ToDo 1' };
    const wrapper = render(
      <ToDo todo={todo} />
    );
    expect(wrapper.html()).toEqual('<li>ToDo 1</li>');
  });
});
```

Again, we've decomposed these components to a point that is ridiculous for the
low level of complexity we have. We don't actually need this flexibility for
such a simple case. But even while writing these trivial tests, design questions
arise:

* The `ToDoLoader` has no idea that it's working with `ToDo`s. Probably this can be
  renamed and become a general purpose component.
* The `ToDoList` has no idea what its children HTML will look like but it is
  represented by a `ul` element that has strict rules in HTML. Either we need to
  inline the `ToDo` element to couple this markup more tightly or make the
  markup more abstract and injectable.

Because we did not immediately turn to mocks and stubs in these tests, we
created a system where the components are decoupled. This means that the design
is more flexible. For instance:

* We could reuse the `ToDoLoader` for other components that need this loading
  behavior.
* We could pass in a different `ToDoDecorator` to render different kinds of `ToDo`s.
* The `ToDoList` component could easily be reused by just renaming `todos` and
  `todo` to something more general.


Automocking in General
----------------------

From Facebook's documentation about shallow rendering:

> Shallow rendering lets you render a component "one level deep" and assert
> facts about what its render method returns, without worrying about the
> behavior of child components...

To clarify the termonology, "child components" here are components that are
rendered inside of the top level element returned from the `render` function.
This includes both components that are passed in as `props.children` to the
component under test and components that are _owned_ by the component under
test. Basically we're mocking every component invocation except the top one.

As a demonstration, shallow rendering will happily render this component and
write a neat little test for it:

```jsx
const MyComponent = (props) => (
  // RangeError: Maximum call stack size exceeded
  <MyComponent />
);

it('all good', () => {
  const wrapper = shallow(<MyComponent />);
  expect(wrapper.equals(<MyComponent />).toBe(true);
});
```

Ember followed a similar course when it took its first stab at component unit
tests in a "sandboxed" environment. Ultimately it found a better approach
by using [integrated component
tests](https://github.com/emberjs/ember-test-helpers/pull/38) by default.

Facebook's Jest testing library had "automocking" as a headline feature but
recently [disabled it by default](https://facebook.github.io/jest/blog/2016/09/01/jest-15.html#disabled-automocking) noting:

> We introduced automocking at Facebook and it worked great for us when unit
> testing was adopted in a large existing code base with few existing tests...

Many mock techniques that hide rigid design and couple themselves to
implementation details are invaluable when you need to just "get coverage" over
existing legacy code.

Buy if your use case is to just "get coverage" then why even have engineers
write the assertions? If you accept that the current code is correct, you can
easily retrofit assertions by rendering a component and then taking a snapshot
of the output. Jest calls this [Snapshot Testing](https://facebook.github.io/jest/blog/2016/07/27/jest-14.html).

From that post;

> ...engineers frequently told us that they spend more time writing a test than the
> component itself. As a result many people stopped writing tests altogether
> which eventually led to instabilities. Engineers told us **all they wanted was
> to make sure their components don't change unexpectedly**.

However many engineers do not see testing as merely a tool to guard against
unexpected changes. Some engineers write unit tests because they value the
design feedback and the documented intent.

As development teams scale and as "code coverage" remains an important metric
it's clear to see that these tools are valuable. But for new code, hear my plea:
let's be mindful with mocking.

