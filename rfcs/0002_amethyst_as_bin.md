# Table of Contents

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
- RFC PR: https://github.com/amethyst/rfcs/pull/6
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
There are four areas of the software that stand to benefit from this

1. Initialization. If we control the main engine initialization sequence then we can avoid handing control over to the user
before the engine is fully initialized.  The biggest benefit this has is that user provided `System::setup()` methods can use
`!Default` resources.  Most commonly, these resources come from the rendering system, such as `ScreenDimensions`.  In order to
gain the most benefits from this we are going to have to introduce a "core dispatcher" concept to the engine.
The core dispatcher would be responsible for input, and rendering, both of which would be optional.  If neither are enabled
then we simply don't run the core dispatcher.

2. Editor. One could imagine the user crate as a pluggable set of code into a variety of possible amethyst executables.
One of these executables could be the standalone engine, and another one could be a scene editor.  This would allow us to provide a
"play" button that starts executing the user code, and since we control the flow of execution now we can also "stop" gameplay
whenever we feel.

3. Scripting. Part of our executable can be identifying, incorporating, and executing code written in foreign programming languages. 
Since we control the flow of execution here we are no longer dependent on the user authoring Rust code in order to identify and
execute their scripts. In theory, you could use Amethyst without needing to know Rust at all.

4. Crossplatform compilation?  It's unclear if this benefit would be achievable as we don't have a proof of concept but it's possible
that this model would allow us to compile a game out to Apple devices without actually owning an Apple device.  It's worth noting
this feature only really matters to hobbyist users, as professionals are going to insist on QAing their build for which they'll still
need actual player hardware.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

When generating a new Amethyst project you'll find a file in your project called `lib.rs`. In this file you'll find a structure called 
`Game` implementing a trait called `GameInterface`.  In this trait will be several functions and types Amethyst depends on you to 
provide.

Additionally you'll find a function called `init_game() -> impl GameInterface` which is responsible for initializing the `Game` object 
Amethyst uses. Do not alter the name, or signature of this function, it is the bridge between your crate and Amethyst.  You can however, 
alter the body of it if you need to do more setup.

Here are the functions and types we expect you to provide:

- `type GameData`

  This is the type used to store your system dispatchers and any thread local data you may have.  We've pre-populated it with a
  simple to use type from Amethyst called `GameData` but you may want to provide your own type here later when your game gets
  more complicated.
  
- `type Event`

  This is the type of event that Amethyst will send to your states.  We've populated it with a default called `StateEvent`.

- `fn first_state() -> impl State<Self::GameData, Self::Event>`
  
  Provides the first state that your game will execute. In the template file we've provided a basic empty state
  and we are returning it in this function.
  
- `fn input_bindings() -> Option<String>`

  Expected to provide either a path to a file in the RON format describing the input bindings for the player, or 
  the input configuration as a string provided in the RON format. It is pre-populated with the path of an a simple file provided
  in the template. If you don't wish to use the Amethyst input system you can return `None` here instead.
  
- `fn game_data() -> impl DataInit<GameData>` 
  
  Provides the core of your game's parallelism.  By default we are returning a `GameDataBuilder` type from here with a few bundles
  added that you will probably find useful.  You can read more about custom GameData types in
  [The Book](https://www.amethyst.rs/book/latest/). You do not need to provide input or rendering facilities here, as the Amethyst
  core dispatcher will take care of that for you.
  
- `fn pipeline_builder(pipeline: PipelineBuilder<Queue<()>>) -> Option<impl PipelineBuild>`
  
  Provides a description of the rendering pipeline.  It is pre-populated with a basic configuration that should
  work for most needs.  If you don't want a window for your game (such as for a headless game server) then you can return `None` here.
  
- `fn post_exit()`

  This function will be executed after your game quits, and the world has been unloaded.  Most games won't need to do anything here but 
  if you, for example need to trigger an upload to a cloud save after the game is over this is the place to do it.

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

During compilation (triggered via a hypothetical cli command `amethyst run`) the amethyst engine links an engine binary against a user 
provided crate which is expected to provide the interface detailed above. The new main method we provide calls these functions at 
appropriate times during initialization of the game and/or editor so as to place the user provided values into the correct locations 
during the startup process.

# Drawbacks
[drawbacks]: #drawbacks

Since this is all very non-conventional we'll have to build, maintain, and support our own tooling for this workflow.
Additionally we will have to provide clear and comprehensive learning materials on how to use said tooling.

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

The primary reason to consider this workflow is that by taking back `fn main()` we can build out features such as an editor and 
scripting in a way that doesn't require our users to be well versed on how to interface their game with our editor.  With this approach
we can provide a simple "launch in editor" button and/or command rather than asking users to alter their code to launch in our editor.

The major alternative is to leave things as they are.  While it's a little easier to maintain this way, we're somewhat limited in
what we can do as a lib. Several people have commented on the issue tracker before "libs shouldn't do this" or "libs shouldn't do that" 
in response to us doing things that are very reasonable for a game engine to need to do. So my takeaway is that we shouldn't be a lib.

# Prior Art
[prior-art]: #prior-art

Some software already using this model is Unreal Engine 4, Unity 3D, Godot, and CryEngine.

Some software not using this model is MonoGame, SDL2, GLFW, and the Ogre rendering engine.

Use of this model seems to be the major distinction between game frameworks and game engines.  Engines are executables, frameworks are
libs.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

- Is the provided rough draft of interface functions sufficient?  Should we add more?  Should we change the signature of the existing
ones?

- What would it look like when a user needs to upgrade the version of Amethyst that they're using?  We can't just use cargo anymore
if we're going with this approach.  Maybe we could keep most of the engine as a lib and provide amethyst_bin projects that use the
amethyst crate on cargo, as well as hooking into a user provided "game crate"?

- Will all user provided crates have the same name?  Should we attempt to build a system that supports dynamic user crate names?

Copyright 2018 Amethyst Developers
