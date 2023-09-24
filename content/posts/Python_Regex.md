+++
title = 'Python_Regex'
date = 2021-10-12T20:59:46+08:00
draft = false
categories= ['Automation']
tags= ['Python','Regex']
+++

## Some Advanced usage on regex

### get float number: 
```
import re
target = 'total growth is : 8%-8.5%'
rate = re.findall('(\d+(?:\.\d+)?)', target)
print(rate)
['8', '8.5']
#\d is any int, + is one to many
#(?:...)
#A non-capturing group allows you to apply quantifiers to part of your regex but does not capture/assign an ID.
#so it means: find int or, if int follows a "." and any int afterwards, combine the int with ".int"
#For example, repeating 1-3 digits and a period 3 times can be done like #this: /(?:\d{1,3}\.){3}\d{1,3}/
#above regex can match string like : 219.254.232.123 or 232.212.222.3434 (can only go until 232.212.222.343), 
#can not match 232.2122.222.3434 or 232.212.22a.3434

#`(\d+(?:\.\d+)?)` is a group, so result will be put in a list. 

#if you just want to match any int or .xxx or xxxx.xxx,just use

rate = re.findall('(\d+(\.\d+)?)', target)
print(rate)
[('8', ''), ('8.5', '.5')]
#which will return a list to you

#if you just want to find a float, use below: 

target = 'total growth is : 8%-8.5% or 9.546%'
rate = re.findall('(\d+\.\d+)', target)
print(rate)
['8.5', '9.546']
```

### A bit deeper:

-   `A(?=B)` ：Match **String A** *ends* with **String B**
    
-   `A(?!B)` ：Match **String A** *NOT end* with **String B**
    
-   `(?<=B)A` ：Match **String A** *start* with **String B**
    
-   `(?<!B)A` ： Match **String A** *NOT start* with **String B**

```
target = 'in 2021 financial year total growth is : 8%-8.5% or 9.546%'

rate = re.findall(r'(\d+(?:\.\d+)?)(?![\d financial])', target)

print(rate)
['8', '8.5', '9.546']

# :! means **NOT end** with 
# ?![\d financial] means number followed by " financial" or a digit, so 2021 is totally excluded. if with only ?![ financial], it wil match 202
```
#### Find any number before %

```
target = 'in 2021 financial year total growth is : 8%-8.5% or 9.546%'
rate = re.findall('\d+(?=%)', target)
print(rate)
['8', '5', '546']

#to find float before % : 
rate = re.findall('(\d+\.\d+)(?=%)', target) #combine previous regex
print(rate)
['8.5', '9.546']

# to find int or float before %: 
rate = re.findall('(\d+(?:\.\d+)?)(?=%)', target)  #combine previous regex
print(rate)
['8', '8.5', '9.546']

# find year: 

rate = re.findall('\d+(?= financial)', target)
print(rate)
['2021']
```
#### No. 2 test: 

-   `A(?!B)` ：Match **String A** *NOT end* with **String B**

```
# find any number not end with " financial"
rate = re.findall('\d+(?![\d financial])', target)
print(rate)
['8', '8', '5', '9', '546']
```
#### No. 3 test:

-  `(?<=B)A` ：Match **String A** *start* with **String B**

```
text = 'Price: $10000.00'
print(re.findall(r'(?<=\$)\d+', text))
['10000']
```

#### No. 4 test:

- `(?<!B)A` ： Match **String A** *NOT start* with **String B**

```
text = 'Price: $10000.00, Quantity: 5'
print(re.findall(r'(?<![\d\$\.])\d+', text))  
['5']
```
# Now come to the import ant one, regex for a IPv4 address

```
IPPattern = re.compile('(?<![\.\d])(?:25[0-5]\.|2[0-4]\d\.|[1]\d\d\.|[1-9]\d\.|[1-9]\.)(?:25[0-5]\.|2[0-4]\d\.|[01]?\d\d?\.){2}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)(?![\.\d])')
```

![IPv4 Regex test result](https://songkou.github.io/posts/python_regex/IPv4_add_regex.jpg)

