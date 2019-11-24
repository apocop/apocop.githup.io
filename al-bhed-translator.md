---
layout: project
title: Al Bhed Translator
permalink: /al-bhed-translator
date: "July 6, 2019"
---

## English ↔ Al Bhed Translator

#### Impetus 

Recently, I started playing Final Fantasy X, and thought it would be a fun little
project to make a translator for Al Bhed.

#### Al Bhed

Al Bhed is the language of the Al Bhed people from the game, Final Fantasy X.
However it's not technically a [constructed language](https://en.wikipedia.org/wiki/Constructed_language), since it's 
a cypher.  This code will be dealing with the English version,
although the code could easily be extended to handle the Japanese-version of 
Al Bhed since the mapping file easily be updated handle additional mapping schema.

#### Basic Usage

The simplest way to translate English to Al Bhed.

```
python translator.py --text="Now comes the age of machina!" 
```

> Huf lusac dra yka uv sylrehy!

To translate Al Bhed into English.


```
python translator.py --text="Oaaryy! Y vmekrd 1000 oaync uvantia!"  --en_to_ab=False
```
> Yeehaa! A flight 1000 years overdue!

#### Break Character

If there's a word that you don't want translated, surround it with asterisks. The 
word will be ignored and the asterisks will be removed.


```
python translator.py --text="Drao druikrd *oui* fana y veaht!!" --en_to_ab=False
```

> They thought oui were a fiend!!

#### Diacritics
While English does not use diacritics, English does has various
loanwords that may feature diacritics such as:
+ tête-à-tête
+ résumé
+ jalapeño

The basic transliteration model would neglect these characters
producing erroneous mixed-tokens— that is, words between English and Al Bhed. 
Engbhed?  Here are the same words as above without considering diacritics.

+ *dêda-à-dêda 
+ *nécisé 
+ *zymybañu

Since it's unknown whether these diacritics have usage in Al Bhed, by default
these diacritics are removed to reflect a more *standard* English. 
However, diacritics can be maintained by passing `--allow_diacritics=True`.
  
 |   English   | AB with Diacritics | AB Without Diacritics |
 |:-----------:|:------------------:|:---------------------:|
 | tête-à-tête |     dâda-ỳ-dâda    |      dada-y-dada      |
 |    résumé   |       nácisá       |         nacisa        |
 |   jalapeño  |      zymybah̃u      |        zymybahu       | 
 
 
 Potential *problems* or benefits is that other language might be transliterated
 and the unsuspecting reader may not realize that it's not the Al Bhed
 they're expecting.  On the other hand, this could be a boon to those that would
 like to maintain diacritics.
 

### Al Bhed Repository

 Try the [Al Bhed Translator](https://github.com/apocop/Al_bhed_translator) for yourself.