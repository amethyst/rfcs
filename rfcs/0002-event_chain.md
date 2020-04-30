# Table of Contents

- [Motivation]
- [Guide Level Explanation](#guide-level-explanation)
- [Reference Level Explanation](#reference-level-explanation)
- [Drawbacks]
- [Rationale and Alternatives](#rationale-and-alternatives)
- [Prior Art](#prior-art)
- [Unresolved Questions](#unresolved-questions)

# Basic Info
[basic]: #basic-info

- Feature Name: event_chaining
- Start Date: 2020-04-30
- RFC PR: (leave this empty until a PR is opened)

# Summary
[summary]: #summary

How we can use event chaining as a way to decouple our different
modules and Systems.

# Motivation
[motivation]: #motivation

- Reduced coupling
- More configurability
- Better separation of concerns
- More!

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation
<details>

[Link to paper](https://www.jojolepro.com/event_chaining.pdf)

</details>

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation

[Link to paper](https://www.jojolepro.com/event_chaining.pdf)

# Drawbacks
[drawbacks]: #drawbacks

[Link to paper](https://www.jojolepro.com/event_chaining.pdf)

# Rationale and Alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

[Link to paper](https://www.jojolepro.com/event_chaining.pdf)

# Prior Art
[prior-art]: #prior-art
<details>
<summary>Discuss previous attempts, both good and bad, and how they relate to this proposal.</summary>
The currently EventRetrigger type in Amethyst was the first proof of concept we did at implementing event chains.
We only used it in the UI crate to get a basic idea of how this can work.

After this, I implemented an example UiDriver in a [fork](https://github.com/jojolepro/amethyst/tree/eventchain) and it worked just fine.
</details>

# Unresolved Questions
[unresolved-questions]: #unresolved-questions
<details>
<summary>Additional questions to consider</summary>
The main unresolved question that remains is how we will make
the Systems/Drivers register to the EventChannels.

We have the possibility of doing it automatically in a SystemDesc,
but this doesn't work with Pausable Systems.
</details>

Copyright 2020 Joël Lupien
