# python合并列表

两个列表：

```python
list1 = [1, 2, 3]
list2 = ['a', 'b', 'c']
```

## "+"号

```python
newlist = list1+list2
print(newlist)
#[1, 2, 3, 'a', 'b', 'c']
```

简单快捷，适用于**直接将两个列表连接在一起的情况**

## extend()

```python
list1.extend(list2)
print(list1)
print(list2)
#[1, 2, 3, 'a', 'b', 'c']
#['a', 'b', 'c']
```

该方法会改变原来的列表，**不会创建新的列表**

## 星号（*）和zip（）函数合并列表

```python
merged_list = []
for pair in zip(list1,list2):
    print(pair)
    for item in pair:
        merged_list.append(item)

print(merged_list)
#(1, 'a')
#(2, 'b')
#(3, 'c')
#[1, 'a', 2, 'b', 3, 'c']
```

这种方法**将两个列表中对应位置的元素合并在一起**。

## 使用列表推导式

```python
merged_list = [item for sublist in [list1, list2] for item in sublist]
```

使用 itertools.chain() 合并列表
`itertools.chain()` 函数可以用来合并任意数量的列表或者其他可迭代对象。

代码示例：

```python
from itertools import chain
list1 = [1, 2, 3]
list2 = [4, 5, 6]
merged_list = list(chain(list1, list2))
print(merged_list) # 输出：[1, 2, 3, 4, 5, 6]
```



这种方法在处理大量列表时非常高效。

## 总结

每种方法都有其特定的应用场景。

使用 `+` 运算符或 `extend()` 方法可以快速合并两个列表，而 `zip()` 函数和列表推导式提供了更多的灵活性，适用于更复杂的情况。

`itertools.chain()` 函数是合并大量列表的高效选择。

