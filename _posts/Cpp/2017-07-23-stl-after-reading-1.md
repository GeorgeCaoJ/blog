---
layout: pageshow
title: STL源码剖析读后知识点整理
category: Cpp
description: 读STL源码剖析的知识点整理
---

## 容器
### 序列式容器
### vector
* vector的内部空间是一块连续线性空间  
* vector通过内部的Alloctor内存分配器动态分配内存。每次对vector加入新的对象时，都可能对内存重新分配。  
* vector的对象总是通过在新的内存空间construct出来的，所以并不是原对象的一个引用。区别于python中的数组（python中的数组对基础数据类型POD，直接是copy了一份，但是对类对象，则是引用而已）
* vector内部有3个指示器，start，finish和end_of_storage，分别表示vector的开始内存位置，已分配对象的尾部位置，和总内存的位置，当vector调用push_back或者insert函数时，都会检查vector中是否有足够空间，如果没有则在原来空间的基础上扩大一倍。  
* 如果初始构造vector指定了内部初始化的对象个数，则内部空间为初始化空间。如`vector<int> vec(3,3)`初始化了3个int型3，则此时的vector.capacity()为3。之后对容器扩容是在3的基数上加倍。  
* vector的每次扩容要做3个动作，构造新总空间，拷贝原对象到新空间，销毁回收原对象空间。其中哪一步出错，都会执行**commit or rollback**策略，要么都操作成功，要么回退到操作前的样子。避免因意外报错，内存泄露。  
* 所以每次vector加入对象的时候都可能触发空间的重新分配（在新的连续空间，与旧的空间没有关系了），因此它的迭代器和指针都会改变，这会引起它们失效，引发未预期的错误。


### list

* list的实现其实就是双向链表，每个节点的内存地址实际上并不连续。  
* list为了满足左闭右开的原则，在尾部链接了一个空节点，此节点的pre指向list的最后一个节点，next指向头节点，所以是一个**环形双向链表**。因此一个空list的内部其实就是一个空节点，它的pre指向自己，next也指向自己。类似的，只包含一个节点的list满足`(end()->next)->next == end()`  
* 每次插入一个节点都是常数时间的操作，每次都只要把链表pre和next指针调整一下。


list中的merge函数输入的前提是两个list都是已经排序的，源码中每次从两个list遍历比较，如果第二的节点比第一个节点小，则将其插入第一个节点前。  
考虑到已经排序的节点，第二个list中节点有可能连续几个都比第一个节点小，是不是可以将其一段插入，而不是每次比较后插入一个节点。  
举例，a为第一list，ai为第一list的第i个节点,b为第二list,bi为第二list的第i节点；b1,b2,b3都小于a1,源码的做法是将其一个个插入a1前。是不是可以将b1,b2,b3一起插入a1前，减少插入节点的次数。  
以下是SGI_STL的源码，注释部分为我觉得可以改进的地方。
```cpp
template<class _Tp, class _Alloc>
void list<_Tp, _Alloc>::merge(list<_Tp, _Alloc>& __x){
    iterator __first1 = begin();
    iterator __last1 = end();
    iterator __first2 = __x.begin();
    iterator __last2 = __x.end();
    whlie(*__first1 != *__last1 && *__first2 != *__last2){
        if (*__first2 < *__first1){
            iterator __next == __first2;

            // GeorgeCaoJ's improvement opinion
            while(*(++next) < *first1){};
            // GeorgeCaoJ's improvement opinion
            
            transfer(__first1, __first2, ++next);
            __first2 = __next;
        }
        else
            ++__first1;
        if (__first2 != __last2)
            transfer(__last1, __first2, __last2);
    }
}

```

### deque

* 双开口的连续线性空间（逻辑连续，实际则不是连续的），实现在两头操作都是常数时间的效率  
* 为了让逻辑连续，而实际不连续，内部的迭代器需要重新设计。deque内部用map(一块连续空间，非STL的map）的形式，保持让多个非连续的空间在逻辑上是连续的，因而在每次进行操作时都需要检查空间是否连续，不连续则要切换空间。
* 内部空间map表的存储的是多个连续空间的地址头部，因此map表的类型为`T**`, 指向实际存储元素的连续地址的头部。
* 当内部map主控空间都填满之后，也要另觅一个更大的空间，和vector的扩容行为相同。
### stack & queue
* 都是基于deque来实现，只是限制了其对外的接口。
* stack.push(n) -> deque.push_back(n), stack.pop() -> deque.pop_back()
* queue.push(n) -> deque.push_front(n), queue.pop() -> deque.pop_back();

### heap

* heap在STL内并没有定义成容器，只算是一种算法，所以包含在**algorithm.h**头文件内。
* heap通常以vector为底层容器，可以实现动态扩容；也能以数组当容器，但是没法扩容。
* 算法实现上是以完全二叉树的数据结构，整个树填满的顺序是从上到下，从左到右，中间没有空隙。因此可以用一个数组来存储，从index为0存储根节点开始，index为i有如下性质：
  1. i节点的父节点在(i-1)/2处
  2. i的左子节点在2*i + 1处
  3. i的右子节点在2*i + 2处
* heap可以分为max-heap(父节点的值都大于其子节点的值)和min-heap(小于)两种
* 堆排序是通过将其转换成最大堆后，一个个从堆中pop最大的值出来，就可以成为有序数列了，其时间复杂度是O(NlogN)。后面看到STL中的快排会用到堆排序

### priority queue
* 优先队列中的元素都是带权值概念，权值中的排在前面，优先出列，但是入列的顺序是随机的
* 优先队列实现是使用最大堆，底部容器默认还是vector

### 关联式容器
### set、multiset、map、multimap底层实现--红黑树RB-tree
* 首先它是二叉搜索树(任何节点的键值都大于左子节点，小于其右子节点),其查找、插入的平均时间为O(logN)。
* 为了防止二叉搜索树的搜索时间恶化到O(N)，于是有了平衡二叉搜索树(AVL-tree),其平衡条件建立在确保整个树的深度在logN，其要求任何节点的左右子树高度相差最多1
* STL使用了另一种平衡二叉搜索树RB-tree，其定义为：
  1. 每个节点非黑即红
  2. 根节点为黑色
  3. 父子节点不得同时为红
  4. 任意节点到达NULL节点的任何路径，包含的黑色节点必须相同
* RB-tree在实现上也采用了类似list的技巧，在头部维护一个空节点，包含3个方向，一个指向整棵树的最左节点，一个指向最右，一个指向根节点

### set
* set的键值等于其实值，且不存在相同键值，set的iterator是一个RB-tree的const iterator，是不允许修改键值的
* 在set中查找，推荐使用set内部定义的find函数，只需O(logN)时间，而调用STL的find函数进行顺序查找时间复杂度是O(N)
### map
* map的每个元素是一个pair类型，拥有实值和键值，其数据结构定义如下：
```cpp
template <class T1, class T2>
struct pair{
    typedef T1 first_type;
    typedef T2 second_type;
    T1 first;
    T2 second;
    pair():first(T1()), second(T2()){}
    pair(const T1& a, const T2& b):first(a), second(b){}
};
```
* 所以其迭代器返回的是一个pair，访问其键值和value值时使用`iter->first`和`iter->second`
* 使用map时需要注意的一点是它的`T2& operator[](const key_type& k){ return (*((insert(value_type(k, T())).first)).second }`，它首先尝试插入一个空构造函数初始化的一个变量，如果存在相同的key，插入失败，返回已经存在的value值；如果插入成功，返回这个刚刚初始化的value值。所以int的默认初始化成0，string默认初始化成""，（区别于python的dictionary，访问不存在的key值会报错）。另外，返回的是引用，因此可以是用来当做左值和右值

### multiset & multimap
* 只是插入RB-tree时候使用的insert_equal()接口，允许插入相同的键值

### hash_set、hash_multiset、hash_map、hash_multimap底层实现——hashtable
* hashtable只需要O(1)的时间复杂度完成插入、删除和搜索。简单是情况可以使用数组来实现最简单的hashtable。假设使用数组来存储5个数字，其中最大的数为65535，那么需要创建65536大小的数组，显然太浪费空间。
* hashtable用hash函数来压缩大小，将大数映射成小数。默认的STL使用**X % TableSize**，但**TableSize**再大也会出现相同的数字取余相等的情况，这就是所谓的碰撞。
* 为了解决碰撞问题，STL使用**开链**策略，就是当遇到碰撞情况，就是把每个索引看成一个链表，碰撞的时候就在链接在这个索引链表的后面。当链表长度够短，其速度也依然能稳定在O(1)的复杂度。
* hashtable表格一般使用vector容器，每个元素称为一个桶(bucket)，当其负载系数(元素个数除以表格大小)大于1的时候，需要扩大表格，同扩大vector操作类似，只是大小是当前表格大小的下一个素数大小。默认的表格大小从素数53起。
* hashtable能处理char*，char,short, int, long和其有符号类型，不能处理string，double，float等类型，必须用户自己定义hash function。
### hash_set、hash_multiset、hash_map、hash_multimap
* hash_set没有自动排序功能，不同与用RB-tree实现的set(map同样）

## 算法
### 迭代器
迭代器将元素的遍历动作进行抽象和统一接口：对于具有连续空间类型的容器，迭代器--和++动作对应了指针的--和++；而对应于非连续空间的容器，如list链表，迭代器--和++对应与链表的指针移动。因此，迭代器也属于泛化的一部分，为算法泛化提供了便利。

### Sort算法
作为最基本的常用的基础算法，STL追求速度和性能上的优化，对sort算法也是有很多优化细节。
* sort使用了三种排序算法：插入排序、快速排序和堆排序
* 当元素个数小于一个阈值(STL默认为16),直接使用插入排序。因为相比于快速排序的递归调用开销(递归要在栈空间保存函数临时变量、返回地址等），插入排序在小数量的元素效果反而比较好
* 当大于阈值时使用快速排序，为了避免输入的数不够随机而导致恶化到O(N^2),使用了**Introspective Sorting(内醒式排序)(极类似于mdeian-of-three QuickSort**,采用头，尾，中央三个元素的中值当做pivot。为了快速取得这三个数，所以迭代器必须使用**RadomAccessIterator**
* 为了避免快速排序分割恶化(递归次数过深),通过元素个数n得到最大分割层数**2^k <= n**的最大k值。分割块内元素较少的时候，其实分割效率比较低，也避免了栈空间溢出的风险，转而对每个分割块进行堆排序。
* 当进行堆排序过后，每个块内的元素基本已经排序完毕，最后调用插入排序对每个块进行最后的检验，只需要比较而一般不会用到插入移动元素。
* 时间复杂度同样是O(NlogN)，为什么优先使用快速排序，而不直接使用堆排序呢？在[知乎](https://www.zhihu.com/question/23873747)上找到一个最简单明了的回答：
> 堆排比较的几乎都不是相邻元素，对cache极不友好。数学上的时间复杂度不代表实际运行时的情况
涉及计算机存储器结构，CPU每次都是对缓存成块进行读取，所以不相邻的元素会频繁在缓存中切换访问的块，所以一个优秀的代码需要考虑缓存友好，考虑到缓存局部性的特点。

