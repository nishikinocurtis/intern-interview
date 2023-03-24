# Effective C++ Notes

1. Prefer `const`, `enum`, `inline`, to `#define`

`#define` is hard to debug, since no label in object code. Also, for float number, `define` may result in multiple copy in obj code.

`char`-based string: declare both const pointer and const char array: `const chat* const p = "Hello World!"`. `const std::string p("Hello World!")` is preferred.

`static const` in class declaration time initialization are not always allowed. This time we need a seperate definition in code. If we want the address, also a definition is needed.

enum-hack: for compiler that doesn't allow in-class initilization, enum value is accepted at where int is needed.

2. use `const` anywhere possible

bitwise const can be cheated, ensure logical const by using mutable

