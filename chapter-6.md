## ISO C++ Standard 

- Implementation-defined behaviors: each implementation is to provide a well-defined and well-documented behavior.
- Unspecified: a range of possible behaviors is possible, but the implementation is not obliged to tell.
- Undefined: no reasonable behavior is required by an implementation. The effect of undefined behavior is unpredictable.

In many cases, we need to rely on implementation defined behaviors to maximise portability - i.e. 16-bit and 32-bit character sets. 

## Types zoo

- Integral types:
    - Char
    - Integer
    - Boolean 
- Arithmetic types: 
    - Integral types
    - Float
    - Double
- Fundamental types:
    - Arithmetic types
    - Void 
- User-defined types: 
    - Enumeration 
    - Class 
- Built-in types: 
    - Fundamental types
    - Pointer
    - Reference 

## Character types: 

- `char`: 8 bit - the choice of signed or unsigned is implementation defined. 
- `signed char`: like char but signed (can take negative values) - range: [-127, 127]
- `unsigned char`: like char but unsigned - range: [0, 255]
- `wchar_t`: size is implementation defined - can hold the largest character set supported by implementation locale.
- `char16_t`: 16 bit char 
- `char32_t`: 32 bit char 

Character types are literal types and support arithmetic and bitwise operations. 

Note that char type being signed or unsigned is implementation defined. This means: 

```C++
char ch = 255; 
int i = ch;
```

This snippet can lead to undefined behavior (in case ch is treated as signed character). 

Char types are distinct, which means mixing of pointers to different char types is not allowed. 

However, variables of three char types can be assigned to one another. However, assigning too large a value to signed char is undefined behavior (overflow for signed types).

## Integer types: 

- By sign: `int`, `unsigned int`, `signed int`
- By length: `short`, `int`, `long`, `long long`

Note that unlike plain `char`, plain `int` is always signed. 

For more control over int types, `<cstdint>` provides types like `int64_t`, etc. 

Size of an int is implementation dependent.

## Void type: 

Only has two use cases: 

- `void f()`: function that returns nothing 
- `void *var`: pointer to object of any/unknown type. 

## Declaration 

- A variable needs to be declared before it can be used. 
- Declaration specifies an interface.
- Definition provides implementation. 
- If it takes memory to represent something, that memory is set aside by its definition.

    - Examples of declarations with definitions: 
    ```C++
    char ch;                            // set aside memory for a char and assign a value of 0 - default value 
    auto count = 1;                     // set aside memory for an int initialised to 1 
    const char* name = "Njal";          // set aside memory for a pointer to char
                                        // set aside memory for a string literal Njal
                                        // initialise the pointer to point to the string literal 
    struct Date {int d, m, y};          // Date is a struct with 3 members 
    int day(Date* p){return p->d};      // day is a function with definition 
    using Point = std::complex<short>   // Point is an alias for std::complex<short>                       
    ```

    - Examples of declaration without definitions: 

    ```C++
    double sqrt(double);
    struct User;
    ```

- There must be exactly one definition for each name in a C++ program. However, there can be many declarations. 
- All declarations of an entity must agree on its type. 
- Definition can specify a value or receive default values. 

## The structure of declaration 

- Optional prefix specifiers (e.g `auto`, `static`).
- Base type (`vector<double>`, `const int`)
- Declarator optionally including a name
    - `*`: prefix - pointer to base type
    - `* const`: prefix - constant pointer 
    - `* volatile`: prefix - volatile pointer 
    - `&`: prefix - lvalue reference to base type 
    - `&&`: prefix - rvalue reference to base type 
    - `auto`: prefix - function using suffix return type 
    - `[]`: postfix - array
    - `()`: postfix - function 
    - `->`: postfix - return from function 
- Optional suffix function specifiers (`const`, `noexcept`)
- Optional initialiser or function body. 

## Scope: 

- `Local`: a name declared in a function or lambda. Scope from its point of declaration to the end of block in which its declaration occurs. 
- `Class`: class member name declared inside a class outside of any function, class, enum, or other namespace. Its scope extends from the opening `{` to the closing `}` of class declaration.
- `Namespace`: defined in a namespace outside of any function, class, enum or other namespaces. Its scope extends from declaration to the end of its namespace. 
- `Global`: defined outside of any function, class, enum class or namespace. The scope extends from its point of declaration to the end of file in which its declaration occurs. 
- `Statement`: defined within the `()` part of a `for`, `while`, `if` or `switch`. Its scope extends from the point of its declaration to the end of the statement. 
- `Function`: a label is in scope from its point of declaration until the end of function. 

Name shadowing - a declaration of a name in an enclosed scope can hide the same name contained in an enclosing scope. This means a name can be redefined to a different entity within a block. After exiting the block, the name resumes its previous meaning.

Hiden global name can be referenced using the scope resolution operator `::`. 

## Initialisation 

- Uniform initialisation is generally preferred over = initialisation: 
    - Does not allow type narrowing - i.e. it forbids
        - float to double 
        - int to char 
        - float to int 
        - int to float 
- Using `auto` allows a variable type to be infered from its initialisation value. 
- Auto initialisation should avoid uniform initialisation unless declared type is a list - i.e. when declare a variable with `auto` type, use `=` instead.
- Most types have a default value: 
    - Integral types - 0 
    - Pointer - nullptr
    - User-defined - default constructor 

## Missing initialiser

- For many types, it is possible to leave out initialiser.
- The only time it makes sense is when creating a large input buffer. 
- If no initialiser is specified, a global/namespace/local static/static member is initialised to `{}` of the appropriate type. 
- Local or dynamic objects are not initialised by default unless they are user-defined types with a default constructor. 
- Initialisation of built in or local variables or dynamic built in can be done with `{}`. 

## Initialiser list 

- Complicated objects may require more than one value as initialiser - usually best handled with `{}`. For instance: 
- Initialised using constructor - explicitly using `(param1, param2, ...)`.
- Note that empty `()` is treated as function declaration.
- Hence for default construction of object, use `{}`. 

## LValues and RValue 

There are two properties that matter for an object when it comes to addressing, copying and moving:
- Has identity: whether the program has a name of, pointer to, or reference to the object so that it is possible to determine if two objects are the same.
- Is movable: whether we are allowed to move the value from one address to another, leaving the object in a valid but unspecified state. 

Combinations: 
 
- i: glvalue 
- i & !m: lvalue
- !i && m: prvalue 
- m: rvalue 
- i & m: xvalue 

## Object lifetime

- `Automatic`: unless otherwise specified, a function-scoped object is destroyed when its name goes out of scope. Automatic object is allocated on the stack. 
- `Static`: has a single address for the object through out its life time - created once at initialisation and goes out of scope when the program terminates. 
- `Free store`: also dynamic - i.e. allocated using new and delete. The lifetime of dynamic objects is controlled by its users. 
- `Temporary`: intermediate result of a computation or an object used to hold a value for a reference to a const argument: lifetime is determined by their use. If bound to a reference, lifetime is that of the reference. Otherwise, live until the end of the full expression they are part of. 
- `Thread local`: life time is bounded to life time of the thread which creates the variable. 