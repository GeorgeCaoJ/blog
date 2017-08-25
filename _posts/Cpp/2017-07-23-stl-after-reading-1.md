---
layout: pageshow
title: STL读后知识点整理
category: Cpp
desceiption: 读STL源码剖析的知识点整理
---

## vector
* vector的内部空间是一块连续线性空间  
* vector通过内部的Alloctor内存分配器动态分配内存。每次对vector加入新的对象时，都可能对内存重新分配。  
* vector的对象总是通过在新的内存空间construct出来的，所以并不是原对象的一个引用。区别于python中的数组（python中的数组对基础数据类型POD，直接是copy了一份，但是对类对象，则是引用而已）
* vector内部有3个指示器，start，finish和end_of_storage，分别表示vector的开始内存位置，已分配对象的尾部位置，和总内存的位置，当vector调用push_back或者insert函数时，都会检查vector中是否有足够空间，如果没有则在原来空间的基础上扩大一倍。  
* 如果初始构造vector指定了内部初始化的对象个数，则内部空间为初始化空间。如`vector<int> vec(3,3)`初始化了3个int型3，则此时的vector.capacity()为3。之后对容器扩容是在3的基数上加倍。  
* vector的每次扩容要做3个动作，构造新总空间，拷贝原对象到新空间，销毁回收原对象空间。其中哪一步出错，都会执行**commit or rollback**策略，要么都操作成功，要么回退到操作前的样子。避免因意外报错，内存泄露。  
* 所以每次vector加入对象的时候都可能触发空间的重新分配（在新的连续空间，与旧的空间没有关系了），因此它的迭代器和指针都会改变，这会引起它们失效，引发未预期的错误。


## list

* list的实现其实就是双向链表，每个节点的内存地址实际上并不连续。  
* list为了满足左闭右开的原则，在尾部链接了一个空几点，此节点的pre指向list的最后一个对象，next指向头对象。因此一个空list的内部其实就是一个空节点，它的pre指向自己，next也指向自己。类似的，只包含一个节点的list满足`(end()->next)->next == end()`  
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

## deque

* 双开口的连续线性空间（逻辑连续，实际则不是连续的），实现在两头操作都是常数时间的效率  
* 为了让逻辑连续，而实际不连续，内部的迭代器需要重新设计。deque内部用map的形式，保持让多个非连续的空间在逻辑上是连续的，因而在每次进行操作时都需要检查空间是否连续，不连续则要切换空间。
* 内部空间map表的存储的是离散空间的地址头部，因此map表的类型为`T**`, 实际的离散存储空间的头部为`T*`
