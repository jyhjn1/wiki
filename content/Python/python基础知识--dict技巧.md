---
title: "python基础知识--如何声明一个字典数组，每一个数组元素是字典dict"
layout: page
date: 2019-01-10 18:00
---

# 写在前面
如何声明一个字典数组,即每个数组元素为一个字典

### 方式

申明两个不同的dict,然后将dict append到list中即可
```python
dict1 = {'multi_match': "q1"}
dict2 = {'multi_match':"q2"}
lista = []

lista.append(dict1)
lista.append(dict2)

输出结果
>>> [{'multi_match': 'q1'}, {'multi_match': 'q2'}]
```

但是正常情况下,可能会使用for循环将一系列的dict添加到list中,如下面的代码
```python
dict1 = {'multi_match': ""}

lista = []
for i in range(5):
    dict1["multi_match"] = i
    lista.append(dict1)
输出结果:

>>> [{'multi_match': 4}, {'multi_match': 4}, {'multi_match': 4}, {'multi_match': 4}, {'multi_match': 4}]
```
***结果来讲,明显是错误的,将for循环中dict1打印出来,每个单个的dict是没有问题的,但是赋值到list中再打印出来就是错误的,后来改写成下面的代码之后***

#### 改写之后
```python
lista = []
for i in range(5):
    dict1 = {'multi_match': ""}
    dict1["multi_match"] = i
    lista.append(dict1)
    del dict1

输出结果:
>>> [{'multi_match': 0}, {'multi_match': 1}, {'multi_match': 2}, {'multi_match': 3}, {'multi_match': 4}]
```

**事实上虽然输出的结果正确,当时自己却不知道如何解释**


## 应用实例
```python
## 改写前
def match_build(query_list, fields_list):
    multi_match_dict = {"query": "", "type": "phrase_prefix", "fields": fields_list}
    match_dict = {"multi_match": multi_match_dict}
    multi_match_list = []

    for i in query_list:
        match_dict["multi_match"]["query"] = i.decode("utf-8")
        multi_match_list.append(match_dict)
    return multi_match_list
```

```python
## 改写后 
def match_build_pro(query_list, fields_list):
    multi_match_list = []

    for i in query_list:
        multi_match_dict = {"query": "", "type": "phrase_prefix", "fields": fields_list}
        match_dict = {"multi_match": multi_match_dict}
        match_dict["multi_match"]["query"] = i.decode("utf-8")
        del multi_match_dict
        multi_match_list.append(match_dict)
    return multi_match_list
```

测试代码
```python
query_list = ["yellow", "black"]
fields_list = ["titles", "summary"]

print match_build(query_list, fields_list)
print match_build_pro(query_list, fields_list)
```
输出结果
```python
# match_build的输出结果:
>>> [{'multi_match': {'query': u'black', 'type': 'phrase_prefix', 'fields': ['titles', 'summary']}}, {'multi_match': {'query': u'black', 'type': 'phrase_prefix', 'fields': ['titles', 'summary']}}]

# match_build_pro的输出结果:
>>> [{'multi_match': {'query': u'yellow', 'type': 'phrase_prefix', 'fields': ['titles', 'summary']}}, {'multi_match': {'query': u'black', 'type': 'phrase_prefix', 'fields': ['titles', 'summary']}}]
```


# OrderedDict中元素排序
OrderedDict对象的字典对象，如果其顺序不同那么Python也会把他们当做是两个不同的对象

```
dd = {'banana': 3, 'apple':4, 'pear': 1, 'orange': 2}
#按key排序
kd = collections.OrderedDict(sorted(dd.items(), key=lambda t: t[0]))
print kd
#按照value排序
vd = collections.OrderedDict(sorted(dd.items(),key=lambda t:t[1]))
print vd

#输出
OrderedDict([('apple', 4), ('banana', 3), ('orange', 2), ('pear', 1)])
OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])
```