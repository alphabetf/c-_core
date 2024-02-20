##### C++核心

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
    return malloc(size;
}
inline void* operator new[](size_t size){
    cout << "global new[]()" << endl;
    return malloc(size;
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
    void* operator new(size_t);
    void operator delete(void*,size_t); /* size_t为可选参数,可写可不写,由编译器传入 */
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

