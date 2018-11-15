# Table of Contents

- [Tracking Issues](#tracking-issue)
- [Motivation](#motivation)
- [Guide Level Explanation](#guide-level-explanation)
- [Reference Level Explanation](#reference-level-explanation)
- [Drawbacks]
- [Rationale and Alternatives](#rationale-and-alternatives)
- [Prior Art](#prior-art)
- [Unresolved Questions](#unresolved-questions)

# Basic Info
[basic]: #basic-info

- Feature Name: amethyst_as_bin
- Start Date: 2018-11-14
- RFC PR: (leave this empty until a PR is opened)
- [Tracking Issue](#tracking-issue): (leave this empty)

After filling out these fields, please proceed to fill out the remaining sections of this document.

# Summary
[summary]: #summary

This RFC is to act as a discussion of the merits and drawbacks of reversing our current relationship with our users.
Currently they create the executable binary, and we create a crate for that binary to use.  However, if we instead
chose to reverse that, and make it so that we provided a binary for which their crate hooked into we can now control
`fn main()`.  After making this transition we would have to be much more dependent on our amethyst toolset than we currently are.

# Motivation
[motivation]: #motivation
There are three areas of the software that stand to benefit from this

1. Initialization. If we control the main engine initialization sequence then we can avoid handing control over to the user
before the engine is fully initialized.  The biggest benefit this has is that user provided `System::setup()` methods can use
`!Default` resources.  Most commonly, these resources come from the rendering system, such as `ScreenDimensions`.  In order to
gain the most benefits from this we are going to have to introduce a "core dispatcher" concept tp the engine.
The core dispatcher would be responsible for input, and rendering, both of which would be optional.  If neither are enabled
then we simply don't run the core dispatcher.

2. Editor. One could imagine the user crate as a pluggable set of code into a variety of possible amethyst executables.
One of these executables could be the standalone engine, and another one could be a scene editor.  This would allow us to provide a
"play" button that starts executing the user code, and since we control the flow of execution now we can also "stop" gameplay
whenever we feel.

3. Scripting. Part of our executable can be identifying, incorporating, and executing code written in foreign languages. Since we
control the flow of execution here we are no longer dependent on the user authoring Rust code in order to identify and
execute their scripts. In theory, you could use Amethyst without needing to know Rust at all.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

When generating a new Amethyst project you'll find a file in your project called `lib.rs`. In this file will be several
functions Amethyst depends on you to provide.  Do not remove them, otherwise your project will not work with Amethyst.

However, you can and are expected to alter the body of the function to change what is returned. Do not change the function names,
parameters, or return types.

Here are the functions and types we expect you to provide:

- `pub type GameData`

  This is the type used to store your system dispatchers and any thread local data you may have.  We've pre-populated it with a
  simple to use type from Amethyst called `GameData` but you may want to provide your own type here later when your game gets
  more complicated.
  
- `pub type Event`

  This is the type of event that Amethyst will send to your states.  We've populated it with a default called `StateEvent`.

- `pub fn first_state() -> impl State<GameData, Event>`
  
  Provides the first state that your game will execute. In the template file we've provided a basic empty state
  and we are returning it in this function.
  
- `pub fn input_bindings() -> String`

  Expected to provide either a path to a file in the RON format describing the input bindings for the player, or 
  the input configuration as a string provided in the RON format. It is pre-populated with the path of an a simple file provided
  in the template. If you don't wish to use the Amethyst input system you can return an empty string here.
  
- `pub fn game_data() -> impl DataInit<GameData>` 
  
  Provides the core of your game's parallelism.  By default we are returning a `GameDataBuilder` type from here with a few bundles
  added that you will probably find useful.  You can read more about custom GameData types in
  [The Book](https://www.amethyst.rs/book/latest/). You do not need to provide input or rendering facilities here, as the Amethyst
  core dispatcher will take care of that for you.
  
- `pub fn pipeline_builder(pipeline: PipelineBuilder<Queue<()>>) -> Option<impl PipelineBuild>`
  
  Provides a description of the rendering pipeline.  It is pre-populated with a basic configuration that should
  work for most needs.  If you don't want a window for your game (such as for a headless game server) then you can return `None` here.

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

During compilation the amethyst engine links against a user provided crate which is expected to provide all of the functions listed above.
The new main method we provide calls these functions at appropriate times during initialization of the game and/or editor so as to place
the user provided values into the correct locations during the startup process.

# Drawbacks
[drawbacks]: #drawbacks

Since this is all very non-conventional we'll have to build, maintain, and support our own tooling for this workflow.
Additionally we will have to provide clear and comprehensive learning materials on how to use said tooling.

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

# Prior Art
[prior-art]: #prior-art
<details>
<summary>Discuss previous attempts, both good and bad, and how they relate to this proposal.</summary>
A few examples of what this can include are:

- For engine, network, web, and rendering proposals: Does this feature exist in other engines and what experience has their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other engines, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other engines.
</details>

# Unresolved Questions
[unresolved-questions]: #unresolved-questions
<details>
<summary>Additional questions to consider</summary>

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?
</details>

Copyright 2018 Amethyst Developers
