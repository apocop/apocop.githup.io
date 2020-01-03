---
layout: post
title: "Let's Make a Tokenizer Part 3"
author: "Daniel Dorado"
date: "Jan 02, 2020"
categories: python tokenization nlp normalization project
---

## Grammar

So as mentioned in the previous posts, the grammar is where rules are
specified.  These grammatical rules will check to see if a token is
not grammatical. If a token passes, it'll pass on the next rule, if it fails
it'll get broken up, and the newly created tokens will begin their journey
through the pipeline. In our grammar, we're going to specify 4 rules:

1. initial_punctuation - separate token-initial punctuation
2. final_punctuation - separate token-final punctuation
3. all_punctuation - separate  token-initial and final punctuation
4. currency - separate currency symbols from quantity amounts

### Language

Since a tokenizer is dealing with a orthographic system, there are a few things
to keep in mind. This grammar is going to focus on en-US, that is English from
the United States. There are different writing standards, in different areas,
and often they conflict. So our rules in this grammar will follow a generic 
American English grammar. In our small grammar this won't matter too much, but
this does change what we consider initial punctuation or final punctuation, how
currency is written. With that being said, this tokenizer will not well, for
Spanish and French.

### Regular Expressions

We'll be defining rules for our grammar using regexes, but the longer the regex
gets the more unwieldy and unreadable they become. So we'll construct them by
building them by assigning smaller sets and characters into the basic to
reusable variables and concatenating them into larger regexes.

So let's start by importing the `re` library, and creating the basic sets we'll
use in our rules.

> Sets are created by placing characters in `[]`.

So lets add the following to our `grammar.py` file.


```
import re

ALPHA = '[A-Z]+'
DIGITS = '[0-9]'
BOS = '^'
EOS = '$'
PLUS = '+'
STAR = "*"
PERIOD = r'\.'
INITIAL_PUNCTUATION = '[\'"]'
FINAL_PUNCTUATION = '[\',!?":.]'
CURRENCY_SYMBOL = '[$£¥€]'
QUESTION_MARK = '?
```

Some of these may seem ridiculous, like `PLUS`, or `PERIOD`, but it
makes it very clear and readable even to someone who later has to read and edit
your grammar. Maybe yourself months after you last looked at this. Let's look at
a few more of these. 

* `ALPHA` is at least one alphabetic character. We'll be compiling our final
regexes with the `re.IGNORECASE` flag, so it isn't necessary to include `a-z`
in the alphabetic set. What does this mean for a token like `café` or `naïve",`?

* `BOS` means *Beginning of String* and `EOS` means *End of String*.

* `INITIAL_PUNCTUATION` punctuation that appears before a word. Note that the
the `'` is escaped and that `\` is neither a part of initial nor final
punctuation.

* `FINAL_PUNCTUATION` punctuation that appears after a word. What about other
punctuation that might appear here? Problem! By including `'` as final
punctuation the possessive plural `cats'` won't be allowed.

* `CURRENCY_SYMBOL` This is the currency symbols the tokenizer will recognize.
So it'll separate `$1.20` and `£1.20` correctly, but not `₴1.20` or `₱1.20`.

### Groups

When a token matches fails a rule, we want to be able to split the token up
into smaller tokens. We'll accomplish this my using regex groups, that is
anything in `()`. Anything we want to potentially separate needs to be in a group.
For example, `'What` needs two groups: one for `'` and `What'`. A punctuation
group and alphabetic string

So let's create a function that takes a regex expression and returns it as a
regex group.

```
def create_group(expression):
    return '(' + expression + ')'
```

This function is mostly for legibility. Notice is uses the characters and sets 
we defined earlier.


```
ALPHA_GROUP = create_group(ALPHA)
INITIAL_PUNCTUATION_GROUP = create_group(INITIAL_PUNCTUATION)
FINAL_PUNCTUATION_GROUP = create_group(FINAL_PUNCTUATION + PLUS)
FINAL_PUNCTUATION_STAR_GROUP = create_group(FINAL_PUNCTUATION + STAR)
CURRENCY_SYMBOL_GROUP = create_group(CURRENCY_SYMBOL)
CURRENCY_GROUP = create_group(DIGITS + PLUS + PERIOD + QUESTION_MARK + DIGITS + '{,2}')
ALPHA_PUNCTUATION_GROUP = create_group(ALPHA + FINAL_PUNCTUATION + STAR)
```

Notice we have two similar groups `FINAL_PUNCTUATION_GROUP` or
`FINAL_PUNCTUATION_STAR_GROUP`. Why we need a difference? Notice that sometimes
`group` is being passed a concatenated string. While it may be easier just to
copy and paste what I've put as code, it would be wise to construct these
yourself so you can grok how they're built up.

### Rules.

Now that we have base character sets and some groups we can define rules.
Notice that these rules are constructed solely of groups. That because
if a token matches a rule, it has to be broken up. Each group represents
the boundary in which a token needs to be split.

```
INITIAL_PUNCTUATION_TOKEN = INITIAL_PUNCTUATION_GROUP + ALPHA_PUNCTUATION_GROUP
FINAL_PUNCTUATION_TOKEN = ALPHA_GROUP + FINAL_PUNCTUATION_GROUP
PUNCTUATION_TOKEN = create_group(FINAL_PUNCTUATION) + FINAL_PUNCTUATION_GROUP
CURRENCY_TOKEN = CURRENCY_SYMBOL_GROUP + CURRENCY_GROUP + FINAL_PUNCTUATION_STAR_GROUP
```

The `PUNCTUATION_TOKEN` rule, notice that we're creating in the definition
since the group isn't being used elsewhere. If later a new rule would benefit
from the same group, it would be better to definite it as a constant variable
like the other groups.

### Rule dictionary

Finally let's make dictionary, where the key is the rule name, and the value is
the string regex rule.

```
RULES_TO_EXPORT = {
    'initial_punctuation_token' : INITIAL_PUNCTUATION_TOKEN,
    'final_punctuation_token' : FINAL_PUNCTUATION_TOKEN,
    'punctuation_token' : PUNCTUATION_TOKEN,
    'currency_token' : CURRENCY_TOKEN,
}
```

Something to consider here is rule order. Since we're not *really* normalizing
text here, order isn't a huge concern, but it is something to keep in mind. In
our case, we want to optimize the tokenizer to run as fast as possible. This
is why the `PUNCTUATION_TOKEN` rule comes after the other rules dealing with
punctuation. Since the other rules have to break up punctuation first, it is
unlikely that a pure punctuation token is going to exist (unless it's an
emoticon), and so checking before it would be broken down causes superfluous
rule checks. On a small dataset there probably wouldn't be a noticeable
difference, but a larger dataset could definitely be influenced.

> You may be concerned that Python dictionaries don't remember ordering, but
for now as of [Python 3.6 they do](https://docs.python.org/3/whatsnew/3.6.html#whatsnew36-compactdict).

### Rule compiling

The order of business is to compile our rules. Let's add a function that does
just that.  This rule compiles our string, but also adds `BOS` and `EOS` to
the string. This guarantees that our rule has to match the entire token and
not just a substring of it.  Also notice, that we pass the `re.IGNORECASE`
flag.

```
def compile_rule(rule):
    """Return a case insensitive rule matching and entire string."""
    return re.compile(BOS + rule + EOS, re.IGNORECASE)
```

So to create our final set of rules, I'll basically create a new `RULES`
dictionary where each value is now compiled using our new function. With
this action, we're almost ready to start building the key component of
our tokenizer—the tokenizer.

```
RULES = {key:compile_rule(value) for (key, value) in RULES_TO_EXPORT.items()}
```

### Testing 1, 2, 3

Testing is a very for text normalization and building grammars. They verify,
that they do what we intend.  For now we're just going to add a simple test
for each rule to verify that the regex we've constructed is the actual
we think we've made are the ones that we actually made. If you're not familiar
with testing or [pytest](https://docs.pytest.org/en/latest/), I highly suggest
you look into it.

In our `test_tokenizer.py` file, import our newly created grammar and
specifically the rules we just made. These tests are pretty basic, just assert
that our regexes are what we think they should be. Any time we make a change to
a rule, the regex here will have to be updated. Otherwise the test will fail.
See if you can make the test fail. It's all too easy.

```
import grammar

RULES = grammar.RULES


def test_inital_punctuation_regex():
    test_regex = RULES['initial_punctuation_token'].pattern
    assert test_regex == """^([\'"])([A-Z]+[\',!?":.]*)$"""

def test_final_punctuation_regex():
    test_regex = RULES['final_punctuation_token'].pattern
    assert test_regex == """^([A-Z]+)([',!?":.]+)$"""

def test_all_punctuation_regex():
    test_regex = RULES['punctuation_token'].pattern
    assert test_regex == """^([',!?":.])([',!?":.]+)$"""

def test_currency_regex():
    test_regex = RULES['currency_token'].pattern
    assert test_regex == r"""^([$£¥€])([0-9]+\.?[0-9]{,2})([',!?":.]*)$"""
```

Once you've got the tests setup. In the open your terminal or command prompt
and type the following command.

```
pytest -v
```

The `-v` flag prints an itemized list of each test and whether it passed or failed.

```
platform win32 -- Python 3.7.4, pytest-5.2.1, py-1.8.0, pluggy-0.13.0 -- C:\Users\$USER\Anaconda3\python.exe
cachedir: .pytest_cache
rootdir: PATH\TO\FILE
plugins: arraydiff-0.3, doctestplus-0.4.0, openfiles-0.4.0, remotedata-0.3.2
collected 4 items

test_tokenizer.py::test_inital_punctuation_regex PASSED                                                          [ 25%]
test_tokenizer.py::test_final_punctuation_regex PASSED                                                           [ 50%]
test_tokenizer.py::test_all_punctuation_regex PASSED                                                             [ 75%]
test_tokenizer.py::test_currency_regex PASSED                                                                    [100%]

================================================== 4 passed in 0.05s ==================================================
```

