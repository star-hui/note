# 1. bs4的简单介绍

使用bs4可以快速找到html文档的内容。
bs4的解析器：需要c语言库，这种查找是最快的。
BeautifulSoup(markup, "lxml")

安装:
> pip install bs4
> from bs4 import BeautifulSoup

# 2. 使用

## 2.1. soup介绍

讲一个html文本传入到html中，就可以生产soup的对象

``` python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>', 'xlxm') 
tag = soup.b  # 生产tag对象，就是
type(tag)
# <class 'bs4.element.Tag'>
```

## 2.2. 四种对象

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种:
> Tag , NavigableString , BeautifulSoup , Comment。  

最重要的是Tag与BeautifulSoup对象

## 2.3. tag对象介绍

1. tag对象的方法:

    ``` python
    tag.name   # 获取标签名字  
    tag.attrs  # 获取标签的属性名，注意的是 html中比如class是多属性的值，获取的属性值是一个列表。 
    tag['属性名']  # 也可以类似字典的方式去获取
    tag.get('属性名') # 另外一种字典获取的方式
    tag.text  # 获取标签中间的文本
    ```
    例子：》
    ``` python
    soup = BeautifulSoup('<meta http-equiv="Cache-Control" content="max-age=300">', 'lxml')
    tag = soup.meta
    tag.name
    tag.text
    tag.http-equiv
    tag.attrs
    tag['http-equiv']  # 直接获取属性值
    tag.get('http-equiv') # 类似字典直接获取属性值
    ```

2. tag对象生成的方法

    1. 直接使用标签名： 支持链式编程:soup.body.a |
    soup.p | soup.a | soup.title
        ``` python
        soup = BeautifulSoup(html, 'lxml')
        print(type(soup.a))
        # <class 'bs4.element.Tag'>
        ```
    2. 使用find  
    查找到第一个就停止，具体参数看下面
    3. 使用find_all  
    查找到所有的匹配，具体参数看下面

3. tag对象find参数详解  
    具有四类参数
