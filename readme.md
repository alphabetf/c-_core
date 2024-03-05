 b##### C++核心

在类中写的简短的成员函数实现，默认会自动隐式加上inline关键字，

如果成员函数不改变类中的成员变量，则应该在**函数后面加上const关键字**

**相同class的各个object互为friends(友元)**

```c++
class complex
{
public:
	complex(double r = 0, double i = 0):re(r),im(i)
    { ... }
    
    int func(const complex& param)
    {
    	return param.re + param.im;    
    }
private: // 私有属性不能被外部成员访问
    double re, im;		
}

/* 使用情景 */
complex c1(2,1);
complex c2;

c2.func(c1);  // 非友元函数却可以使用其他同类对象的私有属性
```

原因：类中的成员函数在构建编译时就能清楚的知道类的内存布局，所以编译器也允许成员函数中具有自身类类型的参数，在使用类成员变量时，其实是基于指针偏移的，所以机缘巧合的造成了这种情况，其实并非友元，但可以理解记忆为：**相同class的各个object互为friends(友元)**

**new的理解与实现:**

C++中的new可以理解为是一个写**法比较特殊的函数**，具体实现源码如下：

```c++
Complex* pc = new Complex(1,2);

void* mem = operator new( sizeof(Complex) );		/* 分配内存，内部调用maclloc（n） */
pc = static_cast<Complex*>(mem);					/* 类型转型 */
pc->Complex::Complex(1,2);							/* 调用构造函数 */
//pc->Complex::Complex(this,1,2);	/* 构造函数隐式的隐藏了一个this指针,this指向分配的内存地址*/
```

**delete的理解与实现:**

C++中的delete也可以理解为是一个**写法比较特殊的函数**，具体源码实现如下：
```c++
String* ps = new String("Hello");
...
delete ps;		

//delete内部实现源码如下
String::~String(ps);		/* 先调用类中的析构函数 */
operator delete(ps);		/* 释放分配的内存，内部调用free(ps) */
```

**delete中的内存泄露：**

```c++
/* 内存结构中会记录数组个数,每一个String对象中都会维护一个指针指向实际的字符串内存地址 */
String* p = new String[3]; 
...
delete[] p;	/* 根据内存结构中记录的数组个数,多次调用析构函数,此处会调用三次析构函数, */
delete p;	/* 只会调用一次析构函数 */
```

**虚析构函数：**只要该类将来会**被继承或成为父类**，就应将该类的析构函数**写成虚析构函数**

```C++
class Base
{
public:
    Base(){};
    virtual ~Base(){cout << "父类的析构函数被执行" << endl;};
    void DoSomething(){cout << "父类dosomething" << endl;};
}

class Derived:public Base
{
public:
    Derived(){};
    virtual ~Derived(){cout << "子类的析构函数被执行" << endl;}
    void DoSomething(){cout << "子类dosomething" << endl;};
}

int main()
{
    Base *p = new Derived; /* 此处产生多态 */
    delete p;	/* 如果Base类不是虚析构则之后调用~Base(),而不会调用~Derived()后在调用~Base() */
    return 0;
}

```

**内部原理&机制**：

​		1，所有被virtual关键字修饰的函数都会被编译进虚函数表，析构函数也不例外

​		2，在一个继承体系中按照析构顺序，所有析构函数会被整合成一个统一的析构函数，并按照一定的规则重新			  命名，同于普通函数，析构函数会被整合而不是重新或者覆盖

**设计模式:TemplateMethod**

```c++
/* 这是一个很久以前写的Application framework */
#include <iostream>
using namespace std;

class CDocument
{
public:
    void OnFileOpen()
	{
    	/* `这是一个算法，每一个cout输出代表着一个实际动作 */
    	cout << "dialog..." << endl;
    	cout << "check file status..." << endl;
    	cout << "open file..." << endl;
    	Serialize();		/* 子类重写该函数后，将会以子类的方式去读取文件 */
    	cout << "close file..." << endl;
    	cout << "update all views..." << endl;
	}
    
    virtual void Serialize(){ }; /* 不知道如何读取文件，所以交给未来的代码去处理 */
}

/* 这是一个现在写的代码，需要使用以前的框架 */
class CMyDoc : public CDocument
{
public:
    virtual void Serialize()	/* 以前写的框架将会调用现在写的函数来读取文件 */
    {
        /* 只有应用程序本身才知道该如何读取自己的文件 */
        cout << "CMyDoc::Serialize()" << endl;
    }
}

/* 使用 */
int main()
{
    CMyDoc myDoc;
    myDoc.OnFileOpen();
}
```

**设计模式:Observer**

```c++
/* 这是订阅中心 */
class Sbuject
{
    int m_value;	/* 这是一个公共数据，所有订阅者都会收到这份相同的数据 */
    verctor<Observer*> m_views; /* 订阅者列表 */
public:
    void attach(Observer* obs)	/* 想要订阅数据,应该先向订阅中心进行注册 */
    {
       m_views.push_back(obs);	
    }
    
    void set_val(int value)
    {
        m_value = value;
        notify();	/* 当数据产生变更时,应及时通知所有订阅者 */
    }
    
    void notify() /* 遍历订阅列表,通知所有订阅者 */
    {
        for(int i = 0; i < m_views.size(); ++i)
        {
            m_views[i]->updata(this, m_value);
        }
    }
};

/* 这是所有观察者的基类,所有观察者都应先在订阅中心进行注册 */
class Observer
{
Public:
    /* 子类应重写该函数,可以不同观察者的自定义数据处理或展示 */
    virtual void updata(Sbuject *sub， int value) = 0;
};
```

**设计模式:Composite**

```c++
/* 假设现在要设计一个文件系统,那么一个目录中可能有文件也可能有目录,则我们需要设计一个目录类，一个文件类
   目录类既可以添加文件类又可以添加目录类自身 */
class Component
{
    int value;
public：
    Component(int val) { value = val; }
    virtual void add(Component*){ }		/* 文件中不能添加文件或者目录,所以默认为空 */
};

/* 这是一个文件类,用于标识文件 */
class Primitive:public Componet
{
public:
    Primitive(int val):Component(val){}
};

/* 这是一个目录类,用于标识目录 */
class Composite:public Component
{
    vector<Component*> c;
public:
    Composite(int val):Component(val){}
    
    void add(Component* elem){	/* 目录中既可以添加文件也可以添加目录 */
        c.push_back(elem);
    }
};
```

**设计模式:Prototype**

```c++
/* 使用场景：让现有的继承体系有能力去创建未来才会出现的的子类 */
#include <iostream.h>
enum imageType
{
    LAST,SPOT
};
/* 父类 */
class Image
{
public:
    virtual void draw() = 0;	
    static Image* findAndClone(imageType);
protected:
    virtual imageType returnType() = 0;
    virtual Image* clone() = 0;	/* 无法知道未来子类的名称,创建一个新的子类必须由子类自己来完成 */
    static void addPrototype(Image *image)	
    {
        _prototype[_nextSlot++] = image;
    }
private:
    /* 静态变量在此处只是声明 */
    static Image* _prototype[10];	/* 一个容器,存储着向父类注册的子类 */
    static int _nextSlot;
};
/* 此处是静态变量的定义 */
Image* Image::_prototype[];
int Image::_nextSlot;

Image* Image::findAndClone(imageType type)
{
    for(int i = 0; i < _nextSlot; i++){
        if(_prototype[i]->returnType() == type){
            return _protorype[i]->clone(); /* new一个新的子类，只能由子类自己来完成 */
        }
    }
}

/* 子类,未来才会出现 */
class LandSatImage:public Image
{
public:
    imageType returnType(){
        return LSAT;
    }
    void draw()
    {
        cout << "LandSatImage::draw" << _id << endl;
    }
    Image* clone(){
        return new LandSatImage(1);	 /* 调用有参构造函数,避免重复向父类注册 */
    }
protected:
    LandSatImage(int dummy){
        _id = _count++;
    }
private:
    static LandSatImage _landSatImage; /* 默认调用自己的私有的无参构造函数 */
    LandSatImage(){
        addPrototype(this);	/* 将维护的静态类_landSatImage注册到父类中 */
    }
    int _id;
    static int _count;
};
/* 此处才是静态变量的定义 */
LandSatImage LandSatImage::_landSatImage; 
int LandSatImage::_count = 1;

/* 子类,未来才会出现 */
class SpotImage:public Image
{
public:
    imageType returnType{
        return SPOT;
    }
    void draw(){
        cout << "SpotImage::draw" << endl;
    }
    Image* clone(){
        return new SpotImage(1); /* 调用有参构造函数,避免重复向父类注册 */
    }
protected:
    SpotImage(int dummy){
        _id = _count++;
    }
private:
    SpotImage(){	/* 将维护的静态类_SpotImage注册到父类中 */
        addPrototype(this);
    }
    static SpotImage _SpotImage; /* 默认调用自己的私有的无参构造函数 */
    int _id;
    static int _count;
};
/* 此处才是静态变量的定义 */
SpotImage SpotImage::_spotImage;
int SpotImage::_count;

/* 以下是测试程序 */
// Simulated stream of creation requests
const int NUM_IMAGES = 8;
imageType input[NUM_IMAGES] =
{
	LSAT, LSAT, LSAT, SPOT, LSAT, SPOT, SPOT, LSAT
};

int main()
{
    Image *images[NUM_IMAGES];
    // Given an image type, find the right prototype, and return a clone
    for (int i = 0; i < NUM_IMAGES; i++)
        images[i] = Image::findAndClone(input[i]);
    // Demonstrate that correct image objects have been cloned
    for (i = 0; i < NUM_IMAGES; i++)
        images[i]->draw();
    // Free the dynamic memory
    for (i = 0; i < NUM_IMAGES; i++)
        delete images[i];
}
```

**转换函数:conversion function**

```c++
class Fraction
{
public:
    Fraction(int num, int den=1):m_numerator(num),m_denominator(den){ }
    /* 固定写法:转换函数,无需写返回类型,此处返回类型就是double类型(不一定要是基本类型,也		可以是任意类型),将类转换为浮点数 */
    operator double() const{ /* 转换函数一般不修改变量，所以应用const修饰 */
        return (double)(m_numerator/m_denominator);
    }
private:
    int m_numerator;	/* 分子 */
    int m_denominator;	/* 分母 */
};

/* 使用 */
Fraction f(3,5);
double d=4+f; /* 编译器会尝试将f转换为double类型,调用operator double()将f转换为0.6 */
```

**非显示的一个参数：non explicit one argument**：

```c++
class Fraction
{
public:
    Fraction(int num, int den=1):m_numerator(num),m_denominator(den){ }

    Fraction operator+(const Fraction& f){
        return Fraction(...);
    }
private:
    int m_numerator;	/* 分子 */
    int m_denominator;	/* 分母 */
};

/* 使用 */
Fraction f(3,5);
double d2=f+4;/* 编译器会尝试将4转换为Fraction类(调用构造)后在调用operator+()函数 */

```

```c++
class Fraction
{
public:
    Fraction(int num, int den=1):m_numerator(num),m_denominator(den){ }
	/* 固定写法:转换函数,无需写返回类型,此处返回类型就是double类型(不一定要是基本类型,也		可以是任意类型),将类转换为浮点数 */
    operator double() const{ /* 转换函数一般不修改变量，所以应用const修饰 */
        return (double)(m_numerator/m_denominator);
    }
    Fraction operator+(const Fraction& f){
        return Fraction(...);
    }
private:
    int m_numerator;	/* 分子 */
    int m_denominator;	/* 分母 */
};

/* 使用 */
Fraction f(3,5);
/* 编译报错:存在二义性错误,编译器可以将f转化为double类型相加后在转化为Fraction类型，也可以将4转换为	  	 Fraction类型,所以存在二义性 */
Fraction d2=f+4;
```

```c++
class Fraction
{
public:
    /* explict关键字由于显示的告诉编译器,该构造函数只能显示的被调用,不能隐式的调用,如将4隐式的转换为		   Fraction类 */
    explict Fraction(int num, int den=1):m_numerator(num),m_denominator(den){ }
	/* 固定写法:转换函数,无需写返回类型,此处返回类型就是double类型(不一定要是基本类型,也可以是任意类型),		 将类转换为浮点数 */
    operator double() const{ /* 转换函数一般不修改变量，所以应用const修饰 */
        return (double)(m_numerator/m_denominator);
    }
    Fraction operator+(const Fraction& f){
        return Fraction(...);
    }
private:
    int m_numerator;	/* 分子 */
    int m_denominator;	/* 分母 */
};

/* 使用 */
Fraction f(3,5);
/* 编译报错:转换失败,将4转换为Fraction类型不被允许,则编译器将f转换为double类型相加后,在转换为Fraction	 类型,但是double类型也无法转换为Fraction类型 */
Fraction d2=f+4;
```

 **模板特化与泛化：**

```c++
/* 模板的泛化,Key可接受任意类型 */
template <class Key>
    struct hash{}

/* 模板的特化 */
template<>
struct hash<char> {
    size_t operator()(char x) const { return x;	}
};

template<>
struct hash<int> {
    size_t operator()(int x) const { return x; }
};

template<>
struct hash<long> {
    size_t operator()(long x) const { return x; }
}

/* 使用 */
cout << hash<long>()(1000); /* 编译器会优先使用特化模板 */
```

**模板的部分特化:个数上的特化**

```c++
/* 泛化的模板 */
template<typename T, typename Alloc=......>
class  vector
{
    ...
};

/* 模板的部分特化:个数上的特化 */
template<typename Alloc=......> 
class vector<bool, Alloc>	/* 只对第一个进行特化 */
{
    ...
}
```

**模板的部分特化:范围上的特化**

```c++
/* 泛化的模板 */
template <typename T>
class C
{
    ...
}

/* 部分特化:范围上的特化 */
template <typename T>
class C<T*>	/* T是指针类型的就使用这部分特化代码 */
{
    ...
}

/* 使用 */
C<string> obj1;		/* 使用的是泛化的模板 */
C<string*> obj2;	/* 使用的是范围特化的模板 */
```

**模板中的模板参数：**

```c++
template <typename T,
		 template <typename T>	/* 模板中的第二依然是一个模板类 */
			class Container
         >
class XCls
{
private:
    Container<T> c;	/* 第二个模板参数会替换Container */
public:
    ...
}

/* 错误使用 */
XCls<string, list> mylist1;	/* list是一个模板容器,但是有第二模板参数,所以需要声明一下在使用 */

/* 定义类型别名,相当于对容器模板具体类型的明确 */
template <typename T>
using Lst = list<T, allocator<T>>;	/* 相当于typedef List<T, allocator<T>> Lst */

/* 明确类型后的正确使用 */
XCls<string, Lst> mylst2;
```

**不定数量的模板参数：**

```c++
void print()	/* 最后被调用的空的重载版本 */
{
    
}

/* 一个数据+一包数据 */
template <typename T, typename... Types> /* 固定书写格式 */
void print(const T& firstArg, const Types&... args) /* 固定书写格式 */
{
    cout << firstArg << endl;
    printg(args...); /* 固定书写格式 */
    // sizeof...(args);	/* 获得后面一包数据的数量 */
}

/* 使用 */
print(7.5, "hello", bitset<16>(377), 42); /* 依次输出 7.5 hello 0000000101111001 42  */
```

**auto关键字：**

```c++
/* auto让编译器自动推导类型,并定义该类型变量 */
list<string> c;
...
list<string>::iterator ite;	
ite = find(c.begin(), c.end(), target);

list<string> c;
...
auto ite = find(c.begin(), c.end(), target); /* 编译器会自动推导ite类型,并定义 */
```

**ranged-base for新型式的for语句：**

```c++
/* 语法格式 */
for(变量 : 容器)
{
    ...
}

/* 使用例子 */
for(int i : {2,3,4,7,9,13,17,19}){
    cout << i << endl;
}

/* 结合auto关键字,让编译器自动推导变量类型 */
vector<double> vec;
...
for(auto elem : vec){
    cout << elem << endl;
}

for(auto& elem : vec){
    elem *= 3;
}
```

**const：**

```c++
/* c++中多个存储同一字符串的“字符串类”共享一个实际字符串内存,但是读时共享,写时复制 */
class template::basic_string<...>中有如下两个成员函数

/* 类似于此处的const只能写在类的成员函数中,全局函数不能使用如此const修饰 */
/* 返回值不在函数重载的考虑范围内,但const修饰在函数重载的考虑范围内,所以此处不会产生二义性 */
charT operator[](size_type pos) const 
{
	/* 不需要考虑写时拷贝 */
}
reference operator[](size_type pos)
{
    /* 需要考虑写时拷贝 */
}

/* 当const成员函数和非const重载版本“同时存在时“,const对象”只能“调用const版本的成员函数，	 非const对象只能调用非const版本的成员函数 */
/* 当const成员函数和非const重载版本“不同时存在时“,const对象”依然只能“调用const版本的成员函	  数,非const对象既可以调用const版本也可以调用非const版本的成员函数 */
const String str("hello world");	
str.print(); /* print()不是const成员函数,所以此处编译会报错 */
```

**全局重载operator new,operator delete:**

```c++
/* 影响范围较大,不建议全局重载,这些重载函数会由编译器来传入参数并调用 */
inline void* operator new(size_t size){
    cout << "global new()" << endl;
    return malloc(size);
}
inline void* operator new[](size_t size){
    cout << "global new[]()" << endl;
    return malloc(size);
}
inline void operator delete(void* ptr)
{
	cout << "global delete()" << endl;  
    free(ptr);
}
inline void operator delete[](void* ptr)
{
	cout << "global delete[]()" << endl;  
    free(ptr);
}
```

**重载类中的operator new,operator delete:**

```c++
class Foo
{
public:
    void* operator new(size_t);
    void operator delete(void*,size_t); /* size_t为可选参数,可写可不写,由编译器传入 */
    ....
}

Foo* p = new Foo;
/* new操作的编译器内部隐式代码如下: */
try{
    void* mem = operator new(sizeof(Foo));	/* 分配内存 */
    p = static_cast<Foo*>(mem);				/* 强制类型转换 */
    p->Foo::Foo();							/* 调用构造函数进行初始化 */
}

delete p;
/* delete操作的编译器内部隐式代码如下: */
p->~Foo();									/* 先调用析构函数 */
operator delete(p);							/* 释放分配的内存 */
```

**重载类中的operator new[],operator delete[]:**

```c++
class Foo
{
public:
    void* operator new[](size_t);
    void operator delete[](void*,size_t); /* size_t为可选参数,可写可不写,由编译器传入 */
    ....
}

Foo* p = new Foo[N];
/* new操作的编译器内部隐式代码如下: */
try{
    void* mem = operator new(sizeof(Foo)*N+4);	/* 分配内存,多4个字节用于记录数组元素个数 */
    p = static_cast<Foo*>(mem);					/* 强制类型转换 */
    p->Foo::Foo();								/* 调用N次构造函数 */
}

delete [] p;
/* delete操作的编译器内部隐式代码如下: */
p->~Foo();									/* 调N次析构函数 */
operator delete(p);							/* 释放分配的内存 */
```

**重载new,delete的完整示例**

```c++
class Foo
{
public:
    int _id;
    long _data;
    string _str;
public:
    Foo():_id(0){
        cout << "default constructor id=" << id << endl; 
    }
    Foo(int i):_id(i) {
        cout << "constructor id=" << id << endl; 
    }
    virtual ~Foo(){
		cout << "destructor id=" << id << endl;
    }
    static void* operator new(size_t size);
    static void  operator delete(void* pdead, size_t size);
    static void* operator new[](size_t size);
    static void  operator delete[](void* pdead, size_t size);
}

void* Foo::operator new(size_t size){
    Foo* p = (Foo*)malloc(size);
    return p;
}
void Foo::operator delete(void* pdead, size_t size){
    free(pdead);
}
void* Foo::operator new[](size_t size){
    Foo* p = (Foo*)malloc(size);
    return p;
}
void Foo::operator delete[](void* pdead, size_t size){
    free(pdead);
}

/* 如果成员函数没有重载版本就使用全局版本 */
Foo* pf = new Foo;
delete pf;

/* 强制使用全局版本 */
Foo* pf = ::new Foo;	/* void* ::operator new(size_t) */
::delete pf;			/* void ::operator delete(void*) */
```

**重载placement new(),placement delete():**

```c++
/* 第一个参数“必须是size_t类型”,且由编译器自动传入,其对应的placement delete版本并不会被编译器自动调用,只有在对应的placement new版本的“构造函数中发生异常”时,编译器才会调用其对应的placement delete版本重载函数,用于释放所分配发内存 */
class Foo
{
public:
    Foo(){	cout << "Foo::Foo()" << endl; }
    Foo(int){ cout << "Foo::Foo(int)" << endl; throw Bad(); } /* 此处故意抛出异常 */
    /* 一般的operator new()重载 */
    void* operator new(size_t size){
        return malloc(size);
    }
    /* 下面是placement new()的重载 */
    void* operator new(size_t size, void* start){ 
        return start;
    }
    void* operator new(size_t  size, long extra){
        return malloc(size+extra)
    }
    void* operator new(size_t  size, long extra, char init){
        return malloc(size+extra)
    }
    
    /* 这是一般的operator delete()重载 */
    void operator delete(void*, size_t){
        cout << "operator delete(void*, size_t)" << endl;
    }
    /* 对应的placement delete重载版本 */
    void operator delete(void*, void*){
        cout << "operator delete(void* void*)" << endl;
    }
    void operator delete(void*, long){
        cout << "operator delete(void*, long)" << endl;
    }
    void operator delete(void*, long, char){
        cout << "operator delete(void*, long, char)" << endl;
    }
}

/* 使用 */
Foo* p1 = new Foo;	/* 调用void* operator new(size_t size) */
Foo* p2 = new(&start) Foo;	/* 调用operator new(size_t size, void* start) */
Foo* p3 = new(100,a) Foo;	/* 调用operator new(size_t  size, long extra, char init) */ 
```

**placement new在标准库中的使用：**

```c++
/* 用于申请内存时,多申请一部分内存,用于存储字符串 */
template<...>
class basic_string
{
private:
    struct Rep{		/* Rep结构体用于管理和记录字符串状态 */
        ...
        void release(){if(-ref == 0) delete this;}
        inline static void* operator new(size_t, size_t);
        inline static void operator delete(void*);
        inlien static Rep* create(size_t);
        ...
    }
    ...
}

/* 重载的palcement new()函数 */
template <class charT, class traits, class Allocator>
inline void* basic_string<charT, traits, Allocato>::Rep::
operator new(size_t s, size_t extra)
{
    return Allocator::allocate(s+extea*sizeof(charT));
}

/* 重载的operator delete()函数 */
template <class charT, class traits, class Allocator>
inline void* basic_string<charT, traits, Allocato>::Rep::
operator delete(void* ptr)
{
    Allocator::dallocate(ptr, sizeof(Rep)+
                         	  reinterpret_cast<Rep*>(ptr)->res 
                         	  * sizeof(charT));
}

template <class charT, class traits, class Allocator>
inline basic_string<charT, traits, Allocato>::Rep* 
basic_string<charT, traits, Allocato>::Rep::create(size_t extra)
{
    extra = frob_size(extra+1);
    Rep* p = new(extra)Rep;	 /* 除了分配Rep内存还要多分配extra字节内存用于存储字符串 */
    ...
}
```

**STL六大核心组件：**

**容器：**用于存储数据，只需关心数据本身，无需关心数据的内存结构

**分配器：**为容器中的数据提供内存的分配和管理

**算法：**用于处理数据并得到结果

**仿函数：**为算法处理提供一些自定义的特殊处理方式

**迭代器：**容器和算法的连接桥梁,泛化的指针,指向容器中的数据

**适配器：**对数据进行适配，用于迭代器，仿函数，和容器

```c++
/* 这是一个STL列子 */
#include <vector>
#include <algorithm>
#include <functional>
#include <iosteram>
using namespace std;

int main()
{
    int ia[6] = {27,210,12,47,109,83};	
    /* 创建一个vector容器,使用alloctor<int>分配器,将数组的起始和结束地址传入 */
    vector<int,allocator<int>> vi(ia, ia+6);
    /* count_if算法,返回在起始和结束范围内满足特定条件的元素数目 */
    /* less<int>(): 仿函数创建的匿名对象 */
    /* bind2nd:适配器,绑定仿函数的第二参数,始终将40传递给仿函数的第二参数 */
    /* not1:函数适配器,将结果取反,此处条件就是大于等于40 */
    cout << count_if(vi.begin(), vi.end(),
                    not1(bind2nd(less<int>(),40)));
    return 0;
}
```

C++标准库所有容器的迭代器遵循**“前闭后开 [ )”**区间

```c++
*(c.begin()) /* 返回的迭代器指针,指向容器的第一个元素地址 */
*(c.end())	 /* 返回的迭代器指针,指向容器的“最后一个元素的后一个”地址 */
```

**typename关键字:**

​	typename关键字**只能用于模板参数或模板定义之中**

​	typename用于**告诉编译器将该名称当成类型而不是变量**

```c++
template<typename _Iter>	/* 此处的typename用于模板参数 */
 /* typename用于告诉编译器这是一个类型(iterator_traits<_Iter>::iterator_category是一个类型) */
 inline typename iterator_traits<_Iter>::iterator_category 
 __iterator_category(const _Iter&)
 {
     /* 返回这个类型的operator()函数结果 */
     return typename Iterator_traits<_Iter>::iterator_category();
 }
```

**allocator分配器:**

```c++
/* STL中的分配器为容器提供具体的内存分配与管理 */
/* vector容器默认使用的是allocator分配器 */
template<typename _Tp, typename _Alloc = std::allocator<_Tp>>
  class vector: protected _Vector_base<_Tp,_Alloc>
  { ... }
/* list容器默认使用的是allocator分配器 */
template<typename _Tp, typename _Alloc = std::allocator<_Tp>>
  class list: protected _List_base<_Tp,_Alloc>
  { ... }
/* deque容器默认使用的是allocator分配器 */      
template<typename _Tp, typename _Alloc = std::allocator<_Tp>>
  class deque: protected _Deque_base<_Tp,_Alloc>
  { ... }
/* set容器默认使用的是allocator分配器 */      
template<typename Key, typename _Compare = std::less<Key>, 
					   typename _Alloc = std::allocator<_Key>>
  class set { ... }
/* map容器默认使用的是allocator分配器 */    
template<typename _Key, typename _Tp, typename _Compare = std::less<_Key>,
			typename _Alloc = std::allocator<std::pair<const _Key, _Tp>>>
  class map { ... }
/* unordered_set容器默认使用的是allocator分配器 */        
template<class _Value, class _Hash = hash<_Value>
    	 class _Pred = std::equal_to<_Value>
         class _Alloc = std::allocator<_Value>>
  class unordered_set { ... }
/* unordered_map容器默认使用的是allocator分配器 */   
template<class _Key, class _Tp,
		 class _Hash = hash<_Key>,
		 class _Pred = std::equal_to<_Key>
         class _Alloc = std::allocator<std::pair<const _Key, _Tp>>>
  class unordered_map { ... }
```

**GCC2.9 allocator<>实现源码：**

```c++
/* GCC2.9的allocator只是以::operator new和operator delete完成allocate()和deallocate(),没有如何特殊设计 */
template <class T>
  class allocator
  {
  public: 
      typedef T		value_type;
      typedef T*	pointer;
      typedef size_t size_type;    
      typedef ptrdiff_t difference_type
      pointer allocate(size_type n){
          return ::allocate((difference_type)n, (pointer)0); 	/* 分配内存 */
      }
      void deallocate(pointer p){ ::deallocate(p); }	/* 回收内存 */
  }
/* allocate实现 */
template <class T>
inline T* allocate(ptrdiff_t size, T*)
{
    set_new_handler(0);
    T* tmp = (T*)(::operator new((size_t)(size*sizeof(T))));	/* 调用全局new() */
    if(tmp == 0){
        cerr << "out of memory" << nedl;
        exit(1);
    }
    return tmp;
}
/* deallocate实现 */
template<class T>
inline void deallocate(T* buffer)
{
    ::operator delete(buffer);		/* 调用全局delete()函数 */
}
```

**C++标准库中提供了多种分配器,allocator<>只是其中一种**

```c++
/* 直接使用分配器进行内存分配 */
int* p;
allocator<int> alloc1;  					/* 创建分配器 */
p = alloc1.allocate(1);						/* 直接使用分配器分配内存 */
alloc1.deallocate(p,1);						/* 归还分配器分配的内存 */
__gun_cxx::malloc_allocator<int> alloc2;	/* 创建分配器 */
p = alloc2.allocate(1);						/* 直接使用分配器分配内存 */
alloc2.deallocate(p,1);						/* 归还分配器分配的内存 */
__gun_cxx::new_allocator<int> alloc2;		/* 创建分配器 */
p = alloc2.allocate(1);						/* 直接使用分配器分配内存 */
alloc2.deallocate(p,1);						/* 归还分配器分配的内存 */
__gun_cxx::__pool_allocator<int> alloc2;	/* 创建分配器 */
p = alloc2.allocate(1);						/* 直接使用分配器分配内存 */
alloc2.deallocate(p,1);						/* 归还分配器分配的内存 */
__gun_cxx::__mt_allocator<int> alloc2;		/* 创建分配器 */
p = alloc2.allocate(1);						/* 直接使用分配器分配内存 */
alloc2.deallocate(p,1);						/* 归还分配器分配的内存 */
__gun_cxx::bitmap_allocator<int> alloc2;	/* 创建分配器 */
p = alloc2.allocate(1);						/* 直接使用分配器分配内存 */
alloc2.deallocate(p,1);						/* 归还分配器分配的内存 */
```

**实现一个自定义的allocator:**

```c++
/* allocator标准接口,需要满足一下条件：
   需提供一组typedef: allocator::value_type;
   					allocator::pointer;
   					allocator::const_pointer;
   					allocator::reference;
   					allocator::const_reference;
   					allocator::size_type;
   					allcoator::difference_type；
   allocator::rebind:allocator内嵌模板,需要定义other成员
   allocator::allocator():构造函数
   allocator::allocator(const allocator&):拷贝构造函数
   template<typename T> allocator::allocator(const allocator<T>&):泛化的拷贝构造函数
   allocator::~allocator():析构函数
   pointer allocator::address(reference x) const:返回对象地址allocator.address(x)相当于&x
   pointer allocator::allocate(size_type n, const void*=0):分配内存空间
   void allocator::deallocator(pointer p,size_type n):释放分配的空间
   size_type allocator::max_size() const:可分配的最大内存空间
   void allocator::construct(pointer p, const T& x):
   					在指定地址处分配内存,相当于new((const void*)P) T(x);
   void allocator::destroy(pointer p):释放指定地址的内存,相当于p->~T();
*/
template <typename T>
struct MyAllocator{
    typedef T 			value_type;
    typedef T* 			pointer;
    typedef const T*	const_pointer
    typedef T&;			reference;
    typedef const T&	const_reference;
    typedef size_t		size_type;
    typedef int			difference_type;
    template<typename U>	/* 内嵌的rebind模板 */
    struct rebind{
        typedef MyAllocator<U> other;
    }
    /* 构造函数 */
    MyAllocator(){ ... }
    MyAllocator(MyAllocator<T> const&){ ... }
    pointer allocate(size_type n, const void* p = 0){	/* 分配n个T类型的内存 */
        T* buffer = (T*)malloc((size_t)(n*sizeof(T)));
        if(buffer == NULL){ /* 错误处理 */
            ...
        }
        return buffer;
    }
    void deallocate(pointer p,size_type n){
        if(p != NULL){
            free(p);
        }
    }
    /* placement construct函数 */
    void construct(pointer p, const T& value){
        new(p) T(value);	/* 需要重载placement new,在地址处构造T */
    }
    /* placement destroy */
    void destroy(pointer p, size_type n){
        p->~T();	/* 调用T的析构函数 */
    }
    /* 最大可分配内存 */
    size_type max_size() const {
        return size_type(UINT_MAX/sizeof(T)));
    }
    pointer address(reference x){
        return (pointer)&x;
    }
    const_pointer const_address(const_reference x){
        return (const_pointer)&x;
    }
 }
/* 使用 */
int elements[] = {1,2,3,4,5};
const int n = sizeof(elements)/sizeof(int);
vector<int,MyAllocator<int>> myvector(elements,elements+n);
```

**Iterator迭代器：**

```c++
/* list容器的迭代器 */
template<class T, class Ref, class Ptr>
class __list_iterator{
    typedef _list_iterator<T, Ref, Ptr> self;
    /* 迭代器原则:迭代器应为算法提供一下5种类型信息 */
    typedef bidirectioal_iterator_tag iterator_category;	/* 迭代器类型 */
    typedef T 	value_type;									/* 迭代器中的值类型 */
    typedef Ptr pointer;									/* 迭代器指针类型 */
    typedef Ref reference;									/* 迭代器的引用类型 */
    typedef ptrdiff_t difference_type;						/* 存储迭代器元素个数的类型 */
    typedef __list_node<T>* link_type;
    link_type node;											/* list容器中存储数据的节点 */
    
    reference operator*() const { return (*node).data; }
    pointer operator->() const { return &(operator*()); }
    /* self是迭代器,返回的是迭代器实体,返回引用类型,因为c++运行前置++连加,如:(++++i) */
    self& operator++()	{node = (link_type)((*node).next); return *this;} //前置++i
    /* 编译器解释时先遇到=和++,所以此处*this被编译器解释为self(迭代器实体)，所以这里调用的是拷贝构造函		 数和operator++(),返回值类型,因为c++不允许后置++连加,如:(i++++,编译报错)   */
    self operator++(int) { self tmp = *this; ++*this; return tmp };	//后置i++
    ...
    __list_iterator(const iterator& x):node(x.node){ ... }  /* 迭代器的拷贝构造函数 */
};
/* list存储数据的节点 */
template<class T>
struct __list_node
{
	typedef void* void_pointer;
	void_pointer prev;
	void_pointer next;
	T data;				
};
/* list容器 */
template <class T， class Alloc = alloc>
class list
{
protected:
    typedef __list_node<T> list_node;
public:
    typedef list_node* link_type;
    typedef __list_iterator<T,T&,T*> iterator;  /* 容器里面定义一个list迭代器类型 */
protected:
    link_type node;
    ...
};

/* 使用 */
list<Foo>::iterator ite;	/* 根据容器中的迭代器类型创建迭代器 */
*ite 						/* 获取一个Foo对象 */
/* 注意:当你对某一个类型实施operator->(),而这种类型又是非内建类型时,编译器在执行完这次operator->()
   后,在返回值基础上继续重复着上面的operator->()操作,直到遇到内建类型,然后在进行成员值存取 */
ite->method();				/* pointer operator->() const { return &(operator*()); } */
ite->field = 7;
```

**Traits特征提取器：**

```c++
/* 由于迭代器可能是自定义类型也可能是一个指针,指针无法为算法提供类型信息,所以需要加入一个中间层Traits,用于提取转化类型信息,间接的为算法提供所需要的类型信息 */
/* list容器的迭代器 */
template<class T, class Ref, class Ptr>
class __list_iterator{
    typedef _list_iterator<T, Ref, Ptr> self;
    /* 迭代器为算法提供了一下5种类型信息 */
    typedef bidirectioal_iterator_tag iterator_category;	/* 迭代器类型 */
    typedef T 	value_type;									/* 迭代器中的值类型 */
    typedef Ptr pointer;									/* 迭代器指针类型 */
    typedef Ref reference;									/* 迭代器的引用类型 */
    typedef ptrdiff_t difference_type;						/* 存储迭代器元素个数的类型 */
	...
}
/* 算法像一下这种方式从类类型迭代器中获取所需类型信息 */
template<typename I>
inline void algorithm(I first, I last)
{
    ...
    I::iterator_category	/* 获取迭代器中的类型信息 */
    I::value_type			/* 获取迭代器中的值类型 */
    I::pointer				/* 获取迭代器中指针类型 */
    I::reference			/* 获取迭代器中的引用类型 */
    I::difference_type		/* 获取迭代器中存储迭代器元素个数的类型*/
    ...
}

/* iterator_traits中间层,用于分离类类型迭代器和非类类型迭代器 */
template<class I>
struct literator_traits{	/* 获取类类型迭代器信息 */
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type value_type;
    typedef typename I::difference_type difference_type;
    typedef typename I::pointer pointer;
    typedef typename I::refernce reference;
}
template<class T>
struct literator_traits<T*>{	/* 获取非类类型迭代器信息 */
    typedef random_access_iterator_tag iterator_category
    typedef T 			value_type;
    typedef ptrdiff_t 	difference_type;
    typedef T* 			pointer;
    typedef T&			reference
}    
template<class T>
struct literator_traits<const T*>{	/* 获取非类类型迭代器信息 */
    typedef random_access_iterator_tag iterator_category
    typedef T 			value_type;		/* 类型是用来声明变量的,所以此处是T,而不是const T */
    typedef ptrdiff_t 	difference_type;
    typedef T* 			pointer;
    typedef T&			reference;
}  
/* 算法可以通过中间层获取类型信息,而无需关注迭代器本身 */
template<typename I, ...>
void algorithm(...){
    typename iterator_traits<I>::value_type v1;
}
```

**容器vector：**

```c++
/* vector容器其本质是一个数组,迭代器是一个指针,GCC2.9Vector部分实现源码如下 */
template<class T， calss Alloc=alloc>
class vector {
public:
    typedef T			 	value_type;
    typedef value_type* 	iterator;		/* vector的迭代器类型是一个指针 */
    typedef value_type& 	reference;
    typedef size_t			size_type;
protected:
    iterator start;			/* 指向数组的起始地址的指针 */
    iterator finish;		/* 指向数组最后一个存储元素的下一个地址的指针 */
    iterator end_of_storage;/* 指向数组末尾的下一个元素的地址 */
public:
    iterator begin() { return start; }
    iterator end() { return finish; }
    size_type size() const { return size_type(end() - begin()); }
    siee_type capacity() const { return size_type(end_of_storage-begin()); }
    bool empty() const { return begin() == end(); }
    reference operator[](size_type n) { return *(begin()+n); }
    reference front() { return *begin(); }
    reference back() { return *(end()-1); }
    void push_back(const T& x) {
        if(finish != end_of_storage){ /* 还有备用存储空间 */
            construct(finish, x);	  /* 将新数据添加到尾部 */
            ++finish;				  /* 更新finish指针 */
        }else{						  /* 没有备用空间,需要扩充vector空间 */
            inser_aux(end(), x);
        }
    }
    void insert_aux(iterator position, const T& x);   /* 辅助函数,还可能被insert()函数调用 */
    ...
};

template<class T， class Alloc>
void vector<T,Alloc>::insert_aux(iterator position, const T& x)
{
    if(finish != end_of_storage){ /* vector还有备用存储空间 */
        construct(finish, *(finish-1));		/* 为insert()函数准备 */
        ++finish;
        T x_copy = x;
        copy_backward(position, finish-2, finish-1);
        *position = x_copy;
    }else{	/* 备用空间已使用完,需要扩充空间 */
        const size_type old_size = size();
        /* 原大小为0,分配1,否则分配为原来的两倍 */
        const size_type len = old_size != 0?2*old_size : 1; 
        
        iterator new_start = data_allocator::allocate(len);	 /* 使用分配器分配新内存 */
        iterator new_finish = new_start;
        try{
            /* 将原vector中的数据拷贝到新的vector内存中 */
            new_finish = uninitialized_copy(start, position,new_start);
            construct(new_finish, x);	/* 写入要插入或push_back进vector的新的元素值 */
            ++new_finish;
            /* 插入还需要拷贝后半段数据进新vector */
            new_finish = uninitialized_copy(position,finish,new_start);
        }catch(...){
            destory(new_start, new_finish);
            data_allocator::deallocate(new_start, len);
            throw;
        }
        destory(begin(), end());/* 析构原来的vector内存 */
        deallocate();
        /* 调整迭代器指针,指向新的内存 */
        start = new_start;
        finish = new_finish;
        end_of_storage = new_start+len;
    }
}

```

**typedef语法细节：**

```c++
int a[100];			//ok
int[100] b;			//fail
typedef int T[100];
T c;				//T类型为int[100]
```

**容器array:**

```c++
template<typename _Tp, std::size_t _Nm>  /* GCC2.9部分源码实现 */
struct array
{
    typedef _Tp 			value_type;
    typedef _Tp*			pointer;
    typedef value_type* 	iterator;	/* 迭代器是指针 */
    
    /* 内部维护一个数组 */
    value_type _M_instance[_Nm ? _Nm : 1];
    
    iterator begin() { return iterator(&_M_instance[0]); }
    iterator end() { return iterator(&_M_instance[_Nm]); }
    ...
}
/* 使用 */
array<int, 10> myarray;
auto ite = myarray.begin();
ite +=3;
cout << *ite;
```

**容器deque:**

```c++
/* GCC2.9源码部分实现 */
/* BufSize是指每一个buffer所能存储的元素个数,其分配规则是,
   如果BufSize有指定大小,则使用指定大小
   如果单个元素大小超过512字节,则BufSize=1,即一个buffer只存储一个元素
   如果单个元素大小小于512字节,则BufSize=512/单个元素大小 */
inline size_t __deque_buf_size(size_t n, size_t sz)
{
    return n != 0 ? n : (sz<512?size_t(512/sz):size_t(1));
}
/* deque内部存储结构:存在一个控制中心map,其实是一个vector数组容器,数据存储在vector中间,方便从两头扩展新的数据,vector中存储的数据是一个个指针,依次指向各个buffer存储块首地址,deque维护着两个迭代器，start和finish这两个迭代器里的node指针分别指向map控制中心vector中的第一个元素的地址和最后一个元素的下一个元素地址,两个迭代器里的first,last和cur指针,分别第一个和最后一个buffer存储块的首地址和最后一个元素的下一个地址，cur则指向第一个和最后一个元素的地址 */
template<class T, class Alloc=alloc, size_t BufSize=0 >
class deque{
public:
    typedef T value_type;
    typedef __deque_iterator<T, T&, T*, BufSize> iterator;
protected:
    typedef pointer* map_pointer;		/* pointer*是T** 类型 */
protected:
    iterator 	start;		/* 一个deque维护着两个迭代器 */
    iterator 	finish;	
    map_pointer map;
    size_type 	map_size;
public:
    iterator begin() { return start; }
    iterator end() { return finish; }
    size_type size() const { return finish-start; }
    reference operator[](size_type n) { return start[difference_type(n)]; }
    reference front() { return *start; }
    reference back() { iterator tmp = finish; --tmp; return *tmp; }
    size_type size() const { return finish - start; }
    bool empty() const { return finish == start; }
    iterator insert(iterator position, const value_type& x){ /* 在position处插入一个元素x */
        if(position.cur == start.cur){	/* 插入点就在deque的最前端 */
            push_front(x);
            return start;
        }else if(position.cur == finish.cur){ /* 插入点在deque的最尾端 */
            push_back(x);
            /* finish中的node指针指向控制中心vector中最后一个元素的下一个元素地址 */
            iterator tmp = finish; 
            --tmp;	/* 指向最后一个元素 */
            return tmp;  /* 返回最后一个元素的迭代器 */
        }else{
            return insert_aux(position,x)	/* 调用辅助函数完成插入动作 */
        }
    }
    ...
};

/* 插入辅助函数 */
template <class T, class Alloc, size_t BufSize>
typename deque<T, Alloc, BufSize>::iterator 
deque<T, Alloc, BufSize>::insert_aux(iterator pos, const value_type& x){
    difference_type index = pos -start;	/* 判断安插点是处在中间偏左还是偏右 */
    value_type x_copy = x;	
    if(index < size()/2){	/* 安插点之前的元素个数较少 */
        push_front(front());
        ...
        copy(front2,pos1,front1); /* 迁移元素 */
    }else{
        push_back(back());
        ...
        copy_backward(pos,back2,back1);
    }
    *pos = x_copy;	/* 插入元素 */
    return pos;
}

/* deque容器的迭代器 */
template<class T， class Ref, class Ptr, size_t BufSize>
struct __deque_iterator{
    typedef random_access_iterator_tag iterator_category; /* 获取迭代器中的类型信息 */
    typedef T	value_type;								  /* 获取迭代器中的值类型 */
    typedef Ptr pointer;								  /* 获取迭代器中的指针类型 */
    typedef Ref reference;								  /* 获取迭代器中的引用类型 */
    typedef ptrdiff_t difference_type;			/* 获取迭代器中存储迭代器元素个数的类型 */
    typedef size_t size_type;	
    typedef T** map_pointer;
    typedef __deque_iterator self;	
    
    T* cur;	 	/* 指向当前要操作的元素 */
    T* first;	/* 指向当前要操作元素所在buffer的起始地址 */
    T* last；   /* 指向当前要操作元素所在buffer的最后一个元素的下一个地址 */
    map_pointer node;	/* 指向当前要操作的buffer的首地址 */
    ...
    reference operator*() const { return *cur; }
    pointer operator->() const { return &(operator*()); }
    /* 两个迭代器之间的距离,是中间相隔完整的buffer的总长度+两头buffer的元素个数 */
    difference_type operator-(const self& x) const {
        return difference_type(buffer_size())*(node-x.node-1)+(cur-first)+(x.last-x.cur);
    }
    void set_node(map_pointer new_node){
        node = new_node; /* 将vector中的下一个元素地址作为新节点 */
        first = *new_node; /* 将first和last指针指向新的buffer区 */
        last = first + difference_type(buffer_size());
    }
    self& operator++(){ 	/* 前置++ */
        ++cur;
        if(cur == last){	 	/* 已经抵达改buffer区的尾端 */
            set_node(node+1);   /* 切换到下一个buffer */
            cur = first;
        }
        return *this;
    }
    self operator++(int){  /* 后置++ */
        self tmp = *this;
        ++*this;
        return tmp;
    }
    self& operator--() {	/* 前置-- */
        if(cur == first){
            set_node(node-1); /* 切换到前一个buffer */
            cur = last;
        }
        --cur;	/* 取得该buffer末尾的最后一个元素 */
        return *this;
    }
    self operator--(int){	/* 后置-- */
        self tmp = *this;
        --*this;
        return tmp;
    }
    self& operator+=(difference_type n){
        difference_type offset = n + ( cur - first);
        if(offset >= 0 && offset < difference_type(buffer_size())){
             cur += n;	/* 在同一个buffer中移动 */
        }else{	/* 在不同的buffer中移动 */
            difference_type node_offset = 
            	 offset > 0 ? offset/difference_type(buffer_size()):
            				 -difference_type((-offset-1)/buffer_size()) -1;
            set_node(node + node_offset);	/* 切换至正确的buffer区 */
            /* 切换至正确的元素 */
            cur = first+(offset-node_offset*difference_type(buffer_size())); 
        }
        return *this;
    }
    self operator+(difference_type n) const {
        self tmp = *this;
        return tmp += n;
    }
    self& operator-=(difference_type n) { return *this += -n; }
    self operator-(difference_type n) const { self tmp = *this; return tmp -= n; }
    reference operator[](difference_type n) const { return *(*this + n); }
};
```

**queue容器:**

```c++
/* 只是对deque容器的一种封装,内部默认维护着一个deque容器 */
template<class T, classn Sequence=deque<T>>  /* queue没有迭代器 */
class queue{
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::size_type size_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
protected:
    Sequence c;	/* 内部默认维护这一个deque容器 */
public:
    bool empty() const { return c.empty(); }
    size_type size() const {return c.size();}
    ...
}
```

**stack容器：**

```c++
/* 只是对deque容器的一种封装,内部默认维护着一个deque容器 */
template<class T, classn Sequence=deque<T>>  /* stack没有迭代器 */
class stack{
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::size_type size_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
protected:
    Sequence c;	/* 内部默认维护这一个deque容器 */
public:
    bool empty() const { return c.empty(); }
    size_type size() const {return c.size();}
    ...
}
```

**rb_tree容器：**

```c++
/* 红黑树中存储的Value值是key加data的组合数据包,Value值可以被修改,但这样会破坏红黑树的存储结构
   红黑树提供两种插入元素的方法:insert_unique(),存在的元素不插入,和insert_equal(),存在的元素,可重复    插入 */
template <class key, 		/* 存储的Key值 */
    	  class Value,  	/* key加data的组合 */
          class KeyOfValue,	/* 如何通过一个value得到一个key */
          class Compare,	/* 如何对key进行比较 */
          class Alloc=alloc>
class rb_tree{
protected:
    typedef __rb_tree_node<Value> rb_tree_node;
    ...
public:
    typedef rb_tree_node* link_type;
    ...
protected:
    size_type node_count;	/* 记录着rb_tree的大小 */
    link_type header;		/* 纪律着红黑树的头 */
    Compare key_compare;	/* 比较key的仿函数 */
    ...
}

template<class Arg, class Result>
struct unary_function{
    typedef Arg 	argument_type
    typedef Result 	result_type;
}
template<class T>
struct identity:public unary_function<T,T>{
    const T& operator()(const T&  x) const { return x; }	
}

template<class Arg1, class Arg2, class Result>
struct binary_function{
   typedef Arg1 first_argument_type;
   typedef Arg2 second_argument_type;
   typedef Result result_type;
}
template<class T>
struct less:public binary_function<T,T,bool>{
    bool operator()(const T& x, const T& y) const{ return x<y; }
}
/* 红黑树的使用 */
rb_tree<int, int, identity<int>, less<int>> itree;
cout << itree.empty() << endl;		// 1
cout << itree.size() << endl;		// 0
itree.insert_unique(3);	/* 插入的值是Value,Value是key和data的组合 */
itree.insert_equal(5);
```

**set,multiset容器:**

```c++
/* set和multiset底层存储结构使用的红黑树,通过使用const_iterator来禁止对元素的修改
   set使用的是insert_unique(),来实现元素的独一无二
   multiset使用的是insert_equal(),来实现元素的重复 */
template<class Key,	/* key+data组合成Value */
		 class Compare = lsee<key>,
	     class Alloc = alloc>
class set{
public:
    typedef Key key_type;
    typedef Key value_type;
    typedef Compare key_compare;
    typedef compare value_compare;
private:
    typedef rb_tree<key_type,value_type,identity<value_type>,key_compare,Alloc> rep_type;
    rep_type t;	/* set容器内部维护着一个红黑树 */
public:
    typedef typename rep_type::const_iterator iterator;	/* set使用的是const迭代器 */
    ...
    /* set的所有操作都是通过红黑树来完成的,所有set更像是一个adapter */
};
```

**map，multimap容器:**

```c++
/* map和multimap也是以红黑树为底部存储结构,同过将key设为const属性来禁止修改key,但可以修改data
   map使用的是insert_unique(),来实现元素的独一无二,operator[]是map独有的操作
   multimap使用的是insert_equal(),来实现元素的重复 */
template<class Key,	/* key */
		 class T,	/* data */
		 class Compare = less<key>, /* 如何比较key值 */
         class Alloc=alloc>
class map{
public:
    typedef Key key_type;
    typedef T 	data_type;
    typedef T   mapped_type;
    typedef pair<const Key, T> value_type; /* 通过设置Key为const属性,来禁止修改Key */
    typedef Compare key_compare;
private:
    typedef 
        rb_tree<key_type,value_type,select1st<value_type>,key_compare,Alloc> rep_type;
    rep_type t;		/* 内部维护一个红黑树 */
public:
    typedef typename rep_type::iterator iterator;	/* 使用红黑树的普通迭代器 */
    ...
}
/* 使用 */
map<int, string,less<int>, alloc> imap;
```

**hashtable容器：**

```c++
/* 底层实现原理:假设我们现在有一个M大小的数组容器,还要一个hashcode()生成算法,任何数据经过hashcode()算法,都会生成一个唯一的哈希码H,对H%M即可得到该数据要存储的数组下标,当H足够多变时,则H%M可能得相同的下标,即发生数据碰撞，所以这个数组中应该存储的是一个个链表(一堆篮子),发生碰撞时将数据挂入相同链表中即可(放入同一个篮子),但是依然存储单一链表过长而降低效率的可能,所以规定,当哈希表中存储的元素"总数"大于数组元素个数(篮子总数),则需要扩充数组大小(增加篮子总数),(GCC2.9将数组扩充到原来大小的2倍附近的质数),所以此处的数组应该是一个vector容器,扩充后应该对哈希表内部所有元素进行重新计算和存放 */
template<class Value>
struct __hashtable_node{
    __hashtable_node* next; /* 这里是一个单向链表 */
    Value val;				/* 存储着数据 */
}
template<class Value,class Key,class HashFcn,
		 class ExtractKey,class EqualKey,class Alloc=alloc>
struct __hashtable_iterator{
    ...
    node* cur;		/* 指向当前操作的链表中的结点 */
    hashtable *ht;	/* 指向整个哈希表的vector容器 */
}
template<class Value, 		/* 要存入哈希表的数据 */
		 class Key, 		/* 哈希表是根据key值来进行元素排列的 */
		 class HashFcn，	   /* hashcode生成算法 */
         class ExtractKey,  /* 如何通过Value值得到key值 */
		 class EqualKey,	/* 如何判断连个Key值相等 */
         class Alloc=alloc>
class hashtbale{
public:
    typedef HashFcn hasher;
    typedef EqualKey key_equal;
    typedef size_t size_type;
private:
    hasher hash;
    key_equal equals;
    ExtractKey get_key;
    
    typedef __hashtable_node<Value> node;	/* 链表节点 */
    vector<node*, Alloc>buckets;	/* 内部维护着一个vector容器,容器中存储的是一个个链表 */
    size_type num_elements;			/* 记录着哈希表中的元素总数 */
public:
    size_type bucket_count() const { return buckets.size(); }
    ...
}
/* 使用 */
hashtable<const char*,
    	  const char*,
     	  hash<const char*>,
    	  identity<const char*>,
		  eqstr,alloc> ht(50,hash<const char*>(),eqstr())); /* 构造函数 */
ht.insert_unique("hello");

struct eqstr{
    bool operator()(const char* s1, const char* s2) const { return strcmp(s1,s2)==0; }
}
inline size_t __st1_hash_string(cosnt char* s){ 	/* hashcode生成算法 */
    unsigned long h = 0;
    for(;*s;++s) { h=5*h+*s;}
    return size_t(h);
}
template<class key>			/* 模板的部分特化 */
struct hash<const char*>{
    size_t operator()(const char* s) const { return __st1_hash_string(s); }
}
```

**C++标准库的算法:函数模板**

```c++
template<typename iterator>
Algorithm(iterator itr1, iterator itr2)	 /* c++标准库算法的形式 */
{
    ...
}
```

**iterator的种类：五种iterator category**

```c++
/* 除了输出迭代器,其他迭代器都是相互继承的 */
struct Input_iterator_tag{ ... };
struct output_itertaor_tag{ ... };
/* 单向迭代器,如:单向链表 */
struct forward_iterator_tag:public input_iterator_tag { ... }; 
/* 双向迭代器,如：双向链表 */
struct bidirectional_iterator_tag:public forward_iterator_tag{ ... };
/* 随机访问迭代器 */
struct random_access_iterator_tag:public bidirectional_iterator_tag{ ... }	
/* 使用typeid可获取迭代器种类 */
#include <typeinfo>	//typeid

iterator itr;
cout << typeid(itr).name(); << endl;  /* 取得迭代器的种类 */
```

**迭代器种类对算法效率的影响：**

```c++
/* 非随机访问迭代器的重载版本 */
template<class InputIterator>
inline iterator_traits<InputIterator>::difference_type
__distance(InputIteratorertor first, InputIterator last, Input_iterator_tag){
    ierator_traits<InputIterator>::difference_type n = 0;
    while(first != last){	/* 依次遍历每一个节点 */
        ++first;
        ++n;
    }
    return n;
}
/* 随机访问迭代器版本 */
template<class RandomAccessIterator>
inline iterator_traits<RandomAccessIterator>::difference_type
__distance(RandomAccessIterator first,RandomAccessIterator 					                        last,random_access_iterator_tag){
    return last-first;
}
/* 计算两个迭代器之间的距离 */
template<class InputIterator> /* 除了输出迭代器,其他迭代器都继承自InputIterator */
inline iterator_traits<InputIterator>::difference_type
distance(InputIterator first, InputIterator last){
    typedef typename iterator_traits<InputIterator>::iterator_category category;
    return __distance(first,last,category())  /* 迭代器种类临时对象 */
}
```

```c++
template<class InputIterator, class Distance> 
inline void __advance(InputIterator& i, Distance n, input_iterator_tag){ 
    while(n--){ ++i; } /* 迭代器只能往前走 */
}    
template<class BidirectionalIterator, class Distance>
inline void __advance(BidirectionalIterator& i, Distance n, bidirectional_iterator_tag){
    if(n > 0){	/* 双向迭代器,即可以往前走也可以往后走 */
        while(n--) { ++i; }
    }else{
        while(n++) { --i; }
    }
}
template<class RandomAccessIterator, class Distance>
inline void __advance(RandomAccessIterator& i, Distance n, random_access_iterator_tag){
    i += n;	/* 随机访问迭代器,随意操作 */
}

template <class itertaor>
inline typename iterator_traits<iterator>::iterator_category
iterator_category(const iterator&){ /* 编译器自动推导参数类型 */
    typedef typename iterator_traits<Iterator>::itertaor_category category;
    return category(); /* 返回临时对象 */
}
/* 将迭代器往前偏移n */
template<class InputItertaor, class Distance>
inline void advance(InputItertaor& i, Distance n){
    __advance(i,n,iterator_category(i));
}
```

**常见算法实现:**

```c++
/* 算法accumulate */
template<class InputIterator, class T>
T accumulate(Inputiterator first, Inputiterator last, T init){
    for(;first != last; ++ first){
        init = init + *first;
    }
    return init;
}
/* 算法accumulate,带仿函数版本 */
template<class InputIterator,class T, class BinaryOperation>
T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation binary_op){
    for(;first != last; ++first){
        init = binary_op(init, *first);
    }
    return init;
}
```

```c++
/* 算法for_each */
template<class InputIterator, class Function>
Function for_each(InputIterator first,InputIterator last, Function F){
    for(;first != last; ++first){
        f(*first);
    }
    return f;
}
```

```c++
/* 算法replace,replace_if,replace_copy */
template<class ForwardIterator, class T>
void replace(ForwardIterator first,ForwardIterator last,
             const T& old_value,const T& new_value){
    for(;first!=last; ++first){ /* 将所有的old_value替换为new_value */
        if(*first == old_value){
            *first = new_value;
        }
    }
}
template<class InputIterator,class OutputIterator,class T>
OutputIterator replace_copy(InputIterator first,InputIterator last,OutputIterator result, 							  const T& old_value,const T& new_value){
    for(;first!=last;++first;++result){ 
        *result = *first==old_value?new_value:*first;
    }
    return result;
}
template<class ForwardIterator, class Predicate, class T>
void replace_if(ForwardIterator first,ForwardIterator last,
                Predicate pred,const T& new_value){
    for(;first!=last;++first){
        if(pred(*first)){
            *first = new_value;
        }
    }
}
```

```c++
/* 算法count,count_if */
template<class InputIterator, class T>
typename iterator_traits<InputIterator>::difference_type
count(InputIterator first, InputIterator last, const T& value){
    for(;first!=last; ++first){
        if(*first == value){
			++n;
        }
    }
    return n;
}
template<class InputIterator, class Predicate>
typename iterator_traits<InputIterator>::difference_type
count_if(InputIterator first, InputIterator last, Predicate pred){
    for(;first!=last；++first){
        if(pred(*first)){
            ++n;
        }
    }
    return n;
}
```

```c++
/* 算法find, find_if */
template<class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value){
    while(first!=last && *first != value){
        ++first;
    }
    return first;  
}
template<class InputIterator, class Predicate>
InputIterator find_if(InputIterator first, InputIterator last, Predicate pred){
    while(first!=last && !(pred(*first))){
        ++first;
    }
    return first;
}
```

```c++
/* 算法binary_search,调用该算法的前提是容器内的元素排序是一个递减序列 */
template<class ForwardIterator, class T>
bool binary_search(ForwardIterator first,ForwardIterator last，
                  const T& val){
    first = std::lower_bound(first,last,val);	/* 查找到第一个小于等于val的值 */
    return (first!=last && !(val < *first)); 
}
/* 在一个递减序列中查找第一个小于等于val的元素 */
template<class ForwardIterator, class T>
ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last,
                            const T& val){
    ForwardIterator it;
    iterator_traits<ForwardIterator>::difference_type count, step;
    count = distance(first,last);
    while(count > 0){
        it = first;
        step = count/2;
        advance(it,step);
        if(*it < val){	
            first = ++it;
            count -= step+1;
        }else{
            count = step;
        }
    }
    return first;
}
```

**仿函数functors：**

```c++
/* 仿函数 */
template<class T>
struct identity:public unary_function<T,T>{
    const T& operator()(const T& x) const { return x; }
};
template<class T1， class T2>
struct pair{
    typedef T1 first_type;
    typedef T2 second_type;
    
    T1 first;
    T2 second;
    pair():first(T1()),second(T2()){};
    pair(const T1& a, const T2& b):first(a),second(b){}；
}
template<class Pair>
struct select1st:public unary_function<Pair,typename Pair::first_type>{
    const typename Pair::first_type& operator()(const Pair& x) const{
        return x.first;
    }
}
template<class Pair>
struct select2nd:public unary_function<Pair,typename Pair::second_type>{
    const typename Pair::second_type& operator()(const Pair& x) const{
        return x.second;
    }
}
```

**仿函数的可适配（adaptable）:**

```c++
/* 仿函数如果想被STL的适配器适配,必须继承以下两个类之一,因为STL的适配器需要通过这些继承的类来获取仿函数的参数类型信息,方便定义变量和使用 */
template<class Arg, class Result> /* 一个传入参数 */
struct unary_function{
    typedef Arg 	argument_type;
   	typedef Result  result_type;
}
template<class Arg1, class Arg2, class Result> /* 两个传入参数 */
struct binary_function{
    typedef Arg1 	first_argument_type;
    typedef Arg2 	second_argument_type;
    typedef Result 	result_type;
}
/* 仿函数通过继承的方式,将参数类型信息传递给继承的类,来给STL的适配器使用 */
template<class T>
struct less:public binary_function<T,T,bool>{
    bool operator()(const T& x， const T& y) const {
        return x < y;
    }
};
```

**容器的适配器：**

```c++
template<class T,class Sequence=deque<T>>
class stack{
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::size_type size_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
protected:
    Sequence c; /* 内部维护着一个其他容器 */
public:
    /* 内部仅仅是对其他容器功能的一个封装 */
    bool empty() const { return c.empty(); }
    size_type size() const { return c.size(); }
    reference top() { return c.back(); }
    const_reference top() const { return c.back(); }
    void push(const value_type& x) { c.push_back(x); }
    void pop() { c.pop_back(); }
}
```

**函数的适配器bind2nd:**

```c++
template<class Operation, class T>
inline binder2nd<Operation>bind2nd(const Operation& op, const T& x){
    typedef typename Operation::second_argument_type arg2_type;
    return binder2nd<Operation>(op,arg2_type(x)); /* 返回一个匿名对象 */
} 
template<class Operation>
class binder2nd:public unary_function<typename Operation::first_argument_type,
									  typename Operation::result_type>{
protected:
    Operation op;	/* 这里相当于创建了一个对象,这里创建对象会调用构造函数 */		
    typename Operation::second_argument_type value;    /* 将传入的两个参数记录起来 */	   
public:
    /* 构造函数 */
    binder2nd(const Operation& x, const typename Operation::second_argument_type& y)
        :op(x),value(y){
            
        };
    typename Operation::result_type
    Operation()(const typename Operation::first_argument_type& x) const{
        return op(x,value);
    };
};
/* count_if算法 */
template<class InputIterator, class Predicate>
typename Iterator_traits<InputIterator>::difference_type 
count_if(InputIterator first, InputIterator last, Predicate pred){
    typename iterator_traits<InputIterator>::difference_type n = 0;
    for(;first !=last){	/* 遍历迭代器 */
        if(pred(*first)){
            ++n;
        }
    }
    return n;
}
/* 使用 */
cout<< count_if(vi.begin(), vi.end(),bind2nd(less<int>(),40));
```

**函数适配器not1:**

```c++
template<class Predicate>
inline unart_negate<Predicate> not1(const Predicate& pred){ /* 模板函数 */
    return unary_negate<Predicate>(pred); /* 返回一个匿名对象 */
}

template<class Prediacte>
class unary_negate:public unary_function<typename Predicate::argument_type,bool>{
protected:
    Predicate pred;	/* 会调用默认的空构造函数 */
public:
    explicit unary_negate(const Predicate& x):pred(x){ /* 拷贝构造函数 */
        
    }
    bool operator()(const typename Predicate::argument_type& x) const {
        return !pred(x); /* 适配器仅仅是起到装饰作用,指示对其他类功能的一种修饰 */
    }
}
/* 使用 */
cout<< count_if(vi.begin(), vi.end(),not1(bind2nd(less<int>(),40));
```

**迭代器适配器:**

```c++
template<class Iterator>
class reverse_iterator{ /* 仅仅只是对正向迭代器功能的一种装饰 */
protected:
    Iterator current;	/* 维护着一个正向迭代器 */
public:
    typedef typename iterator_traits<Iterator>::iterator_category iterator_category;
    typedef typename iterator_traits<Iterator>::value_type value_type;
    typedef Iterator iterator_type;				/* 代表正向迭代器 */
    typedef reverse_iterator<Iterator> self;	/* 代表逆向迭代器 */
public:
    explicit reverse_iterator(iterator_type x):current(x){ }
    reverse_iterator(const self& x):current(x.current){ }
    iterator_type base() const { return current; } 
    /* 对正向迭代器的反向操作 */
    reference operator*() const { Iterator tmp = current; return *--tmp }
    pointer operator->() const { return &(operator*()); }
    self& operator++(){ --current; return *this; }
    self& operator--(){ ++current; return *this; }
    self operator+(difference_type n) const { return self(current -n); }
    self operator-(difference_type n) const { return self(current +n); }
}
/* 使用 */
reverse_iterator rbegin(){ return reverse_iterator(end()); } /* 传入正向迭代器 */
reverse_iterator rbend(){ return reverse_iterator(begin()); } /* 传入正向迭代器 */
```

```c++
template<class InputIterator, class OutputIterator>
OutputIterator copy(InputIterator first,InputIterator last,OutputIterator result){
    while(first!=last){
        *result = *first;	/* result这个类可以重载operator=(),改变=操作行为 */
        ++result; ++first;
    }
    return result;
}
template<class Container>
class insert_iterator{	/* 这个迭代器适配器改变迭代器的赋值行为 */
protected:
    Container* container;
    typename Container::iterator ite;
public:
    typedef output_iterator_tag iterator_category;
    /* 构造函数 */
    insert_iterator(Container& x,typename Container::iterator):container(&x),iter(i){ }
    ...
    insert_iterator<Container>& operator=(const typename Container::value_type& val){
        iter = container->insert(iter,value); /* 将容器的赋值操作改为插入操作 */
        ++iter;
        return *this;
    } 
};
/* 使用 */
list<int>foo,bar;
for(int i = 1; i <= 5; i++){
    foo.push_back(i);
    bar.push_back(i*10);  
}
list<int>::iterator it = foo.begin();
copy(bar.begin(),bar.end(),insert(foo,it)); /* 改变了copy的行为 */
```

```c++
template<class InputIterator, class OutputIterator>
OutputIterator copy(InputIterator first,InputIterator last,OutputIterator result){
    while(first!=last){
        *result = *first;	/* result这个类可以重载operator=(),改变=操作行为 */
        ++result; ++first;
    }
    return result;
}
template<class T, class charT = char, class traits=char_traits<charT>>
class ostream_iterator:public iterator<output_itertaor_tag,void,void,void,void>{
    basic_osteram<charT,traits>* out_stream; /* 使用的输出流 */
    const charT* delim;			/* 每次输出的分隔符 */
public:
    typedef charT char_type;
    typedef traits traits_type;
    typedef basic_ostream<charT,traits>ostream_type;
    /* 构造函数 */
    ostream_iterator(ostream_type& s):out_stream(&s),delim(0){ }
    ostream_iterator(ostream_type& s, const charT* delimiter)
        :out_stream(&s),delim(delimiter){ }
    /* 拷贝构造函数 */
    ostream_iterator(const ostream_iterator<T,charT,traits>& x)
        :out_stream(x.out_stream),delim(x.delim){ }
    ~ostream_iterator(){ }
    /* 将赋值行为适配到输出到out_stream */
    ostream_iterator<T,charT,traits>& operator=(const T& value){ 
        *out_stream << value;
        if(delim != 0){
            *out_stream << delim;
        }
        return *this;
    }
    ostream_iterator<T,charT,traits>& operator*() { return *this; }
    ostream_iterator<T,charT,traits>& operator++() { return *this; } /* 前置++ */
    ostream_iterator<T,charT,traits>& operator++(int) { return *this; } /* 后置++ */
}
/* 使用 */
int main(){
    std::vector<int> myvector;
    for(int i=1;i<10;i++){
        myvector.push_back(i*10);
    }
    std::ostream_iterator<int> out_it(std::cout,",");
    std::copy(myvector.begin(),myvector.end(),out_it); 
    
    return 0;
}
```

```c++
template<class InputIterator, class OutputIterator>
OutputIterator copy(InputIterator first,InputIterator last,OutputIterator result){
    while(first!=last){
        *result = *first;	/* *first操作重载版本operator*() */
        ++result; 			
        ++first;			/* 完成一次读操作 */
    }
    return result;
}
template<class T,class charT=char,
		 class traits=char_traits<charT>,class Distance=ptrdiff_t>
class istream_iterator:public iterator<input_iterator_tag,T,Distance,const T*,const T&>
{
    basic_istream<charT,traits>* in_stream;
    T value;
public:
    typedef charT  char_type;
    typedef traits traits_type;
    typedef basic_istream<charT,traits> istream_type;
    istream_iterator():in_stream(0){}
    istream_iterator(istream_type& s):in_stream(&s){++*this;} /* ++操作已经产生一次读操作 */
    istream_iterator(const isteram_iterator<T,charT,traits,Distance>& x)
        :in_stream(x.in_stream),value(x.value){ }
    ~istream_iterator(){}
    const T& operator*() const { return value }
    const T* operator->() const { return &value }
    istream_iterator<T,charT,traits,Distance>operator++(){ /* 前置++ */
        if(in_stream && !(*in_stream >> value)){ /* 已经完成一次读操作 */
            ins_stream=0；
        }
        return *this;
    }
    istream_iterator<T,charT,traits,Distance> operator++(int){ /* 后置++ */
        istream_iterator<T,charT,traits,Distance> tmp = *this;
        ++*this;
        return tmp;
    }
}
/* 使用 */
istream_iterator<int>iit(cin),eos;
copy(ite,eos,inserter(c,c.begin()));
```

**不定模板参数的使用:**

```c++
/* 利用不定模板参数实现万用的HashCode函数 */
class CustomerHash{
public:
    std::size_t operator()(const Customer& c) const{
        return hash_val(c.fname,c.lname,c.no);
    }
private:
    string fname;
    string lname;
    long no;
}
/* 不定参数模板函数也是可以重载的 */
template<typename... Type>
inline size_t hash_val(const Type&... arg){
    size_t seed = 0;
    hash_val(seed,arg...);
    return seed; /* seed就是最终生成的hashcode */
}
template<typename T，typename... Type>
inline void hash_val(size_t& seed, const T& val, const Type&... arg){
    hash_combine(seed,val); /* 逐一取出模板参数并计算hashcode */
    hash_val(seed,arg...);
}
template<typename T>
inline void hash_val(size_t& seed,const T& val){
    hash_combine(seed,val);
}
template<typename T>
inline void hash_combine(size_t& seed, const T& val){
    seed ^=std::hash<T>()(val)+0x9e3779b9+(seed<<6)+(seed>>2);
}
```

**tuple任意个数类型组合：**

```c++
/* 不定模板参数+自我递归完成可容纳任意类型任意数量组合的类tuple */
template<> class tuple<>{ }; 				/* 空模板参数的特化版本,用于递归结束条件 */
template<typename Head,typename... Tail>
class tuple<Head,Tail...>:private tuple<Tail...>  /* 通过继承完成不定模板参数的分离 */
{
   	typedef tuple<Tail..>inherited;
public:
    tuple(){ }
    tuple(Head v,Tail... vtail):m_head(v),inherited(vtail...){ } /* 不断的自我递归 */
    
    typename Haed::type head(){ return m_head; }
    inherited& tail(){ return *this; }
protected:
    Head m_head; /* 被分离的参数 */
}
/* 使用 */
tuple<int,float,string> t(41,63,"nico");
t.head() /* 取得41 */
t.tail() /* 取得41,63,nico这个数据结构的首 */
t.tail().head() /* 取得6.3 */  
```

**moveable class:**

```c++
class MyString{
public:
    static size_t DCtor;	/* 默认构造函数调用次数 */
    static size_t Ctor;		/* 构造函数调用次数 */
    static size_t CCtor;	/* 拷贝构造函数调用次数 */
    static size_t CAsgn;	/* 拷贝赋值函数调用次数 */
    static size_t MCtor;	/* move构造函数 */
    static size_t MAsgn;	/* move赋值函数 */
    static size_t Dtor;		/* 析构函数 */
private:
    char* _data;
    size_t _len;
    void _init_data(const char* s){
        _data = new char[_len+1];
        memcpy(_data,s,_len);
        _data[_len] = '\0';
public:
     /* 默认构造函数 */
     MyString():_data(NULL),_len(0){
         ++DCtor;
     }
     /* 普通构造函数 */
     MyString(const char* p):_len(strlen(p)){
         ++Ctor;
         _init_data(p);
     }
     /* 拷贝构造函数 */
     MyString(const MyString& str):_len(str._len){
         ++CCtor;
         _init_data(str._data);
     }
     /* 拷贝赋值函数 */
     MyString& operator=(const MyString& str){
         ++CAsgn;
         if(this != &str){
             if(_data) delete _data;
             _len = str._len;
             _init_data(str._data);	/* 深拷贝 */
         }else{
             
         }
         return *this;
     }
     /* move构造函数 */
     MyString(MyString&& str) noexcept:_data(str._data),_len(str._len){
         ++MCtor;	/* 需要确保传入的对象不在会被使用 */
         str._len = 0;
         str._data = NULL;
     }
     /* move赋值函数 */
     MyString& operator=(MyString&& str) noexcept {
         ++MAsgn;
         if(this != &str){ /* 需要确保传入的对象不在会被使用 */
             if(_data) delete _data;
             _len = str._len;
             _data = str._data;
             str.len = 0;
             str._data = null;
		 }
         return *this;
     }
     /* 虚析构函数 */
     virtual ~MyString(){
         if(_data){
             delete _data;
         }
     }
     bool operator<(const MyString& rhs) const{
         return std::string(this->_data) < std::string(rhs._data);
     }
     bool operator==(const MyString& rhs) const {
          return std::string(this->_data) = std::string(rhs._data);
     }
};
size_t MyString::DCtor = 0;
size_t MyString::Ctor = 0;
size_t MyString::CCtor = 0;    
size_t MyString::CAsgn = 0;
size_t MyString::MCtor = 0;
size_t MyString::MAsgn = 0;
size_t MyString::Dtor = 0;
/* 使用 */
std::move(c1)	/* 调用move构造函数 */
```

#### 设计模式:

**面向对象设计原则:**

​		依赖倒置原则: 高层**稳定**模块,不应该依赖于低层**易变**的模块,且二者皆应该依赖于抽象层(**稳定**)

​								 抽象层(**稳定**)不应该依赖于具体实现(变化),具体实现应该依赖于抽象(稳定)

​		开放封闭原则:对扩展开放,对更改封闭(类模块应该是可扩展(继承),但不可修改)

​		单一职责原则:一个类应该隐含着**一个**引起它变化的原因(变化方向应该隐含着类的**责任**)

​		替换原则:		子类应该能够替换父类(is a的关系)

​		接口隔离原则:**接口应该小而完备**,不需要被使用的接口应该尽量不对外暴露

​		**优先使用对象组合**而不是继承: 继承在某种程度上破坏了封装性,子父类耦合度高

​															对象的组合则只需要被组合对象具有良好的接口定义,耦合度低

​		封装变化点: 使用封装来创建对象之间的分界层，让设计者可以在分界层的一侧进行修改，而不会对另一侧产							  生不良的影响，**一侧稳定,一侧变化**,从而实现层次间的松耦合						

​		**针对接口(抽象层)编程**，而不是针对实现编程: 

​							   不将变量类型声明为某个特定的具体类，而是声明为某个接口(抽象层)。

​							   客户程序无需获知对象的具体类型，只需要知道对象所具有的接口(使用抽象层所具有的接口)

​						       减少系统中各部分的依赖关系，从而实现“高内聚、松耦合” 的类型设计方案

##### 设计模式分类:

**组件协作:**通过晚绑定来实现框架与应用程序之间的松耦合

​		**Template Method:**

```c++
/* 应用背景:在某一项任务中,常常有一些固定的操作步骤和流程,但又有部分子步骤的需求常常会发生改变 */
class Library{ /* 应用程序库开发人员,可能很早以前就写好的库 */
public:
    void Run(){  /* 存在较为稳定的执行流程和步骤 */
        Step1();
        if (Step2()) { 	//支持变化 ==> 虚函数的多态调用
            Step3(); 
        }
        for (int i = 0; i < 4; i++){
            Step4(); 	//支持变化 ==> 虚函数的多态调用
        }
        Step5();
    }
	virtual ~Library(){ ... }
protected:
	/* 稳定不变的执行步骤 */
	void Step1() { ... }	
	void Step3() { ... }
	void Step5() { ... }
    /* 易变化的执行步骤,让子类继承,对其进行重写 */
	virtual bool Step2() = 0;
    virtual void Step4() =0; 
};

/* 现在的应用程序开发人员,继承Library并对Library中易变函数进行重写 */
class Application : public Library {
protected:
	virtual bool Step2(){
		//... 子类重写实现
    }
    virtual void Step4() {
		//... 子类重写实现
    }
};

int main(){
	    Library* pLib=new Application(); 
	    lib->Run(); /* 你不要调用我,让我来调用你 */
		delete pLib;
	}
}
```

​		**Strategy:**

```c++
/* 应用背景:在一些任务中,某些对象可能在不同的情况或场景下需要使用不同的算法,如果将这些算法用if.else之类的方式都编码到对象中,将会使得对象本身变得复杂,且相对于不常被调用到的if.else分支,是一种性能负担 */
class TaxStrategy{ /* 计税算法抽象类,易变化的算法应该继承于稳定的抽象类 */
public:
    virtual double Calculate(const Context& context)=0;
    virtual ~TaxStrategy(){}
};

class CNTax : public TaxStrategy{ /* 不同地区的计税算法 */
public:
    virtual double Calculate(const Context& context){ ... }
};
class USTax : public TaxStrategy{ /* 不同地区的计税算法 */
public:
    virtual double Calculate(const Context& context){ ... }
};
class DETax : public TaxStrategy{ /* 不同地区的计税算法 */
public:
    virtual double Calculate(const Context& context){ ... }
};
/* 后期业务变化,新增的计税算法 */
class FRTax : public TaxStrategy{ 
public:
	virtual double Calculate(const Context& context){ ... }
};

class SalesOrder{ /* 稳定的高层模块 */
private:
    TaxStrategy* strategy;	/* 维护着抽象类指针,准备多态的产生 */
public:
    SalesOrder(StrategyFactory* strategyFactory){ /* 根据具体情况创建对应的计税算法类 */
        this->strategy = strategyFactory->NewStrategy();
    }
    ~SalesOrder(){
        delete this->strategy;
    }
    public double CalculateTax(){
        
        Context context(); 
        double val = strategy->Calculate(context); /* 多态调用 */
        /* ... */
    }
};
```

​		**Observer/Event:**

```c++
/* 应用背景:在某一些任务中,我们需要为对象建立一种"通知依赖关系",即目标对象发生改变时其所对应的依赖对象(观察者对象)将得到通知,且这种依赖关系不能过于紧密 */
class IProgress{ /* 抽象类,稳定的模块应该于抽象层 */
public:
	virtual void DoProgress(float value)=0;
	virtual ~IProgress(){}
};
/* 文件分割功能模块,稳定的高层模块 */
class FileSplitter { /* 订阅中,数据发送变化时即会通知观察者 */
	string m_filePath;
	int m_fileNumber;
	/* 这里应该是一个抽象类基类,订阅者只需要继承并重写抽象类子类,即可被订阅中心通知调用 */
	List<IProgress*>  m_iprogressList; /* 订阅者列表 */
public:
	FileSplitter(const string& filePath, int fileNumber) : 
		m_filePath(filePath), 
		m_fileNumber(fileNumber){ /* 构造 */
	}
	void split(){ /* 文件分割功能函数 */
		/* 1.读取大文件 */
		/* 2.分批次向小文件中写入 */
		for (int i = 0; i < m_fileNumber; i++){
			/* ... */
			float progressValue = m_fileNumber;
			progressValue = (i + 1) / progressValue;
            /* ... */
			onProgress(progressValue); /* 发送通知,依次调用订阅者功能函数 */
		}
	}
    /* 向订阅列表添加新的订阅者 */
	void addIProgress(IProgress* iprogress){
		m_iprogressList.push_back(iprogress);
	}
    /* 从订阅列表删除订阅者 */
	void removeIProgress(IProgress* iprogress){
		m_iprogressList.remove(iprogress);
	}
protected:
	virtual void onProgress(float value){ /* 发送通知,虚函数是继续方便被子类重写 */		
		List<IProgress*>::iterator itor=m_iprogressList.begin();
		while (itor != m_iprogressList.end()) /* 遍历订阅列表 */
			(*itor)->DoProgress(value);  	  /* 依次调用订阅者功能函数 */
			itor++;
		}
	}
};

class MainForm : public Form, public IProgress
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;
public:
	void Button1_Click(){
		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());
        FileSplitter splitter(filePath, number);
		ConsoleNotifier cn;
		splitter.addIProgress(this); /* 添加订阅者 */
		splitter.addIProgress(&cn)； /* 添加订阅者 */
		splitter.split();
		splitter.removeIProgress(this);
	}
    /* 订阅者只需要继承抽象类,并重写对应的功能函数,即可被调用中心通知调用 */
	virtual void DoProgress(float value){
		progressBar->setValue(value);
	}
};
/* 订阅者只需要继承抽象类,并重写对应的功能函数,即可被调用中心通知调用 */
class ConsoleNotifier : public IProgress {
public:
	virtual void DoProgress(float value){
		cout << ".";
	}
};
```

**单一职责:**类责任划分不清,盲目使用继承,导致子类数量急剧膨胀,出现大量重复代码

​		**Decorator:**

```c++
/* 应用背景:为了“扩展子类的功能”,过度使用继承,随着扩展子类的增加,各种扩展子类的组合,导致子类数量急剧膨胀 ,盲目继承扩展子类,易违背一个类的单一职责的设计原则 */
class Stream{ 
public：
    virtual char Read(int number)=0;
    virtual void Seek(int position)=0;
    virtual void Write(char data)=0;
    virtual ~Stream(){}
};
/* 业务操作 */
class FileStream: public Stream{ /* 与Stream类职责方向一致 */
public:
    virtual char Read(int number){ ... }	/* 读文件流 */
    virtual void Seek(int position){ ... }  /* 定位文件流 */
    virtual void Write(char data){ ... }    /* 写文件流 */
};
class NetworkStream :public Stream{ /* 与Stream类职责方向一致 */
public:
    virtual char Read(int number){ ... } 	/* 读网络流 */
    virtual void Seek(int position){ ... } 	/* 定位网络流 */
    virtual void Write(char data){ ... } 	/* 写网络流 */  
};
class MemoryStream :public Stream{ /* 与Stream类职责方向一致 */
public:
    virtual char Read(int number){ ... }   /* 读内存流 */
    virtual void Seek(int position){ ... } /* 定位内存流 */
    virtual void Write(char data){ ... }   /* 写内存流 */
};
/* 装饰模式:在现有功能基础上添加一些额外的功能,功能与适配器相似 */
DecoratorStream: public Stream{ /* 继承是为了加入继承体系,这样才能继续被其他类装饰 */
protected:
    Stream* stream; /* 维护一个抽象类指针,在产生多态时才能调用现有基础功能 */
    DecoratorStream(Stream * stm):stream(stm){
    } 
};
/* 扩展的加密功能 */
class CryptoStream: public DecoratorStream { /* 与Stream类职责方向不一致,扩展功能方向 */
public:
    CryptoStream(Stream* stm):DecoratorStream(stm){ ... }
    virtual char Read(int number){
        /* 额外的加密操作... */
        stream->Read(number);  /* 读文件流 */
    }
    virtual void Seek(int position){
        /* 额外的加密操作... */
        stream::Seek(position); /* 定位文件流 */
        /* 额外的加密操作... */
    }
    virtual void Write(byte data){
        /* 额外的加密操作... */
        stream::Write(data);	/* 写文件流 */
        /* 额外的加密操作... */
    }
};
/* 扩展的缓存功能 */
class BufferedStream : public DecoratorStream{ /* 与Stream类职责方向不一致,扩展功能方向 */
public:
    BufferedStream(Stream* stm):DecoratorStream(stm){ ... }    
	/* 同上类似 */
};
void Process(){
    /* 运行时装配,各种扩展功能可相互组合 */
    FileStream* s1=new FileStream(); 			/* 读文件流 */
    CryptoStream* s2=new CryptoStream(s1);  	/* 读文件流+加密 */
    BufferedStream* s3=new BufferedStream(s1);	/* 读文件流+缓存 */
    BufferedStream* s4=new BufferedStream(s2);  /* 读文件流+缓存+加密 */
}
```

​		**Bridge:**

```c++
/* 应用背景:由于某些类型的固有实现逻辑,使得其具有两个甚至多个维度的变化,此时我们需要将其继续抽象分离,使得它们可以相互独立的变化,最终通过抽象类指针进行桥接组合 */
class Messager{ /* 未优化前:原抽象类型 */
public:
    /* 业务实现相关方向 */
    virtual void Login(string username, string password)=0;
    virtual void SendMessage(string message)=0;
    virtual void SendPicture(Image image)=0;
	/* 平台实现相关方向 */
    virtual void PlaySound()=0;
    virtual void DrawShape()=0;
    virtual void WriteText()=0;
    virtual void Connect()=0;
    virtual ~Messager(){}
};
/* 优化后:将存在不同方向职责的抽象类继续分离 */
class Messager{ /* 业务逻辑实现相关抽象类 */
protected:
     MessagerImp* messagerImp; /* 通过该抽象类指针,进行两个变化方向上的桥接 */
public:
    virtual void Login(string username, string password)=0;
    virtual void SendMessage(string message)=0;
    virtual void SendPicture(Image image)=0;
    virtual ~Messager(){}
};
class MessagerImp{ /* 平台实现逻辑相关抽象类 */
public:
    virtual void PlaySound()=0;
    virtual void DrawShape()=0;
    virtual void WriteText()=0;
    virtual void Connect()=0;
    virtual MessagerImp(){}
};
/* 平台实现相关 */
class PCMessagerImp : public MessagerImp{
public:
    virtual void PlaySound(){ ... }
    virtual void DrawShape(){ ... }
    virtual void WriteText(){ ... }
    virtual void Connect(){ ... }
};
class MobileMessagerImp : public MessagerImp{
public:
    virtual void PlaySound(){ ... }
    virtual void DrawShape(){ ... }
    virtual void WriteText(){ ... }
    virtual void Connect(){ ... }
};
/* 业务实现相关 */
class MessagerLite:public Messager { /* 利用平台实现相关抽象类指针进行桥接 */
public:  
    virtual void Login(string username, string password){
        messagerImp->Connect();
        /* ... */
    }
    virtual void SendMessage(string message){  
        messagerImp->WriteText();
        /* ... */
    }
    virtual void SendPicture(Image image){ 
        messagerImp->DrawShape();
        /* ... */
    }
};
class MessagerPerfect:public Messager { /* 利用平台实现相关抽象类指针进行桥接 */
public: 
    virtual void Login(string username, string password){
        messagerImp->PlaySound();
        /* ... */
        messagerImp->Connect();
        /* ... */
    }
    virtual void SendMessage(string message){
        messagerImp->PlaySound();
        /* ... */
        messagerImp->WriteText();
        /* ... */
    }
    virtual void SendPicture(Image image){
        messagerImp->PlaySound();
        /* ... */
        messagerImp->DrawShape();
        /* ... */
    }
};
void Process(){
    //运行时装配
    MessagerImp* mImp=new PCMessagerImp();
    Messager *m =new MessagerPerfect(mImp); /* 注意:上面省略了构造函数 */
}
```

**对象创建:**

​		**Factory Mrthod:**

```c++
```

​		**Abstract Factory:**

```c++
```

​		**Prototype:    **

```c++
```

​		**Builder:**

```c++
```

**对象性能:**

​		**Singleton:**

```c++
```

​		**Flyweight:**

```c++
```

**接口隔离:**

​		**Facade:**

```c++

```

​		**Proxy:**

```c++
```

​		**Mediator:**

```c++
```

​		**Adapter:**

```c++
```

**状态变化:**

​		**Memento:**

```c++
```

​		**State:**

```c++
```

**数据结构:**

​		**Composite:**

```c++
```

​		**Iterator:**

```c++
```

​		**Chain of Resposibility:**

```c++
```

**行为变化:**

​		**Command:**

```c++
```

​		**Visitor:**

```c++
```

**领域问题:**

​		**Interpreter:**

```c++
```

