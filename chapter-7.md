## Pointers 

For a type `T`, `T*` is pointer to type `T`. That is a variable of type `T*` holds the address to an object of type `T`.

Dereferencing (`*`) is the fundamental operation of pointers - i.e take the value at the address pointed to by the pointer. 

Note that pointer addressing mechanism maps to the addressing mechanism of the underlying machine. Most machines can address a byte (8 bit), hence the smallest object that can be allocated and pointed to is a `char`.

```C++
int *pi; // pointer to an integer 
char** pc; // pointer to pointer to character 
int *ap[15]; // array of pointers to int with 15 members 
int (*fp)(char*) // pointer to a function that takes char* as input and returns an int 
int* f(char*) // function that takes a char* argument and returns an int*
```
## Void pointer 

- `void *` usually denotes pointer to object of unknown type. 
- A pointer to any type of object can be assigned to a variable of type `void*` except pointer to a function or a pointer to a member. 
- `void*` can check for equality and inequality, can be assigned to another `void*` and can be convert to another pointer type.
- Users should use `static_cast` to convert a void pointer to an appropriate object pointer (can be unsafe). 

## nullptr 

The literal `nullptr` is a pointer that does not point to any object. `nullptr` literal can be assigned to pointers of any type. There is only one `nullptr` value instead of null values for each pointer type. 

## Arrays 

For a type `T`, `T[size]` is array of size elements of type T. Indexed from `0` to `size - 1`. 

- Accessing out of range is undefined behaviour. Run-time range check is not guaranteed. 
- The number of elements  in the array must be a `constexpr` - i.e must be known at compile time. 
- Can be allocated on the stack or on free store. 

Note that: 
- C++ builtin array is inherently low level and should only be used to implement higher level, better behaved objects like `std::vector` or `std::array`.
- There is no array assignment, and array variable is automatically converted to pointer to the first element. 
- AVOID array interface in function arguments.
- If dynamically allocated, call `delete[]` once and only once after the last use. Usually done reliably if the lifetime of the array is handled by something else - i.e. string or vector or unique_ptr. 

## Array initialiser 

- Initialised using initialiser list.
- If size is not provided, the size of the array will be the number of elements provided in the initialiser list. 
- If the size is provided, and 
    - The number of elements initialised is greater than the size, throws an error. 
    - The number of elements initialised is less than the size, the remaining elements will be 0-initialised. 

## String Literals 

- String literals in C++ is an array of const char terminated with `'\0'`.
- String literal type is `const char[size]` where size is the length of the string literal + 1 (the null terminator).

Note that: 

- In C and older C++, it is possible to assign a string literal to a non-const pointer to char. But this is a common source of error. 

```C++
char* name = "Harry"; // accepted in C and before C++11
name[0] = 'M'; // error - assignment to const
```

- To avoid this - i.e to modify the string, assign to a non const character array instead: 
```C++
char[] name = "Harry";
name[0] = 'M'; // OK
```
- A string literal is statically allocated (not on the stack or the heap). Hence safe to return from a function. Note the return type of the function below.

```C++
const char* greetings(){
    return "Hello World";
}
```

- Compilers can perform string literal pooling (deduplication) but is implementation defined. 

```C++
const char* a = "hello";
const char* b = "hello";
std::cout << (a == b);  // often prints 1 (same address), but not guaranteed by standard
```

## Raw character strings 

- Long string can be broken by whitespace to make it look neater:

```C++
char myString[] = "This is a very long string "
                  "that goes into another line";
// This is equivalent to 
char longString[] = "This is a very long string that goes into another line";
```
- To represent a backslash `\` or a quote character `"`, we need to preceed it with a backslash `\`.
- This can become unmanagable so C++ provides raw string syntax for convenience.
- `R"(CCC)"` to represent the sequence of characters `CCC`.
```C++
char[] myString = R"("quoted string")"; // value is "quoted string"
```

## Pointers into array 

- Array variable automatically degenerates to pointer to the first element in the array.
- Setting the pointer beyond the end of an array is a valid operation. 
- Dereferencing pointer pointing outside of the array is undefined behaviour. 
- The implicit conversion from array to pointer means the size of the array is LOST in the conversion. 
- The integer value of `p` of type `T*` will be `sizeof(T)` less than the integer value of `p + 1`.
- This implies that to traverse an array that does not contain a terminator the way C-style strings do, we must supply the number of elements.

## Pointers and const 

Two constant constructs in C++ 
- `constexpr`: evaluated at compile time 
- `const`: value not modified in scope 

```C++
const double pi = 3.1416; // constant double 
const char ch;      // Error const needs to be initialised 
const int v[] = {1, 2, 3, 4};
v[1] = 4; // Error trying to modify const int
```

`const` modifies a type and does not affect how the constant is allocated: 

```C++
void f(const X* p){
    // p value can't be modified
}

int main(){
    X val; // val can be modified here
    f(&val); // val can't be modified in f
}
```

- Prefixing `const` makes the object the pointer points to constant. Suffixing `const` makes the pointer itself constant. 

```C++
void f(char *p){
    char s[] = "Gorm";
    const char* pc = s; // pointer to constant
    pc[3] = 'g'; //Error cannot modify constant object
    pc = p; // OK

    char *const cp = s; // cp itself is const 
    cp[3] = 'a'; // OK
    cp = p; // Error: cp is constant 

    const char* const cpc = s;
    cpc[3] = 'a'; // Error: constant object 
    cpc = p; // Error constant pointer 
}

```
- An object that is a constant when accessed through one pointer may be variable when accessed in other ways. This is useful for function arguments.
- By declaring pointer argument const, the function is prohibited from modifying the object pointed to. 

```C++
// Find first occurence
const char* strchr(const char* p, char c); // string pointed by p and result are immutable 
char* strchr(char* p, char c); // mutable version
```

- Assigning address pointed to by non-const pointer to a const pointer is valid. 
- Assigning address pointed to by a const pointer to a non-const pointer is forbidden. 

```C++
void main(){
    const int c = 1;
    int* p = &c; // Error - initialise int* with const int*
}
```

## Reference 

- Ergonomic issues of pointers: 
    - Pointer may point to nullptr or sth unexpected. 
    - Passing to function - i.e. `f(&x)`.
    - Protection against possibility of nullptr inelegant.
    - Overload `x + y` instead of `&x + &y`. 
- A reference is an alias for an object. 
- Holds memory address to an object under the hood. 
- Always point to the same object it was initialised with. 
- There is no null reference.
- Accessing a reference uses the same syntax to access the name of an object. 
- Usually used for specifying arguments and return values of functions. 

```C++
template<class T>
class Vector{
    T* elem; 
public: 
    T& operator[](int i){return elem[i];} // return reference to element 
    const T& operator[](int i) const{return elem[i];} // return reference to const element
    void push_back(const T& a){...} // pass reference to element to be added 
};

void f(const vector<double>& v){
    double d1 = v[1]; // copy the value of the double referred to by v.operator[](1) to d1 
    v[2] = 7; // place 7 in the double referred to by v.operator[](2)
    v.push_back(d1); // pass reference to d1 to v.push_back
}
```

There are 3 types of references: 

- `lvalue`: refer to objects whose value we want to change.
- `const` reference: refer to constant objects.
- `rvalue`: refer to objects whose value we do not need to preserve after using (temporary). 

## Lvalue Reference 

- `X&` reference to type `X`. Used as references to lvalues. 

```C++
void f(){
    int var = 1;
    int& r{var}; // r references var 
    int x = r; // x = 1 
    r = 2; // var = 2 
}
```

- A reference needs to be initialised. 
- No operator operates on a reference. Operators instead apply to the underlying object. 
- To get address of the object referred to by a reference, we use `&r`. 

```C++
void f(){
    int var = 0;
    int& r{var};
    ++r; // var = 1 
    int* p = &r; // p points to var
}
```
- Usually used as function argument so we can modify the underlying object through the function. 

```C++
void inc(int& aa){
    ++aa;
}

void f(){
    int x = 1; 
    inc(x); // x = 2
}
```
- It's often best to avoid function that modifies argument. Instead, return values from the function directly. 

```C++
int next(int p){return p + 1;}
void f(){
    int x{1};
    inc(x); // x = 2. Implicit and hard to know
    x = next(x); // x = 3. Explicit and clear
}
```

- References are also used as return type. This is usually used to define functions that can be used on the LHS and RHS of an assignment. 

```C++
template<class K, class V>
class Map{
public:
    V& operator[](const K& key);
    pair<K,V>* begin(){return &elem[0];}
    pair<K,V>* end(){return &elem[0] + elem.size();}
private:
    vector<pair<K,V>>elem; 
}

template<class K, class V>
V& Map<K, V>::operator[](const K& k){
    for (auto& x: elem){
        if (k == elem.first)
            return elem.second;
    }
    elem.push_back({k, V{}});   
    return elem.back().second;
}
```

- Key is passed by reference in case it is expensive to copy. 
- Similarly, value is returned as reference in case it is expensive to copy. 
- Key is constant because we don't want to modify the value of k. 
- Value is non constant in case we want to modify the value. 

### References to variable vs references to const 

- The initialiser of a plain `T&` must be an lvalue of type `T`.
- The initialiser of a `const T&` needs not be an lvalue or even of type `T`.
    - Implicit conversion to T is applied if necessary.
    - Resulting value is placed in a temp var of type T.
    - This temp var is used as the value of initialiser. 

```C++
double& dr = 1; // Error - lvalue is needed.
const double& cdr {1}; // OK

// The latter is equivalent to 
double temp {double{1}};
const double& cdr{temp};
```
- Distinguished because introducing a temporary variable is error prone - an assignment to the variable would be an assignment to the soon to disappear variable temporary variable. 
- No such problem exists for reference to constants.

## Rvalue Reference 

- A non-const lvalue reference refers to an object which the user can write. 
- A const lvalue reference refers to a const object. 
- An rvalue reference refers to a temporary object which the user can modify, assuming the object is never used again. 

We want to know if an object is temporary because we can perform some optimisations - i.e replacing an expensive copy with a cheap move operation. 

- An rvalue reference can bind to an rvalue, but not an lvalue. 

```C++
string var{"Cambridge"};
string f(); 

string& r1{var}; // Ok - lvalue ref binds to lvalue
string& r2{f()}; // Error - f() is rvalue 
string& r3{"Cambridge"}; // Error Cambridge is rvalue 
string&& r4{var}; // Error - binds lvalue to rvalue ref
string && r5{f()}; //OK
string && r6{"Cambridge"}; //OK

const string cr1& {"Cambridge"}; // OK - temporary and bind 
```
- There is no `const T &&`. Most of the benefits of rvalue references are through modifying and moving the referenced temporary object. 
- Both `const T&` and `T&&` can bind to a temporary object. 
    - rvalue (`T&&`) is used as destructive read - i.e. optimising what would otherwise be an expensive copy.
    - const lvalue (`const T&`) is used as reference to a constant - i.e. prevent modification of an argument. 

Consider the following: 

```C++
template<class T>
swap(T& a, T& b){
    T tmp{a}; // two copies of a
    a = b; // two copies of b
    b = tmp; // two copies of a (tmp)
}
```

If `T` is a type that is expensive to copy - i.e. string or vector, this can be expensive. An optimised way: 

```C++
template<class T>
swap(T& a, T& b){
    T tmp {static_cast<T&&>(a)}; //initialisation may write to a
    a = static_cast<T&&>(b); // assignment may write to b
    b = static_cast<T&&>(tmp); // assignment may write to tmp
}
```
The result of `static_cast<T&&>(x)` is an `rvalue` of type `T&&` for `x`. An operation on rvalue can now be optimised for `x`. In particular, if `T` has a move constructor or a move assignment, it will be used: 

```C++
template<class T>
class vector{
    vector (const vector& r); //copy constructor 
    vector (const vector&& r); // move constructor 
}

vector<string> s; 
vector<string> s2{s}; // copy constructor
vector<string> s3{s + "tail";} // move constructor 
```

The use of static cast is verbose and error prone, so there is a similar construct in the stdlib: 

```C++
template<class T>
void swap(T& a, T& b){
    T tmp{std::move(a)};
    a = std::move(b);
    b = std::move(tmp); 
}
```

Note `std::move(x)` does not move x but produces an rvalue reference to x. 

This swap only swap lvalues. To swap rvalues, there can be two overloads: 

```C++
template<class T> void swap(T& a, T&& b);
template<class T> void swap(T&& a, T& b);
```

## Pointer vs Reference 

- Need to change which object to refer to, use pointer.
- Need array of object - use pointer. 
```C++
int x, y;
string& a1[] = {x, y}; // Error - array of reference
string& a2[] = {&x, &y}; // OK
vector<string&> v1 = {x, y}; // Error - vector of reference
vector<string*> v2 = {&x, &y}; // OK
```
- If does not need to change referenced object, use reference. 
- Need operator overloading - use reference. 