---
layout: post
title: "Normalizing Text with Regex Groups in Python"
author: "Daniel Dorado"
date: "November 13, 2019"
categories: python normalization text nlp regex
---

In this post we're going to look at how regex groups can help clean messy text data, and in this specific case by normalizing product numbers.  The data examples come from [RFX]([http://www.sourcinginnovation.com/glossary/RFX.php) instrument bid data.  The challenge here is that while a company may have a concrete way of idenitfy their own products, these product IDs are commonly mistyped when customers request product. We'll see to fix common mistakes by a regular expression that matches all the erroneous forms and uses groups to pull out specific parts from the regex.

### Basic Yamaha Product ID

[Yamaha](https://www.yamaha.com/en/), the world's largest instrument manufactuer, has designed a Product ID system which encodes descriptive information about the product.  Identifing the inherent morphology allows someone to know which instrument it is, but also any special customizations or features it may have. Luckily for us, we don't need to know that.

Let's use the Yamaha Tuba, [YBB-105WC](https://usa.yamaha.com/products/musical_instruments/winds/tubas/ybb-105/specs.html#product-tabs), as our base example. For our purposes the pattern of the Yamaha wind instrument is:

| Brand | Instrument | Hyphen | Series | Feates | Mark/Iteration* |
|:---|:---|:---|:---|:---|:---|
| Y | BB | - | 105 | WC | ∅ |
| Yamaha | Tuba | delimiter |  100 Series | With Case | Product Iteration |

> The Mark or iteration is usally written with Roman Numerals, such as YAS-200AD**II**. No roman numerals means the product is the first iteration!

Since we don't need to pull out this level of detail from IDs in our example, we'll reduce this template even further.

Interestingly enough, this system was so successful that other companies took noticed and started following a similar approach. Note similarity between the Jupiter Tuba [JTU1110](http://jupitermusic.com/us/products/brass/tubas/) and Eastman Tuba [EBB234](https://www.eastmanwinds.com/ebb234). Any guesses on what the first letter stands for?

### Examening the typos

Let's look at the typos we'll be trying to normalize. Let's look at the Yamaha Alto Sax [YAS-200ADII](https://usa.yamaha.com/products/musical_instruments/winds/saxophones/yas-200adii/index.html). This Product might be written a variety of ways such as:

* yas200adii
* yas 200
* yas 200AD
* yas - 200
* yas - 200ADII
* YAS200AD II

And the list goes on.

###  Goal

Our goal is to get all Yamaha product IDs into a canonical form reducing messy data and making them easier to work with.  We will not be correcting typos or incomplete IDs such as **T**FL-222 instead of YFL-222 or YAS-2**80** in place of YAS-200**ADII**. Let's operationalize our normalized form. Ideally:

* ALL CAPS
* No spaces
* Has delininating hyphen
* Mark is optional

This would mean that the previous list of error examples will all be canonicalized as YAS-200II and whichever mark such as 'II' or otherwise will only be included if it's already listed. It's absense will only be considered a typo.
 
### Libraries

We'll be using the [re library](https://docs.python.org/3.8/library/re.html) for regular expressions.
We'll also be using the `Counter` object from [Collections](https://docs.python.org/3.1/library/collections.html#collections.Counter) to manipulate the data easier and generate some statistics.


```python
import re
from collections import Counter
```

### Python Regex Groups

Part of the `re` library, groups allow you select and extract certain subpatterns of a regular expression. So we'll pull out the germane parts of an ID and and normalize them to our standard form.

The syntax of a group is simple:  (?P&lt;group_name&gt;…) where **group_name** is a name we choose for this group and the elipsis is the regex for the group. 

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


```python
yamaha_pattern = r'\b(?P<yamaha_instrument>Y[A-Z]{2})[\s-]*(?P<series>[0-9]+[A-Z]*)\s*(?P<mark>I*)\b'

products = ['YAS-200ADII', 'YAS-200ADII', 'YAS280', 'YAS-200ADII', 'YAS-200ADII', 'YAS-200ADII', 'YAS-200AD', 'YAS200AD', 'YAS-200ADII', 'YAS-200ADII', 'YAS-200ADII', 'YAS-26', 'YAS-26', 'YAS-26', 'YAS-26', 'YAS-26', 'YAS-26', 'YAS26', 'YAC1607', 'YAS - 26', 'YAS-26', 'YAS-26', 'YAS26', 'YAS-26', 'YAS26', 'YAS26', 'YAS-26', 'YAS-26', 'YAS-480', 'YAS - 480', 'YAS-480', 'YAS-480', 'YAS-480', 'YAS 480', 'YAS-480', 'YAS480', 'YAS-480', 'YAS480', 'YAS480', 'YAS-62III', 'YAS 6211', 'YAS-875EX', 'yas-875EXll', 'YAS-875EX', 'YBB-105MWC', 'YBB-105WC', 'YBB 105', 'YBB105', 'YBB - 105', 'YBB105WC', 'YBB-105WC', 'YBB-105WC', 'YBB-105WC', 'YBB-105WC', 'YBB105', 'YBB-105', 'YBB-105WC', 'YBB-105WC', 'YBB105WC', 'YBB 105WC', 'YBB-105WC', 'YBB-105WC', 'YBB-105Wc', 'YBB105WC', 'YBB-105WC', 'YBB105WC', 'YBB105WC', 'YBB105WC', 'YBB-105WC', 'YBB105', 'YBB-105WC', 'YBB 201MWC', 'YBB-201WC', 'YBB-201', 'YBB201WC', 'YBB-202MSWC', 'YBB-202MSWC', 'YBB-202MWC', 'YBB-321', 'YBB-321S', 'YBB-321WC', 'YBB321WC', 'YBB-321', 'YBB321WC', 'YBB321WC', 'YBB-321WC', 'YBB-321wc', 'YBB-321WC', 'YBB-321', 'YBB-321WC', 'YBB-621', 'YBB-641', 'YBB-641WC', 'YBB-641', 'YBB641WC', 'YBB-641', 'YBB641', 'YBB641', 'YBH301M', 'YBH-301M', 'YBH301M', 'YBH301M', 'YBH301MS', 'YBH-301MS', 'YBH-301MS', 'YBH-301MS', 'YBH-301MS', 'YBH301MS', 'YBH301MS', 'YBH 301', 'YBH-301S', 'YBH-301S', 'YBH301S', 'YBL-421G', 'YSL421G', 'YBL421G', 'YBL421G', 'YBL-620G', 'YBL-620G', 'YBL-620G', 'YBL-613H', 'YBL-620G', 'YBL-830', 'YBL-830', 'YBL-830', 'YBS-52', 'YBS-52', 'YBS52', 'YBS-52', 'YBS 52', 'YBS-52', 'YBS52', 'YBS-52', 'YBS-52', 'YBS-52', 'YBS52', 'YBS52', 'YBS52', 'YBS-52', 'YBS52', 'YBS52', 'YBS-52', 'YBS-52', 'YBS-52', 'YBS52', 'YBS-52', 'YBS52', 'YBS-52', 'YBS-52', 'YBS-52', 'YBS-52', 'YBS62', 'YBS-62', 'YBS62', 'YBS-62', 'YBS62', 'YBS-62', 'YCL-200AD', 'YCL-200ADII', 'YCL-200ADII', 'YCL-200ADII', 'YCL-200ADII', 'YCL-200ADII', 'YCL-200AD', 'YCL-20', 'YCL200ADII', 'YCL-200ADII', 'YCL-200ADII', 'YCL-221', 'YCL-221II', 'YCL-221II', 'YCL-221II', 'YCL-221', 'YCL 221', 'YCL 221', 'YCL-221', 'YCL-221', 'YCL-221', 'YCL-221II', 'YCL-221', 'YCL - 221', 'YCL-221', 'YCL-221', 'YCL-221', 'YCL-221II', 'YCL-221II', 'YCL-221', 'YCL-221', 'YCL-221II', 'YCL221II', 'YCL211', 'YCL221', 'YCL221', 'YCL-255', 'YCL 255', 'YCL-255', 'YCL-255', 'YCL-255', 'YCL-255', 'YCL255', 'YCL-255', 'YCL-250', 'YCL255', 'YCL255', 'YCL255', 'YCL255', 'YCL-255', 'YCL250', 'YCL-400AD', 'YCL - 450', 'YCL-450', 'YCL-450N', 'YCL - 450N', 'YCL450N', 'YCL-621', 'YCL-622', 'YCL622II', 'YCL-622ll', 'YCL622II', 'YCL-650', 'YCL-650U', 'YCL-681', 'YCR-2330II', 'YCR2330II', 'YEP-201', 'YEP 201', 'YEP201', 'YEP-201', 'YEP - 201', 'YEP-201', 'YEP-201', 'YEP201', 'YEP201', 'YEP-201', 'YEP 201', 'YEP201', 'YEP-201', 'YEP201', 'YEP201', 'YEP-201', 'YEP-201M', 'YEP201', 'YEP 202', 'YEP-202MS', 'YEP 202MS', 'YEP-202MS', 'YEP-321', 'YEP321', 'YEP-321', 'YEP-321', 'YEP-321', 'YEP321', 'YEP-321', 'YEP-321', 'YEP321', 'YEP-321', 'YEP-321', 'YEP321S', 'YEP-321', 'YEP-321', 'YEP-321', 'YEP-321S', 'YEP-321S', 'YEP-3215', 'YEP321S', 'YEP-321S', 'YEP321S', 'YEP321', 'YEP 321S', 'YEP-642', 'YEP-642SII', 'YEP-642IIS', 'YEP-642SII', 'YEP-642SII', 'YEP642S', 'YFG-812', 'YFH-631G', 'YFH631G', 'YFH-631G', 'YFH--631G', 'YFH-631G', 'YFH631G', 'YFH 2310', 'YFH631G', 'YFH-631G', 'YFH-631GS', 'YFH-631GS', 'YFL-200AD', 'YFL-200AD', 'YFL-200AD', 'YFL-200AD', 'YFL-200ADII', 'YFL-200AD', 'YFL-200AD', 'YFL-200AD', 'YFL-221', 'YFL221', 'YFL-221', 'YFL221', 'YFL 222', 'YFL-222', 'YFL-222', 'YFL-321P', 'YFL-322', 'YFL 361', 'YFL 361', 'YFL - 362', 'YFL-381H', 'YFL381H', 'YFL-382Y', 'YFL-382H', 'YFL-382H', 'YFL-461', 'YFL-462H', 'YFL-481H', 'YFL-577HCT', 'YFL-774HCT', 'YHR-302M', 'YHR - 31411', 'YHR314', 'YHR-314II', 'YHR-314II', 'YHR - 314', 'YHR-312II', 'YHR-314II', 'YHR-314II', 'YHR314', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR-567', 'YHR567', 'YHR-567', 'YHR567D', 'YHR-667', 'YHR667', 'YHR667', 'YHR668DII', 'YHR-668NII', 'YMP-204M', 'YMP204M', 'YMP204MS', 'YMP-204MS', 'YMP-204S', 'YMP-204MS', 'YMP204MS', 'YMP204MS', 'YOB-241', 'YOB-241', 'YOB-241', 'YOB-241', 'YOB-241', 'YOB-241', 'YOB-241', 'YOB-441', 'YOB-441', 'YOB-441', 'YOB441', 'YOB-441A', 'YOB441M', 'YOB-441M', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC-32', 'YPC32', 'YPC-62', 'YPC-62', 'YPC62', 'YPC62', 'YPC-62', 'YPC-62R', 'YPC-62R', 'YPC-81', 'YPC82', 'YSH 301', 'YSH301WC', 'YSH-411', 'YSH-411', 'Ysh411S', 'YSH411', 'YSH-411', 'YSH411', 'YSH411S', 'YSH-411SWC', 'YSH411S', 'YSH441swc', 'YSH411', 'YSH-411WC', 'YSH411', 'YSH-411WC', 'YSH411WC', 'YSH411', 'YSL-200AD', 'YSL-200AD', 'YSL-200AD', 'YSL-200AD', 'YSL-200AD', 'YSL-200AD', 'YSL-354', 'YSL-354', 'YSL 354', 'YSL354', 'YSL354', 'YSL-354', 'YSL-354', 'YSL354', 'YSL-354C', 'YSL354', 'YSL354', 'YSL-446G', 'YSL-446G', 'YSL-447G', 'YSL-447G', 'YSL-448G', 'YSL-448G', 'YSL 448G', 'YSL-448G', 'YSL448G', 'YSL3448G', 'YSL-448G', 'YSL-446G', 'YSL-620', 'YSL-8820', 'YSL-8820', 'YSL-882O', 'YSL882O', 'YSS-475II', 'YSS-475II', 'YSS-475', 'YSS475', 'YSS475', 'YSS475II', 'YSS475', 'YSS-875EX', 'YSS-875EXHG', 'YTR-200ADII', 'YTR-200ADII', 'YTR-200ADII', 'YTR-200ADII', 'YTR-200ADII', 'YTR-200AD', 'YTR200AD', 'YTR-200ADII', 'YTR-200ADII', 'YTR-2330', 'YTR-2330', 'YTR-2330', 'YTR 2330', 'YTR2330', 'YTR-2330', 'YTR 2335', 'YTR 2330', 'YTR2330', 'YTR-2330', 'YTR2330', 'YTR233', 'YTR 2335', 'YTR-2330', 'YTR-300AD', 'YTR 433', 'YTR4335', 'YTR-4335GII', 'YTR-4335GSii', 'YTR-4335GS', 'YTR8335S', 'YTR-8335S', 'YTR-8335IIS', 'YTR-8335', 'YTR8335S', 'YTS 200AD', 'YTS-200ADII', 'YTS-200ADII', 'YTS-200ADII', 'YTS-200AD', 'YTS-200ADII', 'YTS-200ADII', 'YTS26', 'YTS-26', 'YTS23', 'YTS26', 'YTS-26', 'YTS26', 'YTS26', 'YTS-26', 'YTS-26C', 'YTS26', 'YTS-480', 'YTS - 480', 'YTS-480', 'YTS - 480', 'YTS 480', 'YTS-480', 'YTY475', 'YTS475', 'YTS-480', 'YTS-480', 'YTS480', 'YTS480', 'YTS480', 'YTS-480', 'YTS-480', 'YTS-480', 'YTS-62II', 'YTS-62', 'YTS-62III', 'YTS62', 'YTS62', 'YTS6211', 'YTS-62SIII', 'YTS-82ZII', 'YFL-222', 'YFL-362H', 'YFL-482H', 'YCL-621', 'YAS-875EXII', 'YHR-671', 'YSL-882GO', 'YPC62', 'YCL-221', 'YEP-201', 'YEP-321', 'YFL-222', 'YTS-480', 'YCL-255Y', 'YCL-255', 'YBB-105WC', 'YBB-105WC', 'YCL-450N', 'YAS - 480SY', 'YTS-480', 'YTR-8335RS', 'YEP-211', 'YBB-321', 'YBB-105MWC', 'YBB-321WC', 'YEP-321', 'YEP-201', 'YBS-52', 'YTS-480', 'YAS-480C', 'YPC-62', 'YBS-52', 'YTS-475', 'YAS-26', 'YCL-255', 'YEP-201', 'YBS-52', 'YCL-221', 'YPC-32', 'YTS-26', 'YSL-448G', 'YEP-321', 'YCL-221', 'YEP-321', 'YCL-450N', 'YEP-321', 'YAS-480', 'YTR-8310ZZ', 'YEP-202M', 'YEP-321S', 'YEP3215', 'YBS52', 'YBB-105WC', 'YFL-361', 'YBB-105WC', 'YBS-52', 'YAS 662', 'YAS-480', 'YBH-301M', 'YEP-202M', 'YMP-204M', 'YSH-411', 'YMP-204M', 'YBH-301M', 'YEP201', 'ybb321wc', 'ysl-448g', 'yts875ex', 'YMP-204M', 'YBH-301M', 'YAS-62III', 'YBS-62', 'YEP-642II', 'YSL-448G', 'YFL462HY', 'YOB-441', 'YHR668NII', 'YOB841']
```

The Table below describes this regex chunch by chunck. For a more thorough regex review, consult the [re documentation](https://docs.python.org/3.8/library/re.html). Notice 


| Regex component | Explaination |
|:---|:---|
| \b | word boundry |
| (?P&lt;yamaha_instrument&gt;Y[A-Z]{2}) | a group name **yamaha_instrument** that consists of a y followed by two other alphabetic characters |
| [\s-]* | zero or more spaces or hyphens in any order |
| (?P&lt;series&gt;[0-9]+[A-Z]*) | a group named **series** which can consist one or more numbers followed by zero or more alphabetic characters |
| \s* |  zero or more spaces |
| (?P&lt;mark&gt;I*) | named group called **mark** which consists of zero or more i's |
| \b' | word boundry |

`product` is a list of various Yamaha product IDs.  The data comes from RFX data which can be found [here](https://github.com/apocop/normalizers/blob/master/musicbid/data.txt).  To reduce the complexity of this post, I took some IDs from the data rather than extracting it from the raw.

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

# Some statitics.
total_changes = 0
counter = 0
print(f'No.\tOriginal\tNormalized\tChange')
for product in list(data):
    counter += 1
    normalized_product = normalize_yamaha_product_id(product, yamaha_pattern)
    if product.strip() == normalized_product:
        change = False
    else:
        change = True
        total_changes += 1
    print(f'{counter}\t{product.ljust(10)}\t{normalized_product.ljust(10)}\t{change}')
```

    No.	Original	Normalized	Change
    1	YAS-200ADII	YAS-200ADII	False
    2	YAS280    	YAS-280   	True
    3	YAS-200AD 	YAS-200AD 	False
    4	YAS200AD  	YAS-200AD 	True
    5	YAS-26    	YAS-26    	False
    6	YAS26     	YAS-26    	True
    ...
    253	YEP-642II 	YEP-642II 	False
    254	YFL462HY  	YFL-462HY 	True
    255	YHR668NII 	YHR-668NII	True
    256	YOB841    	YOB-841   	True
    

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

    256 unique IDs was reduced to 173 IDs
    That's a decrease of 32.42%!
    

We can make a table comparing the top 10 Product Numbers before normalization and after normalization. 


```python
heading = 'Old'.ljust(10) + 'No.' + '\t'+ 'New'.ljust(10) + 'No.'
print(heading)
print('-'*30)
for num, pair in enumerate(zip(data.most_common(10), normalized_data.most_common(10))):
    print(f'{pair[0][0].ljust(10)}{pair[0][1]}\t{pair[1][0].ljust(10)}{pair[1][1]}')
```

    Old       No.	New       No.
    ------------------------------
    YBS-52    20	YBS-52    31
    YBB-105WC 16	YBB-105WC 24
    YEP-321   16	YEP-201   21
    YCL-221   14	YEP-321   20
    YAS-26    12	YCL-221   19
    YTS-480   11	YAS-26    17
    YBS52     10	YTS-480   17
    YEP-201   10	YCL-255   15
    YCL-255   9	YAS-480   13
    YHR-567   9	YTR-2330  11
    

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
    YCL-221     35.71%
    YAS-26      41.67%
    YTS-480     54.55%
    YCL-255     66.67%
    YAS-480     62.5%
    YTR-2330    83.33%
    

We can see that the new top products all received significant count increases.

### Conclusion

In this post we saw how we could extract substrings using regular expression, and how we could use them to clean up messy data.


