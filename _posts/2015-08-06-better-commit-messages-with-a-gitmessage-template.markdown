---
layout: post
title: "Better Commit Messages with a .gitmessage Template"
date:   2015-08-06 11:53:09
tags: general git
excerpt: "Bring consistency to the structure of your pull requests."
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/better-commit-messages-with-a-gitmessage-template)*

It can be tough for your team to know exactly why they need the changes you're
proposing in a pull request.
What if we could give them more context?
There's an episode of [The Weekly Iteration] where a simple trick was mentioned.
After adopting it myself, I have noticed my co-workers commenting on how useful
these messages have been in providing context.

[The Weekly Iteration]: https://thoughtbot.com/upcase/videos/tips-for-code-review

The trick is simple. Split your PR message into two sections:

    Some awesome title

    Why:

    * ...

    This change addresses the need by:

    * ...

Starting with this structure forces you to answer why the change is necessary
before outlining the changes that you have made towards this goal.

## Automation!

We can tell git to setup our commit messages with this structure.
This is done by setting `commit.template` in `~/.gitconfig`:

```bash
[commit]
  template = ~/.gitmessage
```

Then create `~/.gitmessage` with our new default:

```gitmessage
Why:

*

This change addresses the need by:

*

# 50-character subject line
#
# 72-character wrapped longer description.
```

## Want more?

* This also sets up for a great commit message, check out [5 Useful Tips For
  A Better Commit Message]
* RailsConf 2015: [Implementing a Strong Code-Review Culture]

[Implementing a Strong Code-Review Culture]: https://www.youtube.com/watch?v=PJjmw9TRB7s
[5 Useful Tips For A Better Commit Message]: https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message
