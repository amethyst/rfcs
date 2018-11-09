# Async Support and Futures

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

- Feature Name: async_support
- Start Date: 2018-11-09
- RFC PR: 
- [Tracking Issue](#tracking-issue): 
- [Forum Thread](#forum-discussion): 


# Summary
[summary]: #summary

This RFC is dedicated to the integration of fully async constructs into the synchronized nature of Amethyst.

# Motivation
[motivation]: #motivation
### Why are we doing this?
Amethyst is build with a focus on multithreading, but it isn't true async-ness. There is a loop on the main thread that controls the execution in increments (stepping engine). That means that to modify data, you need to be "in sync" with that execution loop. You cannot modify a resource at any time you want. You have to make sure it is not borrowed and that you are not modifying it during other concurrent modifications. This RFC describes a general way of "syncing" the result of an async computation.
### What use cases does it support?
Anything that can run fully in parallel of the engine's loop.
* Http calls
* Midi controller callbacks
* Network sockets
* Various device events
### What is the expected outcome?
A re-usable and easy way to use such async constructs or callbacks that doesn't take a lot of boilerplate, performance (mutex) or impose a bad programming practice (heavy side-effecting, hardcoded behaviors, etc).

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation
<details>

<summary>Non-technical overview and reasoning for it.</summary>
This change allows to do some action at the end of some async computation. For example, if you are doing a http/POST call using the hyper library, you can get a `Future` back. Shove that future into tokio's reactor, and you got a non-blocking call. Now, however, what if you want to do something once the `Future` completes (or fails)? You can't just write to a resource (which includes entities and storages), because it might be being written at by some `System`.

We need a way to get that result (or side-effects, also called actions) to modify the resources.

There is multiple ways we can achieve this, but I will only be covering the two main methods here.
Both methods rely on the same mechanism: A shared multi-writer queue.

### Option A: 

</details>

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation
<details>
<summary>The technical details and design of the RFC.</summary>
This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.
</details>

# Drawbacks
[drawbacks]: #drawbacks

##### Why should we *not* do this?

If you don't want to be friendly to the user or have more engine-level features I guess.

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