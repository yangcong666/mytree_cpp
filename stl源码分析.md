# stl介绍
标准模板库，是C++的标准库之一，一套基于模板的容器类库，还包括许多常用的算法。

STL提供六大组件，彼此可以组合套用：

- 容器（Containers）：各种数据结构，如：vector、list、deque、set、map。用来存放数据。从实现的角度来看，STL容器是一种class template。
- 算法（algorithms）：各种常用算法，如：sort、search、copy、erase。从实现的角度来看，STL算法是一种 function template。
- 迭代器（iterators）：容器与算法之间的胶合剂，是所谓的“泛型指针”。共有五种类型，以及其他衍生变化。从实现的角度来看，迭代器是一种将 operator*、operator->、operator++、operator- - 等指针相关操作进行重载的class template。所有STL容器都有自己专属的迭代器，只有容器本身才知道如何遍历自己的元素。原生指针(native pointer)也是一种迭代器。
- 仿函数（functors）：行为类似函数，可作为算法的某种策略（policy）。从实现的角度来看，仿函数是一种重载了operator()的class或class template。一般的函数指针也可视为狭义的仿函数。
- 配接器（adapters）：一种用来修饰容器、仿函数、迭代器接口的东西。例如：STL提供的queue 和 stack，虽然看似容器，但其实只能算是一种容器配接器，为它们的底部完全借助deque，所有操作都由底层的deque供应。改变 functors接口者，称为function adapter；改变 container 接口者，称为container adapter；改变iterator接口者，称为iterator adapter。
- 配置器（allocators）：负责空间配置与管理。从实现的角度来看，配置器是一个实现了动态空间配置、空间管理、空间释放的class template。

## 迭代器
## 容器
### vector
底层数据结构是一个动态数组。

默认构造的方式是0， 之后插入按照1 2  4  8  16 二倍扩容。**注（GCC是二倍扩容，VS13是1.5倍扩容。原因可以考虑内存碎片和伙伴系统，内存的浪费）**。扩容后是一片新的内存，需要把旧内存空间中的所有元素都拷贝进新内存空间中去，之后再在新内存空间中的原数据的后面继续进行插入构造新元素，并且同时释放旧内存空间，并且，由于vector 空间的重新配置，导致旧vector的所有迭代器都失效了。
**vector**

用法：

        定义：
            vector<T> vec;

        插入元素：
            vec.push_back(element);
            vec.insert(iterator, element);

        删除元素：
            vec.pop_back();
            vec.erase(iterator);

        修改元素：
            vec[position] = element;

        遍历容器：
            for(auto it = vec.begin(); it != vec.end(); ++it) {......}

        其他：
            vec.empty();    //判断是否空
            vec.size();    // 实际元素
            vec.capacity();    // 容器容量
            vec.begin();    // 获得首迭代器
            vec.end();    // 获得尾迭代器
            vec.clear();    // 清空

实现：

[模拟Vector实现](https://github.com/linw7/Skill-Tree/blob/master/code/my_vector.cpp)

- 线性表，数组实现。
    - 支持随机访问。
    
    - 插入删除操作需要大量移动数据。

- 需要连续的物理存储空间。

- 每当大小不够时，重新分配内存（*2），并复制原内容。

错误避免：

[迭代器失效](https://github.com/linw7/Skill-Tree/blob/master/code/vector_iterator.cpp)

- 插入元素
    - 尾后插入：size < capacity时，首迭代器不失效尾迭代实现（未重新分配空间），size == capacity时，所有迭代器均失效（需要重新分配空间）。
    
    - 中间插入：size < capacity时，首迭代器不失效但插入元素之后所有迭代器失效，size == capacity时，所有迭代器均失效。

- 删除元素
    - 尾后删除：只有尾迭代失效。
    
    - 中间删除：删除位置之后所有迭代失效。
### list
在Vector中如果进行插入和删除操作后迭代器会失效，List有一个重要的性质就是插入和接合操作都不会造成原有的List迭代器失效。而且，再删除一个节点时，也仅有指向被删除元素的那个迭代器失效，其他迭代器不受任何影响。下面来看看List迭代器的源码:
```cpp
template<class T, class Ref, class Ptr>
struct __list_iterator
{
    ....
    typedef bidirectional_iterator_tag iterator_category; //List的迭代器类型为双向迭代器
    typedef __list_node<t>* link_type; 

    // 这个是迭代器实际管理的资源指针
    link_type node;

    // 迭代器构造函数
    __list_iterator(link_type x) : node(x) {}
    __list_iterator() {}
    __list_iterator(const iterator& x) : node(x.node) {}

    // 重载operator *, 返回实际维护的数据
    reference operator*() const { return (*node).data; }
    // 成员调用操作符
    pointer operator->() const { return &(operator*()); }

    // 前缀自加
    self& operator++()
    {
        node = (link_type)((*node).next);
        return *this;
    }

    // 后缀自加, 需要先产生自身的一个副本, 然会再对自身操作, 最后返回副本
    self operator++(int)
    {
        self tmp = *this;
        ++*this;
        return tmp;
    }
```
list构造时首先申请一个list_node节点，作为尾端的空白节点。list中node的值就是该list_node的地址。
```cpp
void push_back(const T& x):list元素尾部增加一个元素x

    void push_front(const T& x):list元素首元素钱添加一个元素X

    void pop_back():删除容器尾元素，当且仅当容器不为空

    void pop_front():删除容器首元素，当且仅当容器不为空

    void remove(const T& x):删除容器中所有元素值等于x的元素

    void clear():删除容器中的所有元素

    iterator insert(iterator it, const T& x ):在迭代器指针it前插入元素x,返回x迭代器指针

    void insert(iterator it,size_type n,const T& x):迭代器指针it前插入n个相同元素x

    void insert(iterator it,const_iterator first,const_iterator last):把[first,last)间的元素插入迭代器指针it前

    iterator erase(iterator it):删除迭代器指针it对应的元素,返回的是it下一个节点的迭代器

    iterator erase(iterator first,iterator last):删除迭代器指针[first,last)间的元素，返回last
```
### deque
deque是连续空间（至少逻辑上看来如此），连续线性空间总令我们联想到array或vector。array无法成长，vector虽可成长，却只能向尾端成长，而且其所谓的成长原是个假象，事实上是（1）另觅更大空间；（2）将原数据复制过去；（3）释放原空间三部曲。如果不是vector每次配置新空间时都有留下一些余裕，其成长假象所带来的代价将是相当高昂。

    deque系由一段一段的定量连续空间构成。一旦有必要在deque的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个deque的头端或尾端。deque的最大任务，便是在这些分段的定量连续空间上，维护其整体连续的假象，并提供随机存取的接口。避开了“重新配置、复制、释放”的轮回，代价则是复杂的迭代器架构。

    受到分段连续线性空间的字面影响，我们可能以为deque的实现复杂度和vector相比虽不中亦不远矣，其实不然。主要因为，既是分段连续线性空间，就必须有中央控制，而为了维持整体连续的假象，数据结构的设计及迭代器前进后退等操作都颇为繁琐。deque的实现代码分量远比vector或list都多得多。

    deque采用一块所谓的map（注意，不是STL的map容器）作为主控。这里所谓map是一小块连续空间，其中每个元素（此处称为一个节点，node）都是指针，指向另一段（较大的）连续线性空间，称为缓冲区。缓冲区才是deque的储存空间主体。SGI STL 允许我们指定缓冲区大小，默认值0表示将使用512 bytes 缓冲区。

deque 的迭代器:
让我们思考一下，deque的迭代器应该具备什么结构，首先，它必须能够指出分段连续空间（亦即缓冲区）在哪里，其次它必须能够判断自己是否已经处于其所在缓冲区的边缘，如果是，一旦前进或后退就必须跳跃至下一个或上一个缓冲区。为了能够正确跳跃，deque必须随时掌握管控中心（map）。所以在迭代器中需要定义：当前元素的指针，当前元素所在缓冲区的起始指针，当前元素所在缓冲区的尾指针，指向map中指向所在缓区地址的指针，分别为cur, first, last, node。

```cpp
deque.push_back(elem);	     //在容器尾部添加一个数据
deque.push_front(elem);	    //在容器头部插入一个数据
deque.pop_back();    		//删除容器最后一个数据
deque.pop_front();	     	//删除容器第一个数据
deque.begin();  //返回容器中第一个元素的迭代器。
deque.end();  //返回容器中最后一个元素之后的迭代器。
deque.rbegin();  //返回容器中倒数第一个元素的迭代器。
deque.rend();   //返回容器中倒数最后一个元素之后的迭代器。
deque.size();	   //返回容器中元素的个数
deque.empty();	   //判断容器是否为空
 
deque.resize(num);   //重新指定容器的长度为num，若容器变长，则以默认值填充新位置。
					//如果容器变短，则末尾超出容器长度的元素被删除。
					
deque.resize(num, elem);  //重新指定容器的长度为num，若容器变长，则以elem值填充新位置。
						//如果容器变短，则末尾超出容器长度的元素被删除。
                        deque.insert(pos,elem);   //在pos位置插入一个elem元素的拷贝，返回新数据的位置。
deque.insert(pos,n,elem);   //在pos位置插入n个elem数据，无返回值。
deque.insert(pos,beg,end);   //在pos位置插入[beg,end)区间的数据，无返回值。
deque.clear();	//移除容器的所有数据
deque.erase(beg,end);  //删除[beg,end)区间的数据，返回下一个数据的位置。
deque.erase(pos);    //删除pos位置的数据，返回下一个数据的位置。
```
### stack
以某种既有容器作为底部结构，将其接口改变，使之符合“先进后出”的特性，形成一个 stack，是很容易做到的。deque 是双向开口的数据结构，若以 deque 为底部结构并封闭其头端开口，便轻而易举地形成了一个 stack。因此，STL便以 deque 作为缺省情况下的 stack 底部结构。

    由于 stack 系以底部容器完成其所有工作，而具有这种“修改某物接口，形成另一种风貌”之性质者，称为 adapter（配接器），因此 stack 往往被归类为容器配接器。

```cpp
stack.empty();   //判断堆栈是否为空
stack.size(); 	     //返回堆栈的大小
stack.top();	  //返回最后一个压入栈元素
stack(const stack &stk);		 //拷贝构造函数
stack& operator=(const stack &stk);	//重载等号操作符
stack.push(elem);   //往栈头添加元素
stack.pop();   //从栈头移除第一个元素

```
### map
用法：

        定义：
            map<T_key, T_value> mymap;

        插入元素：
            mymap.insert(pair<T_key, T_value>(key, value));    // 同key不插入
            mymap.insert(map<T_key, T_value>::value_type(key, value));    // 同key不插入
            mymap[key] = value;    // 同key覆盖

        删除元素：
            mymap.erase(key);    // 按值删
            mymap.erase(iterator);    // 按迭代器删

        修改元素：
            mymap[key] = new_value;

        遍历容器：
              for(auto it = mymap.begin(); it != mymap.end(); ++it) {
                cout << it->first << " => " << it->second << '\n';
              }

实现：

[RBTree实现](https://github.com/linw7/Skill-Tree/tree/master/code/RBTree)

- 树状结构，RBTree实现。
    - 插入删除不需要数据复制。
    
    - 操作复杂度仅跟树高有关。

- RBTree本身也是二叉排序树的一种，key值有序，且唯一。
    - 必须保证key可排序。

基于红黑树实现的map结构（实际上是map, set, multimap，multiset底层均是红黑树），不仅增删数据时不需要移动数据，其所有操作都可以在O(logn)时间范围内完成。另外，基于红黑树的map在通过迭代器遍历时，得到的是key按序排列后的结果，这点特性在很多操作中非常方便。

面试时候现场写红黑树代码的概率几乎为0，但是红黑树一些基本概念还是需要掌握的。

1. 它是二叉排序树（继承二叉排序树特显）：
    - 若左子树不空，则左子树上所有结点的值均小于或等于它的根结点的值。

    - 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值。

    - 左、右子树也分别为二叉排序树。

2. 它满足如下几点要求：
    - 树中所有节点非红即黑。

    - 根节点必为黑节点。

    - 红节点的子节点必为黑（黑节点子节点可为黑）。

    - 从根到NULL的任何路径上黑结点数相同。

3. 查找时间一定可以控制在O(logn)。

4. 红黑树的节点定义如下：
    ```C++
    enum Color {
        RED = 0,
        BLACK = 1
    };
    struct RBTreeNode {
        struct RBTreeNode*left, *right, *parent;
        int key;
        int data;
        Color color;
    };
    ```
所以对红黑树的操作需要满足两点：1.满足二叉排序树的要求；2.满足红黑树自身要求。通常在找到节点通过和根节点比较找到插入位置之后，还需要结合红黑树自身限制条件对子树进行左旋和右旋。

相比于AVL树，红黑树平衡性要稍微差一些，不过创建红黑树时所需的旋转操作也会少很多。相比于最简单的BST，BST最差情况下查找的时间复杂度会上升至O(n)，而红黑树最坏情况下查找效率依旧是O(logn)。所以说红黑树之所以能够在STL及Linux内核中被广泛应用就是因为其折中了两种方案，既减少了树高，又减少了建树时旋转的次数。

从红黑树的定义来看，红黑树从根到NULL的每条路径拥有相同的黑节点数（假设为n），所以最短的路径长度为n（全为黑节点情况）。因为红节点不能连续出现，所以路径最长的情况就是插入最多的红色节点，在黑节点数一致的情况下，最可观的情况就是黑红黑红排列......最长路径不会大于2n，这里路径长就是树高。

### set
SGI STL中的容器set，以RB-Tree作为其底层的实现(rb_tree的大体分析见上文)。在set容器键值key和实值value是相同的，且在容器里面的元素是根据元素的键值自动排序的，同时我们不能修改set容器里面的元素值，所以set的迭代器是采用RB-Tree的const_iterator，不允许用户对其进行修改操作。

### queue
queue是一种先进先出(First In First Out，FIFO)的数据结构，它有两个出口，允许增加元素、移除元素、从最底端加入元素、取得最顶端元素。但除了最底端可以加入、最顶端可以取出外，没有任何其他方法可以存取queue的其他元素，换言之，queue不允许有遍历行为。queue默认以deque为底层容器。
