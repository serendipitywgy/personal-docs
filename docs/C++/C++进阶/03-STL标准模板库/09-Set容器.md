# 【09】Set容器
## 概述
1. **什么是 C++ STL 的 Set**？  
C++ STL（Standard Template Library）中的 Set 是一个关联容器，它包含唯一元素的集合。Set 中的元素默认按升序排列，这是通过特定的比较函数对象实现的。Set 的内部实现通常基于平衡二叉搜索树（如红黑树），这使得元素的插入、删除和查找操作都具有对数时间复杂度。  
2. **Set 的特性**  
●唯一性：Set 中的元素都是唯一的，不允许有重复元素。  
●有序性：Set 中的元素默认按升序排列。  
●自动排序：向 Set 中插入元素时，Set 会自动调整元素的顺序以保持有序性。  
●动态大小：Set 可以动态地添加和删除元素，其大小可以在运行时改变。  
3. **Set 的实现原理**  
Set 的实现通常基于平衡二叉搜索树（如红黑树）。平衡二叉搜索树是一种自平衡的二叉搜索树，它能在插入、删除和查找操作中保持树的平衡，从而实现较高的性能。在平衡二叉搜索树中，左子树上所有结点的值均小于它的根结点的值，右子树上所有结点的值均大于或等于它的根结点的值，并且它的左、右子树也分别为平衡二叉树。这种结构使得在 Set 中进行查找、插入和删除操作的时间复杂度均为 O(log n)。  
4. **如何使用 Set**？  
使用 Set 非常简单，只需要包含 `<set>` 头文件，然后创建 Set 对象并添加元素即可。以下是一个简单的示例：  
```cpp
#include <iostream>  
#include <set>  
  
int main() {  
    // 创建一个空的 Set  
    std::set<int> mySet;  
  
    // 向 Set 中添加元素  
    mySet.insert(10);  
    mySet.insert(20);  
    mySet.insert(30);  
    mySet.insert(40);  
  
    // 遍历 Set 中的元素  
    for (int num : mySet) {  
        std::cout << num << " ";  
    }  
    std::cout << std::endl;  
  
    // 检查元素是否存在于 Set 中  
    if (mySet.find(30) != mySet.end()) {  
        std::cout << "30 is in the set." << std::endl;  
    } else {  
        std::cout << "30 is not in the set." << std::endl;  
    }  
  
    // 删除 Set 中的元素  
    mySet.erase(20);  
  
    // 再次遍历 Set 中的元素  
    for (int num : mySet) {  
        std::cout << num << " ";  
    }  
    std::cout << std::endl;  
  
    return 0;  
}
```
这个示例展示了如何创建一个 Set，如何向其中添加元素，如何遍历元素，如何检查元素是否存在，以及如何删除元素。注意，Set 中的元素默认按升序排列，因此遍历时输出的元素也是有序的。  

## 排序  
自动排序是 std::set 容器的一个内置特性，无需显式调用排序函数。当你向 std::set 中插入元素时，容器会自动调整元素的顺序以保持有序性。下面是一个简单的样例代码，展示了如何使用 std::set 并观察其自动排序的特性：  
```cpp
#include <iostream>  
#include <set>  
  
int main() {  
    // 创建一个空的 set 容器  
    std::set<int> mySet;  
  
    // 向 set 中添加元素  
    mySet.insert(30);  
    mySet.insert(10);  
    mySet.insert(40);  
    mySet.insert(20);  
  
    // 遍历 set 中的元素，观察它们已经自动排序  
    for (int num : mySet) {  
        std::cout << num << " ";  
    }  
    std::cout << std::endl;  
  
    // 输出：10 20 30 40 （自动按升序排列）  
  
    return 0;  
}
```
在这个例子中，尽管我们是以乱序的方式向 mySet 中插入元素（30, 10, 40, 20），但当我们遍历 mySet 时，可以看到元素已经自动按升序排列了（10, 20, 30, 40）。这就是 std::set 的自动排序特性。  

需要注意的是，std::set 中的元素默认是按照小于运算符 < 进行排序的。如果你想要按照其他方式进行排序，你可以提供自定义的比较函数或函数对象给 std::set 的模板参数。例如，如果你想要一个降序排列的 std::set，你可以这样做：  
```cpp
#include <iostream>  
#include <set>  
#include <functional> // 为了使用 std::greater  
  
int main() {  
    // 创建一个降序排列的 set 容器  
    std::set<int, std::greater<int>> mySetDesc;  
  
    // 向 set 中添加元素  
    mySetDesc.insert(30);  
    mySetDesc.insert(10);  
    mySetDesc.insert(40);  
    mySetDesc.insert(20);  
  
    // 遍历 set 中的元素，观察它们是降序排列的  
    for (int num : mySetDesc) {  
        std::cout << num << " ";  
    }  
    std::cout << std::endl;  
  
    // 输出：40 30 20 10 （自动按降序排列）  
  
    return 0;  
}
```

在这个例子中，我们通过将 std::greater`<int>` 作为第二个模板参数传递给 std::set，创建了一个降序排列的 std::set。这样，即使我们以升序的方式向 mySetDesc 中插入元素，它们仍然会在容器中按照降序排列。

**成员函数**

STL Set的变量和成员函数可以通过以下表格进行概述：

| **变量/成员函数**                                       | **描述**                               |
| ------------------------------------------------- | ------------------------------------ |
| **变量**                                            |                                      |
| 元素                                                | Set容器中的元素，它们根据特定的排序准则自动排序，且元素唯一      |
| **成员函数**                                          |                                      |
| begin()                                           | 返回一个指向set容器中第一个元素的迭代器                |
| end()                                             | 返回一个指向set容器“尾部之后”的迭代器                |
| rbegin()                                          | 返回一个指向set容器中最后一个元素的反向迭代器             |
| rend()                                            | 返回一个指向set容器“头部之前”的反向迭代器              |
| size()                                            | 返回set容器中元素的个数                        |
| empty()                                           | 如果set容器为空，则返回true；否则返回false          |
| max_size()                                        | 返回set容器可能包含的最大元素数                    |
| swap(set& other)                                  | 交换两个set容器的内容                         |
| insert(const value_type& val)                     | 在set容器中插入一个元素                        |
| insert(iterator pos, const value_type& val)       | 在set容器的指定位置插入一个元素                    |
| insert(input_iterator first, input_iterator last) | 在set容器中插入一个元素范围                      |
| erase(const key_type& key)                        | 从set容器中删除与指定键值相等的元素                  |
| erase(iterator pos)                               | 从set容器中删除指定迭代器位置的元素                  |
| erase(iterator first, iterator last)              | 从set容器中删除一个元素范围                      |
| find(const key_type& key)                         | 查找set容器中第一个与指定键值相等的元素的迭代器            |
| count(const key_type& key)                        | 返回set容器中与指定键值相等的元素的个数（对于set，结果总是0或1） |
| lower_bound(const key_type& key)                  | 返回一个迭代器，指向set容器中第一个不小于指定键值的元素        |
| upper_bound(const key_type& key)                  | 返回一个迭代器，指向set容器中第一个大于指定键值的元素         |
| equal_range(const key_type& key)                  | 返回一个pair，表示set容器中所有与指定键值相等的元素的范围     |
| clear()                                           | 删除set容器中的所有元素                        |

请注意，上述表格中的value_type、key_type和input_iterator等类型应根据set容器中实际存储的元素类型进行替换。另外，set容器不允许元素重复，且根据元素的键值自动排序。因此，insert函数在插入重复元素时不会执行插入操作，count函数对于set容器来说结果总是0或1。