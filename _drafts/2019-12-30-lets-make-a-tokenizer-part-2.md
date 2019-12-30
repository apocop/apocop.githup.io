---
layout: post
title: "Let's Make a Tokenizer Part 2"
author: "Daniel Dorado"
date: "Dec 29, 2019"
categories: python tokenization nlp normalization project
---

## Exceptions

So let's get started making the `exceptions.py` file. As you see mine is 
pretty basic. It's just a Python dictionary with a few tokens that, for this
example, we're going to going to treat irregularly.  The first step in or
tokenization pipeline is to see if it's in this list.  Notice that the value is
of each entry is a list. We'll iterate through each item and add it to our final
list of tokens. Things you add here will not checked against the rules we define
in our grammar. 

This to consider adding here are more contractions.  I'm not too concerned about, 



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