---
layout: post
title: "Tokenization"
author: "Daniel Dorado"
date: "Dec 21, 2019"
categories: python tokenization nlp normalization
---

### Introduction

Tokenization is an import step in the NLP pipeline. It is often part of the text
normalization process. Many text transformations can't can't be done until the
text is tokenized. This post will examine what is tokenization and its challenges.

## Tokenization

A **token** is a sequence of
characters.  For example, `cat`, `...`, `;)`, `What's`, `R.S.V.P.` are all tokens.
Notice that tokens aren't necessarily *words* like `:)`. This allows us to say 
that all words are tokens, but not all tokens are words. However, functionally
most tokens are words.  The token, `"What?!!!!!` may not be 
useful in its own right, but `"` `What` `?` `!` `!` `!` `!` `!`  might be.
So **tokenization** is the process of splitting a text into tokens (essentially words).


For example `hi how are you?` would become `hi` `how` `are` `you` `?`.


At first glance, this may seem pretty simple.  The string `split()` method
in Python easily splits text into tokens. You might have to knaw off some
punctuation, but it's essentially done.

`Hello, how are you?` in Japanese (according too Google translate) is
`こんにちは、元気ですか？` Japanese doesn't use spaces like English does! This like
`Hello,howareyou?`  This is one loooong token! Punctuation helps a little to
tease tokens apart: `こんにちは` `、` `元気ですか` `？` or `Hello` `,` `howareyou` `?`,
however this still doesn't give you accurate tokens. Often tokenization for these
languages have to be manually done for best results. Although there are a few
algorithms such as (Maximum Matching)[https://medium.com/@anshul16/maximum-matching-word-segmentation-algorithm-python-code-3444fe4bd6f9]
which are successful at tokenizing this kind of text depending on the language.
Languages with shorter words seem to do better. Some languages like Ancient 
Greek and Latin don't use punctuation.  Good thing you probably won't work with
these languages. 

### Ambiguity and Arbitrary Decisions

Sometimes there might be ambiguity as to what constitutes a word.

```
You shouldn't've done that!
```
Is `shouldn't've` one word, two words or three, `should` `not` `have`, what
about `should` `n't` `'ve`?


To properly tokenize text requires good knowledge of the language.  It also
requires making arbitrary decisions. Do we want to consider `what's` as
a single word or two words?  In the end, a decision has to be made based
upon project needs.

Another example is `gonna`. Clearly this word is a single token, but do we 
want it to be? We might also split this as `gon` and `na`.  How might this
affect Part of Speech tagging?

### Punctuation 

In English some tokens require punctuation, `E.T.A.` for example. So if
punctuation in split. You might get an erroneous tokenization of `E.T.A` `.`
or something odder like `E` `.` `T` `.` `A` `.`.  Sometimes the punctuation
may be fulfilling two roles. `I like you Mr.` And what about 
[fo'c's'le](https://en.wikipedia.org/wiki/Forecastle)? Obviously very
important.

#### That Apostrophe S

The `'s` has two meanings in English. It can either signal a contraction or
the genitive clitic. 

```
1. The book is John's.
2, No, it's the woman with the green hat's.
3. But the book's been in the library's collection for a while now.
```
#### Final Apostrophe

Also to note is that English possessive plurals require an apostrophe. `This
is the libraries' book.` I guess this means we can't strip off all 
punctuation...

### Internal punctuation.

Not all languages use punctuation in the same way.  A personal favorite is
[Armenian Tonal Punctuation](https://www.youtube.com/watch?v=vu9_QB9gqTQ),
where punctuation (like !, ? and :) is attached to the syllable in the word that you would like
to emphasize. For example with the question mark: (I'll be using English
English to illustrate the example.)


Standard English punctuation:

What are you doing?

Armenian equivalents:
1. **Wha?t** are you doing.
2. What **a?re** you doing.
3. What are **you?** doing.
4. What are you **do?ing**.

While this articulates precisely what the writer is wanting to emphasize, it
makes processing text more interesting. How should `do?ing` be tokenized. 
You have interwoven tokens. Should `doing` and `do?ing` be considered separate
tokens? Something tells me that autocorrect in Armenian is probably more
frustrating than with other languages.


### Conclusion

By now I hope you're thinking about how you want to deal with tokenization on
for some of your projects.  Know of interesting tokenization situations. Feel
to post them down below in the comments.