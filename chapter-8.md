## Struct

- Structure is an aggregate of elements of different types.
- Struct definition must end with `;`. 

```C++
struct Address{
    int number;
    std::string street; 
    std::string suburb; 
    std::string state; 
    int post_code;    
}; // don't forget the semi-colon
```

- Variables of struct can be initialised with uniform initialiser: 

```C++
int main(){ 
    Address home{18, "Margaret St", "Norwood", "SA", 5067};
    Address new_home{37, "Delbridge Dr", "Sydenham", "VIC", 3037};
    std::cout << home << '\n';
    std::cout << new_home << '\n';
}
```
- Struct pointer members can be accessed using the `->` operator.

```C++
void printAddr(const Address* addr){
    std::cout << addr->name << " " 
              << addr->street << ", "
              << addr->suburb << ", " 
              << addr->state << ", "
              << addr->post_code << '\n';
}
```

- Alternatively, a struct can be passed by referenced and members can be accessed using the `.` operator: 

```C++
void printAddr(const Address& addr){
    std::cout << addr.name << " " 
              << addr.street << ", "
              << addr.suburb << ", " 
              << addr.state << ", "
              << addr.post_code << '\n';
}
```

### Struct layout

- Struct holds its member in the order they are declared. 
- In the previous example, the address value of name is guaranteed to be smaller than the address value of street. 
- The size of a struct is not necessarily the size of its member due to alignment - i.e there can be holes in the structure. 
- When compactness of data is important, lay out structure data members with larger members before smaller ones;

### Struct Name

- The name of a struct becomes available immediately after it has been encountered.
- However, its definition cannot contain an object of that struct itself - i.e. circular definition. 
- To allow 2 structs that refer to each other, we can provide a declaration for one struct first: 

```C++
struct List;
struct Node{
    int value;
    Node* prev;
    Node* next;
    List* container;
};

struct List{
    Node* head;
};
```

### Struct and classes 

In C++, struct is simply a class where all members are public by default. 

### Bit fields

- A `char` is the smallest object that can be independently allocated and addressed in C++. 
- It is however, possible to specify variables that take up less space than a char in a struct using bit field.
- Field needs to specify the number of bits it occupies.
- Field can be unnamed - usually for better layout.

```C++
struct PPN{
    unsigned int PFN: 22; // 22 bit page frame number 
    int: 3; // unnamed field for layout 
    unsigned int CCA: 3; // 3 bit cache coherent algorithm
    bool nonreachable: 1; 
    bool dirty: 1;
    bool valid: 1; 
    bool global: 1;
};

```
- A field must be of integral or enumeration type. 
- It's not possible to take the address of the field. 
- Note: using field may not save space
    - May save data space but lead to larger code space. 
    - Field is a convenient short hand construct for bitwise operator. 

## Union 

- A union is a struct in which all members are allocated at the same address. 
- Hence occupies the space of the largest member. 
- Can hold only 1 value at a time. 
- Union does not provide a check for which field is active and hence error prone.
- In C, union is used for interpreting a byte in different formats (dangerous). However, C++ provides `std::reinterpret_cast` (also dangerous) that can be warned by the compiler.
- Should use `std::variant` instead.

### Union and classes 

Unions are classes with some restrictions: 

- A union cannot be used as a base class
- A union cannot have virtual functions
- A union cannot have members of reference type 
- A union cannot have base class 
- At most one member of a union can have an in-class initialiser. 
- If a union has a member with user-defined constructor, a copy operation, a move operation or a destructor, then that special function is deleted for that union. 

### Example - implementing a discriminate union 

```C++
struct Variant{
    enum class Type {String, Number}; 
    Type type; 
    union {
        double d;
        std::string s; // has user defined constructor, copy assignment, destructor 
    }
    struct BadEntry{}; // For error 
    Variant(const Variant& other){
        if (other.type == Type::String){
            new (&s)(other.s);
        }else{
            d = other.d;
        }
        type = other.type;
    }
    Variant& operator=(const Variant& other){
        if (type == Type::String && other.type == Type::String)
            s = other.s;
        else if (type == Type::String && other.type == Type::Number){
            s.~string();
            d = other.d;
        }
        else if (this->type == Type::Number && other.type == Type::String)
            d = other.d;
        else
            new (&s)(other.s);
        type = other.type;
        return *this;
    }
    double number() const{
        if (type == Type::String) throw BadEntry{};
        return d;
    }
    std::string text() const{
        if (type == Type::Number) throw BadEntry{};
        return s;
    }
    void set_number(double d){
        if (type == Type::String) s.~string();
        this->d = d;
        type = Type::Number;
    }
    void set_text(const std::string& s){
        if (type == Type::String){
            this->s = s;
        }else{
            new(&this->s)(string(s));
            type = Type::String;
        }
    }
    ~Variant(){
        if (type == Type::String)s.~string();
    }
}
```

## Enum

- Enumeration is a type that holds a set of integer values defined by the user.
- Prefer enum class over plain enum
- Enum class is scoped and strongly typed - i.e. does not allow automatic casting.
- Each enumerator must be represented by an underlying integer type. 
- Hence, the underlying type of enum class must be one of the signed or unsigned integer values (char is acceptable);
- By default enumerator values start from 0;
- Can be static_cast to int. 
```C++
enum class Level {Info, Debug, Warning, Error};
static_cast<int> Level::Info == 0; 
static_cast<int> Level::Debug == 1;
static_cast<int> Level::Warning == 2; 
static_cast<int> Level::Error == 0;  
```

- Enumerator values can be initialised using a constant expression of integral type.

```C++
enum class Flags{
    acknowledge = 1;
    paper_empty = 2;
    busy = 4; 
    out_of_black = 8;
    out_of_color = 16;
}
```