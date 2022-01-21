# React Component State vs Redux Store State

## Learning Goals

- Explain interactions between different parts of a React application
- Discuss trade-offs and benefits of one-way data flow

## Introduction

In this lesson, we're going to revisit some of the concepts we have explored so
far. Our focus will be on how components interact with each other and how global
application state can be managed in a predictable, scalable way using actions,
stores and event handlers.

## Component State vs Store State

While our previous lessons extensively focused on moving state **out** of
individual components, we don't always have to. In fact, sometimes it might even
introduce more complexity than needed. Using `useState()` and "local"
component-level state is a perfectly fine choice in most cases.

In general, we should not start out by putting all our state into some form of
global store (or multiple stores).

When architecting a user interface, try to use local state and parent props
**first**. If we end up constantly passing down tons of props, we should
consider connecting the component in question with a Redux store.

E.g. let's say we want to render some form of carousel, something like
[Bootstrap's Carousel component](https://getbootstrap.com/docs/4.0/components/carousel/).

A carousel is a perfect example on where using a store to extract out component
state doesn't necessarily make things easier (or would simply be a massive
overkill).

Writing the essential handler functions for the component in question using
"classical" React-style without **any** "outside" state is trivial:

```js
function Carousel() {
  // We start out rendering the first slide. 0 denotes the index of the
  // active item.
  const [currentSlide, setCurrentSlide] = useState(0);

  /**
   * Handler function that transitions to the next slide in the carousel.
   * This is the function that will be run once the user clicks the "next"
   * button.
   */
  function goNext() {
    setCurrentSlide(prevSlide => prevSlide + 1);
  }

  /**
   * Equivalent to `goNext`. Handler function that will be invoked when clicking
   * the "back" button.
   */
  function goBack() {
    setCurrentSlide(prevSlide => prevSlide - 1);
  }

  return (
    // JSX goes here
  )
}
```

In this case, using the local state of the component has a couple of advantages
over using an external store:

### 1. State Is Bound to the Component

When rendering a very long list of carousels, keeping the state stored in the
store in sync with the _actual_ list of rendered components is hard. Let's say
we render one carousel for each photo "collection" — which could for example be
represented by an array for image sources — keeping the array length in sync
with whatever data structure we would use in the store for representing the
selected slide index is unnecessarily complex. For example, when adding a photo
collection, we would need to add the `currentSlide` property to the store as
well.

Simply distinguishing between "component UI" state and global application state
radically simplified the architecture in the above case, since component state
can by definition not exist without a matching component (and vice versa).

### 2. Simplified Testing

Testing React components is extremely easy compared to other frameworks, such as
Angular. Testing packages like [Enzyme][] from Airbnb allow us to mount
individual components in a test, pass them props, cause state changes, check
what JSX is rendered, etc...

[enzyme]: https://airbnb.io/enzyme/

Using stores doesn't necessarily break this abstraction, but it makes it much
harder to properly test all the possible states that a component can be in,
since a store might contain state that isn't directly consumed by the component
to be tested.

But more importantly, we now need to manage a store during testing. We can use
the same packages and functions like `createStore()` we use to set up Redux with
React, but the tests become more complicated and sometimes less flexible as a
result.

We can also mock it out — some node packages allow us to create a fake store for
the tests. Overall, though, because Redux changes the way data is maintained,
tests need to change accordingly, becoming more complicated.

### 3. Component Reuse

While we focused on implementing our own set of stores, some people prefer to
use Redux, Rx, MobX or some other library for managing state and implementing
unidirectional data flow.

By storing state in an external store, we implicitly couple the component to
whatever architecture we chose for our main application. If we're implementing
an accordion component using [Flux][] (the data flow pattern Redux is based on),
it means everyone using our component will have to use Flux in order to interact
with it (even though it might be hidden through the public API of the
component).

[flux]: https://facebook.github.io/flux/

Hence using component state (and props) instead of stores is the preferred way
when creating reusable components.

## You Might Not Need Redux

Since the release of React hooks, the React Context API and the [`useContext`
hook][usecontext] has emerged as a popular way to manage global state. Some
libraries we've worked with already, such as `react-router` and `react-redux`,
actually use the Context API under the hood — that's how we're able to access
our Redux store state, or information about the browser history, from _any_
component in our component hierarchy without prop drilling.

Before adding Redux to an application, make sure to think about whether or not
your app would benefit from Redux. Check out [this article from the Redux
docs][when should i use redux?] along with the linked articles and discussion
for some help deciding if your app would benefit from Redux!

That said, Redux is very much still [alive and well][redux is not dead yet] and
you are definitely encouraged to try it out in a project so you can learn, and
add Redux to your toolkit!

## Resources

- [Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react)
- [When should I use Redux?][]
- [Redux is Not Dead Yet][]
- [useContext][usecontext]

[usecontext]: https://reactjs.org/docs/hooks-reference.html#usecontext
[when should i use redux?]:
  https://redux.js.org/faq/general#when-should-i-use-redux
[redux is not dead yet]:
  https://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/
