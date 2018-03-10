# ListIterator

`ListIterator`是用于列表的迭代器，允许程序员沿任一方法遍历列表，在迭代期间修改列表，并获取列表中迭代器的当前位置。`ListIterator`没有当前元素，它的光标位置总是位于通过对`previous（）`的调用返回的元素和由`next（）`调用返回的元素之间。 **长度为n的列表的迭代器具有n + 1个可能的光标位置**，如下图所示（^）所示：

```
                     Element(0)   Element(1)   Element(2)   ... Element(n-1)
cursor positions:  ^            ^            ^            ^                  ^

```
需要注意的是：`remove（）`和`set（Object）`方法未按照光标位置进行定义; 它们被定义为对调用`next（）`或`previous（）`返回的最后一个元素进行操作。

该接口的结构如下：

![ListIterator](http://ovn0i3kdg.bkt.clouddn.com/ListIterator.png)

## boolean hasNext();
判断列表迭代器是否有下一个元素。

## E next();
获取下一个元素并将光标向后移动一个。

## boolean hasPrevious();
判断是否有前一个元素。

## E previous();
获取前一个元素并将光标向前移动一个。

## int nextIndex();
返回使用`next`方法将得到的元素的下标，如果迭代器已经在序列的末尾，则返回序列的长度。

## int previousIndex();
返回使用`previous`方法将得到的元素的下标。如果迭代器已经在序列的开头，则返回-1。

## void remove();
从列表中删除`next（）`或`previous（）`返回的最后一个元素（可选操作）。 这个调用只能在每次调用`next`或`previous`进行一次。 只有在最后一次调用`next`或`previous`之后没有调用`add（E）`时才可以进行此操作。


## void set(E e);
用指定的元素（可选操作）替换`next（）`或`previous（）`返回的最后一个元素。 只有在最后一次调用`next`或`previous`之后既不调用`remove（）`也不`add（E）`，则可以进行此调用。

## void add(E e);

将指定的元素插入列表（可选操作）。 元素将插入到将由`next（）`返回的元素之前，如果有的话，并且在由`previous（）`返回的元素之后）。（如果列表不包含任何元素，则新元素将成为列表中的唯一元素。）新元素插入到隐式游标之前：对`next`的后续调用将不受影响，并且对`previous`的后续调用将返回新元素。 （此调用将增加一个调用`nextIndex`或`previousIndex`所返回的值。）
