---
title: Type Deduction
published: true
categories: cpp
tags: c++ type deduction 형식연역 
---

- "Effective Modern C++" 정리 

### 1. 템플릿 연역 규칙

```cpp
// template function  
template <typename T>  
void f(ParamType param);  

// call f function with expr  
f(expr);
```

- 컴파일러는 컴파일도중 expr 을 이용해서 T, ParamType 에 대한 형식을 연역한다. 
- T에 대해 연역된 형식은 expr 과 ParamType 의 형태에도 의존한다. 
- ParamType 의 형태는 세 가지 경우가 있다.  
  > a. ParamType이 포인터 또는 참조 형식(보편 참조 아님) 인 경우  
  > b. ParamType이 보편 참조인 경우  
  > c. ParamType이 포인터도 아니고 참조도 아닌 경우  

- a. ParamType 이 포인터 또는 참조 형식(보편 참조 아님) 인 경우  

```cpp
template <typename T>
void f(T& param);   // param 은 참조 형식

int x = 27;         // x  -> int 
const int cx = x;   // cx -> const int
const int& rx = x;  // rx -> const int 인 x 에 대한 참조 

f(x);               // T -> int, ParamType -> int& 
f(cx);              // T -> const int, ParamType -> const int& 
f(rx);              // T -> const int, ParamType -> const int&
```

```cpp
template <typename T>
void f(const T& param); // param 은 const 에 대한 참조

int x = 27;         // x  -> int 
const int cx = x;   // cx -> const int
const int& rx = x;  // rx -> const int 인 x 에 대한 참조 

f(x);               // T -> int, ParamType -> const int&
f(cx);              // T -> int, ParamType -> const int&
f(rx);              // T -> int, ParamType -> const int&
```

```cpp
template <typename T>
void f(T* param);       // param 은 포인터 

int x = 27;             // x  -> int 
const int* px = &x;     // px -> const int 로서 x를 가리키는 포인터 

f(&x);                  // T -> int, ParamType -> int*
f(px);                  // T -> const int, ParamType -> const int*
```

- b. ParamType 이 보편 참조인 경우  
  expr이 왼값이면 T와 ParamType 둘 다 왼값으로 연역된다.  
  expr이 오른값이면 정상적으로 처리된다. 

```cpp
template <typename T>
void f(T&& param);

int x = 27;         // x  -> int 
const int cx = x;   // cx -> const int
const int& rx = x;  // rx -> const int 인 x 에 대한 참조 

f(x);               // T -> int&, ParamType -> int&
f(cx);              // T -> const int&, ParamType -> const int&
f(rx);              // T -> const int&, ParamType -> const int&
f(27);              // T -> int, ParamType -> int&& 
```

- c. ParamType 이 포인터도 아니고 참조도 아닌 경우  
  > expr형식이 참조이면 참조부분은 무시된다.   
  > expr의 참조성을 무시한 후, 만일 expr 이 const 이면 그 const 역시 무시된다.  
  > 만일 volatile 이면 그것도 무시한다.  

```cpp
template <typename T>
void f(T param);

int x = 27;         // x  -> int 
const int cx = x;   // cx -> const int
const int& rx = x;  // rx -> const int 인 x 에 대한 참조 

f(x);               // T -> int, ParamType -> int
f(cx);              // T -> int, ParamType -> int
f(rx);              // T -> int, ParamType -> int
```

```cpp
template <typename T>
void f(T param);

const char* const ptr = "Fun with pointers";

f(ptr);             // T -> const char*, ParamType -> const char*
```

- 배열을 넘기는 경우  
  배열은 포인터로 붕괴(decay)된다.  
  함수에 값으로 전달되는 배열의 형식은 포인터 형식으로 연역된다.  

```cpp
const char name[] = "J. P. Briggs";
const char* ptrToName = name;   // 배열이 포인터로 붕괴된다.  

template <typename T>
void f(T param);

f(name);                        // T -> const char*, ParamType -> const char* 
```

- 배열에 대한 참조를 넘기는 경우 

```cpp
template <typename T>
void f(T& param);

f(name);        // T -> const char name[13], ParamType -> const char (&)[13]

// 배열의 크기를 컴파일 시점 상수로 돌려주는 템플릿 함수 
tempalte <typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
    return N;
}
```

- 함수 인수  
  배열과 마찬가지로 함수 형식도 함수 포인터로 붕괴할 수 있다. 

```cpp
void someFunc(int, double);

template <typename T>
void f1(T param);

template <typename T>
void f2(T& param);

f1(someFunc);       // T -> void (*)(int, double), ParamType -> void (*)(int, double)

f2(someFunc);       // T -> void (&)(int, double), ParamType -> void (&)(int, double)
```

### 2. auto 형식 연역 규칙  
- "1. 템플릿 연역 규칙" 에서 T 에 해당되는 것이 auto 이며  
  변수 형식 지정자(type specifier)는 ParamType 과 동일한 역할을 한다. 

```cpp
auto x = 27;            // 형식 지정자 -> auto

const auto cx = x;      // 형식 지정자 -> const auto 

const auto& rx = x;     // 형식 지정자 -> const auto& 
```

- auto 도 템플릿 연역 규칙과 마찬가지로 3가지 경우로 나눌 수 있다. 
  > a. 형식 지정자가 포인터나 참조 형식(보편 참조 아님)인 경우
  > b. 형식 지정자가 보편 참조인 경우  
  > c. 형식 지정자가 포인터도 아니고 참조도 아닌 경우  

```cpp
auto x = 27;            // (case c) x -> int 

const auto cx = x;      // (case c) cx -> const int 

const auto& rx = x;     // (case a) rx -> const int&

auto&& uref1 = x;       // (case b) uref1 -> int& 

auto&& uref2 = cx;      // (case b) uref2 -> const int& 

auto&& uref3 = 27;      // (case b) uref3 -> int&& 

```

- 배열과 함수명의 붕괴 
  템플릿 연역 규칙과 동일하다. 

```cpp
const char name[] = "R. N. Briggs";
auto arr1 = name;               // arr1 -> const char*
auto& arr2 = name;              // arr2 -> const char (&)[13]

void someFunc(int, double);     
auto func1 = someFunc;          // func1 -> void (*)(int, double)
auto& func1 = someFunc;         // func2 -> void (&)(int, double)
```

- 템플릿 연력 규칙과 다른 점  
  auto로 선언된 변수를 중괄호 초기치로 초기화 하는 경우 std::initializer_list로 연역된다. 

```cpp
auto x1 = 27;        // 형식은 int, 값은 27   
auto x2(27);         // 형식은 int, 값은 27   

auto x3 = { 27 };    // 형식은 std::initializer_list<int>, 값은 27 
auto x4{ 27 };       // 형식은 int, 값은 27   
```

```cpp
auto x = { 11, 13, 9 };     // x -> std::initializer_list<int>

template <typename T>
void f(T param);
f({ 11, 23, 9 });           // 오류 
```

- 추가 연역 규칙 (C++14)  
  함수 반환 형식과 람다 매개변수 선언도 가능하다.  
  이 연역은 템플릿 형식 연역 규칙들이 적용된다.  
  따라서 중괄호 초기치를 돌려주는 함수의 반환시에는 컴파일 오류가 발생된다. 

```cpp
// C++14
auto createInitList()
{
    return { 1, 2, 3 }; // 오류 
}

std::vector<int> v;
auto resetV = [&v](const auto& newValue) {
    v = newValue;
};

resetV({ 1, 2, 3 });    // 오류 { 1, 2, 3 } 의 형식을 연역할 수 없음   
```

### 3. decltype 
- 템플릿과 auto 의 형식 연역과 다르게 이름이나 표현식의 구체적인 형식을 그대로 말해준다. 

```cpp
const int i = 0;                // decltype(i) -> const int

bool f(const Widget& w);        // decltype(w) -> const Widget&
                                // decltype(f) -> bool(const Widget&)
struct Point {
    int x, y;                   // decltype(Point::x) -> int
};                              // decltype(Point::y) -> int

Widget w;                       // decltype(w) -> Widget

if (f(w)) ...                   // decltype(f(w)) -> bool 

template <typename T>
class vector {
public:
    ...
    T& operator[](std::size_t index);
    ...
};

vector<int> v;                  // decltype(v) -> vector<int>
...
if (v[0] == 0) ...              // decltype(v[0]) -> int&
```

- C++11에서 decltype은 함수의 반환 형식이 그 매개변수 형식들에 의존하는 함수 템플릿을 선언할 때 주로 쓰인다.  

```cpp
// C++11
template <typename Container, typename Index>
auto authAndAccess(Container c, Index i) -> decltype(c[i])
{
    authenticateUser();
    return c[i];
}

// 함수 앞의 auto는 후행 반환 형식(trailing return type) 구문이 쓰임을 나타낸다. 
```

- C++11은 람다 함수가 한 문장으로 이루어져 있다면 그 반환 형식의 연역을 허용한다. 
- C++14는 모든 람다와 모든 함수의 반환 형식 연역을 허용한다.   
  따라서 C++14 에서는 -> decltype 없이 auto 만 남겨 두어도 된다. (되기는 한다.) 

```cpp
// C++14
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index& i)
{
    authenticateUser();
    return c[i];
}

std::deque<int> d;
authAndAccess(d, 5) = 10;

// auto가 지정되면 템플릿 형식 연역을 적용한다. 
// auto 반환 형식 연역 과정에서 참조가 제거 되므로 반환 형식은 int가 된다. 
// 따라서 이 코드는 오른값 int에 10을 재정한다. 
// 따라서 컴파일 오류
```

- 위의 문제를 위해 decltype(auto)가 만들어졌다. 

```cpp
// C++14
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index& i)
{
    authenticateUser();
    return c[i];
}
```

- 변수 선언시에도 초기화 표현식에 decltype 타입 연역 규칙들을 적용할 수 있다. 

```cpp
Widget w;
const Widget& cw = w;           
auto myWidget1 = cw;            // myWidget1 -> Widget 
decltype(auto) myWidget2 = cw;  // mywidget2 -> const Widget&  
```

- 위의 authAndAccess의 리턴 값을 오른쪽 값, 왼쪽 참조 모두 올 수 있게 하려면  

```cpp
// C++14
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
{
    autoenticateUser();
    return std::forward<Container>(c)[i];
}

// C++11
template <typename Container, typename Index>
auto authAndAccess(Container&& c, Index i) 
-> decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

- decltype 추가 설명  
  decltype을 이름에 적용하면 그 이름에 대해 선언된 형식이 산출된다.  
  이름이 아니고 형식이 T인 어떤 왼값 표현식에 대해서는 T& 를 보고한다. 

```cpp
int x = 0;  

decltype(x) y = 0   // decltype(x) -> int 
decltype((x)) z = x // decltype((x)) -> int&

decltype(auto) f1()
{
    int x = 0;
    ...
    return x;       // decltype(x)는 int이므로 f1은 int를 반환 
}

decltype(auto) f2()
{
    int x = 0;
    ...
    return (x);     // decltype((x))는 int&이므로 f2는 int&를 반환 
}
```

### 4. 연역된 형식 파악하는 방법  
- IDE 편집기  
  프로그램 개체(변수, 매개변수, 함수 등) 위에 마우스 커서를 올리면 그 개체의 형식을 표시  

- 컴파일러의 진단 메시지  
  정의 없이 선언만 해둔 클래스 템플릿을 사용하여 오류를 유발한 후 오류 메시지에 나온 타입을 확인한다.  

```cpp
template <typename T>
class TD;

TD<decltype(x)> xType;
TD<decltype(y)> yType;

// class TD에 대한 정의가 없으므로 다음과 같은 오류가 발생된다. 
error: 'xType'은 정의되지 않은 class 'TD<int>'를 사용합니다.
error: 'yType'은 정의되지 않은 class 'TD<const int*>'를 사용합니다. 
```

- 실행 시점 출력-1  
  다음과 같은 형식으로 코드를 작성한다.  

```cpp
std::cout << typeid(x).name() << '\n';
std::cout << typeid(y).name() << '\n';

// int x, int* y 로 정의된 경우 

// gcc, clang 의 경우 
//  x -> i 로 표기 (i -> integer)
//  y -> PKi 로 표기 (PK -> Pointer to const) 

// MS C++ 의 경우 
//  x -> int 
//  y -> int const * 
```
  
- 실행 시점 출력-2  
  다음과 같은 경우 생각대로 나오지 않는다.  

```cpp
std::vector<Widget> createVec();
const auto vw = createVec();
if (!vw.empty()) {
    f(&vw[0]);
}

template <typename T>
void f(const T& param)
{
    std::cout << "T = " << typeid(T).name() << '\n';
    std::cout << "param = " << typeid(param).name() << '\n';
}

// gcc, clang 의 경우 
// T = PK6Widget
// param = PK6Widget 

// MS C++ 의 경우  
// T = class Widget const *
// param = class Widget const * 

// 원하는 값은 
// T = Widget const *
// param = Widget const * const &

// 표준에 따르면
// std::type_info::name은 반드시 주어진 형식을 마치 
// 템플릿 함수에 값 전달 매개 변수로서 전달된 것처럼 취급해야 한다. 
// -> 값 전달이므로 const, volatile, 참조는 빠지게 된다. 
// 따라서 
// T = Widget const * -> Widget const * 
// param = Widget const * const & -> Widget const *  
```

- Boost TypeIndex 라이브러리 사용  

```cpp
#include <boost/type_index.hpp>

template <typename T>
void f(const T& param)
{
    using std::cout;
    using boost::typeidex::type_id_with_cvr;

    cout << "T = " 
        << type_id_with_cvr<T>().pretty_name()
        << '\n';

    cout << "param = "
        << type_id_with_cvr<decltype(param)>().pretty_name()
        << '\n';
}

std::vector<Widget> createVec();
const auto vw = createVec();
if (!vw.empty()) {
    f(&vw[0]);
}

// gcc, clang 
// T = Widget const *
// param = Widget const * const &

// MS C++
// T = class Widget const *
// param = class Widget const * const &
```

- 그래도 이런 것들은 도구일 뿐이므로 형식 연역 규칙들을 잘 숙지하도록 하자. 

