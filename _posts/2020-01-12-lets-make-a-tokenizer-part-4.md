---
layout: post
title: "Let's Make a Tokenizer Part 4"
author: "Daniel Dorado"
date: "Jan 09, 2020"
categories: python tokenization nlp normalization project
---

Congratulations on making it to the last post in making a tokenizer. Let's
jump right in and bring in our imports.  We'll be making use of the `re`
library along to the `exceptions.py` and `grammar.py` files we made earlier.

```
import re
import exceptions
import grammar
```

Let's define a tokenizer class scaffolding and outline the class methods. 
Generally *methods* are functions that are defined inside a class.

```
class Tokenizer:
    def __init__(self):
        pass

    def __tokenize_pipeline(self):
        pass

    def tokenize(self):
        pass
```


>  The double-underscore in `__tokenize_pipeline` indicate that it is a private
method. *Private* meaning for internal use inside the class only. 

Let's look into these methods a little more deeply.

* `__tokenize_pipeline` - Check each token against the rules, breaking it up if
it doesn't pass and recursively call itself on the smaller tokens.

* `tokenize` - Return a list of tokens.


### Instance Variables

So let's modify the the `__init__` method create a few variables upon
object instantiation.

```
def __init__(self):
    self.rules = grammar.RULES
    self.rule_list = list(grammar.RULES)
    self.exception_lexicon = exceptions.LEXICON
    self.accepted_tokens = None
```

Notice we're importing our grammar rules and exceptions lexicon into the
tokenizer.  We also create a variable that will eventually hold our tokens,
`accepted_tokens`. *Accepted* meaning that the token either was in our lexicon
of exceptions or passed every rule defined in our grammar. Notice that we also
generate a list of our grammar rules. This list is in order the order they
were added, thus the order in which they will be applied.

### Tokenize

Let's build our tokenizer function. The reason for the reassignment of
`self.accepted_tokens` as an empty list is to clear the *cache* of tokens so
that when called in succession there won't be tokens from previously
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
hand makes use of lazy loading. A generator yields a single token as needed
needed so extraneous memory is not used. As documents range in size, this
saves memory, which will help prevent the program from slowing. See
[Reduce Memory Usage and Make Your Python Code Faster Using Generators](https://towardsdatascience.com/reduce-memory-usage-and-make-your-python-code-faster-using-generators-bd79dbfeb4c)
for more information. We call the our NLP pipeline on every token in the
generator.  Lastly we return `accepted_tokens`.  At this point the tokenizer
structure is complete. All that remains is to establish the pipeline.


### Tokenize Pipeline

The `tokenize` function breaks a string into individual tokens and then calls
`__tokenize_pipeline` on each one.

```
def __tokenize_pipeline(self, token):
    if token in self.exception_lexicon:
        self.accepted_tokens += self.exception_lexicon[token]
    else:
        continue_matching = True
        rule_index = 0

        while continue_matching:
            rule = self.rule_list[rule_index]
            match = re.match(self.rules[rule], token)
            if match:
                for group in match.groups():
                    if group:
                        self.__tokenize_pipeline(group)
                        continue_matching = False
            else:
                if rule == self.rule_list[-1]:
                    self.accepted_tokens.append(token)
                    continue_matching = False
                else:
                    rule_index += 1
```

First order of business is to check if the token is an exception. If it is,
add the exceptions value to accepted tokens.  Remember the dictionary *key* is
is the exception token and the value is a list with the curated tokens.
`accepted_tokens`.  If a token isn't an exception, we check it against it each
against each rule. If the token fails a rule, (or the token is matched by the
regex), it is immediately broken into its constituent groups defined by the
rule. Then we call `__tokenize_pipeline` recursively on every every subsequent
token. 

> Rather than a *while loop*, a *for loop* is probably more intuitive, but
a while loop avoids having to use a break statement to stop moving on to the
next rule.

Now when a token passes rule.  We check to see if the rule being checked was
the last rule. If it isn't, the `rule_index` is augmented by 1 and the check
is performed with a new rule. If the last rule passes, then the token is added
to `accepted_tokens`, where the process beings again with a new token.

### Demo

The demo here is pretty simple.  We created a few example lines.  Feel free to
include your own. We instantiate a tokenizer object, and call its tokenize
method on every line. Boom! Tokenization. Remember this is in the `demo.py`
file.

```
import tokenizer

lines = [
    "Hi, how are you?",
    "What's up, John?",
    "I hear that Google's looking to purchase #yourcompany for $10 million!",
    "I'm really starting to dig this NLP Notebook blog.",
    "What's your E.T.A.? I really have to know.",
    "I have $43, while you have €3.32.",
]

tokenizer = tokenizer.Tokenizer()

tokenized_lines = [tokenizer.tokenize(line) for line in lines]
for line in tokenized_lines:
    print(*line)
```
```
Hi , how are you ?
What 's up , John ?
I hear that Google's looking to purchase #yourcompany for $ 10 million !
I 'm really starting to dig this NLP Notebook blog .
What 's your E.T.A.? I really have to know .
I have $ 43 , while you have € 3.32 .
```

If you're not sure how to run the file from the terminal or command line. It'll
be something like:

```
python demo.py
```
That is to say:
```
path\to\python path\to\demo.py
```

### Tests
In the last post we added a few tests to verify the regex we created were those
we intended. Now, that the entire tokenizer built. New tests need to be added
to verify it is working as intended. Here are the tests I added, but it
wouldn't hurt to add more. Is the system really working as intended? What
improvements could be made?

```
# Test rules.
def test_inital_punctuation_rule():
    inital_punctuation_testdata = {
        '"Hi' : ['"', 'Hi'],
        '\'Hi' : ['\'', 'Hi'],
        '-Hi' : ['-Hi'],
        '""hi' : ['""hi'],
    }
    for test, answer in inital_punctuation_testdata.items():
        assert TOKENIZER.tokenize(test) == answer

def test_final_punctuation_rule():
    final_punctuation_testdata = {
        'Say,' : ["Say", ','],
        'Hi...' : ['Hi', '.', '.', '.'],
        'Like this:' : ['Like', 'this', ':'],
        'E.T.' : ['E.T.'],
        'etc.' : ['etc', '.'],
    }
    for test, answer in final_punctuation_testdata.items():
        assert TOKENIZER.tokenize(test) == answer


def test_all_punctuation_rule():
    all_punctuation_testdata = {
        '",' : ["\"", ','],
        '\',' : ["\'", ','],
    }
    for test, answer in all_punctuation_testdata.items():
        assert TOKENIZER.tokenize(test) == answer


def test_currency_amount_rule():
    currency_amount_testdata = {
        '$5.00' : ['$', '5.00'],
        '¥32' : ['¥', '32'],
        '¥35.00' : ['¥', '35.00'],
        '€23.00' : ['€', '23.00'],
        '€1.23,' : ['€', '1.23', ','],
    }
    for test, answer in currency_amount_testdata.items():
        assert TOKENIZER.tokenize(test) == answer

def test_exceptions():
    exceptions_testdata = {
        "don't" : ["do", "n't"],
        "isn't" : ["is", "n't"],
        "What's" : ["What", "'s"],
        "what's" : ["what's"],
        "I'm" : ["I", "'m"],
    }
    for test, answer in exceptions_testdata.items():
        assert TOKENIZER.tokenize(test) == answer
```

### Suggestions for Future Tokenizer Updates

1. Add support for borrowed words like café.  How many diacritics does
English usually import?
2. Allow for English plural possessives: e.g. "cats'"
3. Make a rule to split tokens with internal punctuation like E.T.A. better.
4. Add more tests, e.g. sentences.

Remember for every change to the grammar, add new tests. Fortunately, most of
the time, you won't have to build a tokenizer yourself. Don't forget to the
[source code](https://github.com/apocop/NLP-Notebook-Examples/tree/master/2019-12-23-lets-build-a-tokenizer)
is available if you want to compare. Check out a few packages that have
tokenizers. 

* [NLTK](https://www.nltk.org/api/nltk.tokenize.html) has few tokenizers made
for generic and specialized domains like tweets from Twitter.

* [SpaCy](https://spacy.io/)

* [Spark NLP](https://nlp.johnsnowlabs.com/)