## 继承方式及访问权限

    基类中      继承方式             子类中

    public     public继承        => public
    protected  public继承        => protected
    private    public继承        => 子类无权访问

    
    public     protected继承     => protected  
    protected  protected继承     => protected  
    private    protected继承     => 子类无权访问

    
    public     private继承       => private
    protected  private继承       => private
    private    private继承       => 子类无权访问

**总结**<br/>
- 子类public继承不能改变父类的访问权限
- protected继承将父类的public成员变为子类的protected
- 子类private继承将public, protected 变为子类的private
- 父类中的private成员, 不受继承方式的影响,子类永远无法访问
- 对于父类来讲, 尤其是父类的成员函数, 如果你不想外界访问就设成private，如果你想让子类访问就设置成protected，如果你想公开public

## 函数遮蔽
    - 基类和派生类成员的名字一样时会造成遮蔽
    - 不管函数的参数如何，只要名字一样就会造成遮蔽。换句话说，基类成员函数和派生类成员函数不会构成重载，如果派生类有同名函数，
    那么就会遮蔽基类中的所有同名函数，不管它们的参数是否一样。
即使派生类的成员（包括成员变量和成员函数）和基类中成员重名，造成遮蔽，仍然可以访问基类的成员变量和成员函数，不过要加上类名和域解析符。如：
```` cpp
Derived *p = new Derived;
p->Base::fun(); //使用指向派生类的指针p来调用基类Base中的函数fun；
cout << p->Base::i << endl; //输出基类

或者在子类中使用 using Base::fun();
但是如果参数列表也相同, 一样会造成遮蔽.

````

## override
> 当在子类中重载虚函数时, 如果一不小心写错了函数名, 那件事一个全新的函数,他渴望被当前类的子类重载, 是一个全新的函数, 和我们的本意大相径庭, 
> 在函数声明后加override 可以让编译器帮我们检查, 从而避免出错.

## final
**C++11的关键字final有两个用途：禁止虚函数被重写、禁止基类被继承**

```` cpp
#include "final.hpp"
#include <iostream>
 
/////////////////////////////////////////////////
// reference: http://en.cppreference.com/w/cpp/language/final
struct Base {
	virtual void foo();
};
 
struct A : Base {
	virtual void foo() final; // A::foo is final
	// void bar() final; // Error: non-virtual function cannot be final
};
 
struct B final : A { // struct B is final
	// void foo(); // Error: foo cannot be overridden as it's final in A
};
 
// struct C : B { }; // Error: B is final
 
////////////////////////////////////////////////////////
// reference: http://blog.smartbear.com/c-plus-plus/use-c11-inheritance-control-keywords-to-prevent-inconsistencies-in-class-hierarchies/
struct A_ {
	virtual void func() const;
};
 
struct B_ : A_ {
	void func() const override final; //OK
};
 
// struct C_ : B_ { void func()const; }; //error, B::func is final
````

## 继承构造函数

> 子类为完成基类初始化，在C++11之前，需要在初始化列表调用基类的构造函数，从而完成构造函数的传递。如果基类拥有多个构造函数，那么子类也需要实现多个与基
> 类构造函数对应的构造函数。
``` cpp
class Base
{
public:
	Base(int va) :m_value(va), m_c(‘0’){}
	Base(char c) :m_c(c) , m_value(0){}
private:
	int m_value;
	char m_c;
};

class Derived :public Base
{
public:
	//初始化基类需要透传基类的各个构造函数，那么这是很麻烦的
	Derived(int va) :Base(va) {}
	Derived(char c) :Base(c) {}

	//假设派生类只是添加了一个普通的函数
	void display()
	{
//dosomething		
	}
};

```

书写多个派生类构造函数只为传递参数完成基类的初始化，这种方式无疑给开发人员带来麻烦，降低了编码效率。从C++11开始，推出了继承构造函数（Inheriting Constructor），使用using来声明继承基类的构造函数，我们可以这样书写。
``` cpp
class Base
{
public:
	Base(int va) :m_value(va), m_c('0') {}
	Base(char c) :m_c(c), m_value(0) {}
private:
	int m_value;
	char m_c;
};

class Derived :public Base
{
public:
	//使用继承构造函数
	using Base::Base;

	//假设派生类只是添加了一个普通的函数
	void display()
	{
//dosomething		
	}
};
```
上面代码中，我们通过 using Base::Base 把基类构造函数继承到派生类中，不再需要书写多个派生类构造函数来完成基类的初始化。更为巧妙的是，C++11 标准规定，继承构造函数与类的一些默认函数（默认构造、析构、拷贝构造函数等）一样，是隐式声明，如果一个继承构造函数不被相关代码使用，编译器不会为其产生真正的函数代码。这样比通过派生类构造函数“透传构造函数参数”来完成基类初始化的方案，总是需要定义派生类的各种构造函数更加节省目标代码空间。

----

****注意事项****

> 继承构造函数无法初始化派生类数据成员。

继承构造函数的功能是初始化基类，对于派生类数据成员的初始化则无能为力。解决的办法主要有两个：<br/>

一是使用 C++11 特性就地初始化成员变量，可以通过 =、{} 对非静态成员快速地就地初始化，以减少多个构造函数重复初始化变量的工作，注意初始化列表会覆盖就地初始化操作。
``` cpp
class Derived :public Base
{
public:
	//使用继承构造函数
	using Base::Base;

	//假设派生类只是添加了一个普通的函数
	void display()
	{
//dosomething		
	}
private:
//派生类新增数据成员
double m_double{0.0};
};
```

二是新增派生类构造函数，使用构造函数初始化列表初始化。
``` cpp
class Derived :public Base
{
public:
	//使用继承构造函数
	using Base::Base;

	//新增派生类构造函数
	Derived(int a,double b):Base(a),m_double(b){}

	//假设派生类只是添加了一个普通的函数
	void display()
	{
		//dosomething		
	}
private:
	//派生类新增数据成员
	double m_double{0.0};
};
```
相比之下，第二种方法需要新增构造函数，明显没有第一种方法简洁，但第二种方法可由用户控制初始化值，更加灵活。各有优劣，两种方法需结合具体场景使用。

> 构造函数拥有默认值会产生多个构造函数版本，且继承构造函数无法继承基类构造函数的默认参数，所以我们在使用有默认参数构造函数的基类时就必须要小心。
``` cpp
class A
{
public:
	A(int a = 3, double b = 4):m_a(a), m_b(b){}
	void display()
	{
		cout<<m_a<<" "<<m_b<<endl;
	}

private:
	int m_a;
	double m_b;
};

class B:public A
{
public:
using A::A;
};
```
那么A中的构造函数会有下面几个版本：
``` cpp
A()
A(int)
A(int,double)
A(const A&)
```

那么B中对应的继承构造函数将会包含如下几个版本：
``` cpp
B()
B(int)
B(int,double)
B(const B&)
```

可以看出，参数默认值会导致多个构造函数版本的产生，因此在使用时需格外小心。

> 多继承的情况下，继承构造函数会出现“冲突”的情况，因为多个基类中的部分构造函数可能导致派生类中的继承构造函数的函数名与参数相同，即函数签名。考察如下代码：
``` cpp
class A
{
public:
	A(int i){}
};

class B
{
public:
	B(int i){}
};

class C : public A,public B
{
public:
	using A::A;
	using B::B;  //编译出错，重复定义C(int)
	
	//显示定义继承构造函数C(int)
	C(int i):A(i),B(i){}
};
``` 
> 为避免继承构造函数冲突，可以通过显示定义来阻止隐式生成的继承构造函数。
> 此外，使用继承构造函数时，还需要注意：如果基类构造函数被申明为私有成员函数，或者派生类是从虚基类继承而来 ，那么就不能在派生类中申明继承构造函数。

## 虚基类 虚继承






