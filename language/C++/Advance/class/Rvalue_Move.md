## 右值引用，移动语义，移动构造函数和移动赋值运算符
----
### 右值引用

C++的引用允许你为已经存在的对象创建一个新的名字。对新引用所做的访问和修改操作，都会影响它的原型。这叫左值引用。

左值引用只能被绑定在左值上，所以不能这样写：
``` cpp
int& i=42;  // 错误
```
42是一个右值，所以无法绑定，但也有例外，比如可以使用下面的方式将一个右值绑定到一个const左值引用上：
``` cpp
int const& i = 42;
```
这算是钻了标准的一个空子。因为const引用创建一个临时对象，再将左值引用绑定上去。 其允许隐式转换，所以可以这样写：
``` cpp
void print(std::string const& s);
print("hello");  // 创建了临时std::string对象
```
C++11标准添加了右值引用(rvalue reference)，这种引用只能绑定右值，不能绑定左值，它使用两个&&来声明：
``` cpp
int&& i=42;
int j=42;
int&& k=j;  // 编译失败
```
由于这个符号区别于左值引用的符号，所以可以与左值引用一起用于函数重载。

### 移动语义

开发中经常使用左值引用作为函数参数，避免拷贝以提高性能。通常还会加上const修饰符，以避免函数内部对源对象的修改。比如：
``` cpp
// 函数1，接受左值引用
void process_copy(const std::vector<int>& vec_) {
    // do_something
    std::vector<int> vec(vec_); //  不能修改左值，所以要拷贝vector
    vec.push_back(42);
}

// 函数2，接受右值引用
void process_copy(std::vector<int> && vec) {
    vec.push_back(42); // 直接修改右值
}

int main(){
    std::vector<int> data;
    process_copy(data); // 调用函数1
    process_copy(std::vector<int>()); // 调用函数2，临时对象作为右值，函数内部无需拷贝，降低开销
    return 0;
}
```
一般情况下，我们只编写函数1，就已经能满足需求了，它能接受左值引用，也能接受右值引用（传参时右值引用会被转化为const左值引用）。

但是函数内部无法区分调用者传递的是左值还是右值，无论如何都是拷贝后再进行修改等操作。如果传递的是右值，虽然程序可以正确运行，但进行了一次毫无必要的拷贝，如果能直接修改右值，就能节省开销。

于是可以编写第二个函数，它接受右值引用，与函数1形成重载。在传递右值引用时，就会自动调用函数2，这样无需拷贝，可以提高性能。

**这称之为移动语义，将属于main函数块的临时对象的所有权，交给process_copy函数中。**

为什么使用左值引用不能叫做移动呢？

虽然main函数中的对象可以通过data的左值引用将对象本身（而不是拷贝的副本）传递给process_copy函数，但process_copy并不真正拥有对象的所有权，因为调用者可能在之后还会使用该对象，所以process_copy不能修改该对象。

而使用右值引用传递对象，则代表调用者（有意或无意的）保证不会在之后继续使用该对象，所以process_copy可以任意修改该右值。
### 移动构造函数
> 减少不必要的拷贝

许多情况下，类的拷贝构造函数会被隐式调用，比如调用函数时的值传递。
``` cpp
class Person {
private:
    int* data;

public:
    Person() : data(new int[1000000]) {}
    ~Person() { delete [] data; }

    // 拷贝构造函数，需要拷贝动态资源
    Person(const Person& other) : data(new int[1000000]) {
        std::copy(other.data,other.data+1000000,data);
    }
};

void func(Person p){
    // do_something
} 

int main(){
    Person p;
    func(p); // 调用func时，会调用Person的拷贝构造函数来创建实参
    return 0;
}
```
调用func时使用Person的拷贝构造函数创建一个实参。

考虑这样一种情况，使用一个临时的Person对象，作为函数参数传递给func：

int main(){
    func(Person()); // 先创建临时的Person对象，再调用Person的拷贝构造函数来创建实参
    return 0;
}

这里创建的临时对象是一个右值，它作为func函数的参数，但func函数还是忠实的拷贝了它，因为拷贝构造函数的`const Person&`参数可以接收右值。

`Person`内部包含一个很大的动态分配的数组，那么拷贝它的开销会非常大，显然拷贝一个临时对象（连带着拷贝其中的动态资源）是毫无必要的，所以我们应该优化它。

**如果能直接使用Person临时对象内部的动态资源（不是直接使用临时对象本身），而不进行完整的拷贝（不是完全不拷贝），就会节省非常多的开销。 并且不会影响程序的正确性（反正调用者之后也不会用它了，因为是临时对象）**

于是可以编写一个`移动构造函数`，与拷贝构造函数实现重载：
``` cpp
class Person {
private:
    int* data;

public:
    Person() : data(new int[1000000]){}
    ~Person() { delete [] data; }

    // 拷贝构造函数，需要拷贝动态资源
    Person(const Person& other) : data(new int[1000000]) {
        std::copy(other.data,other.data+1000000,data);
    }

    // 移动构造函数，无需拷贝动态资源
    Person(Person&& other) : data(other.data) {
        other.data=nullptr; // 源对象的指针应该置空，以免源对象析构时影响本对象
    }
};

void func(Person p){
    // do_something
} 

int main(){
    Person p;
    func(p); // 调用Person的拷贝构造函数来创建实参
    func(Person()); // 调用Person的移动构造函数来创建实参
    return 0;
}
```
> 移动构造函数接受右值引用，直接获取老数据。

移动构造函数当然产生了新的Person对象，这点与拷贝构造函数无区别。但是并没有拷贝动态分配的资源，而只是将源对象的数据移到新对象中（本例中，仅仅拷贝一个指针）。

> 很重要的一点，将动态数据移动到新对象中后，应该解除与源对象的关系。

在这个例子中，就是把源对象的指针置为nullptr，不然源对象析构时，会将数据释放，影响到本对象。

> 还可以对非临时对象调用移动构造函数。
``` cpp
int main(){
    Person p1;
    func(std::move(p1)); // 调用移动构造函数，应保证之后不再使用p2

    Person p2;
    func(static_cast<X&&>(p2)); // 调用移动构造函数后，应保证之后不再使用p2
    return 0;
}
``` 
std::move()可以提取对象的右值，而static_cast<X&&>将对应变量转换为右值。

这样显示转换为右值之后，应保证之后不再使用该对象。
----

### 管理不可拷贝的资源

****有些类型的构造函数只支持移动构造函数，而不支持拷贝构造函数****

例如，智能指针std::unique_ptr<>的非空实例中，只允许这个指针指向其对象，所以拷贝函数在这里就不能用了(如果使用拷贝函数，就会有两个std::unique_ptr<>指向该对象，不满足std::unique_ptr<>定义)。

但有时我们希望可以转移对象的所有权，所以就需要实现移动构造函数。
``` cpp
#include <iostream>

class Person {
private:
    int* data;

public:
    Person() : data(new int[1000000]){}
    ~Person() { delete [] data; }

    // 删除拷贝构造函数
    Person(const Person& other) = delete;

    // 移动构造函数，无需拷贝动态资源
    Person(Person&& other) : data(other.data) {
        other.data=nullptr; // 源对象的指针应该置空，以免源对象析构时影响本对象
    }
};

void func(Person p){
    // do_something
}

int main(){
    Person p;
    func(p); // 错误，不可拷贝
    func(std::move(p)); // 正确，调用Person的移动构造函数来创建实参
    return 0;
}
```

这样，即使在没有拷贝构造函数的情况下，也能移动资源。

### 移动赋值运算符
移动赋值运算符和移动构造函数行为很接近，也很好理解。
``` cpp

#include <iostream>
using namespace std;

class Person
{
private:
    int age;
    string name;
    int* data;

public:
    Person() : data(new int[1000000]){}
    ~Person() { delete [] data; }

    // 拷贝构造函数
    Person(const Person& p) :
    age(p.age),
    name(p.name),
    data(new int[1000000]){
        std::copy(p.data, p.data+1000000, data);
        cout << "Copy Constructor" << endl;
    }

    // 拷贝赋值运算符
    Person& operator=(const Person& p){
        this->age = p.age;
        this->name = p.name;
        this->data = new int[1000000];
        std::copy(p.data, p.data+1000000, data);
        cout << "Copy Assign" << endl;
        return *this;
    }

    // 移动构造函数
    Person(Person &&p) :
    age(std::move(p.age)),
    name(std::move(p.name)),
    data(p.data){
        p.data=nullptr; // 源对象的指针应该置空，以免源对象析构时影响本对象
        cout << "Move Constructor" << endl;
    }

    // 移动赋值运算符
    Person& operator=(Person &&p){
        this->age = std::move(p.age);
        this->name = std::move(p.name);
        this->data = p.data;
        p.data=nullptr;
        cout << "Move Assign" << endl;
        return *this;
    }
};

int main(){
    Person p1;
    Person p2 = p1; // 拷贝构造函数

    Person p3,p4;
    p3 = p4; // 拷贝赋值运算符

    Person p5;
    Person p6 = std::move(p5); // 移动构造函数

    Person p7,p8;
    p7 = std::move(p8); // 移动赋值运算符

    return 0;
}
```
