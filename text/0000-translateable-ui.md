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

- Feature Name: translateable_prefabs
- Start Date: 2018-11-19
- RFC PR: 
- [Tracking Issue](#tracking-issue): 
- [Forum Thread](#forum-discussion): No.

After filling out these fields, please proceed to fill out the remaining sections of this document.

# Summary
[summary]: #summary

Currently, Amethyst is able to provide prefabs for component hierarchies, for example UIs.
They should predefine what the engine loads and displays in a convenient manner.
Amethyst can also load locales using Fluid, which allows the translation of an application into many languages with ease.
The intention of this RFC is to define a way, how translation functionality can be integrated into prefabs.

# Motivation
[motivation]: #motivation

I want to create a game and translate it into many languages using data-input.

Having to set up a complex logic myself for something I'd consider default behavior is a hindrance.
So, instead of implementing it for every game again and again myself, I want to propose implementing it directly into the engine.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

Amethyst needs to match translation files to UI elements.
In order to do so, the translation key has to be added to the prefab and at least one locale resource has to be loaded,
which offers a translation for that specific key. In a trivial Amethyst application, that would result in loading at least the prefab and the resource. The code in order to do so is trivial and only needs a method call on load.


## Example

```ron
#![enable(implicit_some)]
Text(
    transform: (
        id: "loading",
        anchor: Middle,
        x: 0.,
        y: 0.,
        width: 200.,
        height: 50.,
        transparent: true,
    ),
    text: (
        text: "loading-text",
        translate: true,
        font_size: 25.,
        color: (1., 1., 1., 1.),
        font: File("font/square.ttf", Ttf, ()),
    ),
)
```

```ftl
loading-text = Loading, please wait!
```

```rust
world.exec(|mut creator: UiCreator| {
    creator
        .build("prefabs/my_ui.ron")
        .with_locales("resources/en_US.ftl")
        .create();
});

// or, if no prefabs are used

UiTranslator.translate(world, locale_storage);
```


# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

The original text is the key, and the new text is the translation from the resource file. The translation is done by checking a marker field or component (`translate` field in above example) and match the key against a storage. The translation can either be done by a convenience method on the UiCreator directly, or by calling a translation function directly, which checks the world for translate-able components once.

If no value is found to a key, the text should not be replaced. This way, at least the key is visible and a developer (or QA) knows which translation is still missing.

In order to make the binding between prefab and resource easy, the creator can be extended with a builder pattern, which can add a path or a handle (for simple re-use).

# Drawbacks
[drawbacks]: #drawbacks

I do not see any drawbacks for the proposal itself, since translation is expected of modern games and this RFC adds a simple way to implement it from a game-dev's perspective.

This RFC does not take non-ui texts into account, though, which might also need translation. The above approach is hard to generalize for any component with a text field, and I cannot come up with a good solution to the problem - which might be due to my limited knowledge of Rust and Amethyst.

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

As a web-dev working on web apps, I usually cache the locale file and fill the HTML with attributes marking elements for translation, or directly call a function which returns a translation to a given key. It's quite a simple way and state-of-the art as far as I know (see Angular).

Instead of going with the above described apporach for a technical implementation, the translation could be implemented by caching all entities with a `UiText` component on UI creation, and as soon as the locale resource is loaded, replace the text.

# Prior Art
[prior-art]: #prior-art

Similarities to how locales are inserted in web frameworks.

```html
<h3 i18n>loading-text</h3>
```

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

- Is this the easiest way to combine prefabs and localization?
- How can it be generalized for any (custom?) component with text? Can we use components as markers and traits for that?

Copyright 2018 Amethyst Developers
