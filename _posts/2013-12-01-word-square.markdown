---
layout: post
title:  "Word Squares"
date:   2013-12-01 11:49:33
categories: thoughtbot learn
---

So the second week of [thoughtbot's learn prime][learn] study group is coming to an end and we were assigned the problem of writing a ruby application that generates [word squares][wikiwordsquare]. A word square is like a crossword on steroids. Every column and row of said square needs to contain a valid word. Now this isn't too tough for 3x3 or 4x4 squares but there was an added challenge to post your fastest time for a 6x6 square. I decided to wrap mine in rails (my reasoning was that some in the study group mentioning wanting to experience more rails apps) and you can find it [here on github][word]. One of the imaginary constraints I put on myself when writing this was I wanted every available word square to be just as likely to be found as any other word square. This made it important that picking words were random. I then pulled in the [algorithms gem][algorithms] to gain access to a new data object called a [trie][wikitrie]. I then used this to try a optimize lookup of words from a dictionary.

I also test drove this project. It's fun! It allowed me to very quickly refactor my code which was a new experience for me. The second part of this exercise is we should review each others code and make a suggestion on someone's code on how to improve it. I'm looking forward to seeing what the others in Blue Group have come up with.


[learn]: https://learn.thoughtbot.com/
[wikiwordsquare]: http://en.wikipedia.org/wiki/Word_square
[word]: https://github.com/MattMSumner/word_square
[algorithms]: https://github.com/kanwei/algorithms
[wikitrie]: http://en.wikipedia.org/wiki/Trie
