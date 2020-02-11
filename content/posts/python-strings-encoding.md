---
title: "Python String Encoding"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "python",
    "python2",
    "python3",
    "string",
    "encoding",
]

comment: true
toc: true
auto_collapse_toc: true
---

```python
# coding: utf-8

# If the editor saves the source code as `utf-8` encoding, then the python interpreter
# uses `coding: utf-8` to decode the source code.
# Otherwise, for example, if the editor saves using `latin-1` (aka iso-8859-1) encoding,
# then the python interpreter cannot decode it correctly using `coding: utf-8`.

# `©` code point in latin-1 character set
# Dec Hex Char
# 169 A9  ©
a = '©'

# Depends on the terminal you use to run this code.
print('a in the terminal: {}'.format(a))
# Python2, ©
# Python3, ©

print('type(a): {}'.format(type(a)))
# Python2, <type 'str'>, it is a byte string, the bytes represent of chars.
# Python3, <class 'str'>, it is a unicode string, the code point(numeric) represent of chars.

print('len(a): {}'.format(len(a)))
# Python2, 2
# Python3, 1
print('repr(a): {}'.format(repr(a)))
# Python2, '\xc2\xa9'
# Python3, '©'
# Conclusion: If the editor saves the source code using `utf-8`, then Python interpreter
# decodes the source code using `coding: utf-8`. And Python constructs the `str`
# (it is byte string in python2, unicode string in python3) objects
# based on the decoded chars using its internal representation (CPython implementation).
# The hex utf-8 bytes of `©` is `\xc2\xa9` (https://mothereff.in/utf-8). 
# In Python2, `a` is a bytes represent of `©`. It is 2 bytes.
# In Python3, `a` is a list of unicode code points with only one char in it.

try:
    b = a.decode('utf-8')
    print('repr(b): {}'.format(repr(b)))
    # Python2, u'\xa9', although the raw utf-8 bytes is `\xc2\xa9`, the utf-8 encoding
    # algorithm uses only part of the bits as valid bits.
    # (https://en.wikipedia.org/wiki/UTF-8)
    # Python3, AttributeError due to `a` is a unicode string, cannot be decoded.
except Exception as err:
    print('repr(b): {}'.format(repr(err)))

try:
    c = a.encode('utf-8')
    print('repr(c): {}'.format(repr(c)))
    # Python2, UnicodeDecodeError('ascii', '\xc2\xa9', 0, 1, 'ordinal not in range(128)')
    # Python3, b'\xc2\xa9'
except Exception as err:
    print('repr(c): {}'.format(repr(err)))

d = b'\xc2\xa9'
print('repr(d): {}'.format(repr(d)))
# Python2, '\xc2\xa9'
# Python3, b'\xc2\xa9'

try:
    e = d.decode('utf-8')
    print('repr(e): {}'.format(repr(e)))
    # Python2, u'\xa9'
    # Python3, '©'
except Exception as err:
    print('repr(e): {}'.format(repr(err)))

try:
    f = e.encode('latin-1')
    print('repr(f): {}'.format(repr(f)))
    # Python2, '\xa9'
    # Python3, b'\xa9'
except Exception as err:
    print('repr(f): {}'.format(repr(err)))

```

Everything in a computer is a **byte**,
- why we need the **byte string** and the **unicode string**?
- what exactly is a '**unicode**' string in Python?

The basic idea is the _Unicode Sandwich_, which is that:
1. You will get your **byte strings** into your program and as soon as possible decode them into **Unicode strings**.
2. All of the manipulation and processing of those strings in your program will be done while they are **in unicode form**.
3. Then, on output they should be encode back into **byte strings** and sent on their way.

Many libraries and frameworks in Python3 make following this model very easy as they often give you input from files or the web already decoded to Unicode strings and then allow you to pass output into functions as Unicode strings where they handle the encoding back to bytes.

Where you may run into problems is using lower level libraries that leverage sockets. Some of those socket methods are designed to only take **byte strings**. In Python2, they would accept as input a `str` like `Hello World!` because it was actually a byte string in Python2.

In Python3 you will need to encode the **Unicode** string into **byte string** with a method like `"Hello World!".encode('UTF-8')` when you pass it into the socket function/method or you will get an error telling you that it only takes byte strings.
