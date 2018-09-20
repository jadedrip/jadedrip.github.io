---
title: 析构函数的秘密
layout: post
tags:
  - C++
category: Uncategoried
---
C++ 对虚拟析构函数的解释：

By making the Base class Destructor virtual, both the destructors will be called in order. 
The order of execution of destructor in an inherited class during a clean up is like this. 1. Derived class destructor 2. Base class destructor
要点：

1. 多层继承的时候，只要基类的析构是virtual的。每层 析构都会被调用
2. 如果基层不是 virtual, 那么继承类的析构函数就不会被调用,这可能导致内存泄露

3. 如果基层的析构函数不是 virtual，而 其它层是 virtual， 可能会导致异常。

对3的解释：不同编译器对这个的做法是不同的，当然，不管怎么说，这种情况应该避免。在VC 2005中，非基层的虚拟析构函数，将在基层的非虚拟析构函数之后被调用，但是由于基层的析构函数已经被调用，类已经被释放，因此会导致一个异常,个人感觉是vc的bug。

一个简单的例子：

    #include <iostream>
    using namespace std;
    class Base
    {
    public:
           Base(){ cout<<"Constructor: Base"<<endl;}
           /*virtual*/ ~Base(){ cout<<"Destructor : Base"<<endl;}
    };
     
    class Sec : public Base
    {
    public:
           Sec(){ cout << "Constructor: Sec"<< endl;}
           /*virtual*/~Sec(){ cout << "Destructor: Sec"<< endl;}
    };
    class Derived: public Sec
    {
           //Doing a lot of jobs by extending the functionality
    public:
           Derived(){ cout<<"Constructor: Derived"<<endl;}
           /*virtual*/~Derived(){ cout<<"Destructor : Derived"<<endl;}
    };
    void main()
    {
           Base *Var = new Derived();
           delete Var;
           int i;
           cin >> i;
    }