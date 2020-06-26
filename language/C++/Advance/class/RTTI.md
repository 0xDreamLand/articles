## C++中的2个运算符支持RTTI，即Run Time Type Identification：typeid和dynamic_cast
    要想rtti工作, 基类中必须有一个virtual函数, 才能在运行时根据虚表中的信息判断类型, 而没有虚函数的话,在编译时类型就确定了.
**typeid 主要是用于两个指针是否指向同一类型的对象**
> typeid()运算符和sizeof运算符一样，是C++语言直接支持的。它以一个对象或者类型名作为参数，返回一个对应于该类型的const type_info对象（其实是这个类型
> 对应的type_info对象的引用），表明该对象的确切类型。也可以使用typeid来查看非多态型对象和基本数据类型对象的类型信息。只不过，此时它不会去检索对象的
> vptr和vtable，它们根本就没有这些设施。此时typeid通过编译器维护的信息来返回结果，其结果仍然是操作数静态类型对应的type_info对象。


在多态类的对象中，存在一个虚函数表指针vptr，指向类型对应的虚函数表vtable。vtable的第一项为一个type_info指针，指向该类型对应的type_info对象。vtable从第二项开始，就是该类型的虚函数指针了。对多态类对象应用typeid操作符，需要检索vptr指针，从vtable的第一项获得该对象类型对应的type_info对象。这个过程和虚函数的动态绑定是相同的，它们的代价相同。
**使用typeid的时候，需要注意：typeid()括号中的可以是引用，但使用指针的时候要解引用指针，如typeid(*p)。*p和p的type_info信息是完全不同的。另外，对空指针进行typeid调用，会抛出std::bad_typeid异常**
``` cpp
Human *human1 = new Men();
Human *human2 = new Women();
if(typeid(human1) == typeid(human2)){
//YES, but not our want.
}

```
