# Amethyst RFCs
[Amethyst RFCs]: #amethyst-rfcs

- [Discussing RFCs](https://github.com/amethyst/rfcs/pulls)
- [Accepted RFCs](https://github.com/amethyst/rfcs/tree/master)
- [RFCs Awaiting Implementation](https://github.com/amethyst/rfcs/issues?utf8=✓&q=label%3A%22RFC%3A+Accepted%22+-label%3A%22Status%3A+Working%22+)

## Opening
[Opening]: #opening

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are 'substantial', and we ask that these be put through a
bit of a design process and produce a consensus among the Amethyst community and
the [sub-team]s.

The "Request for Comments" (RFC) process is intended to provide a consistent
and controlled path for new features to enter the engine and associated
libraries, so that all stakeholders can be confident about the direction in which
the engine is evolving.

You may join our [Discord here](https://discord.gg/amethyst) and the [Amethyst Community Forum](https://community.amethyst-engine.org) here: https://community.amethyst-engine.org.

## Table of Contents
[Table of Contents]: #table-of-contents

- [Opening]
- [Table of Contents](#table-of-contents)
- [When you need to follow this process](#when-you-need-to-follow-this-process)
  - [Sub-team Specific Guidelines](#sub-team-specific-guidelines)
- [Before creating an RFC](#before-creating-an-rfc)
- [What the process is](#what-the-process-is)
- [Tracking Issues](#tracking-issues)
  - [Labels and Tags](#labels-and-tags)
- [The RFC life-cycle](#the-rfc-life-cycle)
  - [Reviewing RFCs](#reviewing-rfcs)
  - [Implementing an RFC](#implementing-an-rfc)
  - [RFC Postponement](#rfc-postponement)
- [Help this is all too informal!](#help-this-is-all-too-informal)
- [License](#license)

## When you need to follow this process
[When you need to follow this process]: #when-you-need-to-follow-this-process

You need to follow this process if you intend to make "substantial" changes to
Amethyst, its component crates, or the RFC process itself. The definition of substantial
is somewhat fluid, and to an extent set by the community. Some examples might include:

- A change to a different crate used for physics calculations
- Changes to how the editor communicates with the engine
- Significant protocol changes to the networking library
- Anything that has the potential to produce a substantial negative impact on users

Some changes do not require an RFC:

- Rephrasing, reorganizing, refactoring, or otherwise "changing shape does
  not change meaning".
- Additions that strictly improve objective, numerical quality criteria
  (warning removal, speedup, better platform coverage, more parallelism, trap
  more errors, etc.)
- Additions only likely to be _noticed by_ other developers-of-amethyst,
  invisible to users-of-amethyst.
- More tests and benchmarks.
- Convenience functions or quality of life improvements that do not affect
  current functionality or backwards compatibility.

If you submit a pull request that ends up being generating a lot of discussion or
pushback, you may be asked to submit an RFC. What seems to be a minor change can
have unintended consequences other teams might see.

### Sub-team specific guidelines
[Sub-team specific guidelines]: #sub-team-specific-guidelines

For more details on when an RFC is required for the following areas, please see 
the Amethyst community's [sub-team] specific guidelines for:

- [Networking changes](net_changes.md)
- [Renderer changes](rend_changes.md)
- [Engine changes](engine_changes.md)
- [Web changes](web_changes.md)

## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

A hastily-proposed RFC can hurt its chances of acceptance. Low quality
proposals, proposals for previously-rejected features, or those that don't fit
into the near-term roadmap, may be quickly rejected, which can be demotivating
for the unprepared contributor. Laying some groundwork ahead of the RFC can
make the process smoother.

Although there is no single way to prepare for submitting an RFC, it is
generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the RFC may be desirable; having a consistent
impact on the project requires concerted effort toward consensus-building.

The most common preparations for writing and submitting an RFC include talking
the idea over on the [Discord channel](https://discord.gg/dqe9UYE), specifically
with the applicable sub-team and discussing it on our [Amethyst Community Forums](https://community.amethyst-engine.org)

We welcome all ideas and strive to provide kind but useful feedback. While not
every proposal will make it through, that should in no way discourage anyone
from continuing to make proposals in the future.

If the proposal is promising and someone is new to the RFC process, we will
do our best to find someone to mentor them through the process.

## What the process is
[What the process is]: #what-the-process-is

In short, to get a major feature added to Amethyst, one must first get the RFC
merged into the RFC repository as a markdown file. At that point the RFC is
approved and may be implemented with the goal of eventual inclusion into Amethyst.

- Fork the RFC repo [RFC repository]
- Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is
  descriptive. Don't assign an RFC number yet).
- Fill in the RFC. Put care into the details: RFCs that do not present
  convincing motivation, demonstrate understanding of the impact of the
  design, or are disingenuous about the drawbacks or alternatives tend to be
  poorly-received.
- Submit a pull request. As a pull request the RFC will receive design
  feedback from the larger community, and the author should be prepared to
  revise it in response.
- Each pull request will be labeled with the most relevant [sub-team], which
  will lead to its being triaged by that team in a future meeting and assigned
  to a member of the subteam.
- Build consensus and integrate feedback. RFCs that have broad support are
  much more likely to make progress than those that don't receive any
  comments. Feel free to reach out to the RFC assignee in particular to get
  help identifying stakeholders and obstacles.
- The sub-team will discuss the RFC pull request, as much as possible in the
  comment thread of the pull request itself. Offline discussion will be
  summarized on the pull request comment thread.
- RFCs rarely go through this process unchanged, especially as alternatives
  and drawbacks are shown. You can make edits, big and small, to the RFC to
  clarify or change the design, but make changes as new commits to the pull
  request, and leave a comment on the pull request explaining your changes.
  *Specifically, do not squash or rebase commits after they are visible on the
  pull request.*
- At some point, a member of the subteam will propose a "motion for final
  comment period" (FCP), along with a *disposition* for the RFC (merge, close,
  or postpone).
  - This step is taken when enough of the tradeoffs have been discussed that
  the subteam is in a position to make a decision. That does not require
  consensus amongst all participants in the RFC thread (which is usually
  impossible). However, the argument supporting the disposition on the RFC
  needs to have already been clearly articulated, and there should not be a
  strong consensus *against* that position outside of the subteam. Subteam
  members use their best judgment in taking this step, and the FCP itself
  ensures there is ample time and notification for stakeholders to push back
  if it is made prematurely.
  - For RFCs with lengthy discussion, the motion to FCP is usually preceded by
    a *summary comment* trying to lay out the current state of the discussion
    and major tradeoffs/points of disagreement.
  - Before actually entering FCP, *all* members of the subteam must sign off;
  this is often the point at which many subteam members first review the RFC
  in full depth.
- The FCP lasts ten calendar days, so that it is open for at least 5 business
  days. It is also advertised widely,
  e.g. in [This Week in Amethyst](https://amethyst.rs/blog). This way all
  stakeholders have a chance to lodge any final objections before a decision
  is reached.
- In most cases, the FCP period is quiet, and the RFC is either merged or
  closed. However, sometimes substantial new arguments or ideas are raised,
  the FCP is canceled, and the RFC goes back into development mode.

## Tracking Issues
[tracking-issues]: #tracking-issues
These are issues created on GitHub that track the implementation progress of the RFC.
They link to related issues, have checkboxes, or other structures that make them more
of a meta-issue than an issue for adding a specific feature or fixing a bug.

This issue is one the sub-team will create, not one you need to create.

## Labels and Tags
[labels-tags]: #labels-and-tags

We use the following tags for RFC tracking issues:

- RFC: Proposed
- RFC: Accepted
- RFC: Declined
- RFC: Complete
- RFC: Postponed

See the life-cycle section below for details on what these mean.

## The RFC life-cycle
[The RFC life-cycle]: #the-rfc-life-cycle

When an RFC is first created, a tracking issue is created and given the label
"RFC: Proposed".

Once an RFC becomes approved, the tracking issue will be tagged with the label
"RFC: Accepted". Authors may implement it and submit the feature as a pull
request to the Amethyst repo. Being approved is not a rubber stamp, and in
particular still does not mean the feature will ultimately be merged; it
does mean that in principle all the major stakeholders have agreed to the
feature and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is approved
implies nothing about what priority is assigned to its implementation, nor does
it imply anything about whether an Amethyst developer has been assigned the task of
implementing the feature. While it is not *necessary* that the author of the
RFC also write the implementation, it is by far the most effective way to see
an RFC through to completion: authors should not expect that other project
developers will take on responsibility for implementing their accepted feature.

Modifications to approved RFCs can be done in follow-up pull requests. In
order for a PR that deviates from the RFC standard, a new RFC that follows
the accepted guidelines should be written and contain a link to the prior version.

We strive to write each RFC in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect every
merged RFC to actually reflect what the end result will be at the time of the
next major release.

In general, once accepted, RFCs should not be substantially changed. They should
be as detailed as possible so that only very minor changes are needed as amendments.
More substantial changes should be new RFCs, with a note added to the original RFC.
Exactly what counts as a "very minor change" is up to the sub-team to decide; check
[Sub-team specific guidelines] for more details.

### Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC pull request is up, the sub-team may schedule meetings with the
author and/or relevant stakeholders to discuss the issues in greater detail,
and in some cases the topic may be discussed at a sub-team meeting. In either
case a summary from the meeting will be posted back to the RFC pull request.

A sub-team makes final decisions about RFCs after the benefits and drawbacks
are well understood. These decisions can be made at any time, but the sub-team
will regularly issue decisions. When a decision is made, the RFC pull request
will either be merged or closed. In either case, if the reasoning is not clear
from the discussion in thread, the sub-team will add a comment describing the
rationale for the decision. In the case of the RFC being closed, the tag
"RFC: Declined" will be added to the tracking issue. If it is merged, it will
receive the tag "RFC: Accepted".

### Implementing an RFC
[Implementing an RFC]: #implementing-an-rfc

Some accepted RFCs represent vital features that need to be implemented right
away. Other accepted RFCs can represent features that can wait until some
arbitrary developer feels like doing the work. Every accepted RFC has an
associated issue tracking its implementation in the Amethyst repository; thus that
associated issue can be assigned a priority via the triage process that the
team uses for all issues in the Amethyst repository.

The author of an RFC is not obligated to implement it. Of course, the RFC
author (like any other developer) is welcome to post an implementation for
review after the RFC has been accepted. A prototype or other supporting
materials (data flow diagrams, architecture diagrams, aggregations of
posts from the community) can all help demonstrate the need and the
priority of the RFC.

If you are interested in working on the implementation for an approved RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

Once an RFC has been implemented, the tracking issue will be closed and given
the label "RFC: Complete".

### RFC Postponement
[RFC Postponement]: #rfc-postponement

Some RFC pull requests are tagged with the "RFC: Postponed" label when they are
closed (as part of the rejection process). An RFC closed with "RFC: Postponed" is
marked as such because we want neither to think about evaluating the proposal
nor about implementing the described feature until some time in the future, and
we believe that we can afford to wait until then to do so. Postponed pull
requests may be re-opened when the time is right. We don't have any formal
process for that, you should ask members of the relevant sub-team.

Usually an RFC pull request marked as "RFC: Postponed" has already passed an
informal first round of evaluation, namely the round of "do we think we would
ever possibly consider making this change, as outlined in the RFC pull request,
or some semi-obvious variation of it." (When the answer to the latter question
is "no", then the appropriate response is to close the RFC, not postpone it.)

### Help! This is all too informal
[Help this is all too informal!]: #help-this-is-all-too-informal

The process is intended to be as lightweight as reasonable for the present
circumstances. As usual, we are trying to let the process be driven by
consensus and community norms, not impose more structure than necessary.

[Amethyst Community Forums](https://community.amethyst-engine.org)

## License
[License]: #license

All code in repositories under the amethyst/ organization is Apache 2.0 or MIT
licensed while all assets are Creative Commons Attribution 4.0 licensed if not otherwise stated.

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)
- CC BY-SA 4.0 ([LICENSE-CCBYSA](LICENSE-CCBYSA) or https://creativecommons.org/licenses/by-sa/4.0/)

### Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
