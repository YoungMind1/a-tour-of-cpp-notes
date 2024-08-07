# Who is This Document For?
These notes are most suited for those who are already programming in another programming language (not every basic thing is covered)
# Table of Content
- [Chapter 1 | Basics](#chapter-1--basics)
- [Chapter 2 | User-Defined Types](#chapter-2--user-defined-types)
- [Chapter 3 | Modularity](#chapter-3--modularity)
- [Chapter 4 | Error Handling](#chapter-4--error-handling)
- [Chapter 5 | Classes](#chapter-5--classes)
- [Chapter 6 | Essential Operations](#chapter-6--essential-operations)
- [Chapter 7 | Templates](#chapter-7--templates)
- [Chapter 8 | Concepts and Generic Programming](#chapter-8--concepts-and-generic-programming)
- [Chapter 9 | Library Overview](#chapter-9--library-overview)

# Chapter 1 | Basics
## Function Types
```cpp
// type: double(const vector<double>&, int)
double get(const vector<double>& vec, int index);

// type: char& String::(int)
char& String::operator[](int index);
```

### Boolean Size
Although a boolean is only 1 bit, but it is stored in 8 bit (lowest possible size)

### Specific and Ubiquitous Types
When we want a type of a specific size, we use a standard-library type alias, such as `int32_t`.
## Initializing Variables
If you initialize your variables via `{}` then no information loss will happen.
these information loss are a price paid for keeping c compatibility.
```cpp
int i1 = 7.8; // i1 becomes 7 (surprise?)
int i2 {7.8}; // error: floating-point to integer conversion
```
## Immutability
- `const`'s value is determined in runtime, it is primarily used in interfaces to get a pointer and make the user certain that the state of the object is not going to change.
- `constexpr` is used to specify constants to allow placement of data in read-only memory (where it is unlikely to be corrupted) and for performance. is compile-time, a function can be `constexpr` - but then it has to be simple and have no side effects, the value of `constexpr` is evaluated at compile time.
- `consteval` is used only in compile times. you may use `constexpr` in non constant expressions, but you cannot do the dame with `consteval`.
## Pointers and References
In simple terms: *In an expression, prefix unary `∗` means ‘‘contents of’’ and prefix unary `&` means ‘‘address of.’’*

for each loops in c++ are copying the value, if you don't want to copy, use this:
```c++
void increment()
{
	int v[] = {0,1,2,3,4,5,6,7,8,9};
	for (auto& x : v)
		++x;
}
```
Note: *In a declaration, the unary suffix `&` means ‘‘reference to.’’ A reference is similar to a pointer, except that you don’t need to use a prefix `∗` to access the value referred to by the reference. Also, a reference cannot be made to refer to a different object after its initialization.*
References are particularly useful for specifying function arguments. because you are sure that we are not copying and we are doing the stuff with that thing, and not changing it mid-way.

### Null Pointer
*There is no ‘‘null reference.’’ A reference must refer to a valid object (and implementations assume that it does). There are obscure and clever ways to violate that rule; don’t do that.*, interesting
Use `nullptr` instead of `NULL` OR `0`. due to type system and inference and type deduction. 0 might be an integer!
you can do `if (auto n = v.size(); n!=0)` which is equal to `if (auto n = v.size())`

references and pointers differ, here is a great example:
![how-pointers-work](./images/pointers.png)
![how-references-work](./images/references.png)
remember, we don't need to use `*` to get the value in references, we are working with the values themselves here.
*To access the value pointed to by a pointer, you use `∗`; that is automatically (implicitly) done for a reference.*

# Advice
- Don't use built-in types/features on their own, use them indirectly through standard library.
- Focus on programming techniques, not on language features.
- Use `constexpr` if a function may be evaluated at compile time.
- Use `consteval` if a function must be evaluated at compile time.
- Prefer immutable data.
- Prefer `{}` over `=` for initialization.
- Use `unsigned` only for bit manipulation, it fucks with modular operation, and it doesn't really prevent negative numbers, Have explicit guards for those.
- Use `nullptr` instead of older stuff.

# Chapter 2 | User-Defined Types
### Scope | `delete` operator
Objects allocated on the free store are independent of the scope from which they are created and ‘‘live’’ until they are destroyed using the `delete` operator. objects are allocated on the free store (heap) via `new` keyword.
### Accessing members of structs
To access a struct member, use `.` in references and names, and `->` in pointers.

### Difference Between a *class* and a *struct*
There is no fundamental difference between a struct and a class; a struct is simply a class with members public by default. For example, you can define constructors and other member functions for a struct.

### Define extra operators for enums
```cpp
Traffic_light& operator++(Traffic_light& t) // prefix increment: ++
{
	switch (t) { 
		case Traffic_light::green:  return t=Traffic_light::yellow;
		case Traffic_light::yellow: return t=Traffic_light::red;
		case Traffic_light::red:    return t=Traffic_light::green; 
	}
}
auto signal = Traffic_light::red;
Traffic_light next = ++signal; // next becomes Traffic_light::green
```
You can abbreviate the enum calls in a scope:
```cpp
Traffic_light& operator++(Traffic_light& t) // prefix increment: ++
{
	using enum Traffic_light; // here, we are using Traffic_light
	switch (t) {
		case green: return t=yellow;
		case yellow: return t=red;
		case red: return t=green;
	}
}
```



### Advice
- Use `variant` instead of naked unions.

# Chapter 3 | Modularity
we are separating declarations with definitions. there might be several declarations, but only one definition.
You can achieve modularity via either header files or modules.
both the user code that uses `vector` and the implementation, include the `Vector.h` header file. (to ensure consistency)
### Translation Unit
A `.cpp` file that is compiled by itself (including the `h` files it `#includes`) is called a translation unit. A program can consist of thousands of translation units.
### Disadvantages of header files and `#include`
- Compilation time: If you `#include` `header.h` in 101 translation units, the text of `header.h` will be processed by the compiler 101 times.
- Order dependencies: If we `#include` `header1.h` before `header2.h` the declarations and macros in `header1.h` might affect the meaning of the code in `header2.h`. If instead you `#include` `header2.h` before `header1.h`, it is `header2.h` that might affect the code in `header1.h`.
- Inconsistencies: Defining an entity, such as a type or a function, in one file and then defining it slightly differently in another file, can lead to crashes or subtle errors. This can happen if we – accidentally or deliberately – declare an entity separately in two source files, rather than putting it in a header, or through order dependencies between different header files.
- Transitivity: All code that is needed to express a declaration in a header file must be present in that header file. This leads to massive code bloat as header files `#include` other headers and this results in the user of a header file – accidentally or deliberately – becoming dependent on such implementation details. 

***Usually pass small values by copying them and pass larger ones by sending references.***

Copying large objects on returning them might be expensive. the old way is returning a pointer of them:
```c++
Matrix∗ add(const Matrix& x, const Matrix& y) // complicated and error-prone 20th century style
{
	Matrix∗ p = new Matrix;
	// ... for all *p[i,j], *p[i,j] = x[i,j]+y[i,j] ...
	return p;
}

Matrix m1, m2;
// ...
Matrix∗ m3 = add(m1,m2); // just copy a pointer
// ...
delete m3; // easily forgotten
```
The new way of doing this is by defining `move` constructor. even if we don't do this the compiler usually *elides* copying and constructs the new matrix where it is needed to be constructed.

### Return Type Deduction
Putting `auto` as the return type of functions is easy and convenient; but, a small change to the implementation might change the interface entirely. return type deduction does not offer a solid interface.

### Suffix Return Type
We allow putting return types after the declaration of the function, it is weird but possible!
```cpp
auto mul(int i, double d) -> double { return i∗d; } // Return type is "double"
```

### Unpacking in CPP
```cpp
void incr(map<string,int>& m) // increment the value of each element of m
{
	for (auto& [key,value] : m)
		++value;
}
```
### Advice
- Distinguish between declarations (used as interfaces) and definitions (used as implementations).
- header files should only include the following items:
	- - `#include`s of other header files (possibly with include guards)
	- templates
	- class definitions
	- function declarations
	- `extern` declarations
	- `inline` function definitions
	- `constexpr` definitions
	- `const` definitions
	- `using` alias definitions

# Chapter 4 | Error Handling
If a exception occurs, the code unwinds until it gets to a point that shows interest in handling that kind of exception, in that case it keeps calling the destructors along the way.

You can declare your functions, which you believe that never throw exception, with `noexcept`; I don't know whether it helps with performance or not but if you throw exception in these places, it simply calls terminate, and doesn't even unwind back. it is not always about *not throwing exceptions* sometimes we don't want to recover, *this part of program is designed in a way that any exception is not recoverable*.

### Errors as Values
We can use errors as values when we are expecting that error to occur a lot and when we are expecting the caller to immediately handle the failure.
remember that returning values are much more cheaper than throwing exceptions.

### Assertions
You can come up with your own minimal implementation of a kind of assertion that takes place while testing and does little to non in deployment; But there are not language features for that currently.
For run time though, we have `assert`s. if the program is in debug mode, assertions are checked and they terminate code, if not in debug mode they are not checked.
We have `static_assert` too! they are checked only at compile time! so the values should be there on compile time.

# Chapter 5 | Classes
Functions defined in a class are inlined by default.
```cpp
class myClass {
  public:
    int myFunction();
}

int myClass::myFunction() {
  return 1;
}

  
// myFunction is not inline, while in this, it is:  

class myClass {
  public:
    int myFunction() { return 1; }
}
```
We can tag a function with `const` to show that they are not going to change the underlying object. calling functions that do change the object on const objects is prohibited.

create `constructor`s and `destructor`s in order to avoid naked `new` and `delete` in code. naked `new` and `delete` are bad practices.

This is how function calls are manages for derived classes.
![virtual-tables](./images/vtbl.png)

This virtual call mechanism can be made almost as efficient as the ‘‘normal function call’’
mechanism (within 25%). Its space overhead is one pointer in each object of a class with virtual
functions plus one vtbl for each such class. (if it runs more than once, compiler usually makes some optimizations)

`if (Smiley∗ p = dynamic_cast<Smiley∗>(ps)) {}` this disgusting thing is to check whether `ps` _ which is a `shape`_ a Smiley face or not.
if we use the above thingy with references, then if it is not from our type, it throws an exception.

We can use `unique pointers` in cases when we want to store pointers to our object.
then there is no need to create a destructor for that, it automatically destroys the member objects. (which are typed as `unique_ptr<object_type>`) and it is as efficient as raw pointers.

### Advice
- Use override to make overriding explicit in large class hierarchies.
- Prefer concrete classes over class hierarchies for performance-critical component.

# Chapter 6 | Essential Operations
move, copy, initialization and delete operations can be defined for a class; these are implicitly defined for each class, if you explicitly define some of these, the compiler won't generate the other ones. a good rule of thumb is to define none or all of them.
You can even instruct the compiler to not generate a definition for one of thsee:
```cpp
class Shape {
	public: Shape(const Shape&) =delete; // no copying
	Shape& operator=(const Shape&) =delete; // ...
};
	
void copy(Shape& s1, const Shape& s2)
{
	s1 = s2; // error: Shape copy is deleted
}
```
### Conversion via Constructors
A constructor taking a single argument defines a conversion from its argument type. For example, complex provides a constructor from a double:
```cpp
complex z1 = 3.14; // z1 becomes {3.14,0.0}
complex z2 = z1∗2; // z2 becomes z1*{2.0,0} == {6.28,0.0}
```
You can avoid these via:
```cpp
class Vector {
	public: explicit Vector(int s); // no implicit conversion from int to Vector // ...
};

Vector v1(7); // OK: v1 has 7 elements
Vector v2 = 7; // error: no implicit conversion from int to Vector
```

### Copying Containers
the default copy definition is not good for containers, it simply copies the pointer, now both of variables are pointing to the same object in heap. you should define it yourself.

### Explicit Move
You can be explicit about moving via `std::move()`

Compiler will elide most copies, so most of the times there is no need to define move constructors.

### User Defined Literals
You can define a literal for your custom class! for example `45i` is a complex number with 0 for real part and 45 for imaginary. define them like:
```cpp
constexpr complex<double> operator""i(long double arg) // imaginary literal
{
	return {0,arg};
}
```

### Advice
- If you define `<=>` for a type as non-default, also define `==`
- Return containers by value (relying on copy elision and move for efficiency)
- Write `std::move()` only when you need to explicitly move an object to another scope.

Why? `A class with a virtual function should have a virtual destructor;`

`If a class X has a destructor that performs a nontrivial task, such as free-store deallocation or lock release, the class is likely to need the full complement of functions:`

```c++
class X {
public:
	X(Sometype); // ‘‘ordinar y constructor’’: create an object
	X(); // default constructor
	X(const X&); // copy constructor
	X(X&&); // move constructor
	X& operator=(const X&); // copy assignment: clean up target and copy
	X& operator=(X&&); // move assignment: clean up target and move
	˜X(); // destructor: clean up
};
```

Use explicit type conversions in constructors to eliminate confusion.

# Chapter 7 | Templates

C++'s equivalent to blanket types in rust is concepts.
It sets some constraint for the templates; for example T should be copyable

### Value Template Arguments
In addition to type arguments, a template can take value arguments. For example: 
```cpp
template<typename T, int N>
struct Buffer {
	constexpr int size() { return N; }
	T elem[N];
	// ...
};
```
In the above example we have created a buffer structs which is allocated on the stack!

In c++ to write a proper string we have to add a `s` suffix to the end of the literal: `"Hello"s`. or the compiler might consider the `const *char` type for it.

You cannot use template functions on virtual functions, because the compiler doesn't know all the variations of that function to create a `vtbl` for.

### Function Objects
C++ has somehow a first-class citizen support for functions, they are called function objects:

```cpp
template<typename T>
class Less_than {
	const T val; // value to compare against
public:
	Less_than(const T& v) :val{v} { }
	bool operator()(const T& x) const { return x<val; } // call operator
};
```

### Lambda Functions
`[&](int a){ return a<x; }` this is a lambda function in c++. the `[&]` is a *capture list* it says that all local names used in the lambda expression are going to be accessed via a reference. `[=]` means all names are accessed via copy/value.

### Finally
In c++ we have to deal with a lot of old code and c style codes. We can do the following to ensure the safety of our program (tricking the compiler to use RAII):
```cpp
template <class F>
struct Final_action {
	explicit Final_action(F f) :act(f) {}
	˜Final_action() { act(); }
	F act;
};

template <class F>
[[nodiscard]] auto finally(F f)
{
return Final_action{f};
}

void old_style(int n)
{
	void∗ p = malloc(n∗sizeof(int)); // C-style
	auto act = finally([&]{free(p);}); // call the lambda upon scope exit
	// ...
} // p is implicitly freed upon scope exit
```
In the above code we are putting the required destructor functions into a finally function, that function returns a object in which we have set the destructor functions as its destructor! so when `Final_acion` goes out of scope, the necessary functions are going to be called!

### Compile-time `if`
```cpp
template<typename T>
void update(T& target)
{
	// ...
	if constexpr(is_trivially_copyable_v<T>)
		simple_and_fast(target); // for "plain old data"
	else
		slow_and_safe(target); // for more complex types
	// ...
	}
```

# Chapter 8 | Concepts and Generic Programming
### Concept-based overloading
```cpp
template<forward_iterator Iter>
void advance(Iter p, int n) // move p n elements forward
{
	while (n--)
		++p; // a forward iterator has ++, but not + or +=
}

template<random_access_iterator Iter>
void advance(Iter p, int n) // move p n elements forward
{
	p+=n; // a random-access iterator has +=
}

void user(vector<int>::iterator vip, list<string>::iterator lsp)
{
	advance(vip,10); // uses the fast advance()
	advance(lsp,10); // uses the slow advance()
}
```

### Requirement Clauses
You can specify c++ expressions in the definition of the concept/requirement itself, if the expression is valid code for a given usage of that template, it compiles, otherwise it does not compile.
*requires-expressions* are like raw assembly codes, they should not be visible in the everyday code, we should only see them in the definition of the concepts themselves.
```cpp
template<forward_iterator Iter>
	requires requires(Iter p, int i) { p[i]; p+i; } // Iter has subscripting and integer addition
void advance(Iter p, int n) // move p n elements forward
{
	p+=n;
}
```

### Defining Concepts
```cpp
template<typename T, typename T2 =T>
concept Equality_comparable =
	requires (T a, T2 b) {
		{ a == b } -> Boolean; // compare a T to a T2 with ==
		{ a != b } -> Boolean; // compare a T to a T2 with !=
		{ b == a } -> Boolean; // compare a T2 to a T with ==
		{ b != a } -> Boolean; // compare a T2 to a T with !=
	};
```
It is mostly self explanatory `T2 =T` means that if we don't provide a second template argument, `T2` will be the same as `T`

### Concepts and `auto`
`auto` parameters are kinda like very weak concepts! you can strengthen them by preceding them by a concept!
```cpp
auto twice(Arithmetic auto x) { return x+x; } // just for numbers
auto thrice(auto x) { return x+x+x; } // for anything with a +
```

> Concepts > Only types


*Good abstractions are carefully grown from concrete examples. It is not a good idea to try to ‘‘abstract’’ by trying to prepare for every conceivable need and technique; in that direction lies inelegance and code bloat*

### Variadic Templates
You have seen them in other languages, there is how you can achieve them in C++
```cpp
template<typename T>
concept Printable = requires(T t) { std::cout << t; } // just one operation!

void print()
{
	// what we do for no arguments: nothing
}

template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
	cout << head << ' '; // first, what we do for the head
	print(tail...); // then, what we do for the tail
}
```

You can eliminate the need of the empty call via a compile time if check:
```cpp
template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
	cout << head << ' ';
	if constexpr(sizeof...(tail)> 0)
		print(tail...);
}

```

You can achieve the same via *folds*

### Fold Expressions
You have *left folds* and *right folds*.
```cpp
template<Number... T>
int right_fold(T... v) // Wraps from the highest index
{
	return (v + ... + 0); // add all elements of v starting with 0
}

template<Number... T>
int left_fold(T... v) // Starts with loweset index
{
	return (0 + ... + v); // add all elements of v to 0
}
```

### Forward Argument
If you want to pass down the variadic templates to another function with low run-time overhead you can use `std::forward`
```cpp
template<concepts::InputTransport Transport>
class InputChannel {
public:
	// ...
	InputChannel(Transport::Args&&... transportArgs)
		: _transport(std::forward<TransportArgs>(transportArgs)...)
	{}
	// ...
	Transport _transport;
};
```

# Chapter 9 | Library Overview
nothing!

