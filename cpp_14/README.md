<h2>C++14新特性</h2>

<h4 id="cpp_14_01">函数返回值类型推导</h4>

C++14对函数返回类型推导规则做了优化，先看一段代码：

```CPP
#include <iostream>

using namespace std;

auto func(int i) {
    return i;
}

int main() {
    cout << func(4) << endl;
    return 0;
}
```
使用C++11编译：
```CPP
~/test$ g++ test.cc -std=c++11
test.cc:5:16: error: ‘func’ function uses ‘auto’ type specifier without trailing return type
 auto func(int i) {
                ^
test.cc:5:16: note: deduced return type only available with -std=c++14 or -std=gnu++14
```
上面的代码使用C++11是不能通过编译的，通过编译器输出的信息也可以看见这个特性需要到C++14才被支持。

返回值类型推导也可以用在模板中：

```CPP
#include <iostream>
using namespace std;

template<typename T> auto func(T t) { return t; }

int main() {
    cout << func(4) << endl;
    cout << func(3.4) << endl;
    return 0;
}
```

注意：

）函数内如果有多个return语句，它们必须返回相同的类型，否则编译失败。

```CPP
auto func(bool flag) {
    if (flag) return 1;
    else return 2.3; // error
}
// inconsistent deduction for auto return type: ‘int’ and then ‘double’
```

）如果return语句返回初始化列表，返回值类型推导也会失败

```CPP
auto func() {
    return {1, 2, 3}; // error returning initializer list
}
```

) 如果函数是虚函数，不能使用返回值类型推导
```CPP
struct A {
	// error: virtual function cannot have deduced return type
	virtual auto func() { return 1; } 
}
```

） 返回类型推导可以用在前向声明中，但是在使用它们之前，翻译单元中必须能够得到函数定义
```CPP
auto f();               // declared, not yet defined
auto f() { return 42; } // defined, return type is int

int main() {
	cout << f() << endl;
}
```

）返回类型推导可以用在递归函数中，但是递归调用必须以至少一个返回语句作为先导，以便编译器推导出返回类型。
```CPP
auto sum(int i) {
    if (i == 1)
        return i;              // return int
    else
        return sum(i - 1) + i; // ok
}
```

<br/>
<br/>

<h4 id="cpp_14_02">lambda参数auto</h4>

在C++11中，lambda表达式参数需要使用具体的类型声明：

```CPP
auto f = [] (int a) { return a; }
```

在C++14中，对此进行优化，lambda表达式参数可以直接是auto：

```CPP
auto f = [] (auto a) { return a; };
cout << f(1) << endl;
cout << f(2.3f) << endl;
```

<br/>
<br/>

<h4 id="cpp_14_03">变量模板</h4>

C++14支持变量模板：

```CPP
template<class T>
constexpr T pi = T(3.1415926535897932385L);

int main() {
    cout << pi<int> << endl; // 3
    cout << pi<double> << endl; // 3.14159
    return 0;
}
```

<br/>
<br/>

<h4 id="cpp_14_04">别名模板</h4>

C++14也支持别名模板：

```CPP
template<typename T, typename U>
struct A {
    T t;
    U u;
};

template<typename T>
using B = A<T, int>;

int main() {
    B<double> b;
    b.t = 10;
    b.u = 20;
    cout << b.t << endl;
    cout << b.u << endl;
    return 0;
}
```

<br/>
<br/>

<h4 id="cpp_14_05">constexpr的限制</h4>

C++14相较于C++11对constexpr减少了一些限制：

）C++11中constexpr函数可以使用递归，在C++14中可以使用局部变量和循环

```CPP
constexpr int factorial(int n) { // C++14 和 C++11均可
    return n <= 1 ? 1 : (n * factorial(n - 1));
}
```

在C++14中可以这样做：
```CPP
constexpr int factorial(int n) { // C++11中不可，C++14中可以
    int ret = 0;
    for (int i = 0; i < n; ++i) {
        ret += i;
    }
    return ret;
}
```

）C++11中constexpr函数必须必须把所有东西都放在一个单独的return语句中，而constexpr则无此限制：
```CPP
constexpr int func(bool flag) { // C++14 和 C++11均可
    return 0;
}
```

在C++14中可以这样：
```CPP
constexpr int func(bool flag) { // C++11中不可，C++14中可以
    if (flag) return 1;
    else return 0;
}
```

<br/>
<br/>

<h4 id="cpp_14_06">[[deprecated]]标记</h4>

C++14中增加了deprecated标记，修饰类、变、函数等，当程序中使用到了被其修饰的代码时，编译时被产生警告，用户提示开发者该标记修饰的内容将来可能会被丢弃，尽量不要使用。

```CPP
struct [[deprecated]] A { };

int main() {
    A a;
    return 0;
}
```

当编译时，会出现如下警告：
```CPP
~/test$ g++ test.cc -std=c++14
test.cc: In function ‘int main()’:
test.cc:11:7: warning: ‘A’ is deprecated [-Wdeprecated-declarations]
     A a;
       ^
test.cc:6:23: note: declared here
 struct [[deprecated]] A {
 ```
 
<br/>
<br/>

<h4 id="cpp_14_07">二进制字面量与整形字面量分隔符</h4>

C++14引入了二进制字面量，也引入了分隔符，防止看起来眼花哈~
```CPP
int a = 0b0001'0011'1010;
double b = 3.14'1234'1234'1234;
```

<br/>
<br/>

<h4 id="cpp_14_08">std::make_unique</h4>

C++11中有std::make_shared，却没有std::make_unique，在C++14已经改善。

```CPP
struct A {};
std::unique_ptr<A> ptr = std::make_unique<A>();
```

<br/>
<br/>

<h4 id="cpp_14_08">std::shared_timed_mutex与std::shared_lock</h4>

C++14通过std::shared_timed_mutex和std::shared_lock来实现读写锁，保证多个线程可以同时读，但是写线程必须独立运行，写操作不可以同时和读操作一起进行。

实现方式如下：

```CPP
struct ThreadSafe {
    mutable std::shared_timed_mutex mutex_;
    int value_;

    ThreadSafe() {
        value_ = 0;
    }

    int get() const {
        std::shared_lock<std::shared_timed_mutex> loc(mutex_);
        return value_;
    }

    void increase() {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ += 1;
    }
};
```

<br/>
<br/>

<h4 id="cpp_14_09">std::integer_sequence</h4>
```CPP
template<typename T, T... ints>
void print_sequence(std::integer_sequence<T, ints...> int_seq)
{
    std::cout << "The sequence of size " << int_seq.size() << ": ";
    ((std::cout << ints << ' '), ...);
    std::cout << '\n';
}

int main() {
    print_sequence(std::integer_sequence<int, 9, 2, 5, 1, 9, 1, 6>{});
    return 0;
}
```

输出：

```CPP
7 9 2 5 1 9 1 6
```

std::integer_sequence和std::tuple的配合使用：

```CPP
template <std::size_t... Is, typename F, typename T>
auto map_filter_tuple(F f, T& t) {
    return std::make_tuple(f(std::get<Is>(t))...);
}

template <std::size_t... Is, typename F, typename T>
auto map_filter_tuple(std::index_sequence<Is...>, F f, T& t) {
    return std::make_tuple(f(std::get<Is>(t))...);
}

template <typename S, typename F, typename T>
auto map_filter_tuple(F&& f, T& t) {
    return map_filter_tuple(S{}, std::forward<F>(f), t);
}
```

<br/>
<br/>

<h4 id="cpp_14_10">std::exchange</h4>

直接看代码吧：

```CPP
int main() {
    std::vector<int> v;
    std::exchange(v, {1,2,3,4});
    cout << v.size() << endl;
    for (int a : v) {
        cout << a << " ";
    }
    return 0;
}
```
看样子貌似和std::swap作用相同，那它俩有什么区别呢？

可以看下exchange的实现：

```CPP
template<class T, class U = T>
constexpr T exchange(T& obj, U&& new_value) {
    T old_value = std::move(obj);
    obj = std::forward<U>(new_value);
    return old_value;
}
```

可以看见new_value的值给了obj，而没有对new_value赋值！

<br/>
<br/>

<h4 id="cpp_14_11">std::quoted</h4>

C++14引入std::quoted用于给字符串添加双引号，直接看代码：

```CPP
int main() {
    string str = "hello world";
    cout << str << endl;
    cout << std::quoted(str) << endl;
    return 0;
}
```

编译&输出：

```CPP
~/test$ g++ test.cc -std=c++14
~/test$ ./a.out
hello world
"hello world"
```




