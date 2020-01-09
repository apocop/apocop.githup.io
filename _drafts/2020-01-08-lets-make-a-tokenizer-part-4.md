---
layout: post
title: "Let's Make a Tokenizer Part 4"
author: "Daniel Dorado"
date: "Jan 09, 2020"
categories: python tokenization nlp normalization project
---

### Tokenizer

Congratulations for making it to the last post in making a tokenizer. Let's
jump right in. And bring in our imports.  We'll be making use of the `re`
library along to the `exceptions.py` and `grammar.py` we made earlier.

```
import re
import exceptions
import gramma
```

Let's define a tokenizer class scaffolding and outline the class methods. 
Generally *methods* are functions that defined inside a class.

```
class Tokenizer:
    def __init__(self):
        pass

    def __add_exception(self):
        pass

    def __tokenize_pipeline(self):
        pass

    def tokenize(self):
        pass
```


< The double-underscore in `__tokenize_pipeline` indicate that they are private
methods. Whoever uses this class should only use the `tokenize` method. Private
just means they are for internal use inside the class. Let's look into these
methods a little more deeply.

* `__tokenize_pipeline` - Check each token against the rule, breaking it up if
it doesn't pass and recursively call itself on the smaller tokens.

* `tokenize` The only public function. Returns a list of tokens.


### Instance Variables.

So let's modify the the `__init__` method create a few variables on
instantiation. 

```
def __init__(self):
    self.rules = grammar.RULES
    self.rule_list = list(grammar.RULES)
    self.exception_lexicon = exceptions.LEXICON
    self.accepted_tokens = None
```

Notice we're importing our grammar rules and exceptions list lexicon into the
tokenizer.  We also create a variable that will eventually hold our tokens,
`accepted_tokens`.  Accepted meaning, that the token either was in our lexicon
of exceptions or passed every rule defined in our grammar. Notice that we also
generate a list from our grammar rules. This list is in order the order they
were added, thus the order in which they will be applied.

### Tokenize

Let's build our tokenizer function. The reason for the reassignment of
`self.accepted_tokens` as an empty list is to clear the *cache* of tokens so
that when called in succession you there won't be tokens from previously
processed documents. 

```
def tokenize(self, string):
    self.accepted_tokens = []
    tokens = (token for token in string.split())
    for token in tokens:
        self.__tokenize_pipeline(token)

    return self.accepted_tokens
```

`tokens = (token for token in string.split())` is a pretty interesting. It may
seem at first glace that this just a typical list comprehension, but notice 
than rather than square brackets `[]`, parenthesis are used.  It's a 
generator comprehension. A list comprehension would immediately split the
entire string and load it into memory.  A generator comprehension on the other
hand makes use of lazy loading. A generator yields a single token when it's
needed so extraneous memory is not used. As documents can range in size, this
also us to save memory, which will help prevent the tokenizer from slowing. See
[Reduce Memory Usage and Make Your Python Code Faster Using Generators](https://towardsdatascience.com/reduce-memory-usage-and-make-your-python-code-faster-using-generators-bd79dbfeb4c)
for more information on generators in Python. We call the our NLP pipeline on
every token in the generator.  Lastly we return the `accepted_tokens`.




# Suggestions for Future Edits

1. Add support for borrowed words like café.  How many diacritics does
English usually import?

2. Allow for English plural possessives.