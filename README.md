
# C++: Important Things to Know

## Uniform Initialization (>=C++11)
### Types of (Uniform) Initialization
#### Value Initialization:
```c++
T obj{}; // impl.: fundamental -> 0, user-defined-obj. -> default-constructor
```
#### Direct Initialization:
```c++
T obj{v}; // fundamental -> v, user-defined-obj. -> constructor
T arr[n]{}; // impl. init. of all n elements with def-constr. or 0
T arr[n]{v1,v2,..}; // also skips call of assignment-operator
```
#### Copy Initialization:
```c++
T obj = v; // param. constructor of T will be called, equivalent to: ‘T(v)’
```
- in this case, the compiler will not call the assignment-operator, but it will search for
any parameterized constructor that fits the type of `v`. (At the second expression of
`T v1, v2; v1 = v2;`, the compiler would call the assignment operator. So when there
is no declaration on the left. However, if `v1 = 1;` and there is a constructor of `T` that
accepts integers `(1)`, the constructor would be called first, and then the move
assignment.)

### Pros
- uniform syntax for the initialization of all types
- implicit initialization with empty-brackets (no uninitialized case like in: `T obj;`)
- direct initialization can be used for array types (In _C++98_ it was only possible to do it by assignment: `T arr[n] = {...};`. Since _C++11_: `T arr[n]{...}`)
- it prevents narrowing conversions (warning or error if not explicitly casting)

### Advice
- use uniform-initialization on always on user-defined-types and arrays
- on fundamental types classic initialization can be used for better readability (e.g. `int i = 0;`)
- always prefer initialization (whether it’s uniform or not) over assignment if possible
- on member functions: always use the member initializer-list to initialize members of a class
	- it will prevent the compiler to initialize members via assignment
		- e.g.: (1) `struct A{ T a; A() { a = 42; } };` vs (2) `struct A{ T a; A() : a(42) {} };`
		- (1) will call the default constructor of `T` (`T()`) and the move-assignment operator
		- (2) will directly initialize member a with the `42` (`T(42)`), no assignment operator here.

### Notes
- when using uniform-initialization with auto-deduction (e.g. `auto i{1};`) it can produce a `initializer_list<T>{1}` instead of `T = 1`. This has changed in _C++17_! But for now it’s safer to use copy-initialization together with auto-deduction: `auto i = 1;`.
- see section on [std::initializer_list](#stdinitializer_list-c11)

## std::initializer_list (>=C++11)
### Syntax
```c++
class Container {
	Container(std::initializer_list<int> v) {
	auto it = v.begin();
	while(it != v.end()) this->Add(*(it++));
	}
};
void PrintList(std::initializer_list<int> v) {
	auto it = v.begin();
	while(it != v.end()) cout << *it++ << “, “ << flush;
}
void PrintList2(std::initializer_list<int> v) {
	for(auto x : v) cout << x << “, “ << flush;
}
PrintList({1,2,3,4});
PrintList2({1,2,3,4});
```
### Notes
- allows initialization of user-defined objects in the same way as arrays are initialized
- overloading `initializer_list` constructors enables uniform_initialization on user-
defined objects
- especially useful if custom object is a container
- in case of parameterized constructors, `Obj o{a, b, c};` will already invoke the
constructor implicitly
- same as `Obj i{};` will implicitly invoke the default constructor
- `initializer_list` are mainly used for uniform initialization
- since `initializer_list` supports iterators, it will work with ranged-for loops out-of-the-box
- `initializer_list` is automatically constructed from a braced list of elements
- the type will be `auto`-deduced from the elements in list
- it’s not a true container, but has similar behaviour
- should not be used instead of a container

## Rule of Five (>=C++11)
```c++
class A{
public:
	// constructor
	A() : v{0}, d{1.5}, ptr{new int{}} { }
	// copy constructor
	A(const A& o) : v{o.v}, d{o.d}, ptr{new int{*(o.ptr)}} { }
	// move constructor
	A(A&& o) noexcept : v{std::move(o.v)}, d{std::move(o.d)}, ptr{std::move(o.ptr)}	{
		o.ptr = nullptr;
	}
	// alternative move constructor (more compact)
	A(A&& o) noexcept : v{std::move(o.v)}, d{std::move(o.d)}, ptr{std::exchange(o.ptr, nullptr)} { }
	// destructor
	~A() { delete ptr; }
	// copy-assignment operator
	A& operator=(const A& o) {
		if (this == &o) return *this; // self-assignment guard
		this->v{o.v};
		this->d{o.d};
		delete this->ptr;
		this->ptr = nullptr; // prevent double deletion, if ‘new int’ fails and throws
		this->ptr = new int{*(o.ptr)}};
		return *this;
	}
	// move-assignment operator
	A& operator=(A&& o) noexcept {
		if(this == &o) return *this; // self-assignment guard maybe not needed here
		this->v = std::move(o.v);
		this->d = std::move(o.d);
		delete this->ptr;
		this->ptr = std::move(o.ptr);
		o.ptr = nullptr;
		return *this;
	}
private:
	std::vector<int> v;
	double d;
	int* ptr;
};
```
### Notes
- In move constructor and assignment functions simply <u>move all members</u>, even if the type is fundamental or it is a pointer. There will be no performance bonus, but whenever some types will change, there will be no need to fix the move functions.
- _Core Guideline C.66_: Make move operations `noexcept`
	- e.g. `std::vector::push_back(std::move(obj))` will need the guarantee of the move operation to not throw an exception, else it will use the (slower) copy constructor. This means the move-constructor will need to be set `noexcept` explicitly.
- don’t set `noexcept` if you do something inside the move operation, that can throw!
	- better: Try to avoid throwing exceptions inside constructors and destructors

## Rule of Five and a half (copy-and-swap idiom, C11+ version)
__TODO__
[https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom?answertab=votes#tab-top](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom?answertab=votes#tab-top)

## Include Guidelines
__TODO__
[http://www.cplusplus.com/forum/articles/10627/](http://www.cplusplus.com/forum/articles/10627/)

## Move Semantics (>=C++11)
__TODO__
[https://www.youtube.com/watch?v=St0 NEU5b0o ab channel=CppCon](https://www.youtube.com/watch?v=St0MNEU5b0o&ab_channel=CppCon)
### Notes
- (lvalue) references can <u>only</u> be created from lvalues
- <u>iff</u> the reference is const, (lvalue) references can be created from rvalues
- rvalue references can <u>only</u> be created from rvalues
- rvalue references have been introduced for move semantics
- rvalue references are necessary to detect temporarys in function calls
	- therefore, move semantics will work even if RVO fails (e.g. if the decision of
which object will be returned can only be done at run time)
- lvalues can be converted to rvalues using `std::move(lvalue_var)`
	- this way move semantics can be invoked from lvalues
	- `std::move()` simply performs a `static_cast` to rvalue reference
		- <u>however</u>, be careful since this could create dangling pointers!
		- therefore, don’t use an object after it was moved
- always ensure the rule of five is properly implemented:
	- implementing the rule of five already enables big parts of move semantics
		- exp. 1: if a function returns an object by (r)value, the move assignment will be used
		- exp. 2: if an object is created from a temporary, the move constructor will be used
		- exp. 3: if a function `foo(Obj val)` requires an object `obj` as argument by value.
Then, if foo is called as `foo(std::move(obj))` the object `Obj val` will be created from
the move constructor instead of the copy constructor implicitly
- simple shallow copying instead of moving will lead to problems:
	- two objects would share the same pointers, and if one object would be destroyed
the other object would be left with dangling pointers causing UB
	- that’s why move semantics must always ensure to leave the original object in a
valid but undefined state
		- assign `nullptr` to pointers of the object from which was moved
- when it comes to automatic type deduction `&&` does not refer to rvalue-references:
	- `T&&` where `T` is a function-template type refers to a <u>forwarding reference</u>. At [template argument deduction](https://en.cppreference.com/w/cpp/language/template_argument_deduction) these special template-parameters will always preserve their value-category. `T&&` can therefore be deduced to rvalue-reference (when called by rvalue or rvalue-reference) and to lvalue-reference (when called by lvalue or lvalue-reference). These references are mainly used for [perfect forwarding](#perfect-forwarding).
	- same goes for`auto&&` since it is also a deduced type. However the main application for `auto&&` might be in combination with range-based for loops: In `for(auto&& x : vec) {..}`, the iterator from `vec` can now additionally return rvalues or rvalue-references (which wouldn't bind to non-const `auto&`).
	- see [this](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers) for more information

## Smart Pointers (>=C++11)
### std::unique_ptr
- unique_ptrs handle only one pointer
- copy semantics are `=deleted`, i.e. unique_ptr cannot be copied
- constructor is explicit
	- copy-init. is disabled (use direct-init.)
- can be moved: `foo(std::move(p));`
- use `unique_ptrs` whenever the pointer is only used by one class/object/scope
- prefer `make_unique<T>` over standard constructor
	- (I1) `std::uniqe_ptr<T> p{new T()};`
	- (I2) `std::uniqe_ptr<T> p = std::make_unique<T>();`
	- prefer (I2) since it has (slightly) better performance than (I1)
- use `std::unique_ptr<type>::reset(...)` for reassigning the pointer
- use `std::unique_ptr<type>::get()` to return the pointer
- use unique_ptrs to express that the function/class/scope is owning the pointer
- pass unique_ptrs to functions via:
	- `std::move(p)`: if the ownership should be transferred
	- by-reference or `p.get()` (by-pointer): if you want to use the pointer after the call
	- prefer by-reference over by-pointer
- use `get()` when raw pointers are required as arguments (see below)
### std::shared_ptr
- shared_ptrs can share its underlying pointer
- copy and move semantics are enabled
- a reference counter will keep track of the number of instances holding a ref.
- if ref. counter becomes 0, the memory will be freed
- constructor is explicit
	- copy-init. is disabled (use direct-init.)
- prefer declaration with `make_shared<T>()`, it has better performance ([see this](https://www.grimm-jaud.de/index.php/blog/speicher-performanz-overhead-von-smart-pointern))
- pass shared_ptr by-value or by-reference
	- by-value makes the code more readable and thread-safe
	- by-ref. is (slightly) more performant since in/decreasing ref. count is omitted
	- if performance or thread-safety is not a concern pass by-value, else pass by-ref.
- use shared_ptr if multiple instances need to access the same pointer
- use `std::shared_ptr<type>::reset(...)` for reassigning the pointer
	- calling `reset()` without an argument, this instance will release its ownership,
meaning it should not be used anymore
	- `reset()` is equivalent to setting the shared_ptr `=nullptr`
	- `reset()` will decrement the ref. count
		- in the reassignment case the ref. count will be incremented again (of course)
- use `std::shared_ptr<type>::get()` to return the pointer
### std::weak_ptr
- weak_ptr points to a resource that must not be available anymore (weak assumption)
- weak_ptr is used if a resource could be deleted before it would naturally be
- weak_ptr can only be initialized from shared_ptrs
- init. a weak_ptr will not increment the ref. count of the shared_ptr
- that’s why weak_ptr can also be used to resolve circular references
- exp. for circular references:
```c++
Class B;
Class A{ shared_ptr<B> pB; void setB(shared_ptr<int> b) { pB=b; } };
Class B{ shared_ptr<A> pA; void setA(shared_ptr<int> a) { pA=a; } };
int main() {
	shared_ptr<A> a{new A{}}; shared_ptr<B> b{new B{}};
	a->setB(b); b->setA(a);
}
```
- At the end of main-scope, the shared_ptrs `a` and `b` will be destroyed, but the ref.
count will stay 1. This is because `a` will still contain a ref. of `B` and vice versa
and the ref. count was 2 before deletion (creation of obj + copy into other object).
	- this can be resolved by declaring at least one of the members (of `A` or `B`) as
weak_ptr
	- since weak_ptr will not increment the ref. count at creation,
the associated object will be deleted and mem. freed at the end of
main-scope. The second obj. would then be deleted consequently.
- weak_ptrs can check if the underlying resource (pointer in shared_ptr) is still valid
	- `std::weak_ptr::expired()` returns true if resource is not available anymore
(meaning, the ref. count is 0)
- underlying resource can be accessed by `std::weak_ptr::lock()`
	- it returns a copy of the shared_pointer, which will increment its ref. count
- weak_ptrs can be seen as a safe handle to a shared_ptr
### General Notes
- prefer smart pointers over raw pointers
- prefer unique_ptr over shared_ptr over weak_ptr, depending on situation
- smart pointers overload typical pointer operators, like `->`, `*` and `...=nullptr`
- use raw pointers to express, that a function/class/scope is <u>not</u> owing the pointer
	- the programmer will be aware that the pointer will not be deleted there

## const
### const with value- and pointer-declaration
- `int* p;`
	- `pointer` to `int`
	- `p` can be reassigned and `*p` can be modified
- `const int* p;` or `int const* p;`
	- `pointer` to `const int`
	- `p` can be reassigned, but `*p` cannot be modified
- `int * const p;`
	- `const pointer` to `int`
	- `p` cannot be reassigned, but `*p` can be changed
- `const int* const p;` or `int const* const p;`
	- `const pointer` to `const int`
	- `p` cannot be reassigned and `*p` cannot be modified
### const with member functions
```c++
class A {
public:
	A(int v = 0) : val{v} {}
	int const_func() const { return val; }
	int nonconst_func() { return val; }
private:
	int val;
};
A a1{1};
const A a2{2};
a1.const_func(); // returns 1
a1.nonconst_func(); // returns 1
a2.const_func(); // returns 2
a2.nonconst_func(); // error: non-const functions can only be called by non-const objects
```
### Notes
- read from left to right, e.g.: `int const* const p` -> const pointer to const int
- a const variable or const pointer always need an initializer
- in const member functions, the `this`-object cannot be modified (it’s const)
- for const member functions, the `const` qualifier is required at declaration and definition
- non-const objects can call const- and non-const member functions
- const objects can invoke <u>only</u> const member functions

## static
- only one copy of each static variable exists
- a static member (function) can be called without having an instance of the class
	- `Class::static_member = …;`
	- `Class::static_func(...);`
- static members need to be initialized <u>outside</u> of the class
- static member functions can <u>only</u> access static members and call static member functions

## volatile
__TODO__

## constexpr
### Syntax
```c++
constexpr int i = 42;
constexpr int foo() { return 42; }
constexpr int Add(int x, int y) { return x + y; }
constexpr …
int x = foo(); // will be evaluated at runtime because x is not const
const int y = foo(); // evaluated at compile time
constexpr int y = foo(); // evaluated at compile time
const/constexpr int sum = Add(3,5); // evaluated at compile time
int sum2 = Add(3,5); // evaluated at run time
const/constexpr int sum3 = Add(x,5);//eval. at run time: x might not be known at compile time
```
### Notes
- makes an expression evaluated at compile time
	- therefore the expression will automatically be const, since changes at run time
could not be evaluated at compile time
	- compiler decides if expression will finally be evaluated at compile time
- can increase performance
- but the value computed in the constexpr must be known at compile time, else it will be
evaluated at run time
- like with inline declaration, constexpr must be defined in header file
	- the compiler needs to see the implementation of constexpr, else it will not be
evaluated at compile time
	- actually, all constexpr are defined to be implicitly inline
- can be applied to variable and function declarations
- constexpr functions only accepts and return literal types:
	- void-types, scalar-types, references, classes with constexpr constructors
	- <u>any value involved in constexpr needs to be known at compile time</u>
- before _C++14_ constexpr functions could only consist of one line (return line)
	- with _C++14_ it was relaxed
- [constexpr rules](https://en.cppreference.com/w/cpp/language/constant_expression)
### constexpr vs. inline
- inline will substitute the functions body at calling location
	- avoids function call overhead
	- but function-body will still be evaluated at run time
- constexpr will be evaluated at compile time
	- potentially even more speedup than with inline
	- but constexpr are limited in size, so there are still reasons for inline functions
### constexpr vs. const
- initialization of const variable can be deferred until run time
- constexpr must be initialized at compile time
- all constexpr variables are const, but not vice versa
- use const to indicate that the value cannot be modified
- use constexpr to demand evaluation at compile time
	- use constexpr for optimization purposes only

## C++ Types (>=C++11)
### Object types
- Scalars
	- arithmetic (integral, float)
	- pointers: `T*` for any type `T`
	- enum
	- pointer-to-member
	- `nullptr_t`
- Arrays: `T[]` or `T[N]` for any complete, non-reference type `T`
	- Classes: `class Foo` or `struct Bar`
	- Classes: `class Foo` or `struct Bar`
	- Trivial classes
	- Aggregates
	- POD classes
	- (etc. etc.)
- Unions: `union Zip`
### References types
- `T&`, `T&&` for any object or free-function type `T`
### Function types
- Free functions: `R foo(Arg1, Arg2, ...)`
- Member functions: `R T::foo(Arg1, Arg2, ...)`
### void

## explicit
- if a member function is declared `explicit`, it will disable implicit conversions and copy-
initialization
	- i.e. each time the `explicit` declared constructor would be called implicitly, the
compiler will disallow this
	- exp. 1: if an object would be created from a type which does not fit any constructor,
the compiler will not cast implicitly but emit an error
	- exp2: `T t; t = 1;` would not work if the constructor `T(int)` is declared explicit

## Pointers
- pointers of specific types (e.g., `int *p = &var;`) can only point to that specific type (here: `int`)
- void-pointer (e.g., `void *p = &var;`) can point to any type
	- dereferencing void-pointer: `int i = 0; void *p = &i; int val_of_i = *(int*)p;`
	- try to avoid void-pointers if possible (e.g. by using templates)
- always initialize pointers to avoid UB and prevent compilation errors on some compilers
- use `nullptr` over `NULL`
	- `NULL` is a macro which is defined as 0 by most compilers (`#define NULL (void*)0`)
	- `NULL` is C compatible, `nullptr` is not
	- `nullptr` was introduced with _C++11_ and is type-safe:
		- `NULL` as `(void*)0` is implicitly convertible and comparable to integral types.
This can lead to errors like [here](https://www.geeksforgeeks.org/understanding-nullptr-c/#:~:text=nullptr%20is%20a%20keyword%20that,or%20comparable%20to%20integral%20types.)
		- `nullptr` is a special keyword in _C++11_, hence the compiler will not confuse `nullptr` with the number-literal `0`
	- `nullptr` can be casted to `bool`
	- using uniform-initialization on pointers: `int *p{};` is equal to: `int *p = nullptr;`
- of course: never dereference a null-pointer (whether it was set with `nullptr` or `NULL`)

## References
- a reference will not allocate new memory (unlike pointers), instead they are directly bound to another variable in memory (alias). That’s why a reference:
	- always needs to be initialized (error at `T &ref;`)
	- can only be initialized by lvalues (except for `const T&`)
	- cannot be reassigned
	- cannot be `NULL`/`nullptr`

## Casts
### Notes
- `static_cast<new_type> var;`
	- does not allow to cast between pointers of unrelated types (e.g. `int*` -> `char*`)
	- prefer `static_cast` over `reinterpret_cast`, to cast from and to `void*` (see [here](https://stackoverflow.com/questions/573294/when-to-use-reinterpret-cast))
	- cannot cast away constness and volatility
	- resolved at compile time
- `dynamic_cast<new_type> var;`
	- resolved at run time
	- useful to cast polymorphic classes (e.g. in combination with virtual classes)
		- virtual classes are required if casting from base->derived is required
		- casting from derived->base works always
	- prefer `static_cast` over `reinterpret_cast`, to cast from and to `void*` (see [here](https://stackoverflow.com/questions/573294/when-to-use-reinterpret-cast))
- `reinterpret_cast<new_type> var;`
	- resolved at compile time
	- does allow to cast between pointers of different types
	- cannot cast away constness and volatility
- `const_cast<new_type> var;`
	- converts a const type into an non-const type, not vice versa.
	- be careful, the casted object can then be manipulated
- `(new_type) var;` (C-Style)
	- The C++ compiler will interpret this as one of the above casts, depending on the
object that is to be casted. Therefore, it is not directly visible to the programmer
which cast will be invoked. See the [official standard](https://en.cppreference.com/w/cpp/language/explicit_cast) on C-Style casts.
	- C-Style casts do also not check if the cast is valid
	- C-Style casts will cast away constness and volatility
		- => don’t use C-Style casts in C++
- C++-Style casts will emit errors on illegal cast attempts (not granted for C-Style casts)
- C++-Style casts are better visible and allow for efficient search for casts in the code
- <u>general rule</u>: avoid casting at all if possible
- convert user-defined-types to primitive types by implementing the type-conversion operator
	- e.g. `operator int();` (see section [Overloading Special Operators](#overloading-special-operators))
	- Syntax: `operator <type>()`
		- no arguments
		- no return type
		- operator can be declared `explicit`, this way you explicitly disallow the
compiler to implicitly cast to this type. Explicitly casting (e.g. using
`static_cast`) will however still work

## Enum
### Syntax
```c++
enum Color{RED, GREEN, BLUE}; // RED==0, GREEN==1, BLUE==2
enum Color{RED=1, GREEN=3, BLUE=5}; // RED==1, GREEN==3, BLUE==5
enum Color : char{RED=’r’, GREEN=’g’, BLUE=’b’}; // RED==’r’, GREEN==’g’, BLUE==’b’
```
### Notes
- enumerators are internally represented as <u>undefined integral types</u> (if not further specified)
	- assigning a new value to an enumerator requires static_cast
		- `Color c{RED}; c = static_cast<Color>(2);`
		- that way passing an integer to a function that requires a enum type will fail
		- hence, enums are called symbolic constants -> they restrict to a range
	- when assigning an enumerator to an integer the compiler will implicitly add the static_cast
- enums are bound to the scope they were declared
- prefer using scoped-enums (see below) over traditional enums

## Scoped-Enum (>=C++11)
### Syntax
```c++
/* (Ex1) Color::RED==0, Color::GREEN==1, Color::BLUE==2 */
enum class Color{RED, GREEN, BLUE};
/* (Ex2) Color::RED==1, Color::GREEN==2, Color::BLUE==3 */
enum class Color{RED=1, GREEN=3, BLUE=5};
```
### Notes
- this allows to define multiple enums sharing (at least partially) the same names in the same
scope
- create them by adding the `class` keyword
	- this way the enum will be created inside its own class-scope
- on scoped-enums, assigning a enum-val. to an integer will no longer be implicitly casted
	- `int c = static_cast<int>{Color::RED};`
- prefer using scoped-enums over traditional enums (see above)
- scoped-enums use int as underlying type always
### Example
```c++
enum class Color{RED, GREEN, BLUE};
enum class RGB{RED=1, GREEN=2, BLUE=3};
Color::RED; // ==0
RGB::RED; // ==1
```

## Automatic Type Inference (>=C++11)
### Syntax
```c++
auto <identifier> = <initializer>
```
### Notes
- data-type will be inferred from the initializer value
	- initializer is required
- however, use it rarely since the explicit types usually make the code easier to understand
- use it to avoid long, fully qualified names (e.g. `std::vector<T>::iterator`)
- avoid expressions like `auto i{};’` It can yield unwanted/unexpected results

## Special Literals
- special literals are suffixes appended to literal-values which tell the compiler the data-type to use
### Examples
```c++
10; // int
10l; // long
10u; // unsigned int
1.2; // double
1.2f; // float
L”...”; // wide string (string of wide-chars)
```

## User-Defined Special Literals (>=C++11)
### Syntax
```c++
<return_type> operator””_<literal>(<arguments>)
```
### Notes
- custom literals make the code more readable and safe
- custom literals are defined as operators
	- they can only be defined as global functions, they <u>cannot</u> be member functions
- custom literals need to start with `_`
- built-in special literals cannot be redefined
- return type can be any type (incl. `void`)
- for the arguments, use the largest possible type that can hold the desired values
	- integer types: `unsigned long long`
	- floating types: `long double`
	- character types: `char`, `wchar_t`, `wchar16_t`, `wchar32_t` (depending on what char
should represent)
	- string: `const char*` (compatible with `std::string` and C-strings)
	- these are also the only types that can be suffixed by custom literals
### Example
```c++
struct Distance{ long double dist_km; ... }; // class that holds a distance in kilometer
Distance operator"" _mi(long double val) { return Distance(val * 1.6); }
Distance dist{ 50.0_mi }; // the custom-literal wilk convert the 50 miles to 80 km
cout << dist.dist_km; // ==80.0
```

## Range-Based For Loops (>=C++11)
### Syntax
```c++
for(variable declaration : range) { statement; }
```
### Example
```c++
int arr[]{1,2,3};
for(int x : arr) { /* x will be 1, 2, 3 */ }
for(auto x : {4,5,6}) { /* x will be 4, 5, 6 */ }
for(auto&& x :
```
### Notes
- prefer ranged for-loops over regular for-loops when:
	- no index is required (ranged-for cannot exceed the range)
	- only one pointer
- declaring a reference, like in `for(int &x : arr) {...}` will:
	- prevent copying the next element into `x` at each iteration
	- allow to write into `x` (without `&` you can only read)
- for read only: prefer to use `for(const int &x : arr)`
- for read/write: prefer to use `for(int &x : arr)`
- can be used with `initializer_list`: `for(const int &x : {1,2,3,4})`
- if user-defined classes should support ranged-for loops it needs to provide iterators

## std::begin and std::end (>=C++11)
```c++
int arr[]{1,2,3}
```
- `std::begin(arr)` will return `&arr[0]` (address of first element in arr)
- `std::end(arr)` will return `&arr[3]` (address of last element + 1 in arr)
- can be used for iteration: `int *beg = std::begin(arr); while(beg != std::end(arr)) {beg++; ...}`
- they are both defined in almost all STL headers (e.g. iostream and all STL-containers)

## std::string and std::stringstream
### Notes
- advantage over C-style strings: `std::strings` caches the length of the string
	- `std::string::length()` and `std::string::size()` are O(1)
	- `strlen(cStr)` is O(n)
- use `std::string::c_str()` to get the C-style representation of the string
- use `std::stringstream` when writing to or reading from a string buffer
- use `std::to_string()` to convert any primitive type into a string
	- `std::string s = std::to_string(prim_type_var);`
	- internally it uses `std::stringstream` like below:
		- `std::stringstream ss; ss << prim_type_var; std::string s{ss.str()};`
		- `std::stringstream` overloads the `<<`-operator for all prim. types
- use `std::stoi()` to convert from string to int, or `std::stof()` to convert from string to float
	- there are many functions of this kind implemented, so look them up when needed
### parse comma-sep.-file
```c++
std::string data{“12,14,42”};
std::istringstream iss{data};
std::ostringstream oss;
std::string token;
while(std::getline(iss, token, ’,’)) { oss << token << “\n”; }
```
### remove whitespace
```c++
#include <algorithm>
std::string data{“12,14 , 13, 123”};
data.erase(std::remove(data.begin(), data.end(), ‘ ‘), data.end());
```

## inline
- declare function as `inline` to request the compiler to replace the function call with the function body. This will avoid the function call overhead (copying of arguments and PC addresses)
- use `inline` on functions that are:
	- small (i.e. where the overhead time exceeds the computation time)
	- frequently called (else the effect will not be realized)
- always directly define `inline` functions and do this in the header file (to avoid unresolved external linker error)
	- the compiler need to see the definition of the `inline` function whenever it finds a call
- inlining functions could be part of compiler optimizations (even if not explicitly declared)
- use `inline` functions over macros (macros are error prone)

## Function Pointers
### Syntax
```c++
<ret> Function(args) {...} // definition of Function
<ret> (*fnptr)(args) = &Function // get function pointer from Function
// concrete example
float Add(int a, float b) {...}
float (*ptr)(int, float) = &Add; // ‘&’ before function name is optional
// invocations
(*ptr)(10, 12.2f);
ptr(10,12.2f);
```

## 2D Arrays on Heap
### Example
```c++
std::size_t rows = 2, cols = 3;
int **pArr = new int *[rows];
for(std::size_t i=0; i<rows; ++i)
pArr[i] = new int[cols];
```

## Class Acces-Modifiers
### Syntax
```c++
class <name> : <access modifier> <base class> { … };
```
### Notes
- `class A : public B {};`:
	- all public and protected members of `B` are inherited with same modifiers
- `class A : private B {};`:
	- all public and protected members of B are inherited private
will not be accessible in A
- `class A : protected B {};`:
	- all public and protected members of B are inherited protected
- default access-modifiers:
	- `class A : B { …};` <=> `class A : private B { … };`
	- `struct A : B { …};` <=> `struct A : private B { … };`
	- independent of whether `B` is class or struct
- multiple inheritance possible: `class A : public B : private C { ... };`

## Struct and Class
- only difference to class:
	- Default member access of `struct` is public
	- Default member access of `class` is private
- same as classes, structs can deal with visibility modifiers, virtual and abstract functions,
and inheritance and polymorphism
- functions of base class can be called with `::` operator: `Base::foo();`
- constructors, destructor and assignment-operator are not inherited, you need to provide the
implementation
	- use constructor delegation `A::A(...) : B(...) {}`
	- or force compiler to inherit constructors (_C++11_): `class A : B { public: using B::B; .. };`

## Abstract classes
### Example
```c++
struct B { virtual void foo() = 0; };
struct A : B { void foo() { … } };
B b1; // compile error -> abstract classes cannot be instantiated
A a;
B &b2 = a; // works
```
### Notes
- make a function abstract by declaring it pure virtual: `<u>virtual</u> void foo() <u>= 0</u>;`
- if at least one function is declared pure virtual (abstract), the class will become an
abstract class
- abstract classes cannot be instantiated, but need to be inherited
	- all abstract functions need to be overwritten
- abstract classes work as an interface. It forces the programmer to implement the
given functions (interface)

## Diamond Inheritance
### Example
```c++
class Parent { ... };
class Child1 : virtual Parent { … }; // <- inherit with virtual to force the compile to only
class Child2 : virtual Parent { … }; // <- keep one instance of Parent
class Diamond : Child1, Child2 { … };
```
### Notes
- Problem: Class `Diamond` has two instances of `Parent`. This can lead to ambiguous
definitions
- Solution: All classes that inherit directly from `Parent` (here: Child1, Child2) need to
inherit using the `virtual` specifier
	- it forces the compiler to keep only one instance of the common base class (here: `Parent`)
	- if the common base class does not provide a default constructor, you must
invoke the param. constructor manually via constructor-delegation
	- `Diamond::Diamond() : Child1(...), Child2(...), <u>Parent(...)</u> { … }`
	- Reason: Only one instance of Parent will be created, this will happen at the latest
point of inheritance. So at this point (here at constructing Diamond), the common
base class constructor (here Parent) need to be called
- virtual inheritance internally adds a pointer to the base class instance (vptr) to all classes
that virtual inherit from the base class

## Vtable & Vptr
### Note
- As soon as a member-function inside a class/struct is declared `virtual’` the compiler will
add to the class a virtual table (Vtable) which holds all the addresses of the virtual
functions
	- i.e. heap memory will be used
- additionally the virtual pointer (Vptr) is added as member to the class, which holds the
address of the first function in the Vtable
- if a derived class inherits from a virtual class, a new Vptr and Vtable will be allocated for
each virtual function in the base class, even if not all of them are overwritten
- a virtual function call yields (about) twice the amount of assembler commands than
a non-virtual function call. So for really run-time critical applications this might be
considered
- if members are declared virtual, don’t forget to declare the destructor virtual too!
	- if a base class pointer is deleted that was used in run-time polymorphic
scenario it will call the destructor of the base class only and not the the
destructor of the derived class, unless the base class destructor is
declared virtual

## final (>=C++11)
### Example
```c++
class A final { … };
```
### Note
- a class declared final cannot be inherited
- all attempts in inheriting the class will result in a compiler error
- it is useful to prevent memory leaks when the class is used in a runtime
polymorphic scenario. E.g. the class may not provide a virtual destructor. If you
would inherit the class and then delete the object via a pointer to the base class
the destructor of the derived class would not be called. If the base class have been
final, you wouldn’t even have the chance to inherit from it
- also functions can be specified ‘final’:
```c++
	struct B { virtual void foo(float f); };
struct A : B { void foo(int i) override final; }
struct C : A { void foo(int i) override; } // <- results in compile error
```
- if you don’t want a function to be further overwritten, declare it `final`

## override (>=C++11)
### Example
```c++
struct B { virtual void foo(float f); };
struct A : B { void foo(int i); };
struct C : B { void foo(int i) override; }; // correct would be: void foo(float f) override;
A a;
B &b1 = a;
b1.foo(1.2f) // <- will call foo() from base class B -> foo() in A has other signature so it will
// simply be overloaded but not resolved virtually
C c;
B &b2 = c;
b2.foo(1) // <- would call foo() of C at runtime using the vtable -> but since the signature
// (float -> int) is wrong the compiler will give an error here
```
### Notes
- always use `override` if you know this function overwrites a virtual function
	- it prevents bugs that would else be hard to detect!

## Object Slicing
### Example
```c++
struct B { ... };
struct A : B { ... };
A a;
B b = a; // <- object slicing!
```
### Note
- when a child class object is directly (by value) assigned to a base class object
  the object will be sliced down to the size of the base class object
	- if the size of the child class is bigger than the base class, some part of the
memory would become overwritten which would lead to memory corruption
	- instead the compiler will slice down the size of the object so it fits the size
of the base class
	- in particular, all the attributes that have been added by the child class will be
sliced away
- when you want to call a function of a derived class via a base class instance, use a
pointer or a reference to the base class (<u>no</u> pure object)

## typeinfo (RTTI)
### Example
```c++
#include <typeinfo>
int i{};
const std::type_info &ti = typeid(i);
ti.name(); // will output the typename
```
### Notes
- RTTI == runtime type info
- use it e.g. for save downcasting:
```c++
if(typeid(*ptrBase) == typeid(MyClass)) { ... }
```
- alternatively use `dynamic_cast`:
```c++
MyClass *p = dynamic_cast<MyClass*>(ptrBase);
if (p != nullptr) { … }
```
- however, `dynamic_cast` has usually worse performance since it needs to scan
the class hierarchy at runtime
	- -> prefer `typeid` over `dynamic_cast`
- `typeid` will be resolved at runtime for polymorphic types and at compile time for non-
polymorphic types
- ideally avoid the use of `typeid` (and `dynamic_cast`) if possible
	- the compiler has to add additional extra information to the polymorphic classes
(`typeinfo` object). This object is added alongside the Vtable and is used to resolve
the types at runtime.
	- only use it on polymorphic types <u>and</u> only if necessary to <u>avoid overhead</u>
	- <u>better</u>: adapt your design exploiting polymorphism
- `typeid(*p).name()` will only output the name of the pointed class, if that is polymorphic

## Constructor Delegation
### Syntax
```c++
class A {
public:
	A() : A(0) { }
	A(int i) : member(i) { }
private:
	int member;
};
```

## Post- vs. Pre-Increment
### pre-increment, ++a
```c++
T& class::operator++() {
	this->val++;
	return *this;
}
```
### post-increment, a++:
```c++
T class::operator++(int) {
	T tmp(*this);
	this->val++;
	return tmp;
}
```
### Notes
- <u>prefer pre-increment</u> over post-increment copy
	- post-increment requires creation of temporary
	- this could also be optimized by compiler, but using pre-increment you can be always sure

## Overloading Special Operators
### Syntax
```c++
Class A {
private:
	int i{};
	int *p{};
public:
...
/* omit rule of five */
…
	A& operator++(); // pre-increment, see section post- vs. pre-increment
	A operator++(int); // post-increment, see section post- vs. pre-increment
	int operator+(const A &o) const { return this->i + o.i; } // plus-operator
	friend int operator+(int a, const A &b) const { return a + b.i; } // global plus-operator
	bool operator==(const A &o) const { return this->i == o.i; } // compare-operator
	void operator()() const { std::cout << this->i << std::endl; } // call-operator
	friend std::istream& opeartor>>(std::istream &input, A &o) // istream-operator
	{
		int x; input >> x;
		a.i = x;
		return input;
	}
	friend std::ostream& opeartor<<(std::ostream &output, A &o) // ostream-operator
	{
		out << o.i;
		return output;
	}
	// special operators required for e.g. smar-pointers
	A* operator->() { return this->p; } // arrow-op.: Just an example, use it with ptr-resource!
	A& operator*() { return *this->p; } // deref.-op.: Just an example, use it with ptr-resource!
	// special operators for type conversion into primitive types
	operator int() { return this->i; }
	operator string() { return }
};
```
### Notes
- plus-operator (`operator+(const A &o) const;`) allows for: `A a = A(1)+A(2); A b = A(1) + 2;`
	- `A c = 1 + A(2);` would not work since `int::operator+(const A &o);` is not defined
	- to resolve, global overload operator+: `friend int operator+(int a, const A &b) const;`
- call-operator (operator()) allows any number of arguments
- the <</>>-operators are used to write to i/ostream here. Since in `std::cout << A();`, cout is
on the right, `std::ostream::operator<<(ostream, other)` will be invoked. Therefore, we need
to overload these operators globally
- the `friend` statement, that is used for some operators,
	- makes the function not being part of class A (hence, it’s global)
	- allows the function to access private members of class A
	- -> this way we can global define a function within the class (better readability)
- <u>Rule</u>: if first operand is primitive type or e.g. defined in STL => global overload required
- operators that are not allowed to be overloaded: `.`, `?:`, `.*`, `sizeof`, `#`, casting operators
- custom operators are <u>not</u> supported

## The "Resource Acquisition is Initialization" (RAII) Idiom
__TODO__
[https://en.cppreference.com/w/cpp/language/raii#:~:text=Resource%20Acquisition%20Is%2](https://en.cppreference.com/w/cpp/language/raii#:~:text=Resource%20Acquisition%20Is%20Initialization%20or,in%20limited%20supply)%20to%20the)
[0Initialization%20or,in%20limited%20supply)%20to%20the](https://en.cppreference.com/w/cpp/language/raii#:~:text=Resource%20Acquisition%20Is%20Initialization%20or,in%20limited%20supply)%20to%20the)

## Stack Unwinding
### Examples
```c++
void foo() { int *p = new int[10]; Obj o; throw std::runtime_error(“Error”); delete[] p; }
foo(); // throws an exception. ‘o’ will be destroyed properly same as ‘p’, but the memory
// on the heap allocated by ‘p’ will not be freed
```
### Notes
- If an exception is thrown, the function stack will be destroyed properly
	- i.e. all stack-objects will be destroyed by destructor call
- however, the <u>heap memory will not be freed</u>
- here is where a main benefit of smart-pointers originates in:
	- all <u>stack</u> resources will be destroyed after an exception
	- also local pointers will be destroyed but probably leaving
the heap-memory unreferenced and create memory leaks
	- -> see [RAII](#the-resource-acquisition-is-initialization-raii-idiom) and [smart-pointers](#smart-pointers-c11)
- if an exception is thrown in a constructor, again all <u>stack-objects will be destroyed</u>,
but the heap <u>will no be freed</u> and the <u>destructor will not be called</u>
	- avoid allocation heap-memory in a constructor, instead use smart-pointers or
containers, since they will free the memory if an exception is thrown (since
they are stack-objects and their destructor will be called)
- destructors should <u>never</u> throw an exception
	- destructors are called during stack unwinding, and if a exception is thrown during
stack unwinding, the program will immediately terminate
	- if you cannot avoid throwing an exception in a destructor, than handle the exception
inside the destructor

## noexcept (>=C++11)
### Notes
- can be applied to functions in declaration and definition
- indicates the compiler that no exception will be thrown inside this function
- it enables the more optimization possibilities for the compiler
	- no stack unwinding code overhead needs to be produced
- if a function declared as noexcept will throw an exception the program will terminate
immediately
	- it is unspecified if the stack will be unwinded or not
- `noexcept` ⇔ `noexcept(true)`
- not specifying `noexcept` ⇔ `noexcept(false)`: these functions are called <u>exception neutral</u>
- `noexcept` also works as an operator that tells you whether a function will throw or not
	- `noexcept(foo());`: returns a boolean, true if `foo()` does not throw, false else
	- a function can be marked like this: `void foo2() noexcept(noexcept(foo()));`
- mark all functions which give a <u>strong guarantee that no exception will be thrown</u> as `noexcept(true)`
	- check if functions from libraries will throw before specifying the calling function `noexcept`
- mark move constructors and operators with `noexcept` (if possible) for better optimization
	- e.g. some STL containers will only use move semantics if they are specified `noexcept(true)`
	- implicit move constructors/operators will always be `noexcept(true)`
- destructors should also marked `noexcept` to force the programmer avoid throwing
exceptions in the destructor
	- since _C++11_ all destructors are implicitly marked `noexcept(true)`

## extern
### Examples
```c++
extern int i; // declares that there is a variable named i of type int, defined somewhere else in the program
extern int j = 0; // defines a variable j with external linkage; the extern keyword is redundant here
extern void f(); // declares that there is a function f defined somewhere in the program. extern is redundant, but sometimes considered good style.
extern void f() {;} // defines the function f() declared above. again, the extern keyword is technically redundant here as external linkage is default
extern const int k = 1; // defines a constant int k with value 1 and external linkage. extern is required because const variables have internal linkage by default!
```
### Notes
- The `extern` keyword tells the compiler that a variable is defined in another source module (outside of the current scope).
  The linker then finds this actual declaration and sets up the `extern` variable to point to the correct location
- Variables described by `extern` statements will <u>not</u> have any space allocated for them, as they should be properly defined elsewhere
- If a variable is declared `extern`, and the linker finds <u>no</u> actual declaration of it, it will throw an `"Unresolved external symbol"` error
- `extern` statements are frequently used to allow data to span the scope of multiple files
- When applied to function declarations, the additional `"C"` or `"C++"` string literal will change name mangling when compiling under the opposite language
	- That is, `extern "C" int plain_c_func(int param);` allows C++ code to execute a C library function `plain_c_func`

## Union
### Example
```c++
union U {
	int x;
	char c;
	U() ; x{0}, c{'c'} {} // compile error: only one member can be init. at once
	U() : x{0} {} // OK
};
U u; // member c is initialized
u.x = 1000; // now c is uninitialized and x is initialized
cout << u.c; // UB! in this case the contents of u.x will be interpreted as char
cout << size
```
### Notes
- unions are user-defined objects (like struct and class) where all members share the same
memory space
	- unions always use memory of the size of its biggest member
	- only one member can be initialized at a time
	- it is impossible to get the information from the union which is the current active
member, the programmer needs to keep track manually
		- in example above int is 4 bytes and char 1 byte, therefore the union will
have 4 bytes of memory allocated. But if used with `sizeof(u)`, the size
of the current active member is shown. This can be used to check for active
members.
- use it to save memory (e.g. in embedded systems)
- the members are public by default (like in struct)
- unions can hold user-defined objects, but the union must explicitly implement these
constructors and destructors inside the union
	- in that case you have to implement the placement- new operator
		- `/*u.s is uninitialized std::string*/ new(&u.s) std::string{“hello”};`
	- user-defined members in unions are not destroyed implicitly, so destroy them
manually
	- `/*u.s is std::string*/ u.s.~std::string();`
- unions cannot be inherited or derived, and unions can also not have a base class
	- consequently virtual functions are not allowed

## variant
__TODO__

## Variable Length Arrays (VLA)
### Example
```c++
int foo() { return 42; }
int arr1[100]; // allowed in C++, because size is known at compile time
int arr2[foo()]; // not allowed in C++, because size might not be known at compile time
int *pArr = new int[foo()]; // allowed in C++, because memory is allocated on heap
```
### Notes
- VLAs are not allowed in C++ according to the ISO standard
	- reason: the memory for VLAs is locally allocated on the stack. The stack,
however, has only limited memory available. Therefore, the compiler must know
the size of memory at compile time. Else, at run time a function foo might return
a huge value which could then overflow the stack.
- VLAs are allowed in _C99_ (C Standard), that’s why some C++ compilers (like GCC/G++)
allow them, too
	- but be careful with usage!

## Raw String Literals (>=C++11)
### Examples
```c++
std::string path1{"C:\\path\\to\\file.txt"}; // classis strings: '\' are interpreted as escape symbol -> '\\' is interpreted as '\'-ASCII
std::string path2{R"(C:\path\to\file.txt)"}; // raw string literal: All escape sequences are ignored (since C++11)
std::string path3{R"DELIM(C:\path\to\file"(name)".txt)DELIM"}; // raw string literal with custom delimiter DELIM: Allows for embedding '"'
```
### Notes
- `R"(...)"`: All symbols inside brackets are interpreted as ASCII
	- no escape sequences will be provided
- useful when processing paths, XML, HTML, regular expressions, etc.
- can be used with custom delimiters  (see example above)

## Filesystem Library (>=C++17)
### Examples
```c++
#include <filesystem>
std::filesystem::path p{"/home/user/file.txt"};
p.has_filename(); // returns bool if path ends on a filename
p.filename(); // returns string with the filename if existing
for(const auto &x : p)	{ /* x would is: home, user, file.txt */ }
std::filesystem::directory_iterator beg{path{"/home/user"}}; // '/home/user' is a directory
std::filesystem::directory_iterator end{};
while (beg != end) { /* *beg will iterate over each file and sub-directory in /home/user */ }
std::filesystem::current_path(); // returns path object of current path
std::filesystem::path src{std::filesystem::current_path()};
src /= "file.txt"; // before: src=='/home/user', after: src=='/home/user/file.txt'
```
### Notes
- the `path` class will <u>not</u> check for the validity of the path
- `fstream`

## fstream
### Basic Examples
```c++
std::ifstream input{"file.txt"};
std::ofstream output{"other.txt"};
input.is_open(); // returns true if file is valid and loaded
if (!input) { /* file couldn't be opened */ } // !-operator is overloaded
input.good(); // returns true if all I/O operations were succesfull
input.fail(); // returns true if some I/O operation failed
input.eof(); // returns true if reached end of file
input.bad(); // returns true if stream received irrecoverable errors
input.clear(); // resets stream state flags
input.setstate(std::ios::failbit); // will explicitly set the failbit
std::string line;
while (!std::getline(input, line).eof()) { /* iterate file.txt line by line */}
char ch{};
while (input.get(ch)) { /* iterate file char by char */ }
ch = 'a'; output.put(c); // put char in text file
input.tellg(); // returns current position in input stream (type: size_t)
output.tellp(); // returns current position in output stream (type: size_t)
input.seekg(10); // set input stream position to 10 bytes from current position
output.seekp(10); // set output stream position to 10 bytes from current position
input.close(); // close the file stream (is also done by the destructor)
output.close(); // close the file stream (is also done by the destructor)
```
### Serialization Example
```c++
struct Record { int id; char name[10]; /* etc. */ }; // object to be serialized
void serialize(const Record& rec, std::filesystem::path& path) {
	std::ofstream f{path, std::ios::binary | std::ios::out};
	f.write((const char*)&rec, sizeof(Record));
}
void deserialize(Record &rec, std::filesystem::path& path) {
	std::ifstream f{path, std::ios::binary | std::ios::in};
	f.read((char*)&rec, sizeof(Record));
}
```
### Notes
- streams will be closes in the destructor (at the end of it lifetime)
	- no explicit `.close()` required, but should be preferred
- `tellg()`: tells the position of the get-pointer
	- get-pointer is pointer to current position in input-stream
- `tellp()`: tells the position of the put-pointer
	- put-pointer is pointer to current position in output-stream
- `seekg(n, std::ios::beg)`:  set stream position `n` bytes from begin of file
- `seekg(n, std::ios::cur)`: set stream position `n` bytes from current position (same `seekg(n);`)
- `seekg(-n, std::ios::end)`: set stream position `-n` bytes from end of file (backward)
	- on `ofstream` these functions are called `seekp(...)`. But they behave the same way
- stream state flags:

| Flag    | Meaning                          | Function                   |
| ------- |:--------------------------------:| --------------------------:|
| goodbit | no error                         | `bool good()`              |
| badbit  | irrecoverable stream error       | `bool bad()`               |
| failbit | I/O operation failed             | `bool fail() [operator !]` |
| eofbit  | end of file reached during input | `bool eof()`               |

- stream modes (to be specified when opening a file)

| Mode   | Meaning                                     |
| ------ |:-------------------------------------------:|
| app    | seek to the end before each write operation |
| binary | open in binary mode                         |
| in     | open for reading (default for `ifstream`    |
| out    | open for writing (default for `ofstream`    |
| trunc  | discard file contents before opening        |
| ate    | seek to end after open                      |

## Template Functions
### Examples
```c++
template<typename T>
T foo(T x) { cout << "type: " << typeid(x).name << endl; return x; }
/* auto deduction */
auto r1 = foo(1.2f); // "type: f", r2 is float
auto r2 = foo(12); // "type: i", r2 is int
```
### Notes
- `templates` are function (or class) blueprints for the use with arbitrary datatypes
- an implementation of the `template` will be added by the compiler each time it is used with a new datatype. This process is called template instantiation
	- this happens for each file the `template` will be invoked from
	- this happens implicitly,
		- when a function template is invoked
		- when an address to a function template is used
		- at explicit instantiation
		- when an explicit specialization is created
- if the `template` is only defined but not used, the compiler will not add a implementation at all to the machine code
- `templates` are instantiated at compile-time

## Template Classes
### Examples
```c++
template<typename T>
class Stack {
    T m_buffer[512];
    int m_top{-1};
public:
    Stack() = default;
    Stack(const T& e) { this->Push(e); }

    void Push(const T& e) { m_buffer[++m_top] = e; }
    void Pop() { --m_top; }
    const T& Top() const { return m_buffer[m_top]; }
    bool isEmpty() const { return m_top == -1; }
};
/* ... */
Stack s0; // ERROR: no parameter list that can be used for type deduction
Stack<int> s1; // OK: template type explicitly set, default constructor will be called
Stack s2(100); // OK: will instantiate Stack<int> s2(100). Type is deduced from arguments
s1.Push(42); // Now Stack::Push will be instantiated
```
### Notes
- like with template functions, class templates will be instantiated on use. However <u>only those functions of the template will be instantiated which are used!</u>
	- int the example above only the members, the constructors,  and the Push function are instantiated
- template classes are commonly used for container structures
- class templates can be used with non-type template arguments (see below)

## Why Templates "need" to be implemented in Header
### Notes
- The compiler compiles each `.cpp` file separately and independently.
- Also, when instantiating a template, the compiler creates a new class with the given template argument(s). Therefore, the compiler needs to know the template implementation!
	- If the implementation is in a different `.cpp` file, the compiler cannot see it. All it can see is the header include, which just provides the templated definition.
	- a template is literally a template. A class template is not a class, it's a recipe for creating a new class for each `T` we encounter
- if the template is only defined but not used, the compiler will add no function at all to the source code
### Solutions
- Implement the template class directly in the header. This way the implementation is always included and visible to the compiler
- Declare the template class in a header, and implement it in a body file as done with usual classes. At the bottom of the header file include the body file containing the implementation:
```c++
/* Foo.hpp */
template <typename T>
struct Foo
{
    void doSomething(T param);
};

#include "Foo.cpp" // include body file containing implementation

/* Foo.cpp */
template <typename T>
void Foo<T>::doSomething(T param)
{
    // implementation
}
```
- Explicit Instantiations: Declare the template class in a header, and implement it in a body file as done with usual classes. At the bottom of the body file forward declare instantiations of specific types:
```c++
/* Foo.hpp */
template <typename T>
struct Foo
{
    void doSomething(T param);
};

/* Foo.cpp */
template <typename T>
void Foo<T>::doSomething(T param)
{
    // implementation
}
// explicit instantiations
template class Foo<int>;
template class Foo<float>;
// You will only be able to use Foo with int or float, as these specific classes will be created in Foo.cpp at compile time
```
## Template Argument Deduction
```c++
template<typename T>
void foo(T a) { ... }
template<typename T>
void foo_mult(T a, T b) { ... }
```
### Notes
- process of deducing the argument types from the given arguments
	- e.g., `foo(1.2f)` will be deduced to `foo<float>(1.2f)` by the compiler
- during argument deduction <u>no</u> conversions are performed
	- meaning, when multiple arguments of type `T` exist, all types must match
	- e.g., `foo(1.2f, 14)` will throw compilation error: `T` will be deduced to `float` but `14` is `int` and no conversions are performed at deduction
	- use `static_cast<T>` on arguments, to explicitly type cast: `foo(1.2f, static_cast<float>(14))`
	- or use explicit instantiation: `foo<float>(1.2f, 14)`. It will now implicitly cast the `int`
- the template is instantiated after deduction
- for more information see [template argument deduction](https://en.cppreference.com/w/cpp/language/template_argument_deduction)

## Explicit Specialization of Function Templates
### Example
```c++
template<typename T>
T Max(T x, T y) { return x > y ? x : y; }
// Explicit Instantiation
template int Max(int x, int y);
// Explicit Specialization
template<> const char* Max(const char* x, const char* y) { return stcmpy(x,y) > 0 ? x : y; }
```
### Notes
- allows to specialize templates for specific types
- necessary if specific semantics are not supported on specific types or for optimization purposes
- explicit specified templates are already instantiated!
	- explicitly specifying a template inside a header file would therefore cause a redefinition error
- the empty template argument list at `template<>` is required to tell the compiler to explicit specialize
	- it must be empty!

## Partial Specialization of Function Templates
### Example
```c++
/* partial specialization of function templates is not legal */
template<typename T> void foo(const T& x) { ... }
template<typename T> void f<std::vector<T>>(std::vector<T> const& v) { ... } // ERROR: illegal in C++ !

/* instead, overload the function */
template<typename T> void f(const std::vector<T>& v) { ... }
f(std::vector<T>{}); // invokes overloaded function right above

/* if overloading is not possible 1) */
template<typename T> struct type{}; // empty templated class
template<typename T> T create(type<T>) { return T(); }
template<typename T> std::vector<T> create(type<std::vector<int>>) {
	std::vector<T> v;
	v.reserve(100);
	return v;
}
```
### Notes
- partial specialization of function templates is not supported in C++ natively!
- instead of partially specializing a template function just overload it
	- since overloaded template functions are still templates you will also benefit from lazy instantiation
- if overloading is not possible (T in return-type or nowhere in prototype), then the example above gives a workaround
- either overload or specialize a template function, avoid mixing both because it can result in unexpected behavior!

## Explicit Specialization of Class Templates
### Example
```c++
/* original class */
template<typename T>
class Printer {
	T* m_pData;
public:
	Printer(T* data) : m_pData{data} {}
	void Print() { cout << *m_data << endl; }
	T* GetData() { return m_pData; }
};
/* explicit specialization of entire class */
template<> // empty template argument list is required for "explicit" specialization
class Printer<char*> {
	char* m_pData;
public:
	Printer(char* data) : m_pData{data} {}
	void Print() { cout << m_pData << endl; }
	char* GetData() { return m_pData; }
};
/* explicit specialization of a single class template function */
template<> // empty template argument list is required for "explicit" specialization
void Printer<std::vector<int>>::Print() {
	for(const auto& x : *m_pData) { cout << x; } cout << endl;
}
/* main */
int data{42};
Printer<int> p1{data}; // calls original class

char* str{"Test String"};
Printer<char*> p2{p}; // calls explicit specialization for char*
p2.Print();
char* pData = p2.GetData();

std::vector<int> vec{1,2,3,4};
Printer<std::vector<int>> p3{&vec};
p3.Print(); // calls exlicit specialized Print-function for std::vector
```
### Notes
- as with function templates, explicit specializations of class templates must start with `template<>` (empty template argument list)
- when explicitly specializing  a single function of a template class, the definition must always be outside of the class

## Partial Specialization of Class Templates
### Example
```c++
/* orignal template class */
template<typename T>
class SmartPointer {
	T* m_ptr;
public:
	SmartPointer(T* _ptr) : m_ptr{_ptr} {}
	~SmartPointer() { delete m_ptr; }
	T& operator*() { return *m_ptr; }
	T* operator->() { return m_ptr; }
};
/* partially specialize for use with arrays */
template<typename T>
class SmartPointer<T[]> {
	T* m_ptr;
public:
	SmartPointer(T* _ptr) : m_ptr{_ptr} {}
	~SmartPointer() { delete[] m_ptr; }
	T& operator[](int idx) { return m_ptr[idx]; }
};
/* main */
SmartPointer<int> s1{new int(42)}; // calls original implementation
SmartPointer<int[]> s2{new int[5]}; // calls implementation for partially specialized class
int last = *(s2+4); // ERROR: operator* is not defined on specialized SmartPointer<int[]>. Same goes for operator->
```
### Notes
- note that in the partially specialized example above, the * and -> operators do not exists
- explicit vs partial specialization
	- explicit: all template arguments are specialized
	- partial: a subset of all template arguments are specialized

## Non-Type Template Arguments
### Examples
```c++
template<int size>
void foo() { cout << size << endl; }
template<typename T, int size> // specify size as non-type argument
T Sum(T (&pArr)[size]) // "T (&pArr)[size]" requests a reference to an array of type T and length size
{
	T sum{0};
	for(int i=0; i<size; ++i) sum += pArr[i];
	return sum;
}
// call sample function foo
foo<42>(); // out: "42"
// call Sum which deduce size of array implicitly
int arr[]{1,2,3,4};
int sum = Sum(arr);
```
### Notes
- non-type template arguments must be known at compile time
	- e.g., use constants, statics, or `constexpr`
	- this further implies that  the non-type template arguments <u>cannot</u> be modified!
- common use-case is to specify array sizes as non-type template argument (see above)
	- the size of the array has not to be specified explicitly

## Perfect Forwarding
### Example
```c++
struct ID {
    int m_id;
    ID(const int _id) : m_id{_id} { cout << "ID(int)" << endl; }
    ID(const ID& _id) : m_id{_id.m_id} { cout << "ID(ID&)" << endl; }
    ID(ID&& _id) noexcept : m_id{_id.m_id} { cout << "ID(ID&&)" << endl; }
    ~ID() = default;
};

struct Bad {
    ID m_id1; ID m_id2;
    Bad(const ID& _id1, const ID& _id2) : m_id1{_id1}, m_id2{_id2} { }
    Bad(ID&& _id1, ID&& _id2) : m_id1{std::move(_id1)}, m_id2{std::move(_id2)} { }
};

struct Good {
    ID m_id1; ID m_id2;
    template<typename T1, typename T2>
    Good(T1&& _id1, T2&& _id2) : m_id1{std::forward<T1>(_id1)}, m_id2{std::forward<T2>(_id2)} { }
};

/* ... */
Bad bad1{12, 42}; // call order: (1) ID(int), (2) ID(ID&&)

ID i1{42};         // call order: (1) ID(int)
Bad bad2{12, i1};  // call order: (2) ID(int), (3) ID(ID&), (4) ID(ID&) => copy-constructor is called

ID i2{42};           // call order: (1) ID(int)
Good good1{12, i2};  // call order: (2) ID(int), (3) ID(ID&)
// => 1st arg will be deduced int and m_id1{_id1} will call the constructor of ID,
//    2nd argument will be deduced to ID& and m_id2{_id2} will call the copy constructor

Good good2{12, ID{42}};  // call order: (1) ID(int), (2) ID(int), (3) ID(ID&&)
// => 1st arg will be deduced int and m_id1{_id1} will call the constructor of ID,
//    2nd argument will be deduced to ID&& and m_id2{_id2} will call the move constructor

Good good3{12, 42};  // call order: (1) ID(int), (2) ID(int)
// => all arguments are forwarded to an inplace constructor call, no overhead calls: perfect forwarding
```
### Notes
- technique that always ensures that move-semantics is used when the function is called with temporaries
- in the example above, without perfect forwarding in `Bad`, constructors should be implemented for all possible combinations of reference-types
- in the example above, the call `Good good3{12, 42}` will cause to call the member-constructors only once (inplace)
	- the same way common STL-container methods like `emplace_back()` are implemented
- `T&&` in `template<T> foo(T&& _id)` will be deduced as follows:
	- if `T&` (`lvalue-reference`): `T&&` deduces to `T&` (`lvalue-reference`)
	- if `T&&` (`rvalue-reference`): `T&&` deduces to `T&&` (`rvalue-reference`)
	- rule: `&&` will preserve reference-type at deduction
	- `T&&` is called [forwarding-reference](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)
- `std::forward` will ensure that the argument-types will be preserved
	- `foo(T&& _id) : id{_id}` --> `ID(const ID& _id)`: copy constructor will be called since `_id` is a `lvalue` in the function scope
	- `foo(T&& _id) : id{std::forward<T>(_id)}` --> `ID(ID&& _id)`: if `foo()` is called by temporary, the move constructor will be called since `_id`'s original argument-type (`rvalue-reference`) is preserved by `std::forward`

## Variadic Templates
### Example
```c++
void Print() { cout << endl; } // recursive base case

template<typename T, typename... Params>
void Print(T&& element, Params&&... args)
{
    cout << element;
    if (sizeof...(args) != 0) cout << ", ";
    Print(std::forward<Params>(args)...); // recursive call
}

int main()
{
    Print(1, 3.5, "12", 26.78f); // call Print(int, double, const char*, float)
    // recursive call stack:
    // (1) Print(1, {3.5, "12", 26.78f})
    // (2) Print(3.5, {"12", 26.78f})
    // (3) Print("12", {26.78f})
    // (4) Print(26.78f, {})
    // (5) Print()
}

```
### Notes
- allows to pass a variable number of arguments to a function
- it is indicated by the ellipses (`...`) operator
- in `foo(T... t)`, `t` is called a parameter pack
- parameter packs cannot be iterated, so variadic-functions need to be implemented recursively
- see [Parameter Pack](https://en.cppreference.com/w/cpp/language/parameter_pack) for more information
- alternativley C-Style variadic functions can be implemented using macros like va_start, va_arg, etc.

## Dynamic vs. Static Dispatch
### Exmaples
```c++
/* Version 1: Statically programm two classes, only for reference */

struct Camera1V1
{
  void Open() { cout << "Open Camera 1" << endl; }
  void Close() { cout << "Close Camera 1" << endl; }
  void GetFrame() { cout << "New Frame Camera 1" << endl; }
};

struct Camera2V1
{
  void Open() { cout << "Open Camera 2" << endl; }
  void Close() { cout << "Close Camera 2" << endl; }
  void GetFrame() { cout << "New Frame Camera 2" << endl; }
};

// Usage:

Camera1V1 cam1;
Camera2V1 cam2;
cam1.Open();
cam1.GetFrame();
cam2.GetFrame();

/* Version 2: Dynamic Subclassing */

struct CameraBaseDynamic
{
  void Open() { cout << "Open Camera" << endl; }
  void Close() { cout << "Close Camera" << endl; }

  virtual void GetFrame() = 0;
};

struct Camera1V2 : public CameraBaseDynamic
{
  virtual void GetFrame() override final { cout << "New Frame Camera 1" << endl; }
};

struct Camera2V2 : public CameraBaseDynamic
{
  virtual void GetFrame() override final { cout << "New Frame Camera 2" << endl; }
};

// Usage:

CameraBaseDynamic* cam {}; // == CameraBase* cam = nullptr;
cam = new Camera1V2();
cam->Open();
cam->GetFrame();
cam = new Camera2V2();
cam->GetFrame();

/* Version 3: Static Subclassing */

template<typename T>
struct CameraBaseStatic
{
  void Open() { cout << "Open Camera" << endl; }
  void Close() { cout << "Close Camera" << endl; }

  void GetFrame()
  {
    static_cast<T*>(this)->GetFrameImpl();
  }
};

struct Camera1V3 : public CameraBaseStatic<Camera1V3>
{
  void GetFrameImpl() { cout << "New Frame Camera 1" << endl; }
};

struct Camera2V3 : public CameraBaseStatic<Camera2V3>
{
  void GetFrameImpl() { cout << "New Frame Camera 2" << endl; }
};

Camera1V3 cam;
cam.Open();
cam.GetFrame();
```
### Notes
#### Pros
- Using dynamic or static Subclassing allows **code reuse**.
- Static subclassing does **not** use dynamic dispatch.
#### Cons
- Static subclassing has strange side effects:
  - For `std < c++20` we have no way to enforce an interface on derived classes
    (from c++20 onwards `concepts` can be used in this case)
  - For `std < c++20` impossible to use the same name for derived and base
    function (see `CameraBaseStatic::GetFrame` and `Camera1V3::GetFrameImpl`)

## typedef
### Examples
```c++
typedef unsigned int UINT;
UINT val{};

typedef long long LLONG;
LLONG elem{};

typedef std::vector<std::list<Employee>> Teams;
Teams teams;
Teams::iterator it = teams.begin();

typedef const char* (*ErrorFun)(int); // ErrorFun becomes type of pointers to functions with prototype 'const char* foo(int)'
ErrorFun pfn = GetErrorMessage;

typedef int(&IntArray4)[4];
void foo(IntArray4 array) { for(auto& i : array) cout << i << " "; }
int arr[4]{1,2,3,4};
foo(arr);
```
### Notes
- `typedef` defines a type synonym of an existing type
	- no new types are created!
	- original type name can still be used
- this way complicated or long type definitions can be named more compact and more meaningful
- however usage of `typedef` with templates is limited (see [discussion](#typedef-vs-type-alias) below)
	- use type alias  over typedef!

## Type Alias (>=C++11)
### Examples
```c++
using UINT = unsigned int;
UINT val{};

using LLONG = typedef long long;
LLONG elem{};

using Teams = std::vector<std::list<Employee>>;
Teams teams;
Teams::iterator it = teams.begin();

using ErrorFun = const char* (*)(int); // ErrorFun becomes type of pointers to functions with prototype 'const char* foo(int)'
ErrorFun pfn = GetErrorMessage;

using IntArray4 = int(&)[4];
void foo(IntArray4 array) { for(auto& i : array) cout << i << " "; }
int arr[4]{1,2,3,4};
foo(arr);
```
### Notes
- same as `typedef`, type alias defines a type synonym of an existing type
 	- no new types are created!
	- original type name can still be used
- the syntax of type alias is meant to be more readable/intuitive
- type aliases can also be templated which is not possible with `typedef` (see [discussion](#typedef-vs-type-alias) below)

## typedef vs. Type Alias
### Example
```c++
/* in this scenario typedef and type alias both have the same effect */
typedef std::vector<std::list<std::string>> Names; // typedef
using Names_str = std::vector<std::list<std::string>>; // type alias

/* but unlike typedefs, type aliases can be templated */
template<typename T>
using Names = std::vector<std::list<T>>; // this is not possible with typedef

Names<std::string> names;
Names<Names<std::string>> nnames; // templated type aliases can also be nested
```
### Notes
- type alias has several advantages over typedef:
	- type aliases have uniform syntax which makes the code more readable
	- type aliases can be templated
- since type aliases can be templated, they can be used to hide template arguments (see [here](#type-alias-templates-to-hide-template-arguments)) or ...
- see [here](https://www.nextptr.com/tutorial/ta1193988140/how-cplusplus-using-or-aliasdeclaration-is-better-than-typedef) for more information

## Type Alias to hide Template Arguments
### Example
```c++
/* define template class */
template<typename T, int size>
class _Stack {
    T m_buffer[size];
    int m_top{-1};
public:
    _Stack() = default;
    _Stack(const T& e) { this->Push(e); }

    void Push(const T& e) { m_buffer[++m_top] = e; }
    void Pop() { --m_top; }
    const T& Top() const { return m_buffer[m_top]; }
    bool isEmpty() const { return m_top == -1; }
};
/* type-alias to hide template argument size */
template<typename T>
using Stack = _Stack<T,512>; // allows to use Stack with only one template parameter
/* main */
Stack<int> s1; // calls type-alias

Stack s2(1.2f); // ERROR: for type-aliased templates the type must be set explicitly
Stack<float> s3(1.2f); // OK
```
### Notes
- see [this](https://en.cppreference.com/w/cpp/language/type_alias) for more use-cases of type-aliasing templates

## Type Traits
### Examples
```c++
/* check type trait at run time */
template<typename T>
T Divide1(T a, T b) {
	if(!std::is_floating_point<T>::value) throw std::invalid_argument();
	return a/b;
}
// note:
// even if 'std::is_floating_point<T>::value' is evaluated at compile time,
// 'if(!std::is_floating_point<T>::value)' will still be evaluated at run time!

/* check type trait at compile time */
template<typename T>
T Divide2(T a, T b) {
	static_assert(std::is_floating_point<T>::value, "Divide() only supports floating points");
	return a/b;
}
```
### Notes
- type traits enforces characteristics to the types of template arguments
	- the characteristics of template types are <u>evaluated at compile time</u>
- they are defined in header `<type_traits>` and they can  return a `bool` or a type
- Examples are:
	- `is_void`: checks if a type is `void`
	- `is_null_pointer`: checks if a type is `std::nullptr_t`
	- `is_integral`: checks if a type is an integral type (char, short, int, long, etc.)
	- `is_floating_point`: checks if a type is a floating-point type
	- `is_array`: checks if a type is an array-type
	- see more [here](https://en.cppreference.com/w/cpp/header/type_traits)
- type traits are also useful for template metaprogramming
- type traits can also be used for non-template arguments (normal types)
	- `bool b = std::is_integral<int>::value; // b == true`
- to check the type traits at compile time use [`static_assert`](#static_assert-c11)

## static_assert (>=C++11)
### Example
```c++
/* definition */
static_assert(bool_constexpr, message);
static_assert(bool_constexpr);

/* examples */
static_assert(false, "will never compile");
template <class T>
struct A {
    static_assert(std::is_default_constructible<T>::value, "T requires default constructor");
    static_assert(std::is_move_constructible<T>::value, "T requires move constructor(s)");
};
```
### Notes
- like normal asserts but it is evaluated at compile time
- if an `static_assert` evaluates false the code will not compile
- a custom message can be passed to the `static_assert` call which will be displayed at compiler error message
- in the last example above, `class T` is enforced to have default and a move constructors. If `A<T>` is called with `T` that does not fulfill these traits, the code <u>will</u> not compile
- `static_assert` will of course not be visible in final machine code, they are treated only at compile time
- see [this](https://en.cppreference.com/w/cpp/language/static_assert) for more information

## Variadic Functions (C-Style)
### TODO
- see [this](https://en.cppreference.com/w/c/variadic)

## Callbacks: Function Pointers vs. Function Objects
### Example
```c++
/* sorting algorithm framework */
template<typename T, int size, typename Comparator>
void Sort(T (&arr)[size], Comparator comp) // template param Comparator can be deduced to function pointer and functors
{
    for(int i=0; i<size-1; ++i)
        for(int j=0; j<size-1; ++j)
            if(comp(arr[j],arr[j+1]))
            {
                T temp   = std::move(arr[j]);
                arr[j]   = std::move(arr[j+1]);
                arr[j+1] = std::move(temp);
            }
}
/* comparator functions */
bool fAscending(int x, int y) { return x > y; }
bool fDescending(int x, int y) { return x < y; }
/* comparator functors */
struct Ascending { bool operator()(int x, int y) { return x > y; } };
struct Descending { bool operator()(int x, int y) { return x < y; } };
/* main */
int arr[]{1, 5, 3, 7, 2};
// call with function pointers
Sort(arr, fAscending); // 1,2,3,4,7
Sort(arr, fDescending); // 7,4,3,2,1
// call with function objects
Sort(arr, Ascending()); // 1,2,3,4,7
Sort(arr, Descending()); // 7,4,3,2,1
```
### Notes
- using function pointers is hard to optimize for the compiler
	- the address of the pointer might be determined at run time
	- this often disallows the compiler to inline the call
	- function objects on the other side are comparatively easy to optimize/inline
- a function object (functor) is a `struct`/`class` that overloads the function call operator `operator()(..)`
	- call to overloaded function call operator resembles a global function call
	- this way functors can be used as callbacks instead of function pointers
	- functors must be specified at compile time and can therefore be optimized better than function pointers by the compiler
		- functors are often more efficient
	- since functors are defined in a `struct`/`class` they can provide a state (e.g. keeping track of some entity). This is impossible for function pointers and allows functors a wider range of applications
		- functors are more flexible to use
- <u>prefer to use function objects over function pointers</u>

## Lambda Expressions (>=C++11)
### Examples
```c++
/* simple lambda functions */
[](){ cout<<"this is a inplace lambda expr" << endl; }(); // out: "this is a inplace lambda expr"

[](int x) {
	x *= 2;
	[](int x) { cout << x << endl; }(x); // nesting lambdas is possible
}(5); // out: 10

auto fn = [](){ cout<<"this is a lambda expr in a variable"<<endl; };
fn(); // out: "this is a lambda expr in a variable"

auto sum1 = [](int x, int y) { return x+y; }; // iff the return type equals the argument types then there is no need to specify the return type
auto r1 = sum1(4,2); // r1 = 6;

auto sum2 = [](int x, int y)->float { return static_cast<float>(x+y); }; // return type must be specified here since it differs from argument types
auto r2 = sum1(4,2); // r2 = 6.f;

auto sum3 = [](auto x, auto y) { return x+y; }; // works with any type
auto r3 = sum(5.2f,4.8f); // r3 = 10.f

template<typename T, int size, typename Callback>
void ForEach(T (&arr)[size], Callback operation) { // template param Callback can be deduced to function pointer and functors/lambda expressions
	for(int i=0; i<size; ++i) operation(arr[i]);
}
int arr[]{1,2,3,4};
ForEach(arr, [](auto x){ cout<< x << " "; }); // call lambda inplace

/* lambda functions with references */
ForEach(arr, [](auto& x){ x +=1; }); // arr: 2,3,4,5

/* lambda functions with capture list */
int offset{5};
ForEach(arr, [offset](auto& x){ x +=offset; }); // arr: 6,7,8,9

ForEach(arr, [offset](auto& x){ x +=offset; offset++; }); // ERROR lambda functions are const by default => offset cannot be modified

// mutable removes constness
ForEach(arr, [offset](auto& x)mutable { x +=offset; offset++; }); // arr: 6,8,10,12

// capture variable by reference so that captured values can be returned from lambda
int sum{};
ForEach(arr, [&sum](auto& x)mutable { sum += x; });
cout << sum << endl; // out: 10

int sum2{};
ForEach(arr, [&](auto& x)mutable { sum2 += x; }); // [&] captures all variables in enclosing scope by reference -> all variables in scope can be used and modified inside lambda
cout << sum2 << endl; // out: 10

// [=] captures all variables in enclosing scope by value
ForEach(arr, [=](auto& x) { cout<<x+offset<<" "; }); // arr: 6,7,8,9

int sum3{};
ForEach(arr, [=,&sum3](auto& x)mutable { sum3 += x+offset;  }); // [=,&sum3] captures all scope variables by value except sum3 which is captured by reference
cout << sum3 << endl; // out: 30

int sum4{};
ForEach(arr, [&,offset](auto& x)mutable { sum4 += x+offset;  }); // [&, offset] captures all scope variables by reference except offset which is captured by value
cout << sum4 << endl; // out: 30

/* generalized capture (>=C++14) */
// trivial example
[y=sum4](int x) { return y+x; }(5); // out: 35

// C++14 now supports capturing a varibale by move
auto u = std::make_unique<int>(2);
ForEach(arr, [o=std::move(u)](auto x){ cout << x+(*o) << " "; }); // arr: 3,4,5,6
```
### Notes
- a lambda expression defines an anonymous function object
- it is an syntactic shortcut for a function pointer
	- internally a `struct` is created and the lambda function will be implemented inside `operator()(..)`
- can be used for simple callbacks without internal state
- lambda expression can be passed as argument
- lambda expression can accept parameters and return values
- lambda expressions can capture variables from the enclosing scope
	- `[var]`: capture variable `var` by value
	- `[=]`: capture all scope variables by value
	- `[&var]`: capture `var` by reference
	- `[&]`: capture all scope variables by reference
	- `[&,var]`: capture all scope variables by reference except `var` which is captured by value
 	- `[=,&var]`: capture all scope variables by value except `var` which is captured by reference
 	- `[this]`: capture all member variables (i.e. capture `this`) if lambda is used inside a member function
 	- note: <u>global and static variables are automatically captured</u>
 	- note: only lambda expression with empty capture list (`[]`) can implicitly be decomposed to a function pointer
	 	- <u>if a lambda is passed to a function pointer argument, the capture list must be empty (`[]`)!</u>
- the capture list can only capture variables which are defined in the same scope <u>before</u> the lambda call
	- this holds even when capturing all scope variables with `[=]` or `[&]`
- C++14 added generalized capture  support which allows to define a new variable inside the capture list
	- this now allows to capture variables by move
- <u>when possible prefer lambda expression over function objects</u>
