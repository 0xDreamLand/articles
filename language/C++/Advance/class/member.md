## 类成员函数的指针（非静态）

指向类成员函数的指针与普通函数指针的区别在于，前者需要匹配函数的参数类型和个数以及返回值类型，还要匹配该函数指针所属的类类型。
这是因为非静态的成员函数必须被绑定到一个类的对象或者指针上，才能得到被调用对象的this指针，然后才能调用指针所指的成员函数（所有类的对象都有自己数据成员的拷贝，但是成员函数都是共用的，为了区分是谁调用了成员函数，就必须有this指针，this指针是隐式的添加到函数参数列表里去的）。

> 返回值 (类名::*指针类型名)(参数列表) = &类名::成员函数名;

- 注意：这里的这个&符号是比较重要的：不加&，编译器会认为是在这里调用成员函数，所以需要给出参数列表，否则会报错；加了&，才认为是要获取函数指针。这是C++专门做了区别对待。
- 注意：这里的前面一对括号是很重要的，因为()的优先级高于成员操作符指针的优先级。

直接来看一个示例吧：
``` cpp
#include <iostream>
using namespace std;
class Calculation
{
public:
    int add(int a,int b){ //非静态函数
        return  a + b;
    }
};

int (Calculation::*FuncCal)(int,int);

int main()
{
    FuncCal = &Calculation::add;
    Calculation * calPtr = new Calculation;
    int ret = (calPtr->*FuncCal)(1,2);  //通过指针调用

    Calculation cal;
    int ret2 = (cal.*FuncCal)(3,4);  //通过对象调用

    cout << "ret = " << ret << endl;
    cout << "ret2 = " << ret2 << endl;
    return 0;
}
```
----

## 指向类的静态函数的指针

****类的静态成员函数和普通函数的函数指针的区别在于，他们是不依赖于具体对象的，所有实例化的对象都共享同一个静态成员，所以静态成员也没有this指针的概念。****

所以，指向类的静态成员的指针就是普通的指针。
``` cpp
class Calculation
{
public:
    static int add(int a,int b){ //非静态函数
        return  a + b;
    }
};

int (*FuncCal)(int,int);

int main()
{
    FuncCal = &Calculation::add;
    int ret = (*FuncCal)(1,2);  //直接引用
    int ret2 = FuncCal(3,4);  //直接引用

    cout << "ret = " << ret << endl;
    cout << "ret2 = " << ret2 << endl;
    return 0;
}
```

> 总结以上两种情况的区别：

    如果是类的静态成员函数，那么使用函数指针和普通函数指针没区别，使用方法一样
    如果是类的非静态成员函数，那么使用函数指针需要加一个类限制一下。

使用函数指针，很多情况下是用在函数的参数中，在一些复杂的计算，如果需要重复调用，并且每次调用的函数不一样，那么这时候使用函数指针就很方便了，可以减少代码量。
