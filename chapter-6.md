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
- Optional suffix function specifiers (`const`, `noexcept`)
- Optional initialiser or function body. 

