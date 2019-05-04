---
layout: post
title: "Shell Script Suggestions for Speedy Setups"
date: 2017-01-12 15:55:30
tags: shell productivity happiness internal-tools
excerpt: >
  Is your team setup for success? Do they have the tools to deliver working
  software? Step zero is being able to get setup quickly.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/shell-script-suggestions-for-speedy-setups)*

We aim to rotate onto new projects every 2-4 months. This gives us the benefit
of fresh eyes on each project every so often. Every time we rotate someone new
onto a project, they bring new energy and make a positive impact on aspects that
the current team have become complacent to.

Getting a new developer setup quickly is important for making sure they feel
equipped to succeed. If getting a laptop setup with your codebase takes more
then 30 minutes on average then there is room to improve!

Ideally, if I already have all the application's dependencies this should take
closer to 5 minutes. Thirty minutes is attainable for most products and any
longer is going to directly affect my first impressions of the codebase. I
always want a new team member's introduction to the product to be a joyful one.

We've written before about having a [`bin/setup` script][bin-setup] to automate
this process. There's even a default `bin/setup` script shipped as part of all
new rails applications. Here are some tips I've recently learnt to improve first
time setups.

[bin-setup]: https://thoughtbot.com/blog/bin-setup

## The Rule of Silence

Be respectful of everyone's time and attention. [The rule of silence][silence]
is simply that "when a program has nothing surprising, interesting or useful to
say, it should say nothing". I want as little information as possible when
everything is working as expected. I aim for the following examples to fulfil on
this ideal.

[silence]: http://www.linfo.org/rule_of_silence.html

## Dependencies

Every app has dependencies, be it Elasticsearch, Redis or even a certain
version of Node.js. Let us start by making sure these are installed or prompt
our script user to get them:

```sh
#!/bin/sh

# Exit if any subcommand fails
set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NO_COLOR='\033[0m'
CLEAR_LINE='\r\033[K'

printf "[1/6]🔎   checking dependencies"

if ! command -v node > /dev/null; then
  printf "${CLEAR_LINE}💀${RED}   You must install node on your system before setup can continue${NO_COLOR}\n"
  printf "ℹ️   On macOS🍎 you should 'brew install node'\n"
  exit -1
fi

if [[ $(node --version) != "v4.6.0" ]]; then
  printf "${CLEAR_LINE}⚠️${YELLOW}   You are not using a known working version of node.${NO_COLOR}\n"
  printf "ℹ️   This might not be a problem but if you're having issues, try installing 4.6.0\n"
  printf "[1/6]🔎   checking dependencies"
fi
```

Checkout [this excellent description][not-which] of why we use `command -v`
instead of `which`. This script checks if you have node installed. If not,
you're prompted to get it. If you have it installed it will then check the
version and leave a helpful warning if your local version doesn't match the
known working version.

[not-which]: https://stackoverflow.com/questions/592620/check-if-a-program-exists-from-a-bash-script/677212#677212

### A Side Note on Joy

You'll notice two elements in this script to improve readability.

The first is colors! Colors are a great way to indicate which output I need to
care about. Green generally means all is well. Yellow is a potential issue and
should be avoided at your own peril. And finally, Red states that there's a
problem.

The second is the careful and thoughtful application of emoji. These powerful
characters have the ability to punctuate the message your trying to convey.
Notice the lack of color on lines that are purely informational. By tactfully
prefixing these lines with ℹ️ we allow users to quickly understand the essence of
the message.

A word of warning: despite popular opinion, emoji can be overused. I can give no
advice when that line is crossed but I suggest you follow your ❤️.

## Installing Packages/Gems/Addons

Once we've determined that all dependencies are installed we can automate
installing packages:

```sh
#!/bin/sh

# Exit if any subcommand fails
set -e

CLEAR_LINE='\r\033[K'

printf "${CLEAR_LINE}[2/6]⏳   Installing yarn packages"

yarn > /dev/null
```

There are many tools that do not follow the rule of silence. To fix this we can
redirect the output to `/dev/null`. This is to keep our setup as noiseless as
possible. This pattern should be applied on a case by case basis. It is
sometimes desirable to allow the tool you're using to give feedback but often
there is a lot more being output then I'm interested in. `> /dev/null` will only
redirect output to stdout, program errors are written to stderr and so will
still be output.

## Your Special Snowflake Specific Steps

Let's say you have something very specific to your application such as cloning
internal dependencies. You might write a script to pull down all the required
repos:

```sh
#!/bin/sh

# Exit if any subcommand fails
set -e

DIR=$PWD

. ./bin/colors
CLEAR_LINE='\r\033[K'

function clone {
  printf "⏳   $1 is being cloned"
  cd $DIR/..

  if [ ! -d $1 ]; then
    # don't stop the script if there is an error cloning from github
    set +e
    git clone $2 > /dev/null

    # if the last command gives a non-zero exit code we should warn the user
    if [[ $? != 0 ]]; then
      printf "${CLEAR_LINE}❌${RED}   $1 failed to clone! You might not have permissions.${NO_COLOR}\n"
    fi
    set -e
  fi

  cd $DIR
}

clone dependency_one git@github.com:my_org/dependency_one.git
clone dependency_two git@github.com:my_org/dependency_two.git
```

In this case I don't want to stop the script from completing, but I do want the
user to know they do not have a complete setup. We're using `set +e` to make
sure the script doesn't stop if it hits and error and then manually checking for
a non-zero exit code.

## Pulling it all together

![](https://images.thoughtbot.com/blog-vellum-image-uploads/CNwHzunvSMe2jJBjtf3T_bin:setup.gif)

I like to continue this pattern of having scripts with a single responsibility
and calling them all together in `bin/setup`:

```sh
#!/bin/sh

# Exit if any subcommand fails
set -e

CLEAR_LINE='\r\033[K'

. ./bin/colors

./bin/setup_steps/check_dependencies
./bin/setup_steps/install_plugins
./bin/setup_steps/setup_ssl
./bin/setup_steps/pull_environment_vars_from_heroku

printf "${CLEAR_LINE}[6/6]🎉${GREEN}   Finished!${NO_COLOR}\n"
```
