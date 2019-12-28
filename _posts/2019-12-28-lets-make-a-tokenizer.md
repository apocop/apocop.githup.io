---
layout: post
title: "Let's Make a Tokenizer Part 1"
author: "Daniel Dorado"
date: "Dec 28, 2019"
categories: python tokenization nlp normalization project
---

## Introduction

So in this post, we'll start making a simple English tokenizer in pure Python.
Simple in the sense that we're not going to try and aim for perfect
tokenization, but the most common kinds of tokens we might see. In preparation,
I suggest you might want to brush up on [Python regular expressions](https://docs.python.org/3/library/re.html)
since we'll be dealing with them here.  We will be building a tokenizer class, so
hopefully you'll be comfortable with [classes](https://docs.python.org/3/tutorial/classes.html).
Feel free to peruse the [final code](https://github.com/apocop/NLP-Notebook-Examples/tree/master/2019-12-23-lets-build-a-tokenizer)
for this project.

### Set up the project.

First step is to create a new folder and add the following python files:

```
1. tokenizer.py
2. grammar.py
3. exceptions.py
4. test_tokenizer.py
5. demo.py
```

Honestly, we could probably get away with just two files. One for tests, and
one for everything else, but I'd like to make this extendable to some degree.
Meaning you could build multiple grammars for various languages... or purposes
and easily import it into your tokenizer.

#### File Overview

Let's take a moment to get acquainted with our files.

#### Tokenizer
The tokenizer file is the main file. Here we'll build the tokenizer object,
and its internal logic.

#### Grammar
The grammar file is where we'll create the *rules* describing which kinds of
tokens are **NOT** allowed. For example `"hi` or `you",` should not be allowed.
These rules will be declared with regular expressions. However, since regexes
can be difficult to build and decipher as they get longer, we'll build them up
piece by piece into manageable parts with various test to make sure they work
as intended.

#### Exceptions
Exceptions will hold a lexicon for tokens where we want a custom tokenization.
For example, `isn't` may map to `is` `n't`.  It might also be a good idea to
hold common abbreviations in the exceptions lexicon. Ours will only hold a 
couple for the example, but feel free to fill yours up. 

#### Test_Tokenizer
We'll be using the [Pytest](https://docs.pytest.org/en/latest/) framework to
test our grammar rules and tokenizer. This may seem somewhat unnecessary for
a toy example, but it will allow us to see just how good or bad or tokenizer
preforms. And should we make any changes to our grammar to work better, we'll
need to verify the change works with tests, and that there are no unintended
consequences. The more tests the better!

#### Demo

This file will be a simple file where we'll import our tokenizer and see it
in action, but running it over several sentences. This file will demo our
tokenizer.

#### Next Post

In the next post we'll start with adding our exceptions.