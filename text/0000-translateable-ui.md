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
```


# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

The translation is done by caching all entities with a `UiText` component on UI creation, and as soon as the locale resource is loaded, replace the text. The original text is the key, and the new text is the translation from the resource file.

If no value is found to a key, the text should not be replaced. This way, text which should not be translated can simply be kept as text in the prefab.

In order to make the binding between prefab and resource easy, the creator can be extended with a builder pattern, which can add a path or a handle (for simple re-use).

# Drawbacks
[drawbacks]: #drawbacks

I do not see any drawbacks, since translation is expected of modern games and this RFC adds a simple way to implement it from a game-dev's perspective.

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

As a web-dev working on web apps, I usually cache the locale file and fill the HTML with attributes marking elements for translation, or directly call a function which returns a translation to a given key. It's quite a simple way and state-of-the art as far as I know (see Angular), however, since we know that UiText elements will contain all the text and there's no complex DOM in Amethyst, we can simplify the process and directly write the translation-key into the text field without marking them first.

Instead of going with the above described apporach for a technical implementation, the translation could be done using a marker (additional field?), which is unset after a successful translation. The Ui builder might expose a way to default the marker to either `true` or `false`.

# Prior Art
[prior-art]: #prior-art

Similarities to how locales are inserted in web frameworks.

```html
<h3 i18n>loading-text</h3>
```

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

Is this the easiest way to combine prefabs and localization?

Copyright 2018 Amethyst Developers
