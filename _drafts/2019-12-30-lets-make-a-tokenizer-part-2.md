---
layout: post
title: "Let's Make a Tokenizer Part 2"
author: "Daniel Dorado"
date: "Dec 29, 2019"
categories: python tokenization nlp normalization project
---

## Exceptions

So let's get started making the `exceptions.py` file. As you see this, is 
pretty basic. It's just a Python dictionary with a few tokens that, for this
example, we're going to going to treat as exceptions.  The first step in our
tokenization pipeline is to see if it's in this list. If a word is in this 
list we'll add the associated tokens in to our final tokens list. Since
everything here is an exception, the *approved tokens here* will not be
checked against our grammar rules.

Things to consider adding here are more contractions.  This tokenizer isn't
concerned about capitalization so `What's` and `what's` are treated
separately.  I won't worry about it in this example, but feel free to
adjust the tokenizer as needed.

While, I'm not adding too many exceptions for this example, experiment.
What kind of other things might be included in here.

It's not uncommon to have a lexicon for common abbreviations. Like `Mr.`
and `U.K.`. As needs arise you may finding yourself needing to add to
this list.

```
"""
Exception lexicon for tokenizer.
"""

LEXICON = {
    "don't" : ["do", "n't"],
    "isn't" : ["is", "n't"],
    "What's" : ["What", "'s"],
    "I'm" : ["I", "'m"],
}

```

## Next Post

In the next post we'll start building the grammar and write tests to ensure the
rules are encapsulated as intended.