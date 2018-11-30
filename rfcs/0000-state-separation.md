# Table of Contents

- [Tracking Issues](#tracking-issue)
  - [Amethyst Community Forum Discussion](#forum-discussion)
- [Motivation]
- [Guide Level Explanation](#guide-level-explanation)
- [Reference Level Explanation](#reference-level-explanation)
- [Drawbacks]
- [Rationale and Alternatives](#rationale-and-alternatives)
- [Prior Art](#prior-art)
- [Unresolved Questions](#unresolved-questions)

# Basic Info
[basic]: #basic-info

- Feature Name: state_separation
- Start Date: 2018-11-30
- RFC PR: (leave this empty until a PR is opened)
- [Tracking Issue](#tracking-issue): (leave this empty)
- [Forum Thread](#forum-discussion)

# Summary
[summary]: #summary

Create a clean separation between states, and the callbacks (code) associated with them.

## Amethyst Community Forum Discussion
[forum-discussion]: #forum-discussion

https://community.amethyst-engine.org/t/request-for-comments-simplified-state-machine/143/3

# Motivation
[motivation]: #motivation

* Improve ergonomics and re-usability of state code.
* Track the current state in a manner which is available through the `World`.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

Here's an overview of what I'm proposing, followed by dedicated sections where I try to walk
through how each change works and what motivated them:

* States and their callbacks are separated.
* Introducing a `State` derive to construct highly efficient storage for state enums.
* Getting the current state from a system.
* Introducing global callbacks.
* Custom GameData in states is deprecated.
* Modifying states is done through a `States` resource.
* Refactor amethyst_test.

## States and their callbacks are separated

State implements the `trait State<T, E: Send + Sync + 'static>` trait.

T is very commonly some variation of `GameData<'a, 'b>`. This can be customized (see below), but the only real requirement is that it somehow encapsulates a `Dispatcher`.
Every state is then responsible for driving the dispatcher through their `update` function.

This is primarily done by implementing `SimpleState`, and then relying on the blanket implementation of this to perform the dispatch.

This change instead introduces the following:

* A simplified `StateCallback<S, E>` trait, where `S: State<E>`, and `E` is the event.
* `Application` is now responsible for driving a single `Dispatcher` through a locally stored `GameData`.

The second part is a big one. That means there's no need to make use of callback abstractions (e.g. `SimpleState`), which makes the number of abstractions used for implementing callbacks fewer.

Here is a list of some of the signatures of `StateCallback`:

```rust
trait StateCallback<S, E> {
    fn on_start(&mut self, _: &mut World) {
    }

    fn update(&mut self, _: &mut World) -> Trans<S> {
        Trans::None
    }

    fn handle_event(&mut self, _: &mut World, _: &E) -> Trans<S> {
        Trans::None
    }

    // snip
}
```

Most notably `Trans` has been simplified to only take the `State`, and we no longer receive a parameterized `StateData<T>`.
Events are passed by reference, since they are passed to multiple callbacks (shadow, global, state).

This also means that we need to _associate_ a state with its corresponding callback.
This is done when building the application like this:

```rust
let mut game = Application::build("./assets")?
    .with_state(State::Main, MainState)?
    .build(GameDataBuilder::default())?;

game.run();
```

Note that there are a couple of types which come with `State` implementations:
* `&'static str`
* `u8`, `u32`, `u64`
* `i8`, `i32`, `i64`
* `()` (the empty tuple)

Note that the last one can be used as a "default state" when you only have one in your game:

```rust
let mut game = Application::build("./assets")?
    .with_state((), MainState)?
    .build(GameDataBuilder::default())?;

game.run();
```

The initial state is determined by its `Default` implementation. So `()` would be `()`, `&str` would be `""` (the empty string).

It's also trivial to implement a HashMap-based state implementation as long as your state implements `Hash + PartialEq + Eq`:

```rust
#[derive(Debug, Clone, Hash, PartialEq, Eq)]
struct SomeStruct {
    // snip
}

impl<E> State<E> for SomeStruct {
    type Storage = MapStateStorage<Self, E>;
}
```

For enum-based states there's an even more efficient alternative, which is covered in the next section.

## The `State` derive

The most natural way to model states is to use an enum with unit variants. Unit variants are variants which have no structure or tuple body, like this:

```rust
enum State {
  Loading,
  Main,
}
```

To provide a `State` implementation for this enum you can use the `State` derive:

```rust
#[derive(State, Debug, Clone)]
enum State {
  Loading,
  Main,
}
```

This will provide the `State` implementation, and implement `Default` to return the first element.

When using this derive, the Storage implementation is also specialized to an implementation like this:

```rust
struct State_Storage<E> {
  f1: Option<Box<dyn amethyst::StateCallback<State, E>>>,
  f2: Option<Box<dyn amethyst::StateCallback<State, E>>>,
}

impl<E> amethyst::StateStorage<State, E> for State_Storage<E> {
    fn get_mut(&mut self, value: &State) -> Option<&mut Box<dyn amethyst::StateCallback<State, E>>> {
        match *self {
            State::Loading => self.f1.as_mut(),
            State::Main => self.f2.as_mut(),
        }
    }

    // snip
}
```

This effectively inilnes the callbacks in the state machine.

## Getting the current state from a System

Since states and callbacks are separated, it is now possible to query the current state from a system.
You do this by adding `Read<'r, State>` to `SystemData`:

```rust
struct MySystem;

impl<'r> System<'r> for MySystem {
    type SystemData = Read<'r, State>;
}
```

Note that this is maintained internally in `Application` using a global callback that has implemented the `changed` function.

## Introducing global callbacks

Since we no longer have a common ancestor in `SimpleState` I introduced a new mechanism in the state machine called "global callbacks".

These are callbacks that are fired for _all_ states, in particular it is suitable to solve some specific problems in `amethyst_test` where we need to automatically drive state transitions on each update.

```rust
/// A callback that is registered for all events.
/// This is typically used for bookkeeping specific things.
pub trait GlobalCallback<S, E> {
    /// Fired when state machine has been started.
    fn started(&mut self, world: &mut World) {}

    /// Fired when state machine has been stopped.
    fn stopped(&mut self, world: &mut World) {}

    /// Fired when state has changed, and what it was changed to.
    fn changed(&mut self, world: &mut World, state: &S) {}

    /// Fired on events.
    ///
    /// If multiple callbacks would result in a state transition, they will be applied one after
    /// another in an undetermined order.
    fn handle_event(&mut self, world: &mut World, _: &E) -> Trans<S> {
        Trans::None
    }

    /// Fired on fixed updates.
    fn fixed_update(&mut self, world: &mut World) -> Trans<S> {
        Trans::None
    }

    /// Fired on updates.
    fn update(&mut self, world: &mut World) -> Trans<S> {
        Trans::None
    }
}
```

## Custom GameData is deprecated

Custom GameData is a bit of an awkward fit.
And making use of it involves implementing a large number of traits and necessitated making the StateCallback trait harder than it should be.

We generally have two places that are readily available for storing data:
* A `Resource`, which is associated with the `World`.
* In the state callbacks (since we have `&mut self` access in the callbacks).
* In the global callbacks (same reason as above).

The biggest motivation to implement custom game data seemed to be to make use of multiple dispatchers.
But this is fraught with its own set of problems.
Splitting the dispatchers means your resources can't be accessed in parallel, effectively causing a barrier between them.

The GameData seemed like an akward fit into all of this, so I decided to try and deprecate it.
The combination of the World, callback, and global callback storage has been sufficient to quite nicely address all architecture patterns I've looked into so far.

## Modifying states is done through a `States` resource.

The only feature that couldn't be addressed using the above was adding states to the state machine.

This became necessary since it's a feature that is used in the `renderable` example to asynchronously load assets, and defer creation of the `Example` state until those assets have been loaded.

Since we no longer return a full state in `Trans`, I decided to implement a resource called `States<S, E>` that can be used to associate new state callbacks.

If you look at the `renderable` example in this PR you should get a feel for how it works.

Similarly as before, we only maintain one instance of the callbacks per state.
When a state is added here, it is correctly lifecycled, old state is `on_stop`:ed, new state is `on_start`:ed.

Here's an example taken from the `examples/renderable`:

```rust
world.write_resource::<States<_, _>>().new_state(
    State::Example,
    Example {
        scene: self.prefab.as_ref().unwrap().clone(),
    },
);
```

## Refactor amethyst_test

Since a large number of things changed in how applications are constructed, `amethyst_test` needed to be refactored as well.

I'm not gonna cover too much about what happened, but if you are curious, the tests and API docs should have been updated.

But briefly:
* All methods prefixed with `do_*` are run _in order_.
* `with_effect`, `with_assertion`, and `with_setup` all were the same alias. So they have been replaced with `do_fn`.
* You can switch state using `do_state(&S)`.
* A `do_wait(usize)` function has been introduced to wait a number of frames before progressing.

A simple test case would now look like this:

```rust
AmethystApplication::blank()
    .with_state("first", FunctionState::new(|world| {
        world.add_resource(ApplicationResource);
    }))
    .do_state("first")
    .do_wait(1)
    .do_fn(|world| {
        /// Reading the resource should be fine.
        world.read_resource::<ApplicationResource>();
    })
    .run()
    .expect("an error")
```

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

Too much to list, I encourage you to look at the refactoring done in association with building the
prototype.

You can find a diff here:
https://github.com/amethyst/amethyst/compare/master...udoprog:enum-state-machine

# Drawbacks
[drawbacks]: #drawbacks

This causes a disruption in how we currently write and associate state handlers.
Users will be required to refactor their existing code to follow the new design.

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

The overall design came out of trying to simplify state callbacks as much as possible while still
retaining their identity.

An alternative to "naming states" is to extend the existing `State` trait to include this
information as something like an associated function or constant.
This was rejected because it increases the complexity of the trait, while one of the goal is to
decrease it as much as possible.

Not doing this leads to existing and new users of Amethyst having to cope with the relatively high
complexity associated with the existing State hierarchies and custom GameData.

# Prior Art
[prior-art]: #prior-art

This is inherent to the design of Amethyst's take on Application/State/System. The author is not
aware of any relevant prior art.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

* Can these changes be incorporated incrementally?
* Should we really deprecate custom GameData?
* Are there additional simplifications we can do to `StateCallback`?
* Final naming of traits.

Copyright 2018 Amethyst Developers
