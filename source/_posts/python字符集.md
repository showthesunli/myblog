---
title: python字符集
date: 2020-09-04 02:30:01
tags:
- python
- 字符集
categories:
- python
---

## Unicode是什么？

Unicode是一种字符集，设计初衷是将全世界所有的字符都放到这一个集合中。只要所有的电脑都支持这一个字符集，就不再会出现乱码了。想的挺美好。

### 为么还有乱码？

既然只要电脑都支持Unicode,为什么还有乱码的出现？Unicode只是一个字符集，也就是一个唯一的字符只跟一个唯一的数字匹配。这个唯一的数字叫做码点。那么问题来了，计算机存储数据都是bit位，那么计算机以什么样的字节序列来读取码点呢？换句话说，当计算机读取到0x41，这段数据被解释成“A”还是“41”呢？为了解决这个问题，现在就需要编码方式登场了。最为人熟知的Unicode编码方式有三种UTF-8,UTF-16,UTF-32。此外还有GB2312，GBK，GB18030等等，不过这些方式不属于Unicode字符集。乱码就是因为在数据编码和解码的过程中使用了不同的编码方式造成的。为么会有这么多的编码方式？可以看[这里](https://www.freebuf.com/articles/others-articles/25623.html)

### python对Unicode的支持

python3.0以来，开始全面支持Unicode字符集。默认的python源码文件以UTF-8编码。看看下面的例子（混合流编程）:


```python
我是谁 = '大肚子李大嘴'
print(我是谁)
```

    大肚子李大嘴


python支持转义字符。可以避免在源文件中出现非ASCII字符。具体使用方式及说明看下面的代码：


```python
print('\N{GREEK CAPITAL LETTER DELTA}') #使用字符名称
print('\u0394')#使用双字节长度的码点
print('\U00000394')#使用四字节长度的码点
```

    Δ
    Δ
    Δ


python支持从二进制数据解码为字符串。根据`errors`参数的不同，会进行不同的错误处理。


```python
#strict表示被解码的数据必须符合UTF-8编码，否则会抛出UnicodeDecodeError异常
b'\x80abc'.decode('UTF-8','strict')
```


    ---------------------------------------------------------------------------
    
    UnicodeDecodeError                        Traceback (most recent call last)
    
    <ipython-input-222-15907f3c629d> in <module>
          1 #strict表示被解码的数据必须符合UTF-8编码，否则会抛出UnicodeDecodeError异常
    ----> 2 b'\x80abc'.decode('UTF-8','strict')


    UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 0: invalid start byte



```python
#�的unicode码点是U+FFFD，所以在乱码中经常会看见这个符号，这个符号的名称叫做REPLACE CHARACTER
print(b'\x80abc'.decode('UTF-8','replace'))

print('\N{REPLACEMENT CHARACTER}')
```

    �abc
    �



```python
#插入一个\xNN，即插入一个转义字符反斜杠'\'
b'\x80abc'.decode('UTF-8','backslashreplace')
```




    '\\x80abc'




```python
#忽略掉不能被目标编码方式解码的数据
b'\x80abc'.decode('UTF-8','ignore')
```




    'abc'



内置函数`chr()`可以将一个整型数字转换为一个Unicode字符,这个整型参数为这个字符在Unicode中的码点。相反的操作使用内置函数`ord()`


```python
ord('中')
```




    20013




```python
chr(20013)
```




    '中'




```python
hex(20013)
```




    '0x4e2d'




```python
'\u4e2d'
```




    '中'



python支持从字符串编码为字节。与`bytes.decode()`相反的方法是`str.encode()`。`str.encode()`返回一个`bytes`对象，代表了str在对应的编码方式编码后产生的字节数据。


```python
u = chr(20013) + 'abcd' + chr(22269)
print(u)
u.encode('UTF-8')

```

    中abcd国





    b'\xe4\xb8\xadabcd\xe5\x9b\xbd'




```python
#默认的strick模式，抛出编码异常
u.encode('ASCII')
```


    ---------------------------------------------------------------------------
    
    UnicodeEncodeError                        Traceback (most recent call last)
    
    <ipython-input-243-b5d2794c2ab5> in <module>
    ----> 1 u.encode('ASCII')


    UnicodeEncodeError: 'ascii' codec can't encode character '\u4e2d' in position 0: ordinal not in range(128)



```python
#以？代替无法编码的数据
u.encode('ASCII','replace')
```




    b'?abcd?'




```python
#无法编码的数据，插入一个XML字符的转义。
u.encode('ASCII','xmlcharrefreplace')
```




    b'&#20013;abcd&#22269;'




```python
ord('中')
```




    20013




```python
#忽略掉无法编译的数据
u.encode('ascii','ignore')
```




    b'abcd'



`str.encode()`的`errs`参数等于`backslashreplace`时需要详细说明。


```python
u = chr(409608)+chr(40960)+chr(128)+chr(65)+chr(8)+'abcd'+chr(1972)
u.encode('ascii','backslashreplace')
```




    b'\\U00064008\\ua000\\x80A\x08abcd\\u07b4'



在python中，字节串仅能使用ascii表示。而ascii码采用单字节编码，并且仅使用了前7位。所以128-255号码点没有编码。对大于128的字符，需要进行转义。
`backslashreplace`参数使得`str.encode()`函数在编码错误的时候，会使用`\uhhhh \Uhhhhhhhh \xhh`这三种转义方式的字符串来进行编码。这些转义字符串就成了数据本身，包括转义符`\`。所以必须对转义符本身进行转义，即`backslashreplace`名称的由来，转义符替换。
而`chr(8)`这个字符属于ascii码，不会编码失败，但其是不可见字符，所以字节串中用`\x08`来转义了，并且不需要添加转义字符。