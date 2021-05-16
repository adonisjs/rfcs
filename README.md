# Adonis RFCs
> The process is inspired by the [Vuejs RFC process](https://github.com/vuejs/rfcs). Currently, we are testing out the RFC process and may tweak it as we go along and learn new things.

Many small changes to the existing APIs and bug fixes are usually done without creating any RFC. However, we may ask you to create a formal RFC for modifications of the following nature.

- Adding an entirely new API or module to the ecosystem.
- Changing the behavior of an existing feature.
- Features that are not clear and require more discussion and community involvement.

## Why do you need to do this?

We appreciate you taking out the time and sharing your ideas to improve the framework. However, as AdonisJS is growing and getting used in production, we have to be more responsible and careful with the changes or features we add to the framework.

Also, adding a new feature to the framework core or the official set of packages means creating more work for the framework creator/core team. We may not always be interested in executing new ideas or maintaining them in the long term, as there is always some ongoing work.

With that said, we still encourage you to share your ideas with us and see if we can bring them together to life.

## Taking the process seriously

The RFC process is not only about sharing your wishlist of features. You should do some proper research and create an actionable RFC. 

#### ❌ Bad example of an RFC

In the following example, you are transferring the weight of research and finalizing the implementation level details. ** These kinds of RFCs will be closed without any explanation**.

```
Please add support for filesystem-based routing. It will be helpful for me and others as well.

These are some frameworks that do it.
```

#### ✅ A better example of an RFC

```
- Share the feature you want the framework to have
- Share the basic usage examples
- Share if there are going to be any performance issues by introducing the feature
- If a variation of that API already exists, then have a conversation with the core team and try to understand why they opted for the other variation
- Share design and implementation details
- Share unknows - We will be more than happy to discuss them in-depth with you
- Share know limitations
```

## What the process is
In short, to get a major feature added to AdonisJS, one must first get the
RFC merged into the RFC repo as a markdown file. At that point, the RFC
is 'active' and may be implemented to achieve eventual inclusion
into AdonisJS.

* Fork the RFC repo https://github.com/adonisjs/rfcs

* Copy `0000-template.md` to `active-rfcs/0000-my-feature.md` (where
'my-feature' is descriptive. Don't assign an RFC number yet).

* Fill in the RFC. Put care into the details: **RFCs that do not
present convincing motivation, demonstrate understanding of the
impact of the design, or are disingenuous about the drawbacks or
alternatives tend to be poorly received**.

* Submit a pull request. Make sure to follow the pull request template and open a corresponding discussion thread.

* Build consensus and integrate feedback in the discussion thread. RFCs with broad support are much more likely to make progress than those who don't receive any comments.

* Eventually, the [core team] will decide whether the RFC is a candidate
for inclusion in AdonisJS.

* An RFC can be modified based upon feedback from the [core team] and community. Significant modifications may trigger a new final comment period.

* An RFC may be rejected after the public discussion has settled
and comments have been made summarizing the rationale for rejection. A member of the [core team] should then close the RFC's associated pull request.

* An RFC may be accepted at the close of its final comment period. A [core team] member will merge the RFC's associated pull request, at which point the RFC will become 'active'.

## Details on Active RFCs

Once an RFC becomes active, then authors may implement it and submit the
feature as a pull request to the applicable AdonisJS repo. Becoming 'active' is not a rubber stamp, and in particular, it still does not mean we will ultimately merge the feature; it does mean that the core team has agreed to it in principle and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and 'active' implies nothing about what priority is assigned to its implementation or whether anybody is currently working on it.

We can do modifications to active RFCs in follow-up PRs. We strive
to write each RFC in a manner that will reflect the final design of
the feature; but the nature of the process means that we cannot expect
every merged RFC to reflect what the end result will be at
the time of the next major release; therefore, we try to keep each RFC
document somewhat in sync with the language feature as planned,
tracking such changes via follow-up pull requests to the document.

## Implementing an RFC

The author of an RFC is not obligated to implement it. Of course, the RFC author (like any other developer) is welcome to post an implementation for review after the RFC has been accepted.

An active RFC should have the link to the implementation PR listed if there is one. You should conduct feedback on the actual implementation in the implementation PR instead of the original RFC PR.

If you are interested in working on the implementation for an 'active'
RFC, but cannot determine if someone else is already working on it,
feel free to ask (e.g., by leaving a comment on the associated issue).

## Reviewing RFC's

Members of the [core team] will attempt to review some set of open RFC
pull requests regularly. If a core team member believes an RFC PR is ready to be accepted into active status, they can approve the PR using GitHub's review feature to signal their approval of the RFC.

