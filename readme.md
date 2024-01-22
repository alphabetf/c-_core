##### C++核心

在类中写的简短的成员函数实现，默认会自动隐式加上inline关键字，

如果成员函数不改变类中的成员变量，则应该在**函数后面加上const关键字**

**相同发class的各个object互为friends(友元)**

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

原因：类中的成员函数在构建编译时就能清楚的知道类的内存布局，所以编译器也允许成员函数中具有自身类类型的参数，在使用类成员变量时，其实是基于指针偏移的，所以机缘巧合的造成了这种情况，其实并非友元，但可以理解记忆为：**相同发class的各个object互为friends(友元)**

