---
layout: post
title: "Normalizing Text with Regex Groups in Python"
author: "Daniel Dorado"
date: "November 13, 2019"
categories: python normalization text nlp regex
---

# Normalizing Text with Regex Groups in Python


In this post we're going to look at how regex groups can help clean messy text data, and in this specific case by normalizing product numbers.  The data comes from [RFX]([http://www.sourcinginnovation.com/glossary/RFX.php) instrument bids.  The challenge here is that while a company may have a concrete way of identify their own products, these product IDs are commonly mistyped when customers request product. We'll see to fix common mistakes by a regular expression that matches all the erroneous forms and uses groups to pull out specific parts from the regex.

### Basic Yamaha Product ID

[Yamaha](https://www.yamaha.com/en/), the world's largest instrument manufacturer, has designed a Product ID system which encodes descriptive information about the product.  Identifying the inherent morphology allows someone to know which instrument it is, but also any special customizations or features it may have. Luckily for us, we don't need to know that.

Let's use the Yamaha Tuba, [YBB-105WC](https://usa.yamaha.com/products/musical_instruments/winds/tubas/ybb-105/specs.html#product-tabs), as our base example. For our purposes the pattern of the Yamaha wind instrument is:

| Brand | Instrument | Hyphen | Series | Features | Mark/Iteration* |
|:---|:---|:---|:---|:---|:---|
| Y | BB | - | 105 | WC | ∅ |
| Yamaha | Tuba | delimiter |  100 Series | With Case | Product Iteration |

> The Mark or iteration is usually written with Roman Numerals, such as YAS-200AD**II**. No roman numerals means the product is the first iteration!

Since we don't need to pull out this level of detail from IDs in our example, we'll reduce this template even further.

Interestingly enough, this system was so successful that other companies noticed and started following a similar approach. Note the similarity between the Jupiter Tuba [JTU1110](http://jupitermusic.com/us/products/brass/tubas/) and Eastman Tuba [EBB234](https://www.eastmanwinds.com/ebb234). Any guesses on what the first letter stands for?

### Examening the typos

Let's look at the typos we'll be trying to normalize. Let's look at the Yamaha Alto Sax [YAS-200ADII](https://usa.yamaha.com/products/musical_instruments/winds/saxophones/yas-200adii/index.html). This Product ID might be written a variety of ways such as:

* yas200adii
* yas 200
* YAS 200AD
* yas - 200
* yas - 200ADII
* YAS200AD II

And the list goes on.

###  Goal

Our goal is to get all Yamaha product IDs into a canonical form reducing messy data and making them easier to work with.  We will not be correcting typos or incomplete IDs such as **T**FL-222 instead of YFL-222 or YAS-2**80** in place of YAS-200**ADII**. Let's operationalize our normalized form. Ideally:

* ALL CAPS
* No spaces
* Has delineating hyphen
* Mark (as in 'mark Ⅰ' or 'mark Ⅱ') is optional, but present if available

This would mean that the previous list of error examples will all be canonicalized as YAS-200II and whichever mark such as 'II' or otherwise will only be included if it's already listed. It's absence will only be considered a typo.
 
### Libraries

We'll be using the [re library](https://docs.python.org/3.8/library/re.html) for regular expressions.
We'll also be using the `Counter` object from [Collections](https://docs.python.org/3.1/library/collections.html#collections.Counter) to manipulate the data easier and generate some statistics.


```python
import codecs
import re
from collections import Counter
```

### Python Regex Groups

Part of the `re` library, groups allow you select and extract certain sub-patterns of a regular expression. So we'll pull out the germane parts of an ID and and normalize them to our standard form.

The syntax of a group is simple:  (?P&lt;group_name&gt;…) where **group_name** is a name we choose for this group and the ellipsis is the regex for the group. 

For example, let's separate 'Y' from 'FL' in 'YFL'. We assign 'FYL' to `string` and for our pattern we make two groups in one regular expression. One named `yamaha` and the other `instrument`.


```python
string = 'YFL'
pattern = '(?P<yamaha>y)(?P<instrument>[a-z]+)'
ex = re.match(pattern, string, re.IGNORECASE)
```

We can examen specific groups by using the `group()` method and pass it the group name as a string.  Please note that `re.match()` was passed the `re.IGNORECASE` flag so that the regex is case insensitive.


```python
print(ex.group('yamaha'))
print(ex.group('instrument'))
```

    Y
    FL
    

Simple enough, now for something more complex. Let's designed a regex that'll encompass the various iterations of Yamaha product ID. We're going to break it into 3 groups: `yamaha_instrument`, `series`, and `mark`.

Note that the first group `yamaha_instrument` we'll be conflating both the brand 'Yamaha' and 'instrument since we don't have a practical reason to distinguish them. `Series` will also conflate any \[numeric]+[alpha]* pattern after the delimiter (that is hyphen in our canonical form). The group, `mark`, only will come to play if the the mark is separated by a space from the `series`.

We'll first use the regex to extract all the Yamaha IDs from the data


```python


yamaha_pattern = r'\b(?P<yamaha_instrument>Y[A-Z]{2})[\s-]*(?P<series>[0-9]+[A-Z]*)\s*(?P<mark>I*)\b'

with codecs.open('yamaha_skus.txt', 'r', encoding='utf8') as f:
    products = f.read().split('\n')
    

print(f'Total Number of Yamaha IDs: {len(products)}')
print(f'Sample of Product IDs: {products[:4]}')
```

    Total Number of Yamaha IDs: 596
    Sample of Product IDs: ['YAC1607', 'YAS - 26', 'YAS - 480', 'YAS - 480SY']
    

The Table below describes this regex chunk by chunk. For a more thorough regex review, consult the [re documentation](https://docs.python.org/3.8/library/re.html). Notice 


| Regex component | Explanation |
|:---|:---|
| \b | word boundary |
| (?P&lt;yamaha_instrument&gt;Y[A-Z]{2}) | a group name **yamaha_instrument** that consists of a y followed by two other alphabetic characters |
| [\s-]* | zero or more spaces or hyphens in any order |
| (?P&lt;series&gt;[0-9]+[A-Z]*) | a group named **series** which can consist one or more numbers followed by zero or more alphabetic characters |
| \s* |  zero or more spaces |
| (?P&lt;mark&gt;I*) | named group called **mark** which consists of zero or more i's |
| \b' | word boundary |

`products` is a list of various Yamaha product IDs.  The data comes from RFX data which can be found [here](https://github.com/apocop/apocop.github.io/blob/master/_jupyter/2019-11-13-python-group-regex/yamaha_skus.txt).

Now that we have a regex that will separate all these product IDs, we'll create a function that takes a product id and normalizes it.


```python
def normalize_yamaha_product_id(product, pattern):
    'Return idealized form of Yamaha product ID'
    product_id = re.match(pattern, product, re.IGNORECASE)
    
    # The three groups.
    yamaha_instrument = product_id.group('yamaha_instrument').upper()
    series = product_id.group('series').upper()
    mark = product_id.group('mark').upper()
    
    # Put extracted groups into final normalized form.
    return f'{ yamaha_instrument}-{series}{mark}'
```

Let's test our function and make sure it preforms well. With these basic tests it seems to work well. Be sure to mess around with your own test case, and try to make it fail.  What happens, for example, if you have 'yas - 200 ADII'? 





```python
# Test: (input, correct_ouput)
test = [
    ('yas200adii', 'YAS-200ADII'),
    ('yas 200', 'YAS-200'),
    ('yas 200AD', 'YAS-200AD'),
    ('yas - 200ADII', 'YAS-200ADII'),
    ('YAS200AD II', 'YAS-200ADII')
]

for input, output in test:
    test_case = normalize_yamaha_product_id(input, yamaha_pattern)
    if test_case == output:
        print(f'Pass' )
    else:
        print(f'Fail: "{test_case}" is not equal to "{output}')

    

```

    Pass
    Pass
    Pass
    Pass
    Pass
    

Everything passed! Now that we have tested our function and know it works.  Let's use it to normalize the IDs in `products`. We initialize a `Counter` object with `products`.  This gives us an object where we can easily get the unique IDs and counts  the various instances each individual ID. Let's normalize just the unique IDs.  The 'Change' column is *True* if the input is different than the output and *False* if there is no difference.


```python
data = Counter(products)

total_changes = 0
counter = 0
print(f'No.\tOriginal\tNormalized\tChange')

ids = []
for product in list(data):
    counter += 1
    normalized_product = normalize_yamaha_product_id(product, yamaha_pattern)
    if product == normalized_product:
        change = False
        ids.append((counter, product, normalized_product, change))
    else:
        change = True
        total_changes += 1
        ids.append((counter, product, normalized_product, change))

for counter, product, normalized_product, change in ids:
    if counter < 10:
        print(f'{counter}\t{product.ljust(10)}\t{normalized_product.ljust(10)}\t{change}')  

```

    No.	Original	Normalized	Change
    1	YAC1607   	YAC-1607  	True
    2	YAS - 26  	YAS-26    	True
    3	YAS - 480 	YAS-480   	True
    4	YAS - 480SY	YAS-480SY 	True
    5	YAS 480   	YAS-480   	True
    6	YAS 6211  	YAS-6211  	True
    7	YAS 662 III	YAS-662III	True
    8	YAS-200AD II	YAS-200ADII	True
    9	YAS-200ADII	YAS-200ADII	False
    

Feel free to look through the results. It appears that the function is working as intended. 

### Analysis

Let's examen how this function changed the data. For comparison we'll also create a new `Counter` with the normalized forms. 


```python
normalized_data = Counter([normalize_yamaha_product_id(product, yamaha_pattern) for product in products])

num_of_products = len(list(data))
num_of_uniq_products = len(list(normalized_data))

percentage_decrease = round((num_of_products - num_of_uniq_products) / num_of_products * 100,2)

print(f'{num_of_products} unique IDs was reduced to {num_of_uniq_products} IDs')
print(f'That\'s a decrease of {percentage_decrease}%!')
```

    260 unique IDs was reduced to 173 IDs
    That's a decrease of 33.46%!
    

We can make a table comparing the top 10 Product Numbers before normalization and after normalization. 


```python
heading = 'No.'.ljust(5) + 'Old ID'.ljust(10) + 'Cnt'.ljust(5) + 'New ID'.ljust(10) + 'Cnt'
print(heading)
print('-'*30)

counter = 0
for old, new in zip(data.most_common(10), normalized_data.most_common(10)):
    counter += 1
    old_id    = old[0].ljust(10)
    old_count = str(old[1]).ljust(5)
    new_id    = new[0].ljust(10)
    new_count = str(new[1])
    print(f'{ str(counter).ljust(5) }{ old_id }{ old_count }{ new_id }{ new_count }')
```

    No.  Old ID    Cnt  New ID    Cnt
    ------------------------------
    1    YBS-52    20   YBS-52    31
    2    YBB-105WC 16   YBB-105WC 24
    3    YEP-321   16   YEP-201   21
    4    YAS-26    12   YEP-321   20
    5    YCL-221   11   YAS-26    17
    6    YTS-480   11   YTS-480   17
    7    YBS52     10   YCL-221   15
    8    YEP-201   10   YCL-255   15
    9    YCL-255   9    YAS-480   13
    10   YHR-567   9    YCL-221II 12
    

From the table we can see that the YBS-52 is still the most common item requested, and that 'YBS52' has been removed from the list.  As a few other items such as the YCL-255 and YHR-576 were removed from the Top 10, this new Top 10 list provides more accurate insights.

Now let's look at the percentage increases of the Top 10 items. To get the percentage change:

$$\frac{\text{new number} - \text{old number}}{\text{old number}} \times{100} $$


```python
print('Product ID'.ljust(11), 'Percentage Change')
print('-'*30)
for product, new_count in normalized_data.most_common(10):
    old_count = data[product]
    percentage_increase = round((new_count - old_count) / old_count * 100, 2)
    print(f'{product.ljust(12)}{percentage_increase}%')
```

    Product ID  Percentage Change
    ------------------------------
    YBS-52      55.0%
    YBB-105WC   50.0%
    YEP-201     110.0%
    YEP-321     25.0%
    YAS-26      41.67%
    YTS-480     54.55%
    YCL-221     36.36%
    YCL-255     66.67%
    YAS-480     62.5%
    YCL-221II   71.43%
    

We can see that the new top products all received significant count increases when we started counting more accurately.

### Conclusion

In this post we saw how we could extract substrings using regular expression named groups, and how we could use them to clean messy data. The cleaner the data, the easier it is to model and extract practical and actionable insights.
