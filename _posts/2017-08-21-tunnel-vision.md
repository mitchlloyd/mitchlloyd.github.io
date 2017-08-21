---
layout: post
title:  "Component State: Overcoming Tunnel Vision"
date:   2017-08-20
---

Redux can eliminate the need for component state by storing all data in one
immutable object. However, Redux introduces new complexity to manage global
state and actions. For domain resources (usually fetched over HTTP) a global
store enables sophisticated caching strategies. But for application state,
it's often more trouble than it's worth.

React lets us organize applications by the features in our domain (e.g.
ProjectPage, Search, ValidatedField) rather than by technical patterns (e.g.
views, containers, actions, reducers). Effective product teams work in terms
of features, not abstract technical domains.

The constraint of working with component state puts pressure on our designs.
Developers must decide which components should own state and then connect
parent and child components to that state with data props and callbacks.
Because components can make API calls, manage internal state, and render HTML
they often grow with every new feature.

Let's focus on one design pressure we often feel when using component state:
tunneling data though layers of components.

Tunneling Application State
---------------------------

Most front-end developers know that they could forego global state if they
were willing to pass every bit of state through every component. This
practice is sometimes called "tunneling".

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=tunneling-component.jsx"></script>

Tunneling properties through intermediate components starts small. But every
time a developer extracts a new component, they must tunnel all the
properties through that new component. It's like they're being punished
every time they improve the design.

Let's look at an example of a search component that follows a naive design
path.

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=before-search.jsx"></script>

Already there's a little boilerplate in our `render` method. We're mapping
names (e.g. `onFilterClear` to `this.handleFilterClear`) without providing
any new information. It's redundant but it's a reasonable way to expose a
public API to `SearchLayout`.

So now let's check out the `SearchLayout` component:

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=before-search-layout.jsx"></script>

`SearchLayout` has some presentation concerns (notice the heading and the
horizontal rule), but it also has to divvy up props from its owner to its
children. This time we're mapping props that our component doesn't care about
to other props that our component doesn't care about. This is where we start
to feel the "tunneling" pain.

Search, meet SearchField
---------------------------------

`SearchLayout` shouldn't know how to connect the prop names between `Search`
and `SearchField`. It should only care about presentation things: headings,
horizontal rules, and other stylistic elements.

So let's pull the configuration of `SearchField` and `SearchResult` up the stack,
right where that information lives: inside `Search`.

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=between-search.jsx"></script>

So what does `SearchLayout` look like now?

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=after-search-layout.jsx"></script>

`SearchLayout` decides where to put the `field` prop and `results` prop, but
no longer needs to worry about the props exchanged between `SearchField`,
`SearchResults`, and `Search`.

Is this _actually_ better?
--------------------------------

Are we really improving our design or just moving stuff around?

I argue this design is better because we've eliminated the repetition of
tunneling our properties through the `SearchLayout`. We now map the `Search`
properties to owned components once rather than twice. We've also identified
an implicit responsibility (configuration) that was shared between two
components and isolated it inside of `Search`.

However, if we look at the `Search` component before and after this
refactoring we notice that it's more complicated. Although `SearchLayout` is
now dead simple, `Search` knows about three components (`SearchLayout`,
`SearchField`, and `SearchResults`) rather than one (`SearchLayout`). Because
`Search` is the top-level component in our `Search` domain, it's easy for
responsibilities to collect here.

`Search` handles both the concern of querying and the concern of
coordinating communication between our search components. Let's extract a
new component to handle our lower-level, "querying" concern.

Introducing the SearchQuery Component
------------------------------------

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=after-search-query.jsx"></script>

An alternative, and probably a more popular option, is to create a higher
order component that accepts `Search` as an argument and renders it with
`props.query`. I prefer the render callback because it declaratively shows
how all of our search components work together inside of `Search`.

Notice that I cheated a little. Not only did I extract a component to handle
querying, I also bundling the props and callbacks into a domain object that
I'll call `query` inside of `Search`.

`Search` is looking pretty good at this point.

<script src="https://gist.github.com/mitchlloyd/555d7a655a1d72e6f2dae80d00083ebb.js?file=after-search.jsx"></script>

We setout to remove some `prop` tunneling and at the same time dramatically
increased the flexibility of our design. It would be trivial to create
another `Search` component that uses a different `SearchQuery` component to
accommodate a different backend. We could inject a different presentation for
our search results into our `SearchLayout` component. The flexibility comes
with a cost: separating concerns creates more abstract pieces to learn about.
But in this case, the level of abstraction in each component is more
consistent, making our components easier to understand.

Goodbye Tunneling
-----------------

If you squint a little you could see `query` as a little Redux bundle with
one immutable state object and 3 actions (setFilter, clearFilter, setTerm)
that we dispatch by calling.

By creating a domain object and a component dedicated to composition, we
removed the pain of tunneling state though intermediate components. As a
happy side-effect we created a design that is more flexible, has constant
levels of abstraction, and is easier to test.
