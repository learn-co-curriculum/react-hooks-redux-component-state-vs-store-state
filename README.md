# Components, Events, Actions, and Stores

## Overview

In this lesson we're going to revisit some of the concepts we have explored so far. Our focus will be on how components interact with each other and how global application state can be managed in a predictable, scalable way using actions, stores and event handlers.

## Objectives

1. Review interactions between different parts of a React application
2. Discuss trade-offs and benefits of one-way data flow

## Component State vs Store State

While our previous lessons extensively focused on moving state **out** of individual components, we don't always have to. In fact, sometimes it might even introduce more complexity than needed. Using `setState` and "local" component-level state is a perfectly fine choice in most cases.

In general, we should not start out by putting all our state into some form of global store (or multiple stores).

When architecting a user interface, try to use local state and parent props **first**. If we end up constantly passing down tons of props, we should consider connecting the component in question with the respective Flux store.

E.g. let's say we want to render some form of carousel, something like the [Bootstrap's Carousel component] (http://getbootstrap.com/javascript/#carousel):


A carousel is a perfect example on where using a store to extract out component state doesn't necessarily make things easier (or would simply be a massive overkill).

Writing the essential handler functions for the component in question using "classical" React-style without **any** "outside" state is trivial:

```js
class Carousel extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      // We start out rendering the first slide. 0 denotes the index of the
      // active item.
      currentSlide: 0,
    };
  }
  /**
   * Handler function that transitions to the next slide in the carousel.
   * This is the function that will be run once the user clicks the "next"
   * button.
   */
  goNext = () => {
    const previousSlide = this.state.currentSlide;
    this.setState({ currentSlide: previousSlide + 1 });
  }
  /**
   * Equivalent to `goNext`. Handler function that will be invoked when clicking
   * the "back" button.
   */
  goBack = () => {
    const previousSlide = this.state.currentSlide;
    this.setState({ currentSlide: previousSlide - 1 });
  }
  render() {
    // Magic goes here
  }
}
```

In this case, using the local state of the component has a couple of advantages over using an external store:

1. The state is **by definition** bound to the component

When rendering a very long list of carousels, keeping the state stored in the store in sync with the _actual_ list of rendered components is hard. Let's say  we render one carousel for each photo "collection" — which could for example be represented by an array for image sources — keeping the array length in sync with whatever data structure we would use in the store for representing the selected slide index is unnecessarily complex. For example, when adding a photo collection, we need to add the `currentSlide` property to the store as well.

Simply distinguishing between "component UI" state and global application state radically simplified the architecture in the above case, since component state can by definition not exist without a matching component (and vice versa).

2. Simplified Testing

Testing React components is extremely easy compared to other frameworks, such as Angular. Reason being that React components are by definition declarative, while Angular heavily relies on imperative APIs.

Using stores doesn't necessarily break this abstraction, but it makes it much harder to properly test all the possible states that a component can be in, since a store might contain state that isn't directly consumed by the component to be tested.

But more importantly, we now need to manage a store during testing. This means we need to make sure we **always** restore it to its previous state before every test case, otherwise this can lead to hard to debug failed tests. In Mocha, we can use `beforeEach` to run a function before every test case (`it(...)`).

Instead of restoring the store's state, we can also mock it out. This eliminates the need to reset the store.

3. Reusing the component is possible

While we focused on implementing our own set of stores, some people prefer to use Redux, Rx, mobx or some other library for managing state and implementing unidirectional data flow.

By storing state in an external store, we implicitly couple the component to whatever architecture we chose for our main application. If we're implementing an accordion component using Flux, it means everyone using our component will have to use Flux in order to interact with it (even though it might be hidden though the public API of the component).

Hence using component state (and props) instead of stores is the preferred way when creating reusable components.

## Connecting components using lifecycle methods

So far, we oftentimes connect "connected" components to stores by adding an event listener on the store in the component's lifecycle methods:

```js
class MyCounter extends React.Component {
  componentDidMount() {
    this.removeListener = store.addListener(count => {
      this.setState({ count });
    });
  }
  componentWillUnmount() {
    this.removeListener();
  }
}
```

In `componentDidMount` we add an event listener that updates the component's state based on the store's state, thus triggering a re-rendering.

In `componentWillUnmount` we remove the event listener. Typically `addListener` returns a function that — when invoked — removes the event listener from the store. In other cases people might subclass the `EventEmitter` class directly in order to implement a store, hence there are cases in which we actually have to call `removeListener` on the store itself.

Whether or not we use `componentDidMount` instead of `componentWillMount` doesn't make a difference here. We can call `setState` in both.

## Presentational vs Container Components

**Container components** are components that are directly **connected** to our store, e.g. using the `addListener` method. They are primarily concerned with holding state, managing it and passing it to other components using props.

Container components have handler functions that dispatch actions or mutate state. They contain the "business logic" of our application.

In single page apps, a good rule of thumb is to make each page of your application (or component attached to a sub-route) a container component. While it isn't necessarily a bad idea to use nested container components, passing props to pure components tends to be easier to test and reason about.

**Presentational components** are modular, reusable (and typically small) components that are concerned with "how stuff looks". They are not connected to a particular store, can be used in different applications and are usually stateless (as in "pure").

Usually UI elements (with a bit of interaction) are presentational components and therefore not concerned with the actual state of the application. E.g. a modal, accordion, or button should not be container components.

Deciding whether or not something should be a container or presentational component is not a definitive decision. Making presentational components stateful by wiring them up to a store is usually quite easy and gets rid of a lot of indirection. For example passing down a lot of different props 5 levels deep is much more error prone than simply connecting the "leaf" component to the store.

It also means we don't need to rerender all the components in between the presentational leaf component that is due to be rendered and the intermediate components that simply pass down the state via props from the container component.

## Resources

- [Interactivity and Dynamic UIs](https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html)
- [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367#.7v3xs9al2)
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.jp0dni40i)

[carousel]: assets/carousel.png "Bootstrap Carousel"

<p class='util--hide'>View <a href='https://learn.co/lessons/react-components-events-actions-and-stores'>Components, Events, Actions And Stores</a> on Learn.co and start learning to code for free.</p>
