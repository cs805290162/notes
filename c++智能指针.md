智能指针主要分为shared_ptr、unique_ptr和weak_ptr三种，使用时需要引用头文件<memory>。

C++98 中还有auto_ptr，基本被淘汰了，不推荐使用。而 C++11 中shared_ptr和weak_ptr都是参考boost库实现的。


1 shared_ptr的初始化
1、最安全的分配和使用动态内存的方法是调用一个名为 make_shared 的标准库函数。 此函数在动态内存中分配一个对象并初始化它，返回指向此对象的 shared_ptr。与智能指针一样，make_shared 也定义在头文件 memory 中。
2、shared_ptr的一个最大的陷阱是循环引用，循环引用会导致堆内存无法正确释放，导致内存泄漏。
3、不要用一个原始的指针初始化多个shared_ptr，多个shared_ptr无关联，一个引用为0释放内存，会造成空指针问题。
2 weak_ptr如何解决相互引用的问题, 对std::weak_ptr的相互引用，不会导致计数的增加
要想解决上面循环引用的问题，只能引入新的智能指针std::weak_ptr。std::weak_ptr有什么特点呢？与std::shared_ptr最大的差别是在赋值的时候，不会引起智能指针计数增加。

• weak_ptr被设计为与shared_ptr共同工作，可以从一个shared_ptr或者另一个weak_ptr对象构造，获得资源的观测权。但weak_ptr没有共享资源，它的构造不会引起指针引用计数的增加。
• 同样，在weak_ptr析构时也不会导致引用计数的减少，它只是一个静静地观察者。weak_ptr没有重载operator*和->，这是特意的，因为它不共享指针，不能操作资源，这是它弱的原因。
• 如要操作资源，则必须使用一个非常重要的成员函数lock()从被观测的shared_ptr获得一个可用的shared_ptr对象，从而操作资源。
当我们创建一个weak_ptr时，要用一个shared_ptr来初始化它：

来自 <https://www.cnblogs.com/linuxandmcu/p/10409723.html> 


3 unique_ptr独占的智能指针

不允许赋值和拷贝操作，只能够移动。可以使用move函数移动。

不能拷贝 unique_ptr 的规则有一个例外：我们可以拷贝或赋值一个将要被销毁的 unique_ptr。

unique_ptr<int> clone (int p) { unique_ptr<int> ret(new int (p)); // ... return ret; }

对于上面这段代码，编译器都知道要返回的对象将要被销毁。在此情况下，编译器执行一种特殊的“拷贝”，在《C++ Primer》13.6.2节（第473页）中有介绍。
