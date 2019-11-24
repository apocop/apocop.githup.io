# Internationalization & Localization in Python

In this post, we'll look briefly into Internationalization and Localization in Python using the `gettext` library.  Localization and Internationalization, often spelt I18N and L10N respectively, while somewhat related, are distinct.  An overly simplistic comparison would be that I18N often is done by engineers from the beginning by adding support for foreign language scripts, different date formats, number formatting... which in turn allows the program to be localized.  Localization can include translation, or even how the the product is preceived in a different regions, countries, etc.

There's a somewhat humorous story 

For our purposes, we'll be laying some internationalization infrastructure to allow our program to be localized.  We'll be localizing our program into Spanish... and looking at Hebrew.

The simple program is called `greeting.py` and prints out a few lines of text.


```python
def greet():
    'Prints out greeting message.'
    age = 25
    print('Hi')
    print("What's up?")
    print('I am %s years old!' % (age))
    print('\n')

greet()
```

    Hi
    What's up?
    I am 25 years old!
    
    
    

Simple enough. Let's highlight the procedure we'll need to follow.

1. Mark strings which should be translated as 'translatable'.
2. Extract translatable strings.
3. Translate strings
4. Use `gettext` to 'get' string translations.

### 1. Marking Translatable Strings

So translatable strings are essentially user facing strings. Strings such a comments or variables should generally be left alone. To mark a string as translatable, we place the function `_()` around it from `gettext`.  So we `import gettext` and install `greeting`.


```
import gettext
gettext.install('greeting')

def greet():
    'Prints out greeting message.'
    age = 25
    print(_('Hi'))
    print(_("What's up?"))
    print(_('I am %s years old!' % (age)))
    print(_('\n'))

greet()
```

### Extracting Translatable Strings
At this point we there's no change to the program's output.  Great! Now we can extract each of the translatable strings.
Part of the Standard Python Library, we'll use the following bash/cmd command to extract all the strings.

`C:\Users\$USER\Anaconda3\Tools\i18n\pygettext.py -d greeting greeting.py`

Of course your location of `pygettext.py` will be different, but you can tell, it's located in the `Tools\i18n` folder of your Python installation. After running the command you'll see a new new file: `greeting.pot`  Examening the file, we see a header, metadata, and the extracted strings, with a blank string below.  It's these blank strings where we set the translations.

```
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR ORGANIZATION
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"POT-Creation-Date: 2019-11-23 15:31-0800\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=cp1252\n"
"Content-Transfer-Encoding: 8bit\n"
"Generated-By: pygettext.py 1.5\n"


#: greeting.py:7
msgid "Hi"
msgstr ""

#: greeting.py:8
msgid "What's up?"
msgstr ""

#: greeting.py:9
msgid "I am %s years old!"
msgstr ""


```

### 2. Translating Strings

Now this file, `greeting.pot` would be handed off to a translator to provide the appropriate translations.  It's important to note `charset=cp1252\n"`. `cp1252` needs to be changed to `utf-8` to avoid rendering issues of alternative scripts. I won't be filling in any of the other metadata for this example.  Since our example is small we can just supply the appropriate translations, but on a larger project, a translator might use translations editor such as [Poedit](https://poedit.net/).

> Note how `%s` is used in the translation and strings.  [Python F-Strings](https://realpython.com/python-f-strings/) are not supported yet.

```
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR ORGANIZATION
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"POT-Creation-Date: 2019-11-23 09:54-0800\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=utf-8\n"
"Content-Transfer-Encoding: 8bit\n"
"Generated-By: pygettext.py 1.5\n"


#: greeting.py:7
msgid "Hi"
msgstr "Hola"

#: greeting.py:8
msgid "What's up?"
msgstr "¿Qué tal?"

#: greeting.py:9
msgid "I am %s years old!"
msgstr "¡Tengo %s años!"

```

#### Language Tags

The next step requires a litte knowledge of [language tags](https://www.w3.org/International/articles/language-tags/).
A language tag is a two (sometimes three) letter designation ([ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)) used to identify a language.  For example, 'en' is stands for English while, 'es' is Spanish.  However, for finer gradation if you needed to localize for two regions such as English from the UK vs the English from the US, the local or region code can be used.  For example, en-GB (English of Great Britian) vs en-US. For our purposes, we'll just be using 'es', for a generic Spanish.

### Integrating Translations

Without translations finished and a basic understanding of language tags we can move on.  Much like we used the `pygettext.py` tool to extract translations, there was another tool it the same folder, `msgfmt.py`.  It takes a `*.po` file as an argument outputs a binary `*.mo` file. Fortunately our `greeting.pot`file just needs to be renamed with the `*.pot` extension.  Once you rename it, we'll need to place in a specified folder for `gettext` to find it.

So let's lets create a few new directories. From the workspace, add the following: `locale/es/LC_MESSAGES` and add your newly renamed `greeting.po` file. Note our language tag, `es` in the path. Navagate to this folder and run `msgfmt.py` passing in the new file name sans extension.

`
python C:\Users\$USER\Anaconda3\Tools\i18n\msgfmt.py greeting
`

Once you create this you should have two files in the LC_MESSAGES folder:

1. greeting.po
2. greeting.mo

### Switching languages

With the final infrastructure in place we can write a short function that will allows us to change between languages.
Going back to our greeting.py, we'll write a new function that will allows us to switch between languages.



```python
def select_language(language):
    lang = gettext.translation('greeting', localedir='locale', languages=[language], fallback=True)
    lang.install()
```

Let's examen the `gettext.translation()` method a little closer. `localdir` is the where our local directory is located in the current workspace. `languages` corresponds to our language tag, in this case it'll be 'es'.  By setting `fallback=True`, if a string requires a translation, but one isn't available for the corresponding language it'll return the hardcoded string. In this case English, even though we didnt' set up an 'en' locale. The following code demonstrates the program language being switched from English to Spanish and back to English. By passing an invalid language tag, it 'fellback' back to English.


```python
import gettext

gettext.install('greeting')

def greet():
    age = 25
    print(_('Hi'))
    print(_("What's up?"))
    print(_('I am %s years old!') % (age))
    print('\n')

def select_language(language):
    lang = gettext.translation('greeting', localedir='locale', languages=[language], fallback=True)
    lang.install()


# Default language 'English'.
greet()

# Change language to Spanish.
select_language('es')
greet()

# Change language to English.
select_language('en')
greet()
```

```
Hi
What's up?
I am 25 years old!


Hola
¿Qué tal?
¡Tengo 25 años!


Hi
What's up?
I am 25 years old!
```

### Final Thoughts

Great! We successfully internationalized and localized our program for a Spanish Speaking audience. Unfortunately, not all languages are so easy.  Here's one example, a right-to-left (RTL) language Hebrew: ISO code. `he`. Right-to-left that meaning the language is written and read from right to left. I've gone ahead created and translated a new `*.po` file and appropriate locale folder. You can see the [final code]().

```
#: greeting.py:7
msgid "Hi"
msgstr "שלום"

#: greeting.py:8
msgid "What's up?"
msgstr "מה נשמע?"

#: greeting.py:9
msgid "I am %s years old!"
msgstr "אני בן %s!"
```
Compare the translations to the terminal output:
[]()

At a cursory glance everything may look fine but if you look closer you can see that output from the terminal is backwards! It has rendered a RTL language as left-to-right! If copied and pasted into another space, the same backwards text may render correctly, depending on the rendering engine. 

In fact, the rendering of the Hebrew in this post will be wrong.  Like English, punctuation like the question mark in Hebrew is at the end of a sentence, so it should be on the left side of the sentence, but notice it's on the right! Note the last sentence also renders correctly, **"אני בן %s!"**  It's roughly equivalent of to "%s years old I am".

Here's what the sentences would like like if rendered correctly

